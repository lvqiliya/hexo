---
title: Soul 网关学习笔记10-divide插件原理
abbrlink: 868a7138
date: 2021-01-26 21:22:24
tags:
categories:
---
探究 divide 插件原理
<!--more-->

## 调用链

分别启动 `soul-admin`、`soul-bootstrap`、`soul-examples-http` 三个模块，然后发起 GET 请求：`http://localhost:9195/http/order/findById?id=007`，得到下面关键日志。

```log
o.d.soul.plugin.base.AbstractSoulPlugin  : divide selector success match , selector name :/http
o.d.soul.plugin.base.AbstractSoulPlugin  : divide rule success match , rule name :/http/order/findById
o.d.s.plugin.httpclient.WebClientPlugin  : The request urlPath is http://localhost:8188/order/findById?id=007, retryTimes is 0
```

根据日志可以看出先后执行了抽象类 `AbstractSoulPlugin` 和类 `WebClientPlugin` 的逻辑，日志打印的方法是 `AbstractSoulPlugin#execute` 中调用的，于是打上断点进行跟踪。再次发起 GET 请求。发现请求首先进入的方法是 `SoulWebHandler$DefaultSoulPluginChain#execute`。

```java
public final class SoulWebHandler implements WebHandler {
    ......
    private static class DefaultSoulPluginChain implements SoulPluginChain {
        ......
        public Mono<Void> execute(final ServerWebExchange exchange) {
            return Mono.defer(() -> {
                if (this.index < plugins.size()) {
                    SoulPlugin plugin = plugins.get(this.index++);
                    Boolean skip = plugin.skip(exchange);
                    if (skip) {
                        return this.execute(exchange);
                    }
                    return plugin.execute(exchange, this);
                }
                return Mono.empty();
            });
        }
    }
}
```

暂且不再向上追溯，分析如何具体调用 divide 插件中的内容。在抽象类 `AbstractSoulPlugin` 中首先对插件状态进行了判断，如果没有开启则直接跳过执行力逻辑重新回到 `SoulWebHandler$DefaultSoulPluginChain#execute`，如果发现插件开启则调用 `DividePlugin#doExecute`。

```java
public abstract class AbstractSoulPlugin implements SoulPlugin {
    // 模板方法，具体逻辑由子类实现
    protected abstract Mono<Void> doExecute(ServerWebExchange exchange, SoulPluginChain chain, SelectorData selector, RuleData rule);

    public Mono<Void> execute(final ServerWebExchange exchange, final SoulPluginChain chain) {
        String pluginName = named();
        final PluginData pluginData = BaseDataCache.getInstance().obtainPluginData(pluginName);
        if (pluginData != null && pluginData.getEnabled()) {
            final Collection<SelectorData> selectors = BaseDataCache.getInstance().obtainSelectorData(pluginName);
            ......
            final SelectorData selectorData = matchSelector(exchange, selectors);
            ......
            final List<RuleData> rules = BaseDataCache.getInstance().obtainRuleData(selectorData.getId());
            ......
            return doExecute(exchange, chain, selectorData, rule);
        }
        return chain.execute(exchange);
    }
    ......
}
```

紧接着分析子类逻辑，方法 `doExecute` 主要是将请求的内容封装到类 `ServerWebExchange` 中。

```java
public class DividePlugin extends AbstractSoulPlugin {

    @Override
    protected Mono<Void> doExecute(final ServerWebExchange exchange, final SoulPluginChain chain, final SelectorData selector, final RuleData rule) {
        final SoulContext soulContext = exchange.getAttribute(Constants.CONTEXT);
        assert soulContext != null;
        final DivideRuleHandle ruleHandle = GsonUtils.getInstance().fromJson(rule.getHandle(), DivideRuleHandle.class);
        final List<DivideUpstream> upstreamList = UpstreamCacheManager.getInstance().findUpstreamListBySelectorId(selector.getId());
        // 判空
        if (CollectionUtils.isEmpty(upstreamList)) {
            log.error("divide upstream configuration error： {}", rule.toString());
            Object error = SoulResultWrap.error(SoulResultEnum.CANNOT_FIND_URL.getCode(), SoulResultEnum.CANNOT_FIND_URL.getMsg(), null);
            return WebFluxResultUtils.result(exchange, error);
        }
        final String ip = Objects.requireNonNull(exchange.getRequest().getRemoteAddress()).getAddress().getHostAddress();
        DivideUpstream divideUpstream = LoadBalanceUtils.selector(upstreamList, ruleHandle.getLoadBalance(), ip);
        // 判空
        if (Objects.isNull(divideUpstream)) {
            log.error("divide has no upstream");
            Object error = SoulResultWrap.error(SoulResultEnum.CANNOT_FIND_URL.getCode(), SoulResultEnum.CANNOT_FIND_URL.getMsg(), null);
            return WebFluxResultUtils.result(exchange, error);
        }
        // set the http url
        String domain = buildDomain(divideUpstream);
        String realURL = buildRealURL(domain, soulContext, exchange);
        exchange.getAttributes().put(Constants.HTTP_URL, realURL);
        // set the http timeout
        exchange.getAttributes().put(Constants.HTTP_TIME_OUT, ruleHandle.getTimeout());
        exchange.getAttributes().put(Constants.HTTP_RETRY, ruleHandle.getRetry());
        return chain.execute(exchange);
    }
}
```

