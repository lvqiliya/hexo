---
title: Soul 网关学习笔记04-sofa插件初体验
abbrlink: 114ca0a8
date: 2021-01-18 22:17:50
tags: soul
categories: 源码学习
---
使用 sofa 插件，体验 sofa 代理
<!--more-->
## 启动项目

运行 soul-admin 模块的主函数，启动 soul-admin。

运行 soul-bootstrap 模块的主函数，启动 soul-bootstrap。

使用 admin 账号登录管理台，点击 PluginList-sofa 模块，此时选择器列表（SelectorList）和规则列表（RuleList）均为空。可以手动添加选择器和规则。

启动 zookeeper 服务。

检查 soul-bootstrap 模块下 sofa 插件引入是否完备。

进入 soul-examples 模块下的 sofa 子模块启动项目。

## 注册流程

```log
2021-01-18 22:50:41.387  INFO 13784 --- [pool-2-thread-1] o.d.s.client.common.utils.RegisterUtils  : sofa client register success: {"appName":"sofa","contextPath":"/sofa","path":"/sofa/insert","pathDesc":"Insert a row of data","rpcType":"sofa","serviceName":"org.dromara.soul.examples.dubbo.api.service.DubboTestService","methodName":"insert","ruleName":"/sofa/insert","parameterTypes":"org.dromara.soul.examples.dubbo.api.entity.DubboTest","rpcExt":"{\"loadbalance\":\"hash\",\"retries\":3,\"timeout\":-1}","enabled":true} 
2021-01-18 22:50:41.467  INFO 13784 --- [pool-2-thread-1] o.d.s.client.common.utils.RegisterUtils  : sofa client register success: {"appName":"sofa","contextPath":"/sofa","path":"/sofa/findById","pathDesc":"Find by Id","rpcType":"sofa","serviceName":"org.dromara.soul.examples.dubbo.api.service.DubboTestService","methodName":"findById","ruleName":"/sofa/findById","parameterTypes":"java.lang.String","rpcExt":"{\"loadbalance\":\"hash\",\"retries\":3,\"timeout\":-1}","enabled":true} 
2021-01-18 22:50:41.567  INFO 13784 --- [pool-2-thread-1] o.d.s.client.common.utils.RegisterUtils  : sofa client register success: {"appName":"sofa","contextPath":"/sofa","path":"/sofa/findAll","pathDesc":"Get all data","rpcType":"sofa","serviceName":"org.dromara.soul.examples.dubbo.api.service.DubboTestService","methodName":"findAll","ruleName":"/sofa/findAll","parameterTypes":"","rpcExt":"{\"loadbalance\":\"hash\",\"retries\":3,\"timeout\":-1}","enabled":true} 
```

根据以上日志找到注册 sofa 服务的代码。

```java
public static void doRegister(final String json, final String url, final RpcTypeEnum rpcTypeEnum) {
    try {
        String result = OkHttpTools.getInstance().post(url, json);
        if (AdminConstants.SUCCESS.equals(result)) {
            log.info("{} client register success: {} ", rpcTypeEnum.getName(), json);
        } else {
            log.error("{} client register error: {} ", rpcTypeEnum.getName(), json);
        }
    } catch (IOException e) {
        log.error("cannot register soul admin param, url: {}, request body: {}", url, json, e);
    }
}
```

分析方法签名可得，`doRegister` 方法的调用者传入了需要发送的 json 报文、url 地址以及 RPC 类型，由于是注册 sofa 模块，猜想 `rpcTypeEnum = sofa`。于是查看调用者代码。

```java
public class SofaServiceBeanPostProcessor implements BeanPostProcessor {
    ......
    private final String url;
    public SofaServiceBeanPostProcessor(final SofaConfig sofaConfig) {
        ......
        url = sofaConfig.getAdminUrl() + "/soul-client/sofa-register";
        ......
    }

    @Override
    public Object postProcessAfterInitialization(final Object bean, final String beanName) throws BeansException {
        if (bean instanceof ServiceFactoryBean) {
            executorService.execute(() -> handler((ServiceFactoryBean) bean));
        }
        return bean;
    }

    private void handler(final ServiceFactoryBean serviceBean) {
        ......
        final Method[] methods = ReflectionUtils.getUniqueDeclaredMethods(clazz);
        for (Method method : methods) {
            SoulSofaClient soulSofaClient = method.getAnnotation(SoulSofaClient.class);
            if (Objects.nonNull(soulSofaClient)) {
                RegisterUtils.doRegister(buildJsonParams(serviceBean, soulSofaClient, method), url, RpcTypeEnum.SOFA);
            }
        }
    }
}
```

