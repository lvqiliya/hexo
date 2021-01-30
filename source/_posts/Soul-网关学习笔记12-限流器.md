---
title: Soul 网关学习笔记12-限流器
abbrlink: f080064
date: 2021-01-30 22:19:40
tags:
categories:
---
使用 hystrix、resilience4j、sentinel 插件
<!--more-->

## 配置 hystrix

`soul-bootstrap` 模块默认已经引入了 `hystrix` 的相关依赖，此处就不再赘述。启动 `soul-admin` 模块，在插件管理中开启 `hystrix` 插件，然后启动 `soul-bootstrap` 模块。

进入【插件列表-hystrix】，新增选择器和规则。

- hystrix 选择器

![hystrix 选择器](/images/soul/admin-hystrix-selector.png)

- hystrix 规则

![hystrix 规则](/images/soul/admin-hystrix-rule.png)

## hystrix 源码分析

发送请求 `http://localhost:9195/http/order/findById?id=007`，查看日志。

```log
o.d.soul.plugin.base.AbstractSoulPlugin  : hystrix selector success match , selector name :rongduan
o.d.soul.plugin.base.AbstractSoulPlugin  : hystrix rule success match , rule name :findById
o.d.soul.plugin.base.AbstractSoulPlugin  : divide selector success match , selector name :/http
o.d.soul.plugin.base.AbstractSoulPlugin  : divide rule success match , rule name :/http/order/findById
o.d.s.plugin.httpclient.WebClientPlugin  : The request urlPath is http://localhost:8188/order/findById?id=007, retryTimes is 0
```

在启用插件 `hystrix` 之前，请求仅打印了 `divide` 相关内容。开启并配置 `hystrix` 之后，抽象类 `AbstractSoulPlugin` 中的日志打印生效了。于是在抽象类中打上断点，得到以下调用栈信息。

![hystrix 规则](/images/soul/stack-trace-hystrix.doExecute.png)

于是进入子类 `HystrixPlugin` 查看关键代码。

```java
public class HystrixPlugin extends AbstractSoulPlugin {
    @Override
    protected Mono<Void> doExecute(final ServerWebExchange exchange, final SoulPluginChain chain, final SelectorData selector, final RuleData rule) {
        final SoulContext soulContext = exchange.getAttribute(Constants.CONTEXT);
        assert soulContext != null;
        final HystrixHandle hystrixHandle = GsonUtils.getInstance().fromJson(rule.getHandle(), HystrixHandle.class);
        if (StringUtils.isBlank(hystrixHandle.getGroupKey())) {
            hystrixHandle.setGroupKey(Objects.requireNonNull(soulContext).getModule());
        }
        if (StringUtils.isBlank(hystrixHandle.getCommandKey())) {
            hystrixHandle.setCommandKey(Objects.requireNonNull(soulContext).getMethod());
        }
        Command command = fetchCommand(hystrixHandle, exchange, chain);
        return Mono.create(s -> {
            Subscription sub = command.fetchObservable().subscribe(s::success,
                    s::error, s::success);
            s.onCancel(sub::unsubscribe);
            if (command.isCircuitBreakerOpen()) {
                log.error("hystrix execute have circuitBreaker is Open! groupKey:{},commandKey:{}", hystrixHandle.getGroupKey(), hystrixHandle.getCommandKey());
            }
        }).doOnError(throwable -> {
            log.error("hystrix execute exception:", throwable);
            exchange.getAttributes().put(Constants.CLIENT_RESPONSE_RESULT_TYPE, ResultEnum.ERROR.getName());
            chain.execute(exchange);
        }).then();
    }

    private Command fetchCommand(final HystrixHandle hystrixHandle, final ServerWebExchange exchange, final SoulPluginChain chain) {
        if (hystrixHandle.getExecutionIsolationStrategy() == HystrixIsolationModeEnum.SEMAPHORE.getCode()) {
            return new HystrixCommand(HystrixBuilder.build(hystrixHandle),
                exchange, chain, hystrixHandle.getCallBackUri());
        }
        return new HystrixCommandOnThread(HystrixBuilder.buildForHystrixCommand(hystrixHandle),
            exchange, chain, hystrixHandle.getCallBackUri());
    }
    ......
}
```

