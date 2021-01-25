---
title: Soul 网关学习笔记09-nacos同步数据
abbrlink: 3b61fa21
date: 2021-01-25 21:29:04
tags:
categories:
---
探究 nacos 同步数据源码
<!--more-->

## 配置

- `soul-bootstrap` 模块引入依赖，如下：

```xml
    <dependency>
        <groupId>org.dromara</groupId>
        <artifactId>soul-spring-boot-starter-sync-data-nacos</artifactId>
        <version>${project.version}</version>
    </dependency>
```

- `soul-bootstrap` 模块配置文件修改，如下：

```yml
soul:
  sync:
    nacos:
      url: localhost:8848
      namespace: 1c10d748-af86-43b9-8265-75f487d20c6c
      acm:
        enabled: false
        endpoint: acm.aliyun.com
        namespace:
        accessKey:
        secretKey:
```

- `soul-admin` 模块配置文件修改，如下：

```yml
soul:
  sync:
    nacos:
      url: localhost:8848
      namespace: 1c10d748-af86-43b9-8265-75f487d20c6c
      acm:
        enabled: false
        endpoint: acm.aliyun.com
        namespace:
        accessKey:
        secretKey:
```

## `soul-admin` 模块

模块初始化时，加载数据同步配置类 `DataSyncConfiguration`，通过配置类中声明的静态内部类加载 `nacos` 相关配置和监听。值得注意的是 `@Import(NacosConfiguration.class)`，通过工厂类 `NacosFactory` 创建了实例 `ConfigService`。

```java
@Configuration
public class DataSyncConfiguration {
    ......
    @Configuration
    @ConditionalOnProperty(prefix = "soul.sync.nacos", name = "url")
    @Import(NacosConfiguration.class)
    static class NacosListener {
        @Bean
        @ConditionalOnMissingBean(NacosDataChangedListener.class)
        public DataChangedListener nacosDataChangedListener(final ConfigService configService) {
            return new NacosDataChangedListener(configService);
        }
        @Bean
        @ConditionalOnMissingBean(NacosDataInit.class)
        public NacosDataInit nacosDataInit(final ConfigService configService, final SyncDataService syncDataService) {
            return new NacosDataInit(configService, syncDataService);
        }
    }
    ......
}
```

从控制台页面触发插件或选择器或规则修改时，会进入监听类相关逻辑。

```java
public class NacosDataChangedListener implements DataChangedListener {
    ......
    @Override
    public void onAppAuthChanged(final List<AppAuthData> changed, final DataEventTypeEnum eventType) {......}
    @Override
    public void onPluginChanged(final List<PluginData> changed, final DataEventTypeEnum eventType) {......}
    @Override
    public void onSelectorChanged(final List<SelectorData> changed, final DataEventTypeEnum eventType) {......}
    @Override
    public void onMetaDataChanged(final List<MetaData> changed, final DataEventTypeEnum eventType) {......}
    @Override
    public void onRuleChanged(final List<RuleData> changed, final DataEventTypeEnum eventType) {......}
    ......
}
```

以修改规则为例，`onRuleChanged` 方法根据事件类型处理收到的规则内容，然后发布修改后的内容。

```java
public class NacosDataChangedListener implements DataChangedListener {
    ......
    private void publishConfig(final String dataId, final Object data) {
        configService.publishConfig(dataId, NacosPathConstants.GROUP, GsonUtils.getInstance().toJson(data));
    }
    ......
}
```

通过初始化时创建的类 `ConfigService`，一层层调用，最终通过 http 请求将修改的数据发布到网关。

## `soul-bootstrap` 模块

启动日志部分如下；

```log
INFO 7204 --- [           main] d.s.s.s.s.d.n.NacosSyncDataConfiguration : you use nacos sync soul data.......
```

模块初始化时，加载配置类 `NacosSyncDataConfiguration`，实例化 nacos 同步数据处理类 `NacosSyncDataService`，其类关系结构如图所示：

![类 NacosSyncDataService 结构图](/images/soul/diagram-NacosSyncDataService.png)

查看源码，构造方法首先实例化了父类 `NacosCacheHandler`，然后执行方法 `start`。方法 `watcherData` 注册配置文件中所有内容，通过的方法 `ConfigService#getConfigAndSignListener` 读取 nacos 中的数据。

```java
public class NacosSyncDataService extends NacosCacheHandler implements AutoCloseable, SyncDataService {
    public NacosSyncDataService(final ConfigService configService, final PluginDataSubscriber pluginDataSubscriber,
                                final List<MetaDataSubscriber> metaDataSubscribers, final List<AuthDataSubscriber> authDataSubscribers) {
        super(configService, pluginDataSubscriber, metaDataSubscribers, authDataSubscribers);
        start();
    }

    public void start() {
        watcherData(PLUGIN_DATA_ID, this::updatePluginMap);
        watcherData(SELECTOR_DATA_ID, this::updateSelectorMap);
        watcherData(RULE_DATA_ID, this::updateRuleMap);
        watcherData(META_DATA_ID, this::updateMetaDataMap);
        watcherData(AUTH_DATA_ID, this::updateAuthMap);
    }
    ......
}
```

仍然以修改规则为例，`NacosCacheHandler#updateRuleMap` 收到数据同步请求，通过 `CommonPluginDataSubscriber#onSubscribe` 将数据缓存到 `BaseDataCache` 中。
