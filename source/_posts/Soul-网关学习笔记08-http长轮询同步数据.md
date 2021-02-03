---
title: Soul 网关学习笔记08-http长轮询同步数据
abbrlink: 54370cbe
date: 2021-01-24 20:45:27
tags:
categories:
---
探究 http 长轮询同步数据源码
<!--more-->

## 配置

- `soul-bootstrap` 模块引入依赖，如下：

```xml
    <dependency>
      <groupId>org.dromara</groupId>
      <artifactId>soul-spring-boot-starter-sync-data-http</artifactId>
      <version>${last.version}</version>
    </dependency>
```

- `soul-bootstrap` 模块配置文件修改，如下：

```yml
soul :
  sync:
    http:
      url: http://localhost:9095
```

- `soul-admin` 模块配置文件修改，如下：

```yml
soul:
  sync:
    http:
      enabled: true
```

## `soul-admin` 模块

### 启动

```log
INFO 16128 --- [           main] o.d.s.a.l.AbstractDataChangedListener    : update config cache[APP_AUTH], old: null, updated: {group='APP_AUTH', md5='d751713988987e9331980363e24189ce', lastModifyTime=1611491946818}
INFO 16128 --- [           main] o.d.s.a.l.AbstractDataChangedListener    : update config cache[PLUGIN], old: null, updated: {group='PLUGIN', md5='25948545f811d0141479f2be00bbbf08', lastModifyTime=1611491946829}
INFO 16128 --- [           main] o.d.s.a.l.AbstractDataChangedListener    : update config cache[RULE], old: null, updated: {group='RULE', md5='8f211189a0fa50f466e84d71676bd489', lastModifyTime=1611491946938}
INFO 16128 --- [           main] o.d.s.a.l.AbstractDataChangedListener    : update config cache[SELECTOR], old: null, updated: {group='SELECTOR', md5='49d6c1625ebb5504ed442822b3ceb61f', lastModifyTime=1611491946950}
INFO 16128 --- [           main] o.d.s.a.l.AbstractDataChangedListener    : update config cache[META_DATA], old: null, updated: {group='META_DATA', md5='75b739c6c58f1848c54b770894c5a19a', lastModifyTime=1611491946957}
INFO 16128 --- [           main] a.l.h.HttpLongPollingDataChangedListener : http sync strategy refresh interval: 300000ms
```

根据日志找到 `soul-admin` 模块中的抽象类 `AbstractDataChangedListener` 和类 `HttpLongPollingDataChangedListener`。类之间关系有如图关系：

![类 HttpLongPollingDataChangedListener 结构图](/images/soul/diagram-HttpLongPollingDataChangedListener.png)

- `AbstractDataChangedListener` 关键代码

```java
public abstract class AbstractDataChangedListener implements DataChangedListener, InitializingBean {
    ......
    @Override
    public final void afterPropertiesSet() {
        updateAppAuthCache();
        updatePluginCache();
        updateRuleCache();
        updateSelectorCache();
        updateMetaDataCache();
        afterInitialize();
    }

    protected abstract void afterInitialize();
    ......
}
```

由于实现了接口 `InitializingBean`，故项目启动之后会执行方法 `afterPropertiesSet`。首先分别更新 `APP_AUTH`、`PLUGIN`、`RULE`、`SELECTOR`、`META_DATA` 的缓存，然后执行初始化方法 `afterInitialize`，该抽象方法由子类实现具体逻辑。

- `HttpLongPollingDataChangedListener` 关键代码

```java
public class HttpLongPollingDataChangedListener extends AbstractDataChangedListener {
    ......
    @Override
    protected void afterInitialize() {
        // 刷新时间 5 min
        long syncInterval = httpSyncProperties.getRefreshInterval().toMillis();
        // Periodically check the data for changes and update the cache
        scheduler.scheduleWithFixedDelay(() -> {
            log.info("http sync strategy refresh config start.");
            try {
                this.refreshLocalCache();
                log.info("http sync strategy refresh config success.");
            } catch (Exception e) {
                log.error("http sync strategy refresh config error!", e);
            }
        }, syncInterval, syncInterval, TimeUnit.MILLISECONDS);
        log.info("http sync strategy refresh interval: {}ms", syncInterval);
    }

    private void refreshLocalCache() {
        this.updateAppAuthCache();
        this.updatePluginCache();
        this.updateRuleCache();
        this.updateSelectorCache();
        this.updateMetaDataCache();
    }
    ......
```