跟踪到这算是解决了日志的前两句。

> 疑问：如何从抽象类 `AbstractSoulPlugin` 跳转逻辑进入到 `WebClientPlugin` 尚未弄清楚。

虽然存有疑问，但仍然可以根据日志进行断点分析。进入类 `WebClientPlugin`。

```java
public class WebClientPlugin implements SoulPlugin {
    ......
    public Mono<Void> execute(final ServerWebExchange exchange, final SoulPluginChain chain) {
        final SoulContext soulContext = exchange.getAttribute(Constants.CONTEXT);
        assert soulContext != null;
        String urlPath = exchange.getAttribute(Constants.HTTP_URL);
        if (StringUtils.isEmpty(urlPath)) {
            Object error = SoulResultWrap.error(SoulResultEnum.CANNOT_FIND_URL.getCode(), SoulResultEnum.CANNOT_FIND_URL.getMsg(), null);
            return WebFluxResultUtils.result(exchange, error);
        }
        long timeout = (long) Optional.ofNullable(exchange.getAttribute(Constants.HTTP_TIME_OUT)).orElse(3000L);
        int retryTimes = (int) Optional.ofNullable(exchange.getAttribute(Constants.HTTP_RETRY)).orElse(0);
        log.info("The request urlPath is {}, retryTimes is {}", urlPath, retryTimes);
        HttpMethod method = HttpMethod.valueOf(exchange.getRequest().getMethodValue());
        WebClient.RequestBodySpec requestBodySpec = webClient.method(method).uri(urlPath);
        return handleRequestBody(requestBodySpec, exchange, timeout, retryTimes, chain);
    }
    ......
    private Mono<Void> handleRequestBody(final WebClient.RequestBodySpec requestBodySpec,
                                         final ServerWebExchange exchange,
                                         final long timeout,
                                         final int retryTimes,
                                         final SoulPluginChain chain) {
        return requestBodySpec.headers(httpHeaders -> {
            httpHeaders.addAll(exchange.getRequest().getHeaders());
            httpHeaders.remove(HttpHeaders.HOST);
        })
                .contentType(buildMediaType(exchange))
                .body(BodyInserters.fromDataBuffers(exchange.getRequest().getBody()))
                .exchange()
                .doOnError(e -> log.error(e.getMessage()))
                .timeout(Duration.ofMillis(timeout))
                .retryWhen(Retry.onlyIf(x -> x.exception() instanceof ConnectTimeoutException)
                    .retryMax(retryTimes)
                    .backoff(Backoff.exponential(Duration.ofMillis(200), Duration.ofSeconds(20), 2, true)))
                .flatMap(e -> doNext(e, exchange, chain));

    }

    private Mono<Void> doNext(final ClientResponse res, final ServerWebExchange exchange, final SoulPluginChain chain) {
        if (res.statusCode().is2xxSuccessful()) {
            exchange.getAttributes().put(Constants.CLIENT_RESPONSE_RESULT_TYPE, ResultEnum.SUCCESS.getName());
        } else {
            exchange.getAttributes().put(Constants.CLIENT_RESPONSE_RESULT_TYPE, ResultEnum.ERROR.getName());
        }
        exchange.getAttributes().put(Constants.CLIENT_RESPONSE_ATTR, res);
        return chain.execute(exchange);
    }
}
```

一路向下探究，发现语句 `chain.execute(exchange)` 再一次出现。到目前为止还是没有弄清楚是如何调用到方法 `WebClientResponsePlugin#execute`，但是通过向所有实现了接口 `SoulPlugin` 的类中打断点，还是证实最终进入到该方法中。