在 `SofaServiceBeanPostProcessor` 的构造方法中为 url 属性赋值了 `admin 模块地址/soul-client/sofa-register`。同时根据 `handler` 方法中对于 `doRegister` 的调用证明了之前猜想。

根据 url 地址找到 admin 模块中对应的代码，一直深入到最底层。

```java
@Service("soulClientRegisterService")
public class SoulClientRegisterServiceImpl implements SoulClientRegisterService {
    ......
    @Override
    public String registerSofa(final MetaDataDTO dto) {
        MetaDataDO byPath = metaDataMapper.findByPath(dto.getPath());
        if (Objects.nonNull(byPath)
                && (!byPath.getMethodName().equals(dto.getMethodName())
                || !byPath.getServiceName().equals(dto.getServiceName()))) {
            return "you path already exist!";
        }
        final MetaDataDO exist = metaDataMapper.findByServiceNameAndMethod(dto.getServiceName(), dto.getMethodName());
        saveOrUpdateMetaData(exist, dto);
        String selectorId = handlerSofaSelector(dto);
        handlerSofaRule(selectorId, dto, exist);
        return SoulResultMessage.SUCCESS;
    }
    ......
}
```

`registerSofa` 方法返回的是注册结果，如果一切正常则返回 **SUCCESS**。

- `saveOrUpdateMetaData` 方法负责保存和更新元数据到数据库的表 meta_data 里，然后把元数据发布出去。`ApplicationEventPublisher#publishEvent`的源代码和发布机制还需进一步探究，本文不再叙述。

- `handlerSofaSelector` 方法负责注册选择器、选择器条件并拿到选择器，提供选择器的 id 给 `handlerSofaRule` 使用。选择器和选择器条件入库之后，调用 `ApplicationEventPublisher#publishEvent` 进行发布。

- `handlerSofaRule` 方法负责注册规则、规则条件。入库之后，调用 `ApplicationEventPublisher#publishEvent` 进行发布。

附上官方针对该模块的数据库表 UML 图。

![数据库表 UML 图](/images/soul/soul-db.png)

## 问题及解决

### 注册失败

前几次启动 sofa 模块，测试类的几个标注了 `@SoulDubboClient` 的方法并没有注册成功。

```log
2021-01-18 23:18:53.895 ERROR 8032 --- [pool-1-thread-1] o.d.s.client.common.utils.RegisterUtils  : sofa client register error: {"appName":"sofa","contextPath":"/sofa","path":"/sofa/insert","pathDesc":"Insert a row of data","rpcType":"sofa","serviceName":"org.dromara.soul.examples.dubbo.api.service.DubboTestService","methodName":"insert","ruleName":"/sofa/insert","parameterTypes":"org.dromara.soul.examples.dubbo.api.entity.DubboTest","rpcExt":"{\"loadbalance\":\"hash\",\"retries\":3,\"timeout\":-1}","enabled":true} 
2021-01-18 23:18:53.908 ERROR 8032 --- [pool-1-thread-1] o.d.s.client.common.utils.RegisterUtils  : sofa client register error: {"appName":"sofa","contextPath":"/sofa","path":"/sofa/findById","pathDesc":"Find by Id","rpcType":"sofa","serviceName":"org.dromara.soul.examples.dubbo.api.service.DubboTestService","methodName":"findById","ruleName":"/sofa/findById","parameterTypes":"java.lang.String","rpcExt":"{\"loadbalance\":\"hash\",\"retries\":3,\"timeout\":-1}","enabled":true} 
2021-01-18 23:18:53.924 ERROR 8032 --- [pool-1-thread-1] o.d.s.client.common.utils.RegisterUtils  : sofa client register error: {"appName":"sofa","contextPath":"/sofa","path":"/sofa/findAll","pathDesc":"Get all data","rpcType":"sofa","serviceName":"org.dromara.soul.examples.dubbo.api.service.DubboTestService","methodName":"findAll","ruleName":"/sofa/findAll","parameterTypes":"","rpcExt":"{\"loadbalance\":\"hash\",\"retries\":3,\"timeout\":-1}","enabled":true} 
```

根据本文第二部分[注册流程](#注册流程)的分析，最底层的代码方法或者之前或者之后的逻辑出现了问题，于是设置断点进行调试，发现第 12 行是根据 serviceName 和 methodName 进入数据库查找。当 sofa 注册失败时，该代码能查到两条数据，因为不是查列表，所以报错了。进入数据库查看表 meta_data，根据同一个 serviceName 和 methodName 进行搜索，会检索到之前体验 dubbo 模块时插入的两条关于 dubbo 的数据。

解决办法：清空表 meta_data，重启 sofa 模块即可。