方法 `afterInitialize` 首先获取了指定刷新时间，然后利用线程池延迟定时更新缓存，更新操作复用了抽象父类的逻辑。

### 流程分析

- 根据日志断点分析

从页面随意修改divide插件中的选择器或这规则，笔者修改了规则，观察日志出现以下内容：

```log
INFO 2012 --- [-long-polling-2] a.l.h.HttpLongPollingDataChangedListener : send response with the changed group,ip=127.0.0.1, group=RULE, changeTime=1611501194871
```

根据日志找到打印位置：

```java
public class HttpLongPollingDataChangedListener extends AbstractDataChangedListener {
    ......
    protected void afterRuleChanged(final List<RuleData> changed, final DataEventTypeEnum eventType) {
        scheduler.execute(new DataChangeTask(ConfigGroupEnum.RULE));
    }
    ......
    class DataChangeTask implements Runnable {
        ......
        @Override
        public void run() {
            for (Iterator<LongPollingClient> iter = clients.iterator(); iter.hasNext();) {
                LongPollingClient client = iter.next();
                iter.remove();
                client.sendResponse(Collections.singletonList(groupKey));
                log.info("send response with the changed group,ip={}, group={}, changeTime={}", client.ip, groupKey, changeTime);
            }
        }
    }
}
```

在日志记录行打上断点，并再次修改选择器或规则，查看调用栈：

![stack-trace-DataChangeTask.png](/images/soul/stack-trace-DataChangeTask.png)

`DataChangedEventDispatcher#onApplicationEvent` 监听到了这次修改并对其做出处理。

> 疑问
> 其一：修改规则之后跟踪调用栈，当执行到上述源码 `afterRuleChanged` 时，其参数哪里用到？
> 其二：new 出来的 `DataChangeTask` 中使用迭代器操作了 `clients`，何时向其添加的 `LongPollingClient`？

- 根据官方文档断点分析

![官方图 http-long-polling](/images/soul/http-long-polling.png)

由官方图 **http-long-polling** 可得 `soul-admin` 模块的入口在类 `ConfigController`，查看该类及调用类的关键源码。

```java
public class ConfigController {
    ......
    @PostMapping(value = "/listener")
    public void listener(final HttpServletRequest request, final HttpServletResponse response) {
        longPollingListener.doLongPolling(request, response);
    }
}

public class HttpLongPollingDataChangedListener extends AbstractDataChangedListener {
    private final BlockingQueue<LongPollingClient> clients;
    ......
    public void doLongPolling(final HttpServletRequest request, final HttpServletResponse response) {

        // compare group md5
        List<ConfigGroupEnum> changedGroup = compareChangedGroup(request);
        String clientIp = getRemoteIp(request);

        // response immediately.
        if (CollectionUtils.isNotEmpty(changedGroup)) {
            this.generateResponse(response, changedGroup);
            log.info("send response with the changed group, ip={}, group={}", clientIp, changedGroup);
            return;
        }

        // listen for configuration changed.
        final AsyncContext asyncContext = request.startAsync();

        // AsyncContext.settimeout() does not timeout properly, so you have to control it yourself
        asyncContext.setTimeout(0L);

        // block client's thread.
        // HttpConstants.SERVER_MAX_HOLD_TIMEOUT = 60 MILLISECONDS
        scheduler.execute(new LongPollingClient(asyncContext, clientIp, HttpConstants.SERVER_MAX_HOLD_TIMEOUT));
    }
    ......
    class LongPollingClient implements Runnable {
        ......
        LongPollingClient(final AsyncContext ac, final String ip, final long timeoutTime) {
            this.asyncContext = ac;
            this.ip = ip;
            this.timeoutTime = timeoutTime;
        }

        @Override
        public void run() {
            this.asyncTimeoutFuture = scheduler.schedule(() -> {
                clients.remove(LongPollingClient.this);
                List<ConfigGroupEnum> changedGroups = compareChangedGroup((HttpServletRequest) asyncContext.getRequest());
                sendResponse(changedGroups);
            }, timeoutTime, TimeUnit.MILLISECONDS);
            clients.add(this);
        }
        ......
    }
}
```