```java
public class WebClientResponsePlugin implements SoulPlugin {
    @Override
    public Mono<Void> execute(final ServerWebExchange exchange, final SoulPluginChain chain) {
        return chain.execute(exchange).then(Mono.defer(() -> {
            ServerHttpResponse response = exchange.getResponse();
            ClientResponse clientResponse = exchange.getAttribute(Constants.CLIENT_RESPONSE_ATTR);
            if (Objects.isNull(clientResponse)
                    || response.getStatusCode() == HttpStatus.BAD_GATEWAY
                    || response.getStatusCode() == HttpStatus.INTERNAL_SERVER_ERROR) {
                Object error = SoulResultWrap.error(SoulResultEnum.SERVICE_RESULT_ERROR.getCode(), SoulResultEnum.SERVICE_RESULT_ERROR.getMsg(), null);
                return WebFluxResultUtils.result(exchange, error);
            }
            if (response.getStatusCode() == HttpStatus.GATEWAY_TIMEOUT) {
                Object error = SoulResultWrap.error(SoulResultEnum.SERVICE_TIMEOUT.getCode(), SoulResultEnum.SERVICE_TIMEOUT.getMsg(), null);
                return WebFluxResultUtils.result(exchange, error);
            }
            response.setStatusCode(clientResponse.statusCode());
            response.getCookies().putAll(clientResponse.cookies());
            response.getHeaders().putAll(clientResponse.headers().asHttpHeaders());
            return response.writeWith(clientResponse.body(BodyExtractors.toDataBuffers()));
        }));
    }
    ......
}
```

明显的，这一步是最终一步，之后的逻辑就是返回应答。

---
未完待续

---
28日更新

## 负载均衡

通过前面的分析，已经弄清楚了 `divide` 插件的调用链，但是在分析抽象类 AbstractSoulPlugin 的时候并没有深入探究是如何匹配选择器和如何绑定规则。笔者分别以 `8188` 和 `8189` 两个端口启动 `SoulTestHttpApplication`，然后进入控制台页面，点击插件列表中的 `divide` 并修改选择器，在配置项中新增 host、protocol、ip:port、weight。

> 猜想：既然可以在选择器中添加配置并且设定权重，那一定会有一个机制来选择调用哪一个 ip 和端口。

```java
final SelectorData selectorData = matchSelector(exchange, selectors);

private SelectorData matchSelector(final ServerWebExchange exchange, final Collection<SelectorData> selectors) {
    return selectors.stream()
            .filter(selector -> selector.getEnabled() && filterSelector(selector, exchange))
            .findFirst().orElse(null);
}

private Boolean filterSelector(final SelectorData selector, final ServerWebExchange exchange) {
    if (selector.getType() == SelectorTypeEnum.CUSTOM_FLOW.getCode()) {
        if (CollectionUtils.isEmpty(selector.getConditionList())) {
            return false;
        }
        return MatchStrategyUtils.match(selector.getMatchMode(), selector.getConditionList(), exchange);
    }
    return true;
}
```

由以上代码可得以下结论：

1. 使用方法 filter 过滤选择器，筛选出选择器是启用状态且 filterSelector 返回 true 的选择器；
2. 方法 filterSelector 调用了 `MatchStrategyUtils#match` 返回一个布尔值协助判断。

由于插件 divide 中没有定义多个选择器，第一个进入方法 `MatchStrategyUtils#match` 进行匹配的选择器必定返回 true，此处先略去该方法的深入分析。选择器的匹配完成后就是匹配规则。

```java
private RuleData matchRule(final ServerWebExchange exchange, final Collection<RuleData> rules) {
    return rules.stream()
            .filter(rule -> filterRule(rule, exchange))
            .findFirst().orElse(null);
}

private Boolean filterRule(final RuleData ruleData, final ServerWebExchange exchange) {
    return ruleData.getEnabled() && MatchStrategyUtils.match(ruleData.getMatchMode(), ruleData.getConditionDataList(), exchange);
}
```

结论与选择器部分相似，不同的是，当启动项目时候注册了多条规则，那么此处通过方法 `filter` 过滤时必定要将规则列表中的数据一一判断，默认注册了 5 条规则，那应该会执行 5 次方法 `filterRule`。

前文提出的**猜想**到目前为止尚未得到证实，而匹配选择器和规则的逻辑中并不涉及负载均衡，故而进入方法 `DividePlugin#doExecute` 继续跟踪查看。调用链部分的分析中有一个关键的工具类没有提及，根据其命名看，必定和负载均衡有直接关系。关键代码如下：

```java
DivideUpstream divideUpstream = LoadBalanceUtils.selector(upstreamList, ruleHandle.getLoadBalance(), ip);

public class LoadBalanceUtils {
    public static DivideUpstream selector(final List<DivideUpstream> upstreamList, final String algorithm, final String ip) {
        LoadBalance loadBalance = ExtensionLoader.getExtensionLoader(LoadBalance.class).getJoin(algorithm);
        return loadBalance.select(upstreamList, ip);
    }
}
```

