---
title: Soul 网关学习笔记03-dubbo插件初体验
abbrlink: 1592c754
date: 2021-01-16 19:44:50
tags:
categories:
---
使用 divide 插件，体验 dubbo 代理
<!--more-->
## 启动项目

运行 soul-admin 模块的主函数，启动 soul-admin。

运行 soul-bootstrap 模块的主函数，启动 soul-bootstrap。

使用 admin 账号登录管理台，点击 PluginList-dubbo 模块，此时选择器列表（SelectorList）和规则列表（RuleList）均为空。可以手动添加选择器和规则。

检查 soul-bootstrap 模块下 dubbo 插件引入是否完备。

进入 soul-examples 模块下的 dubbo 子模块。

dubbo 服务接入网关三要素：

- 引入 soul 支持的依赖

```xml
<dependency>
    <groupId>org.dromara</groupId>
    <artifactId>soul-spring-boot-starter-client-alibaba-dubbo</artifactId>
    <version>${soul.version}</version>
</dependency>
```

- `application.yml` 中配置

```yml
soul:
  dubbo:
    adminUrl: http://localhost:9095
    contextPath: /dubbo
    appName: dubbo
```

- 代码中使用注解 `@SoulDubboClient`

## 源码初步分析

根据猫大人反复提到的假设猜想原则，假设 dubbo 插件源端进入方法是 `AlibabaDubboPlugin#doExecute`，使用断点进行验证，假设成立。

基于该方法向上回溯，由 `SoulWebHandler#handle` 中创建的静态内部类 `DefaultSoulPluginChain` 进行调用，进入插件抽象类 `AbstractSoulPlugin`，通过执行 `AbstractSoulPlugin#execute` 方法，调用子类的 `doExecute` 方法，即 `AlibabaDubboPlugin#doExecute`。

此处的设计模式是模板模式，抽象类定义最基本的处理流程，具体处理逻辑由子类实现。`AbstractSoulPlugin` 中的基本流程分别进行了匹配选择器、过滤选择器、匹配规则、过滤规则、调用子类执行逻辑。

```java
// 匹配选择器
final SelectorData selectorData = matchSelector(exchange, selectors);
// 过滤选择器
private SelectorData matchSelector(final ServerWebExchange exchange, final Collection<SelectorData> selectors) {
    return selectors.stream()
            .filter(selector -> selector.getEnabled() && filterSelector(selector, exchange))
            .findFirst().orElse(null);
}
// 匹配规则
RuleData rule;
......
rule = matchRule(exchange, rules);
// 过滤规则
private RuleData matchRule(final ServerWebExchange exchange, final Collection<RuleData> rules) {
    return rules.stream()
            .filter(rule -> filterRule(rule, exchange))
            .findFirst().orElse(null);
}
// 调用子类执行逻辑
return doExecute(exchange, chain, selectorData, rule);
```

接着 `AlibabaDubboPlugin#doExecute` 调用了 `AlibabaDubboProxyService#genericInvoker`，最终通过

```java
return genericService.$invoke(metaData.getMethodName(), pair.getLeft(), pair.getRight());
```

至此完成流程调用。

至于再更进一步的源码逻辑梳理，则还需时间进行调试。
