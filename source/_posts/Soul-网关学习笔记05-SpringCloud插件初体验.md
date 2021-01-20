---
title: Soul 网关学习笔记05-SpringCloud插件初体验
tags: soul
categories: 源码学习
abbrlink: 2a526027
date: 2021-01-19 22:47:52
---
使用 SpringCloud 插件，体验 SpringCloud 代理
<!--more-->
## 启动项目

运行 soul-admin 模块的主函数，启动 soul-admin，在插件管理中打开 springCloud 插件。

进入 soul-bootstrap 模块，在 pom.xml 引入关于 springCloud 的依赖，同时引入 eureka 相关依赖作为 springCloud 的注册中心。

运行 soul-bootstrap 模块的主函数，启动 soul-bootstrap。

进入 soul-examples 模块下的 eureka 子模块启动项目。

进入 soul-examples 模块下的 springCloud 子模块启动项目。

## 源码分析

发送 `GET` 请求 `http://localhost:9195/springcloud/order/findById?id=007`。目前能追溯到最原始的请求来自于一下代码。

```java
public final class SoulWebHandler implements WebHandler {
    ......
    @Override
    public Mono<Void> handle(@NonNull final ServerWebExchange exchange) {
        MetricsTrackerFacade.getInstance().counterInc(MetricsLabelEnum.REQUEST_TOTAL.getName());
        Optional<HistogramMetricsTrackerDelegate> startTimer = MetricsTrackerFacade.getInstance().histogramStartTimer(MetricsLabelEnum.REQUEST_LATENCY.getName());
        return new DefaultSoulPluginChain(plugins).execute(exchange).subscribeOn(scheduler)
                .doOnSuccess(t -> startTimer.ifPresent(time -> MetricsTrackerFacade.getInstance().histogramObserveDuration(time)));
    }
    ......
    private static class DefaultSoulPluginChain implements SoulPluginChain {
        ......
        @Override
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

通过调用静态内部类 `DefaultSoulPluginChain` 的 `execute` 方法遍历插件集。其实这里 `Mono.defer()` 的用法并未看懂，但是根据 debug 的现象看，的确是遍历了插件集的内容。每一次遍历都进入抽象类 `AbstractSoulPlugin` 中，进行一次判断。

```java
public abstract class AbstractSoulPlugin implements SoulPlugin {
    protected abstract Mono<Void> doExecute(ServerWebExchange exchange, SoulPluginChain chain, SelectorData selector, RuleData rule);

    @Override
    public Mono<Void> execute(final ServerWebExchange exchange, final SoulPluginChain chain) {
        String pluginName = named();
        final PluginData pluginData = BaseDataCache.getInstance().obtainPluginData(pluginName);
        if (pluginData != null && pluginData.getEnabled()) {
            final Collection<SelectorData> selectors = BaseDataCache.getInstance().obtainSelectorData(pluginName);
            if (CollectionUtils.isEmpty(selectors)) {
                return handleSelectorIsNull(pluginName, exchange, chain);
            }
            final SelectorData selectorData = matchSelector(exchange, selectors);
            if (Objects.isNull(selectorData)) {
                return handleSelectorIsNull(pluginName, exchange, chain);
            }
            selectorLog(selectorData, pluginName);
            final List<RuleData> rules = BaseDataCache.getInstance().obtainRuleData(selectorData.getId());
            if (CollectionUtils.isEmpty(rules)) {
                return handleRuleIsNull(pluginName, exchange, chain);
            }
            RuleData rule;
            if (selectorData.getType() == SelectorTypeEnum.FULL_FLOW.getCode()) {
                //get last
                rule = rules.get(rules.size() - 1);
            } else {
                rule = matchRule(exchange, rules);
            }
            if (Objects.isNull(rule)) {
                return handleRuleIsNull(pluginName, exchange, chain);
            }
            ruleLog(rule, pluginName);
            return doExecute(exchange, chain, selectorData, rule);
        }
        return chain.execute(exchange);
    }
    ......
}
```

根据插件名字绑定插件数据，值得注意的是 `obtainPluginData` 方法实际上是一个从 `PLUGIN_MAP` 中取值的操作，`PLUGIN_MAP` 是基本数据缓存类中的一个 concurrentHashMap。

> 猜想：BaseDataCache 类一定是在启动 bootstrap 模块时进行缓存操作。暂不验证。

判断若插件开启，则进入判断内的逻辑：

1. 根据插件名获取选择器集合；
2. 匹配选择器；
3. 根据选择器获取规则集合；
4. 匹配规则；
5. doExecute()。

`doExecute` 方法利用了模板模式，由子类 `SpringCloudPlugin` 具体实现了该方法的逻辑。

> 问题：如何匹配到子类 `SpringCloudPlugin` 需进一步探究。

```java
public class SpringCloudPlugin extends AbstractSoulPlugin {
    ......
    @Override
    protected Mono<Void> doExecute(final ServerWebExchange exchange, final SoulPluginChain chain, final SelectorData selector, final RuleData rule) {
        if (Objects.isNull(rule)) {
            return Mono.empty();
        }
        final SoulContext soulContext = exchange.getAttribute(Constants.CONTEXT);
        assert soulContext != null;
        final SpringCloudRuleHandle ruleHandle = GsonUtils.getInstance().fromJson(rule.getHandle(), SpringCloudRuleHandle.class);
        final SpringCloudSelectorHandle selectorHandle = GsonUtils.getInstance().fromJson(selector.getHandle(), SpringCloudSelectorHandle.class);
        if (StringUtils.isBlank(selectorHandle.getServiceId()) || StringUtils.isBlank(ruleHandle.getPath())) {
            Object error = SoulResultWrap.error(SoulResultEnum.CANNOT_CONFIG_SPRINGCLOUD_SERVICEID.getCode(), SoulResultEnum.CANNOT_CONFIG_SPRINGCLOUD_SERVICEID.getMsg(), null);
            return WebFluxResultUtils.result(exchange, error);
        }

        final ServiceInstance serviceInstance = loadBalancer.choose(selectorHandle.getServiceId());
        if (Objects.isNull(serviceInstance)) {
            Object error = SoulResultWrap.error(SoulResultEnum.SPRINGCLOUD_SERVICEID_IS_ERROR.getCode(), SoulResultEnum.SPRINGCLOUD_SERVICEID_IS_ERROR.getMsg(), null);
            return WebFluxResultUtils.result(exchange, error);
        }
        final URI uri = loadBalancer.reconstructURI(serviceInstance, URI.create(soulContext.getRealUrl()));

        String realURL = buildRealURL(uri.toASCIIString(), soulContext.getHttpMethod(), exchange.getRequest().getURI().getQuery());

        exchange.getAttributes().put(Constants.HTTP_URL, realURL);
        //set time out.
        exchange.getAttributes().put(Constants.HTTP_TIME_OUT, ruleHandle.getTimeout());
        return chain.execute(exchange);
    }
    ......
}
```

经过**一系列加工**，`exchange` 的属性中放入了多个关键要素，如 `realURL`，本例中则为 `http://192.168.10.89:8884/order/findById?id=007`。

最后 `chain.execute(exchange)` 语句成功绕晕笔者，断点下去，代码又回到最初静态内部类 `DefaultSoulPluginChain`。
