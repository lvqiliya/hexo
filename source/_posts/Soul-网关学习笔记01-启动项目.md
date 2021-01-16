---
title: Soul 网关学习笔记01-启动项目
date: 2021-01-16 19:34:07
tags:
categories:
---
拉取项目并运行
<!--more-->
## 运行环境

- Windows 10
- IDEA
- JDK 8
- MySQL 8.0

## 运行项目

直接通过 IDEA 拉取项目并打开项目，等待依赖下载完成，配置好项目 JDK 即可，不再赘述。

修改 admin 模块 application.yml，添加本地 MySQL 数据库密码，并在 url 的值最后加上 *&useSSL=false*，避免出现以下大段警告代码。

```java
** BEGIN NESTED EXCEPTION **

javax.net.ssl.SSLException
MESSAGE: closing inbound before receiving peer's close_notify
STACKTRACE:
javax.net.ssl.SSLException: closing inbound before receiving peer's close_notify
at sun.security.ssl.Alert.createSSLException(Alert.java:133)
at sun.security.ssl.Alert.createSSLException(Alert.java:117)
......
```

启动 admin 模块并以 admin 用户登录，成功无异常。

启动 bootstrap 模块，成功无异常。

## 阅读文档

由于学识有限，官方文档所介绍的功能并不能马上理解，多数描述并不能带来直观感受，但笔者比较感兴趣是以下两点：

- 内置丰富的插件支持，鉴权，限流，熔断，防火墙等等。
- 流量配置动态化，性能极高，网关消耗在 1~2ms。

目前项目中不断有抗风险的需求提出，但至于如何去实现，笔者不是很明白。虽然可以通过接收接口请求，通过一系列的逻辑去查库判断，但是很不优雅且基本上没有高性能可说。

代码耗时一直是我笔者比较好奇的点，几毫秒的响应速度让人很想了解是如何达到的。

> Soul作为网关，为了提供更高的响应速度，所有的配置都缓存在JVM的Hashmap中
> Soul 网关在启动时，会从从配置服务同步配置数据，并且支持推拉模式获取配置变更信息，并且更新本地缓存。而管理员在管理后台，变更用户、规则、插件、流量配置，通过推拉模式将变更信息同步给 Soul 网关

热拔插模式应该是除了性能之外，给用户体验最直观的特性。

## 术语理解

- 插件-包括监控、鉴权、限流、分发等
- 选择器-插件具有多个选择器，但选择器本身笔者目前没有更细致的理解
- 规则-选择器可选择多个规则，但具体规则还需研究源码