以上源码大意时将待执行任务放入了阻塞队列中，然后通过线程池延时 60 MILLISECONDS 后执行异步任务并移除任务。

---
> 疑问：
> 以上两个不同的断点分析有何联系和区别？

---
未完待续

---
2月3日更新

## `soul-bootstrap` 模块

既然是探究 http 长轮询同步数据，那么直接找到同步数据模块 `soul-sync-data-center`，查看子模块 `soul-sync-data-http`。查看配置类 `HttpSyncDataConfiguration`。

```java
@Configuration
@ConditionalOnClass(HttpSyncDataService.class)
@ConditionalOnProperty(prefix = "soul.sync.http", name = "url")
@Slf4j
public class HttpSyncDataConfiguration {
    @Bean
    public SyncDataService httpSyncDataService(final ObjectProvider<HttpConfig> httpConfig, final ObjectProvider<PluginDataSubscriber> pluginSubscriber,
                                           final ObjectProvider<List<MetaDataSubscriber>> metaSubscribers, final ObjectProvider<List<AuthDataSubscriber>> authSubscribers) {
        log.info("you use http long pull sync soul data");
        return new HttpSyncDataService(Objects.requireNonNull(httpConfig.getIfAvailable()), Objects.requireNonNull(pluginSubscriber.getIfAvailable()),
                metaSubscribers.getIfAvailable(Collections::emptyList), authSubscribers.getIfAvailable(Collections::emptyList));
    }
    @Bean
    @ConfigurationProperties(prefix = "soul.sync.http")
    public HttpConfig httpConfig() {
        return new HttpConfig();
    }
}
```

配置类读取了配置文件中同步数据的配置，并初始化了 http 同步数据类 `HttpSyncDataService`。

```java
public class HttpSyncDataService implements SyncDataService, AutoCloseable {
    ......
    public HttpSyncDataService(final HttpConfig httpConfig, final PluginDataSubscriber pluginDataSubscriber,
                               final List<MetaDataSubscriber> metaDataSubscribers, final List<AuthDataSubscriber> authDataSubscribers) {
        this.factory = new DataRefreshFactory(pluginDataSubscriber, metaDataSubscribers, authDataSubscribers);
        this.httpConfig = httpConfig;
        this.serverList = Lists.newArrayList(Splitter.on(",").split(httpConfig.getUrl()));
        this.httpClient = createRestTemplate();
        this.start();
    }
    ......
    private void start() {
        // It could be initialized multiple times, so you need to control that.
        if (RUNNING.compareAndSet(false, true)) {
            // fetch all group configs.
            this.fetchGroupConfig(ConfigGroupEnum.values());
            int threadSize = serverList.size();
            this.executor = new ThreadPoolExecutor(threadSize, threadSize, 60L, TimeUnit.SECONDS,
                    new LinkedBlockingQueue<>(),
                    SoulThreadFactory.create("http-long-polling", true));
            // start long polling, each server creates a thread to listen for changes.
            this.serverList.forEach(server -> this.executor.execute(new HttpLongPollingTask(server)));
        } else {
            log.info("soul http long polling was started, executor=[{}]", executor);
        }
    }
}
```

构造函数的最后一步调用了方法 `start`，通过原子类的CAS方法 `RUNNING` 改为 `true`。进入判断，首先调用方法 `fetchGroupConfig`，然后初始化了一个核心线程数、最大线程数都为配置文件字段 `soul.sync.http.urls` 大小的线程池，然后循环 url 执行 `HttpLongPollingTask`。

