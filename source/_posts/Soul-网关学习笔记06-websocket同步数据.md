---
title: Soul 网关学习笔记06-websocket同步数据
tags: soul
categories: 源码学习
abbrlink: e756f54c
date: 2021-01-20 22:58:34
---
探究 websocket 同步数据源码
<!--more-->
## 配置

> ...... 但是，有一点需要注意的是，`soul-web` 和 `soul-admin` 必须使用相同的同步机制。

根据以上描述，检查 `soul-bootstrap` 和 `soul-admin` 模块的配置文件。补充说明一下 `soul-web` 和 `soul-bootstrap` 之间的关系，在 `soul-bootstrap` 中引入了 `soul-spring-boot-starter-gateway` 模块，而该模块依赖于 `soul-web` 模块。观察项目代码结构，`soul-bootstrap` 除了主函数仅剩下两个类。

> 猜想：`soul-bootstrap` 模块中一系列配置都放在 `soul-web` 模块。

一个简单的证明。启动 `soul-bootstrap` 模块时，控制台打印 soul logo 的逻辑就在 `soul-web` 模块下。

```log
2021-01-20 23:34:46.842  INFO 13700 --- [main] org.dromara.soul.web.logo.SoulLogo       : 

     | | 
 ___  ___  _   _| | 
/ __|/ _ \| | | | |
\__ \ (_) | |_| | |
|___/\___/ \__,_|_|

 :: Soul :: (v2.0.2)
```

- `soul-bootstrap` 模块启用 `websocket` 进行数据同步

```yml
soul :
    sync:
        websocket :
  urls: ws://localhost:9095/websocket
```

- `soul-admin` 模块启用 `websocket` 进行数据同步

```yml
soul:
  sync:
    websocket:
      enabled: true
```

## 启动

```log
[main] o.d.s.b.SoulBootstrapApplication         : Starting SoulBootstrapApplication on DESKTOP-E96NI96 with PID 9860
[main] o.d.s.b.SoulBootstrapApplication         : The following profiles are active: local
[main] .s.d.r.c.RepositoryConfigurationDelegate : Multiple Spring Data modules found, entering strict repository configuration mode!
[main] .s.d.r.c.RepositoryConfigurationDelegate : Bootstrapping Spring Data Redis repositories in DEFAULT mode.
[main] .s.d.r.c.RepositoryConfigurationDelegate : Finished Spring Data repository scanning in 13ms. Found 0 Redis repository interfaces.
[main] o.d.s.w.configuration.SoulConfiguration  : load plugin:[global] [org.dromara.soul.plugin.global.GlobalPlugin]
[main] o.d.s.w.configuration.SoulConfiguration  : load plugin:[sign] [org.dromara.soul.plugin.sign.SignPlugin]
[main] o.d.s.w.configuration.SoulConfiguration  : load plugin:[waf] [org.dromara.soul.plugin.waf.WafPlugin]
[main] o.d.s.w.configuration.SoulConfiguration  : load plugin:[rate_limiter] [org.dromara.soul.plugin.ratelimiter.RateLimiterPlugin]
[main] o.d.s.w.configuration.SoulConfiguration  : load plugin:[hystrix] [org.dromara.soul.plugin.hystrix.HystrixPlugin]
[main] o.d.s.w.configuration.SoulConfiguration  : load plugin:[resilience4j] [org.dromara.soul.plugin.resilience4j.Resilience4JPlugin]
[main] o.d.s.w.configuration.SoulConfiguration  : load plugin:[divide] [org.dromara.soul.plugin.divide.DividePlugin]
[main] o.d.s.w.configuration.SoulConfiguration  : load plugin:[webClient] [org.dromara.soul.plugin.httpclient.WebClientPlugin]
[main] o.d.s.w.configuration.SoulConfiguration  : load plugin:[divide] [org.dromara.soul.plugin.divide.websocket.WebSocketPlugin]
[main] o.d.s.w.configuration.SoulConfiguration  : load plugin:[monitor] [org.dromara.soul.plugin.monitor.MonitorPlugin]
[main] o.d.s.w.configuration.SoulConfiguration  : load plugin:[response] [org.dromara.soul.plugin.httpclient.response.WebClientResponsePlugin]
[main] b.s.s.d.w.WebsocketSyncDataConfiguration : you use websocket sync soul data.......
[main] o.d.s.p.s.d.w.WebsocketSyncDataService   : websocket connection is successful.....
```

