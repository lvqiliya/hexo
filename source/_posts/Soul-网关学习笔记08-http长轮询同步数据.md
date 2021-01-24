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