```java
    class HttpLongPollingTask implements Runnable {
        private String server;
        private final int retryTimes = 3;
        HttpLongPollingTask(final String server) {
            this.server = server;
        }
        @Override
        public void run() {
            while (RUNNING.get()) {
                for (int time = 1; time <= retryTimes; time++) {
                    try {
                        doLongPolling(server);
                    } catch (Exception e) {
                        // print warnning log.
                        if (time < retryTimes) {
                            log.warn("Long polling failed, tried {} times, {} times left, will be suspended for a while! {}",
                                    time, retryTimes - time, e.getMessage());
                            ThreadUtils.sleep(TimeUnit.SECONDS, 5);
                            continue;
                        }
                        // print error, then suspended for a while.
                        log.error("Long polling failed, try again after 5 minutes!", e);
                        ThreadUtils.sleep(TimeUnit.MINUTES, 5);
                    }
                }
            }
            log.warn("Stop http long polling.");
        }
    }
```

方法 `run` 中，因为 `RUNNING` 的值为 `true`，所以是死循环。

进入死循环并根据 `retryTimes` 再次循环执行方法 `doLongPolling` 用于同步数据。

```java
    private void doLongPolling(final String server) {
        MultiValueMap<String, String> params = new LinkedMultiValueMap<>(8);
        for (ConfigGroupEnum group : ConfigGroupEnum.values()) {
            ConfigData<?> cacheConfig = factory.cacheConfigData(group);
            String value = String.join(",", cacheConfig.getMd5(), String.valueOf(cacheConfig.getLastModifyTime()));
            params.put(group.name(), Lists.newArrayList(value));
        }
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);
        HttpEntity httpEntity = new HttpEntity(params, headers);
        String listenerUrl = server + "/configs/listener";
        log.debug("request listener configs: [{}]", listenerUrl);
        JsonArray groupJson = null;
        try {
            String json = this.httpClient.postForEntity(listenerUrl, httpEntity, String.class).getBody();
            log.debug("listener result: [{}]", json);
            groupJson = GSON.fromJson(json, JsonObject.class).getAsJsonArray("data");
        } catch (RestClientException e) {
            String message = String.format("listener configs fail, server:[%s], %s", server, e.getMessage());
            throw new SoulException(message, e);
        }
        if (groupJson != null) {
            // fetch group configuration async.
            ConfigGroupEnum[] changedGroups = GSON.fromJson(groupJson, ConfigGroupEnum[].class);
            if (ArrayUtils.isNotEmpty(changedGroups)) {
                log.info("Group config changed: {}", Arrays.toString(changedGroups));
                this.doFetchGroupConfig(server, changedGroups);
            }
        }
    }

    private void doFetchGroupConfig(final String server, final ConfigGroupEnum... groups) {
        StringBuilder params = new StringBuilder();
        for (ConfigGroupEnum groupKey : groups) {
            params.append("groupKeys").append("=").append(groupKey.name()).append("&");
        }
        String url = server + "/configs/fetch?" + StringUtils.removeEnd(params.toString(), "&");
        log.info("request configs: [{}]", url);
        String json = null;
        try {
            json = this.httpClient.getForObject(url, String.class);
        } catch (RestClientException e) {
            String message = String.format("fetch config fail from server[%s], %s", url, e.getMessage());
            log.warn(message);
            throw new SoulException(message, e);
        }
        // update local cache
        boolean updated = this.updateCacheWithJson(json);
        if (updated) {
            log.info("get latest configs: [{}]", json);
            return;
        }
        // not updated. it is likely that the current config server has not been updated yet. wait a moment.
        log.info("The config of the server[{}] has not been updated or is out of date. Wait for 30s to listen for changes again.", server);
        ThreadUtils.sleep(TimeUnit.SECONDS, 30);
    }
```

方法 `doFetchGroupConfig` 也被方法 `fetchGroupConfig` 调用。

更新数据完成之后会睡眠 30s，然后重新进入死循环。