分析启动日志不难发现，主函数根据配置文件，先后运行了 `SoulConfiguration`、`WebsocketSyncDataConfiguration`、`WebsocketSyncDataService` 这三个类。查看源码发现 `WebsocketSyncDataConfiguration` 调用了 `WebsocketSyncDataService`。

```java
public class WebsocketSyncDataConfiguration {
    // 读取配置文件中 websocket 的值，即 ws://localhost:9095/websocket
    @Bean
    @ConfigurationProperties(prefix = "soul.sync.websocket")
    public WebsocketConfig websocketConfig() {
        // WebsocketConfig 仅包含一个私有属性 urls
        return new WebsocketConfig();
    }

    @Bean
    public SyncDataService websocketSyncDataService(
            // ws://localhost:9095/websocket
            final ObjectProvider<WebsocketConfig> websocketConfig,
            final ObjectProvider<PluginDataSubscriber> pluginSubscriber,
            final ObjectProvider<List<MetaDataSubscriber>> metaSubscribers,
            final ObjectProvider<List<AuthDataSubscriber>> authSubscribers) {
        log.info("you use websocket sync soul data.......");
        return new WebsocketSyncDataService(websocketConfig.getIfAvailable(WebsocketConfig::new), pluginSubscriber.getIfAvailable(),
     metaSubscribers.getIfAvailable(Collections::emptyList), authSubscribers.getIfAvailable(Collections::emptyList));
    }
}
```

> 问题：pluginSubscriber、metaSubscribers、authSubscribers 分别从哪里来，分别是什么内容？

接着进入 `WebsocketSyncDataService` 源码部分。

```java
public class WebsocketSyncDataService implements SyncDataService, AutoCloseable {
    ......
    public WebsocketSyncDataService(final WebsocketConfig websocketConfig,
                                    final PluginDataSubscriber pluginDataSubscriber,
                                    final List<MetaDataSubscriber> metaDataSubscribers,
                                    final List<AuthDataSubscriber> authDataSubscribers) {
        String[] urls = StringUtils.split(websocketConfig.getUrls(), ",");
        executor = new ScheduledThreadPoolExecutor(urls.length, SoulThreadFactory.create("websocket-connect", true));
        for (String url : urls) {
            try {
                clients.add(new SoulWebsocketClient(new URI(url), Objects.requireNonNull(pluginDataSubscriber), metaDataSubscribers, authDataSubscribers));
            } catch (URISyntaxException e) {
                log.error("websocket url({}) is error", url, e);
            }
        }
        try {
            for (WebSocketClient client : clients) {
                boolean success = client.connectBlocking(3000, TimeUnit.MILLISECONDS);
                if (success) {
                    log.info("websocket connection is successful.....");
                } else {
                    log.error("websocket connection is error.....");
                }
                executor.scheduleAtFixedRate(() -> {
                    try {
                        if (client.isClosed()) {
                            boolean reconnectSuccess = client.reconnectBlocking();
                            if (reconnectSuccess) {
                                log.info("websocket reconnect is successful.....");
                            } else {
                                log.error("websocket reconnection is error.....");
                            }
                        }
                    } catch (InterruptedException e) {
                        log.error("websocket connect is error :{}", e.getMessage());
                    }
                }, 10, 30, TimeUnit.SECONDS);
            }
            /* client.setProxy(new Proxy(Proxy.Type.HTTP, new InetSocketAddress("proxyaddress", 80)));*/
        } catch (InterruptedException e) {
            log.info("websocket connection...exception....", e);
        }
    }
    ......
}
```

明显的，此处创建了一个 `SoulWebsocketClient`，然后将其丢入线程池类，每 30s 判断一次 `websocket` 服务是否连接正常。

> 猜想：数据同步将从类 `SoulWebsocketClient` 开始。

重启 `soul-bootstrap` 并在类 `SoulWebsocketClient` 中进行断点调试。断点在节选代码片段第九行。

```java
public final class SoulWebsocketClient extends WebSocketClient {
    ......
    @SuppressWarnings("ALL")
    private void handleResult(final String result) {
        WebsocketData websocketData = GsonUtils.getInstance().fromJson(result, WebsocketData.class);
        ConfigGroupEnum groupEnum = ConfigGroupEnum.acquireByName(websocketData.getGroupType());
        String eventType = websocketData.getEventType();
        String json = GsonUtils.getInstance().toJson(websocketData.getData());
        websocketDataHandler.executor(groupEnum, json, eventType);
    }
}
```

成功进入断点，查看线程栈，证明猜想正确。

![SoulWebsocketClient-debug-01](/images/soul/SoulWebsocketClient-debug-01.png)