明显的，方法 `LoadBalanceUtils#selector` 根据入参 `algorithm` 去找到负载均衡算法实现类，然后调用实现类中的 `select` 方法选择到底调用哪一个 ip 和端口。入参 `algorithm` 是在规则中配置的负载策略，本文使用的规则默认都是 `random`。

查看接口 `LoadBalance` 的 `random` 算法实现类。

```java
public class RandomLoadBalance extends AbstractLoadBalance {
    public DivideUpstream doSelect(final List<DivideUpstream> upstreamList, final String ip) {
        int totalWeight = calculateTotalWeight(upstreamList);
        boolean sameWeight = isAllUpStreamSameWeight(upstreamList);
        if (totalWeight > 0 && !sameWeight) {
            return random(totalWeight, upstreamList);
        }
        // If the weights are the same or the weights are 0 then random
        return random(upstreamList);
    }
    ......
}
```

至此，通过负载均衡算法 `RandomLoadBalance#doSelect` 完成了调用具体的 ip 和端口，提出的猜想得到证实。

---
未完待续

---
29日更新

## 探活机制

停掉 `soul-examples-http` 服务后，`soul-admin` 模块的日志打印了以下错误信息。

```java
o.d.s.a.s.impl.UpstreamCheckService      : check the url=localhost:8188 is fail 
```

进入类 `UpstreamCheckService` 查看日志打印的为止并查看逻辑。

```java
public class UpstreamCheckService {
    ......
    private void check(final String selectorName, final List<DivideUpstream> upstreamList) {
        List<DivideUpstream> successList = Lists.newArrayListWithCapacity(upstreamList.size());
        for (DivideUpstream divideUpstream : upstreamList) {
            final boolean pass = UpstreamCheckUtils.checkUrl(divideUpstream.getUpstreamUrl());
            if (pass) {
                if (!divideUpstream.isStatus()) {
                    divideUpstream.setTimestamp(System.currentTimeMillis());
                    divideUpstream.setStatus(true);
                    log.info("UpstreamCacheManager check success the url: {}, host: {} ", divideUpstream.getUpstreamUrl(), divideUpstream.getUpstreamHost());
                }
                successList.add(divideUpstream);
            } else {
                divideUpstream.setStatus(false);
                log.error("check the url={} is fail ", divideUpstream.getUpstreamUrl());
            }
        }
        if (successList.size() == upstreamList.size()) {
            return;
        }
        if (successList.size() > 0) {
            UPSTREAM_MAP.put(selectorName, successList);
            updateSelectorHandler(selectorName, successList);
        } else {
            UPSTREAM_MAP.remove(selectorName);
            updateSelectorHandler(selectorName, null);
        }
    }
    ......
}
```

在方法 `check` 中打上断点并启动 `soul-examples-http` 服务，查看调用栈信息。

![UpstreamCheckService#check 调用栈](/images/soul/stack-trace-UpstreamCheckService#check.png)

于是追踪到调用方法 `check` 的代码。

```java
public class UpstreamCheckService {
    ......
    @PostConstruct
    public void setup() {
        ......
        if (check) {
            new ScheduledThreadPoolExecutor(Runtime.getRuntime().availableProcessors(), SoulThreadFactory.create("scheduled-upstream-task", false))
                    .scheduleWithFixedDelay(this::scheduled, 10, scheduledTime, TimeUnit.SECONDS);
        }
    }
    ......
    private void scheduled() {
        if (UPSTREAM_MAP.size() > 0) {
            UPSTREAM_MAP.forEach(this::check);
        }
    }
    ......
}
```

明显的，在方法 `setup` 中初始化了一个线程池，每10秒执行一次方法 `scheduled`，从而调用方法 `check`。其具体逻辑如下：

1. 当缓存 UPSTREAM_MAP 中个数大于 0 时，遍历每项数据并检查；
2. 如果 url 是 ip，则使用 socket#connect 连接实现探活；如果不是，则使用 InetAddress#isReachable 来进行探活；
3. 如果发现 `divideUpstream` 已死，首先从 UPSTREAM_MAP 移除，然后再通过 ApplicationEventPublisher#publishEvent 发布出去。

补充说明，在方法 `setup` 上标注了 `@PostConstruct`，官方文档中的描述如下：

> The PostConstruct annotation is used on a method that needs to be executed after dependency injection is done to perform any initialization.

简单来说就是类 `UpstreamCheckService` 作为一个 bean 依赖注入后，方法 `setup` 需要第一时间执行。
