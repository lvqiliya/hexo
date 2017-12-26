---
title: Development environment configuration
date: 2017-12-26 22:53:26
tags: [Java, Maven, Tomcat, IDEA]
categories: 白科技
---
整理的环境配置
<!--more-->
# 开篇废话
其实配置开发环境并不是很常用，毕竟重装系统的频率也没有那么高。今天配置了一次，到网上到处找比较规范的配置，但是各写各的且看到的都不满意，所以自己来写以便下次配置环境查看。
# 开发环境配置
## Java
- 安装java，跳过不说。
- 新建系统变量
> `JAVA_HOME` `C:\Program Files\Java\jdk1.8.0_101`
>
> `CLASSPATH` `.;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar;`

- 添加系统变量
> `Path` `%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;`

- 校验是否成功
> `java -version`
>
> `javac`

## Tomcat
- 下载tomcat二进制压缩版，[tomcat官方传送门](http://tomcat.apache.org/)。
- 新建系统变量
> `CATALINA_HOME` `D:\Program Files\apache-tomcat-8.5.24`

- 添加系统变量
> `Path` `%CATALINA_HOME%\bin;`

- 校验是否成功
> 启动 *D:\Program Files\apache-tomcat-8.5.24\bin\startup.bat*
>
> 打开 [http://localhost:8080](http://localhost:8080)

## Maven
- 下载maven二进制压缩版，[maven官方传送门](http://maven.apache.org/)。
- 新建系统变量
> `MAVEN_HOME` `D:\Program Files\apache-maven-3.5.2`

- 添加系统变量
> `Path` `%MAVEN_HOME%\bin;`

- 校验是否成功
> `mvn -v`

- 配置setting.xml
> *C:\Users\lvqiliya* 中新建文件夹，命名`.m2.`，copy一份setting.xml到.m2文件夹里面
>
> 修改本地仓库 & 添加阿里云镜像
```xml
<localRepository>D:\Program Files\apache-maven-3.5.2\repository</localRepository>

<mirror>
      <id>alimaven</id>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <mirrorOf>central</mirrorOf>
</mirror>
```

# IDEA配置
## 忽略.idea、.mvn和*.iml
- 路径`Files >> Setting >> Editor >> File Types`
- Ignore files and folders中添加`.idea;.mvn;*.iml`