紧接着查看类 `websocketDataHandler`。

```java
public class WebsocketDataHandler {
    private static final EnumMap<ConfigGroupEnum, DataHandler> ENUM_MAP = new EnumMap<>(ConfigGroupEnum.class);

    /**
     * Instantiates a new Websocket data handler.
     *
     * @param pluginDataSubscriber the plugin data subscriber
     * @param metaDataSubscribers  the meta data subscribers
     * @param authDataSubscribers  the auth data subscribers
     */
    public WebsocketDataHandler(final PluginDataSubscriber pluginDataSubscriber,
                                final List<MetaDataSubscriber> metaDataSubscribers,
                                final List<AuthDataSubscriber> authDataSubscribers) {
        ENUM_MAP.put(ConfigGroupEnum.PLUGIN, new PluginDataHandler(pluginDataSubscriber));
        ENUM_MAP.put(ConfigGroupEnum.SELECTOR, new SelectorDataHandler(pluginDataSubscriber));
        ENUM_MAP.put(ConfigGroupEnum.RULE, new RuleDataHandler(pluginDataSubscriber));
        ENUM_MAP.put(ConfigGroupEnum.APP_AUTH, new AuthDataHandler(authDataSubscribers));
        ENUM_MAP.put(ConfigGroupEnum.META_DATA, new MetaDataHandler(metaDataSubscribers));
    }

    public void executor(final ConfigGroupEnum type, final String json, final String eventType) {
        ENUM_MAP.get(type).handle(json, eventType);
    }
}
```

所有的数据同步将从 `handle` 方法为起点。

---
21日更新

## 调用链解析

书接上文，通过 `WebsocketDataHandler` 的构造器可以看到，参数包括了三个 Subscriber。接口 `MetaDataSubscriber` 和接口 `AuthDataSubscriber` 都定义了 `onSubscribe`、`unSubscribe`、`refresh` 三个方法，它们的实现类分别实现了前两个方法，后一个方法是 Java 8 引入了默认方法，目前源码中没有给定实现。接口 `PluginDataSubscriber` 定义了三组，每组四个，共十二个默认方法。分别包括对插件的处理、对选择器的处理、对规则的处理。其实现类完成了具体的逻辑处理——更新和删除操作。

```java
public interface PluginDataSubscriber {
    default void onSubscribe(PluginData pluginData) {}
    default void unSubscribe(PluginData pluginData) {}
    default void refreshPluginDataAll() {}
    default void refreshPluginDataSelf(List<PluginData> pluginDataList) {}
    default void onSelectorSubscribe(SelectorData selectorData) {}
    default void unSelectorSubscribe(SelectorData selectorData) {}
    default void refreshSelectorDataAll() {}
    default void refreshSelectorDataSelf(List<SelectorData> selectorDataList) {}
    default void onRuleSubscribe(RuleData ruleData) {}
    default void unRuleSubscribe(RuleData ruleData) {}
    default void refreshRuleDataAll() {}
    default void refreshRuleDataSelf(List<RuleData> ruleDataList) {}
}
```

上文所提到的 `handle` 方法具体实现逻辑是在抽象类 `AbstractDataHandler`，除此之外该类还定义了四个抽象方法。众所周知子类需具体实现抽象方法，根据 `handle` 方法传入的参数 `type` 自动选择具体子类去实现刷新、创建、删除等操作。

```java
public abstract class AbstractDataHandler<T> implements DataHandler {
    protected abstract List<T> convert(String json);
    protected abstract void doRefresh(List<T> dataList);
    protected abstract void doUpdate(List<T> dataList);
    protected abstract void doDelete(List<T> dataList);

    @Override
    public void handle(final String json, final String eventType) {
        List<T> dataList = convert(json);
        if (CollectionUtils.isNotEmpty(dataList)) {
            DataEventTypeEnum eventTypeEnum = DataEventTypeEnum.acquireByName(eventType);
            switch (eventTypeEnum) {
                case REFRESH:
                case MYSELF:
                    doRefresh(dataList);
                    break;
                case UPDATE:
                case CREATE:
                    doUpdate(dataList);
                    break;
                case DELETE:
                    doDelete(dataList);
                    break;
                default:
                    break;
            }
        }
    }
}
```

启动 `soul-bootstrap` 模块时，插件处理的事件类型为 `MYSELF`，根据抽象类的判断逻辑，执行 `PluginDataHandler#doRefresh` 方法。先执行公共插件数据订阅者类 `CommonPluginDataSubscriber` 的刷新方法，即清理掉缓存类 `BaseDataCache` 中现有的插件数据。然后将插件数据再次缓存到 `BaseDataCache`中。

