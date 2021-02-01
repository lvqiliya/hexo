---
title: Soul 网关学习笔记13-限流器
abbrlink: e49b92c1
date: 2021-02-01 23:40:21
tags:
categories:
---
使用 ratelimiter、waf、rewrite、contextpath 插件
<!--more-->

## ratelimiter

启动 `soul-admin` 模块，在插件管理中开启 `ratelimiter` 插件，然后启动 `soul-bootstrap` 模块。

进入【插件列表-ratelimiter】，新增选择器和规则。

发送请求 `http://localhost:9195/http/order/findById?id=007`，查看日志。这一步跟熔断器部分的分析一样，都是通过抽象类 `AbstractSoulPlugin` 进入子类的 `doExecute`，具体分析该方法中执行了什么内容。

```java
public class RewritePlugin extends AbstractSoulPlugin {
    @Override
    protected Mono<Void> doExecute(final ServerWebExchange exchange, final SoulPluginChain chain, final SelectorData selector, final RuleData rule) {
        final String handle = rule.getHandle();
        final RateLimiterHandle limiterHandle = GsonUtils.getInstance().fromJson(handle, RateLimiterHandle.class);
        return redisRateLimiter.isAllowed(rule.getId(), limiterHandle.getReplenishRate(), limiterHandle.getBurstCapacity())
                .flatMap(response -> {
                    if (!response.isAllowed()) {
                        exchange.getResponse().setStatusCode(HttpStatus.TOO_MANY_REQUESTS);
                        Object error = SoulResultWrap.error(SoulResultEnum.TOO_MANY_REQUESTS.getCode(), SoulResultEnum.TOO_MANY_REQUESTS.getMsg(), null);
                        return WebFluxResultUtils.result(exchange, error);
                    }
                    return chain.execute(exchange);
                });
    }
}

public class RedisRateLimiter {
    ......
    @SuppressWarnings("unchecked")
    public Mono<RateLimiterResponse> isAllowed(final String id, final double replenishRate, final double burstCapacity) {
        if (!this.initialized.get()) {
            throw new IllegalStateException("RedisRateLimiter is not initialized");
        }
        List<String> keys = getKeys(id);
        List<String> scriptArgs = Arrays.asList(replenishRate + "", burstCapacity + "", Instant.now().getEpochSecond() + "", "1");
        Flux<List<Long>> resultFlux = Singleton.INST.get(ReactiveRedisTemplate.class).execute(this.script, keys, scriptArgs);
        return resultFlux.onErrorResume(throwable -> Flux.just(Arrays.asList(1L, -1L)))
                .reduce(new ArrayList<Long>(), (longs, l) -> {
                    longs.addAll(l);
                    return longs;
                }).map(results -> {
                    boolean allowed = results.get(0) == 1L;
                    Long tokensLeft = results.get(1);
                    RateLimiterResponse rateLimiterResponse = new RateLimiterResponse(allowed, tokensLeft);
                    log.info("RateLimiter response:{}", rateLimiterResponse.toString());
                    return rateLimiterResponse;
                }).doOnError(throwable -> log.error("Error determining if user allowed from redis:{}", throwable.getMessage()));
    }
    ......
}
```

## waf

启动 `soul-admin` 模块，在插件管理中开启 `waf` 插件，然后启动 `soul-bootstrap` 模块。

进入【插件列表-waf】，新增选择器和规则。

发送请求 `http://localhost:9195/http/order/findById?id=007`，查看日志。这一步跟熔断器部分的分析一样，都是通过抽象类 `AbstractSoulPlugin` 进入子类的 `doExecute`，具体分析该方法中执行了什么内容。

