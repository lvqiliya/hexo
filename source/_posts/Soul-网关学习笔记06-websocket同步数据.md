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
......
clients.add(new SoulWebsocketClient(new URI(url), Objects.requireNonNull(pluginDataSubscriber), metaDataSubscribers, authDataSubscribers));
......
```

明显的，此处创建了一个 `SoulWebsocketClient`。

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
    ......
    public void executor(final ConfigGroupEnum type, final String json, final String eventType) {
        ENUM_MAP.get(type).handle(json, eventType);
    }
}
```

所有的数据同步将从 `handle` 方法为起点。

***未完待续***