```java
public class PluginDataHandler extends AbstractDataHandler<PluginData> {
    ......
    protected void doRefresh(final List<PluginData> dataList) {
        pluginDataSubscriber.refreshPluginDataSelf(dataList);
        dataList.forEach(pluginDataSubscriber::onSubscribe);
    }
    ......
}
```

同理，对于选择器和规则的处理与插件的处理逻辑一样，不再赘述。三者的核心处理逻辑则是以下代码。

```java
public class CommonPluginDataSubscriber implements PluginDataSubscriber {
    ......
    private <T> void subscribeDataHandler(final T classData, final DataEventTypeEnum dataType) {
        Optional.ofNullable(classData).ifPresent(data -> {
            if (data instanceof PluginData) {
                PluginData pluginData = (PluginData) data;
                if (dataType == DataEventTypeEnum.UPDATE) {
                    BaseDataCache.getInstance().cachePluginData(pluginData);
                    Optional.ofNullable(handlerMap.get(pluginData.getName())).ifPresent(handler -> handler.handlerPlugin(pluginData));
                } else if (dataType == DataEventTypeEnum.DELETE) {
                    BaseDataCache.getInstance().removePluginData(pluginData);
                    Optional.ofNullable(handlerMap.get(pluginData.getName())).ifPresent(handler -> handler.removePlugin(pluginData));
                }
            } else if (data instanceof SelectorData) {
                SelectorData selectorData = (SelectorData) data;
                if (dataType == DataEventTypeEnum.UPDATE) {
                    BaseDataCache.getInstance().cacheSelectData(selectorData);
                    Optional.ofNullable(handlerMap.get(selectorData.getPluginName())).ifPresent(handler -> handler.handlerSelector(selectorData));
                } else if (dataType == DataEventTypeEnum.DELETE) {
                    BaseDataCache.getInstance().removeSelectData(selectorData);
                    Optional.ofNullable(handlerMap.get(selectorData.getPluginName())).ifPresent(handler -> handler.removeSelector(selectorData));
                }
            } else if (data instanceof RuleData) {
                RuleData ruleData = (RuleData) data;
                if (dataType == DataEventTypeEnum.UPDATE) {
                    BaseDataCache.getInstance().cacheRuleData(ruleData);
                    Optional.ofNullable(handlerMap.get(ruleData.getPluginName())).ifPresent(handler -> handler.handlerRule(ruleData));
                } else if (dataType == DataEventTypeEnum.DELETE) {
                    BaseDataCache.getInstance().removeRuleData(ruleData);
                    Optional.ofNullable(handlerMap.get(ruleData.getPluginName())).ifPresent(handler -> handler.removeRule(ruleData));
                }
            }
        });
    }
}
```

接下来需要针对 `handler.handlerPlugin(pluginData)`、`handler.handlerSelector(selectorData)`、`handler.handlerRule(ruleData)` 三者进行分析。

---
未完待续

---
2月4日更新

## `soul-admin` 模块

查看 soul-admin 模块的配置类 DataSyncConfiguration。以下代码是 `websocket` 相关的初始化。

```java
@Configuration
public class DataSyncConfiguration {
    ......
    @Configuration
    @ConditionalOnProperty(name = "soul.sync.websocket.enabled", havingValue = "true", matchIfMissing = true)
    @EnableConfigurationProperties(WebsocketSyncProperties.class)
    static class WebsocketListener {
        @Bean
        @ConditionalOnMissingBean(WebsocketDataChangedListener.class)
        public DataChangedListener websocketDataChangedListener() {
            return new WebsocketDataChangedListener();
        }

        @Bean
        @ConditionalOnMissingBean(WebsocketCollector.class)
        public WebsocketCollector websocketCollector() {
            return new WebsocketCollector();
        }

        @Bean
        @ConditionalOnMissingBean(ServerEndpointExporter.class)
        public ServerEndpointExporter serverEndpointExporter() {
            return new ServerEndpointExporter();
        }
    }
}
```

在深入探究实例化的三个类之前，需要特殊说明一下类 `DataChangedEventDispatcher`。该类实现了 `ApplicationListener`，利用Spring事件监听机制，循环接口 `DataChangedListener` 的实现类——其中包括配置类实例化的 `WebsocketDataChangedListener`——调用具体的处理逻辑。

