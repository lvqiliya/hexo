---
title: Soul 网关学习笔记07-zookeeper同步数据
abbrlink: d6f6030f
date: 2021-01-22 22:45:24
tags:
categories:
---
探究 zookeeper 同步数据源码
<!--more-->
## 配置

- `soul-bootstrap` 模块引入依赖，如下：

```xml
     <dependency>
          <groupId>org.dromara</groupId>
           <artifactId>soul-spring-boot-starter-sync-data-zookeeper</artifactId>
           <version>${last.version}</version>
     </dependency>
```

- `soul-bootstrap` 模块配置文件修改，如下：

```yml
soul:
  sync:
    zookeeper:
      url: localhost:2181
      sessionTimeout: 5000
      connectionTimeout: 2000
```

- `soul-admin` 模块配置文件修改，如下：

```yml
soul:
  sync:
    zookeeper:
      url: localhost:2181
      sessionTimeout: 5000
      connectionTimeout: 2000
```

## 启动

- 运行 `zookeeper`，Windows 环境下双击运行 `zkServer.cmd`。
- 启动 `soul-admin` 模块，调用 `zookeeper` 服务。

```log
5692 --- [-localhost:2181] org.I0Itec.zkclient.ZkEventThread        : Starting ZkClient event thread.
......
5692 --- [localhost:2181)] org.apache.zookeeper.ClientCnxn          : Opening socket connection to server localhost/127.0.0.1:2181. Will not attempt to authenticate using SASL (unknown error)
5692 --- [localhost:2181)] org.apache.zookeeper.ClientCnxn          : Socket connection established, initiating session, client: /127.0.0.1:63443, server: localhost/127.0.0.1:2181
5692 --- [localhost:2181)] org.apache.zookeeper.ClientCnxn          : Session establishment complete on server localhost/127.0.0.1:2181, sessionid = 0x1000554f83b0003, negotiated timeout = 5000
[ain-EventThread] org.I0Itec.zkclient.ZkClient             : zookeeper state changed (SyncConnected)
```

- 启动 `soul-bootstrap` 模块，调用 `zookeeper` 服务。

```log
9700 --- [           main] s.b.s.d.z.ZookeeperSyncDataConfiguration : you use zookeeper sync soul data.......
9700 --- [-localhost:2181] org.I0Itec.zkclient.ZkEventThread        : Starting ZkClient event thread.
......
9700 --- [localhost:2181)] org.apache.zookeeper.ClientCnxn          : Opening socket connection to server localhost/127.0.0.1:2181. Will not attempt to authenticate using SASL (unknown error)
9700 --- [localhost:2181)] org.apache.zookeeper.ClientCnxn          : Socket connection established, initiating session, client: /127.0.0.1:63477, server: localhost/127.0.0.1:2181
9700 --- [localhost:2181)] org.apache.zookeeper.ClientCnxn          : Session establishment complete on server localhost/127.0.0.1:2181, sessionid = 0x1000554f83b0004, negotiated timeout = 5000
9700 --- [ain-EventThread] org.I0Itec.zkclient.ZkClient             : zookeeper state changed (SyncConnected)
```

## 同步数据源码解析

从 `soul-bootstrap` 模块启动日志入手，启动服务时自动装配了 zookeeper 同步数据配置类 `ZookeeperSyncDataConfiguration`。

```java
public class ZookeeperSyncDataConfiguration {

    /**
     * Sync data service sync data service.
     *
     * @param zkClient          the zk client
     * @param pluginSubscriber the plugin subscriber
     * @param metaSubscribers   the meta subscribers
     * @param authSubscribers   the auth subscribers
     * @return the sync data service
     */
    @Bean
    public SyncDataService syncDataService(final ObjectProvider<ZkClient> zkClient, final ObjectProvider<PluginDataSubscriber> pluginSubscriber,
                                           final ObjectProvider<List<MetaDataSubscriber>> metaSubscribers, final ObjectProvider<List<AuthDataSubscriber>> authSubscribers) {
        log.info("you use zookeeper sync soul data.......");
        return new ZookeeperSyncDataService(zkClient.getIfAvailable(), pluginSubscriber.getIfAvailable(),
                metaSubscribers.getIfAvailable(Collections::emptyList), authSubscribers.getIfAvailable(Collections::emptyList));
    }

    @Bean
    public ZkClient zkClient(final ZookeeperConfig zookeeperConfig) {
        return new ZkClient(zookeeperConfig.getUrl(), zookeeperConfig.getSessionTimeout(), zookeeperConfig.getConnectionTimeout());
    }
}
```

实例化 `ZookeeperSyncDataService`。

```java
public class ZookeeperSyncDataService implements SyncDataService, AutoCloseable {
    ......
    public ZookeeperSyncDataService(final ZkClient zkClient, final PluginDataSubscriber pluginDataSubscriber,
                                    final List<MetaDataSubscriber> metaDataSubscribers, final List<AuthDataSubscriber> authDataSubscribers) {
        this.zkClient = zkClient;
        this.pluginDataSubscriber = pluginDataSubscriber;
        this.metaDataSubscribers = metaDataSubscribers;
        this.authDataSubscribers = authDataSubscribers;
        watcherData();
        watchAppAuth();
        watchMetaData();
    }
    ......
}
```

顺序执行方法 `watcherData`、`watchAppAuth`、`watchMetaData`。

- 跟踪方法 `watcherData` 的源码，配合注释查看调用链，如下：

```java
    private void watcherData() {
        final String pluginParent = ZkPathConstants.PLUGIN_PARENT;
        // 从 zookeeper 获取节点列表
        List<String> pluginZKs = zkClientGetChildren(pluginParent);
        for (String pluginName : pluginZKs) {
            // 处理所有从 zookeeper 获取的插件内容
            watcherAll(pluginName);
        }
        // zookeeper 客户端订阅节点变化
        zkClient.subscribeChildChanges(pluginParent, (parentPath, currentChildren) -> {
            if (CollectionUtils.isNotEmpty(currentChildren)) {
                for (String pluginName : currentChildren) {
                    watcherAll(pluginName);
                }
            }
        });
    }

    private void watcherAll(final String pluginName) {
        watcherPlugin(pluginName);
        watcherSelector(pluginName);
        watcherRule(pluginName);
    }

    private void watcherPlugin(final String pluginName) {
        String pluginPath = ZkPathConstants.buildPluginPath(pluginName);
        if (!zkClient.exists(pluginPath)) {
            zkClient.createPersistent(pluginPath, true);
        }
        // 缓存插件数据
        cachePluginData(zkClient.readData(pluginPath));
        // 订阅插件数据
        subscribePluginDataChanges(pluginPath, pluginName);
    }

    private void subscribePluginDataChanges(final String pluginPath, final String pluginName) {
        zkClient.subscribeDataChanges(pluginPath, new IZkDataListener() {
            @Override
            public void handleDataChange(final String dataPath, final Object data) {
                // 调用 CommonPluginDataSubscriber 类的 onSubscribe 方法，具体可见系列文章 06
                Optional.ofNullable(data)
                        .ifPresent(d -> Optional.ofNullable(pluginDataSubscriber).ifPresent(e -> e.onSubscribe((PluginData) d)));
            }
            ......
        });
    }
```

查看调用栈：

![watcherData方法调用栈](/images/soul/watcherData.png)

***未完待续***
