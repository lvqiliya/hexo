---
title: Soul 网关学习笔记10-divide插件原理
abbrlink: 868a7138
date: 2021-01-26 21:22:24
tags:
categories:
---
探究 divide 插件原理
<!--more-->

分别启动 `soul-admin`、`soul-bootstrap`、`soul-examples-http` 三个模块，然后发起 GET 请求：http://localhost:9195/http/order/findById?id=007，得到下面关键日志。

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
