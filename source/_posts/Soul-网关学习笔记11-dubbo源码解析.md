---
title: Soul 网关学习笔记11-dubbo源码解析
abbrlink: ed5b9336
date: 2021-01-27 23:11:32
tags:
categories:
---
使用 dubbo 插件，源码浅析
<!--more-->

## 注解 @SoulDubboClient

项目启动时自动装配了类 `SoulAlibabaDubboClientConfiguration`。

```java
@Configuration
public class SoulAlibabaDubboClientConfiguration {

    @Bean
    public AlibabaDubboServiceBeanPostProcessor alibabaDubboServiceBeanPostProcessor(final DubboConfig dubboConfig) {
        return new AlibabaDubboServiceBeanPostProcessor(dubboConfig);
    }

    @Bean
    @ConfigurationProperties(prefix = "soul.dubbo")
    public DubboConfig dubboConfig() {
        return new DubboConfig();
    }
}
```

除了加载 properties 文件中的内容外，还实例化了类 `AlibabaDubboServiceBeanPostProcessor`，该类实现了接口 `ApplicationListener` 以实现方法 `onApplicationEvent` 用来注册选择器和规则。

```java
public class AlibabaDubboServiceBeanPostProcessor implements ApplicationListener<ContextRefreshedEvent> {
    ......
    private void handler(final ServiceBean<?> serviceBean) {
        Class<?> clazz = serviceBean.getRef().getClass();
        if (ClassUtils.isCglibProxyClass(clazz)) {
            String superClassName = clazz.getGenericSuperclass().getTypeName();
            try {
                clazz = Class.forName(superClassName);
            } catch (ClassNotFoundException e) {
                log.error(String.format("class not found: %s", superClassName));
                return;
            }
        }
        final Method[] methods = ReflectionUtils.getUniqueDeclaredMethods(clazz);
        for (Method method : methods) {
            SoulDubboClient soulDubboClient = method.getAnnotation(SoulDubboClient.class);
            if (Objects.nonNull(soulDubboClient)) {
                RegisterUtils.doRegister(buildJsonParams(serviceBean, soulDubboClient, method), url, RpcTypeEnum.DUBBO);
            }
        }
    }
    ......
}
```

方法 `handler` 先获取类上的 `@SoulDubboClient` 注解，然后通过 http 请求远程调用，将标注了注解的方法作为规则注册到 `soul-admin` 模块。

## 匹配原理

```java
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

    private RuleData matchRule(final ServerWebExchange exchange, final Collection<RuleData> rules) {
        return rules.stream()
                .filter(rule -> filterRule(rule, exchange))
                .findFirst().orElse(null);
    }

    private Boolean filterRule(final RuleData ruleData, final ServerWebExchange exchange) {
        return ruleData.getEnabled() && MatchStrategyUtils.match(ruleData.getMatchMode(), ruleData.getConditionDataList(), exchange);
    }
```

此处需要提到的是匹配策略工具类 `MatchStrategyUtils`，

```java
public interface MatchStrategy {
    Boolean match(List<ConditionData> conditionDataList, ServerWebExchange exchange);
}
```

该接口分别是实现了两个子类，`AndMatchStrategy` 和 `OrMatchStrategy`。