```java
public class WafPlugin extends AbstractSoulPlugin {

    @Override
    protected Mono<Void> doExecute(final ServerWebExchange exchange, final SoulPluginChain chain, final SelectorData selector, final RuleData rule) {
        WafConfig wafConfig = Singleton.INST.get(WafConfig.class);
        if (Objects.isNull(selector) && Objects.isNull(rule)) {
            if (WafModelEnum.BLACK.getName().equals(wafConfig.getModel())) {
                return chain.execute(exchange);
            }
            exchange.getResponse().setStatusCode(HttpStatus.FORBIDDEN);
            Object error = SoulResultWrap.error(403, Constants.REJECT_MSG, null);
            return WebFluxResultUtils.result(exchange, error);
        }
        String handle = rule.getHandle();
        WafHandle wafHandle = GsonUtils.getInstance().fromJson(handle, WafHandle.class);
        if (Objects.isNull(wafHandle) || StringUtils.isBlank(wafHandle.getPermission())) {
            log.error("waf handler can not configuration：{}", handle);
            return chain.execute(exchange);
        }
        if (WafEnum.REJECT.getName().equals(wafHandle.getPermission())) {
            exchange.getResponse().setStatusCode(HttpStatus.FORBIDDEN);
            Object error = SoulResultWrap.error(Integer.parseInt(wafHandle.getStatusCode()), Constants.REJECT_MSG, null);
            return WebFluxResultUtils.result(exchange, error);
        }
        return chain.execute(exchange);
    }
    ......
}
```

简单来说，方法 `doExecute` 通过多次判空或者判断权限选择继续执行下一个插件还是直接返回拒绝信息。

## rewrite

启动 `soul-admin` 模块，在插件管理中开启 `rewrite` 插件，然后启动 `soul-bootstrap` 模块。

进入【插件列表-rewrite】，新增选择器和规则。

发送请求 `http://localhost:9195/http/order/findById?id=007`，查看日志。这一步跟熔断器部分的分析一样，都是通过抽象类 `AbstractSoulPlugin` 进入子类的 `doExecute`，具体分析该方法中执行了什么内容。

```java
public class RewritePlugin extends AbstractSoulPlugin {
    @Override
    protected Mono<Void> doExecute(final ServerWebExchange exchange, final SoulPluginChain chain, final SelectorData selector, final RuleData rule) {
        String handle = rule.getHandle();
        final RewriteHandle rewriteHandle = GsonUtils.getInstance().fromJson(handle, RewriteHandle.class);
        if (Objects.isNull(rewriteHandle) || StringUtils.isBlank(rewriteHandle.getRewriteURI())) {
            log.error("uri rewrite rule can not configuration：{}", handle);
            return chain.execute(exchange);
        }
        exchange.getAttributes().put(Constants.REWRITE_URI, rewriteHandle.getRewriteURI());
        return chain.execute(exchange);
    }
}
```

## contextpath

启动 `soul-admin` 模块，在插件管理中开启 `contextpath` 插件，然后启动 `soul-bootstrap` 模块。

进入【插件列表-contextpath】，新增选择器和规则。

发送请求 `http://localhost:9195/http/order/findById?id=007`，查看日志。这一步跟熔断器部分的分析一样，都是通过抽象类 `AbstractSoulPlugin` 进入子类的 `doExecute`，具体分析该方法中执行了什么内容。

```java
public class ContextPathMappingPlugin extends AbstractSoulPlugin {

    @Override
    protected Mono<Void> doExecute(final ServerWebExchange exchange, final SoulPluginChain chain, final SelectorData selector, final RuleData rule) {
        final SoulContext soulContext = exchange.getAttribute(Constants.CONTEXT);
        assert soulContext != null;
        final String handle = rule.getHandle();
        final ContextMappingHandle contextMappingHandle = GsonUtils.getInstance().fromJson(handle, ContextMappingHandle.class);
        if (Objects.isNull(contextMappingHandle) || StringUtils.isBlank(contextMappingHandle.getContextPath())) {
            log.error("context path mapping rule configuration is null ：{}", rule);
            return chain.execute(exchange);
        }
        //check the context path illegal
        if (!soulContext.getPath().startsWith(contextMappingHandle.getContextPath())) {
            Object error = SoulResultWrap.error(SoulResultEnum.CONTEXT_PATH_ERROR.getCode(), SoulResultEnum.CONTEXT_PATH_ERROR.getMsg(), null);
            return WebFluxResultUtils.result(exchange, error);
        }
        this.buildContextPath(soulContext, contextMappingHandle);
        return chain.execute(exchange);
    }
    ......
}
```