处理流程如下：

1. 将规则中的信息解析到类 `HystrixHandle`，并针对分组 key 和命令 key 进行判空处理；
2. 方法 `fetchCommand` 根据隔离策略选择实例化具体 `Command` 实现类，该实现类同时继承了 `hystrix-core.jar` 的类 `AbstractCommand`:
3. 流式写法返回 `Mono`。

至于第三步具体处理逻辑，暂时还没有看得十分明白，此处暂时略过。

## resilience4j 源码分析

进入类 `Resilience4JPlugin` 之前的流程和 `hystrix` 一样，返回的都是 `Mono`。

不同的是，它会根据一个标识 `circuit enable` 的值进行判断，然后分别执行不同的方法。

```java
public class Resilience4JPlugin extends AbstractSoulPlugin {
    ......
    @Override
    protected Mono<Void> doExecute(final ServerWebExchange exchange, final SoulPluginChain chain, final SelectorData selector, final RuleData rule) {
        final SoulContext soulContext = exchange.getAttribute(Constants.CONTEXT);
        assert soulContext != null;
        Resilience4JHandle resilience4JHandle = GsonUtils.getGson().fromJson(rule.getHandle(), Resilience4JHandle.class);
        if (resilience4JHandle.getCircuitEnable() == 1) {
            return combined(exchange, chain, rule);
        }
        return rateLimiter(exchange, chain, rule);
    }

    private Mono<Void> rateLimiter(final ServerWebExchange exchange, final SoulPluginChain chain, final RuleData rule) {
        return ratelimiterExecutor.run(
                chain.execute(exchange), fallback(ratelimiterExecutor, exchange, null), Resilience4JBuilder.build(rule))
                .onErrorResume(throwable -> ratelimiterExecutor.withoutFallback(exchange, throwable));
    }

    private Mono<Void> combined(final ServerWebExchange exchange, final SoulPluginChain chain, final RuleData rule) {
        Resilience4JConf conf = Resilience4JBuilder.build(rule);
        return combinedExecutor.run(
                chain.execute(exchange).doOnSuccess(v -> {
                    if (exchange.getResponse().getStatusCode() != HttpStatus.OK) {
                        HttpStatus status = exchange.getResponse().getStatusCode();
                        exchange.getResponse().setStatusCode(null);
                        throw new CircuitBreakerStatusCodeException(status);
                    }
                }), fallback(combinedExecutor, exchange, conf.getFallBackUri()), conf);
    }
```

## sentinel 源码分析

进入类 `SentinelPlugin` 之前的流程和 `hystrix` 一样，返回的都是 `Mono`。

```java
public class SentinelPlugin extends AbstractSoulPlugin {
    ......
    @Override
    protected Mono<Void> doExecute(final ServerWebExchange exchange, final SoulPluginChain chain, final SelectorData selector, final RuleData rule) {
        final SoulContext soulContext = exchange.getAttribute(Constants.CONTEXT);
        assert soulContext != null;
        String resourceName = SentinelRuleHandle.getResourceName(rule);
        SentinelHandle sentinelHandle = GsonUtils.getInstance().fromJson(rule.getHandle(), SentinelHandle.class);
        return chain.execute(exchange).transform(new SentinelReactorTransformer<>(resourceName)).doOnSuccess(v -> {
            if (exchange.getResponse().getStatusCode() != HttpStatus.OK) {
                HttpStatus status = exchange.getResponse().getStatusCode();
                exchange.getResponse().setStatusCode(null);
                throw new SentinelFallbackException(status);
            }
        }).onErrorResume(throwable -> sentinelFallbackHandler.fallback(exchange, UriUtils.createUri(sentinelHandle.getFallbackUri()), throwable));
    }
    ......
```

---
三个限流器的具体差异尚未进行深一步探究。