```java
@Component
public class DataChangedEventDispatcher implements ApplicationListener<DataChangedEvent>, InitializingBean {

    private ApplicationContext applicationContext;

    private List<DataChangedListener> listeners;

    public DataChangedEventDispatcher(final ApplicationContext applicationContext) {
        this.applicationContext = applicationContext;
    }

    @Override
    @SuppressWarnings("unchecked")
    public void onApplicationEvent(final DataChangedEvent event) {
        for (DataChangedListener listener : listeners) {
            switch (event.getGroupKey()) {
                case APP_AUTH:
                    listener.onAppAuthChanged((List<AppAuthData>) event.getSource(), event.getEventType());
                    break;
                case PLUGIN:
                    listener.onPluginChanged((List<PluginData>) event.getSource(), event.getEventType());
                    break;
                case RULE:
                    listener.onRuleChanged((List<RuleData>) event.getSource(), event.getEventType());
                    break;
                case SELECTOR:
                    listener.onSelectorChanged((List<SelectorData>) event.getSource(), event.getEventType());
                    break;
                case META_DATA:
                    listener.onMetaDataChanged((List<MetaData>) event.getSource(), event.getEventType());
                    break;
                default:
                    throw new IllegalStateException("Unexpected value: " + event.getGroupKey());
            }
        }
    }

    @Override
    public void afterPropertiesSet() {
        Collection<DataChangedListener> listenerBeans = applicationContext.getBeansOfType(DataChangedListener.class).values();
        this.listeners = Collections.unmodifiableList(new ArrayList<>(listenerBeans));
    }
}
```

- `WebsocketDataChangedListener`

当监听到数据变化时，该类调用方法 `WebsocketCollector#send`，将变化的数据发给网关。核心代码如下。

```java
public class WebsocketDataChangedListener implements DataChangedListener {

    @Override
    public void onPluginChanged(final List<PluginData> pluginDataList, final DataEventTypeEnum eventType) {
        WebsocketData<PluginData> websocketData =
                new WebsocketData<>(ConfigGroupEnum.PLUGIN.name(), eventType.name(), pluginDataList);
        WebsocketCollector.send(GsonUtils.getInstance().toJson(websocketData), eventType);
    }

    @Override
    public void onSelectorChanged(final List<SelectorData> selectorDataList, final DataEventTypeEnum eventType) {
        WebsocketData<SelectorData> websocketData =
                new WebsocketData<>(ConfigGroupEnum.SELECTOR.name(), eventType.name(), selectorDataList);
        WebsocketCollector.send(GsonUtils.getInstance().toJson(websocketData), eventType);
    }

    @Override
    public void onRuleChanged(final List<RuleData> ruleDataList, final DataEventTypeEnum eventType) {
        WebsocketData<RuleData> configData =
                new WebsocketData<>(ConfigGroupEnum.RULE.name(), eventType.name(), ruleDataList);
        WebsocketCollector.send(GsonUtils.getInstance().toJson(configData), eventType);
    }

    @Override
    public void onAppAuthChanged(final List<AppAuthData> appAuthDataList, final DataEventTypeEnum eventType) {
        WebsocketData<AppAuthData> configData =
                new WebsocketData<>(ConfigGroupEnum.APP_AUTH.name(), eventType.name(), appAuthDataList);
        WebsocketCollector.send(GsonUtils.getInstance().toJson(configData), eventType);
    }

    @Override
    public void onMetaDataChanged(final List<MetaData> metaDataList, final DataEventTypeEnum eventType) {
        WebsocketData<MetaData> configData =
                new WebsocketData<>(ConfigGroupEnum.META_DATA.name(), eventType.name(), metaDataList);
        WebsocketCollector.send(GsonUtils.getInstance().toJson(configData), eventType);
    }
}
```

- `WebsocketCollector`

前面提到了当监听到数据变化时，调用 `send` 方法同步数据到网关。

```java
@ServerEndpoint("/websocket")
public class WebsocketCollector {
    ......
    public static void send(final String message, final DataEventTypeEnum type) {
        if (StringUtils.isNotBlank(message)) {
            if (DataEventTypeEnum.MYSELF == type) {
                Session session = (Session) ThreadLocalUtil.get(SESSION_KEY);
                if (session != null) {
                    sendMessageBySession(session, message);
                }
            } else {
                SESSION_SET.forEach(session -> sendMessageBySession(session, message));
            }
        }
    }

    private static void sendMessageBySession(final Session session, final String message) {
        try {
            session.getBasicRemote().sendText(message);
        } catch (IOException e) {
            log.error("websocket send result is exception: ", e);
        }
    }
}
```

- `ServerEndpointExporter`
