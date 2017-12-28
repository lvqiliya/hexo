---
title: Development environment configuration
date: 2017-12-26 22:53:26
tags: [Java, Maven, Tomcat, MySQL, Gradle, IDEA]
categories: 白科技
---
整理的环境配置
<!--more-->
# 开篇废话
其实配置开发环境并不是很常用，毕竟重装系统的频率也没有那么高。今天配置了一次，到网上到处找比较规范的配置，但是各写各的且看到的都不满意，所以自己来写以便下次配置环境查看。
# 开发环境配置
## Java
- 安装java，跳过不说。
- 新建系统变量：
> `JAVA_HOME` `C:\Program Files\Java\jdk1.8.0_101`
>
> `CLASSPATH` `.;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar;`

- 添加系统变量：
> `Path` `%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;`

- 校验是否成功：
> `java -version`
>
> `javac`

## Tomcat
- 下载tomcat二进制压缩版，[tomcat官方传送门](http://tomcat.apache.org/)。
- 新建系统变量：
> `CATALINA_HOME` `D:\Program Files\apache-tomcat-8.5.24`

- 添加系统变量：
> `Path` `%CATALINA_HOME%\bin;`

- 校验是否成功：
> 启动 *D:\Program Files\apache-tomcat-8.5.24\bin\startup.bat*
>
> 打开 [http://localhost:8080](http://localhost:8080)

## Maven
- 下载maven二进制压缩版，[maven官方传送门](http://maven.apache.org/)。
- 新建系统变量：
> `MAVEN_HOME` `D:\Program Files\apache-maven-3.5.2`

- 添加系统变量：
> `Path` `%MAVEN_HOME%\bin;`

- 校验是否成功：
> `mvn -v`

- 配置setting.xml：
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

## Gradle
- 下载gradle二进制压缩版，[gradle官方传送门](https://gradle.org/install/#manually)。
- 新建系统变量：
> `GRADLE_HOME` `D:\Program Files\gradle-4.4.1`

- 添加系统变量：
> `Path` `%GRADLE_HOME%\bin;`

- 校验是否成功：
> `gradle -v`

- 建 *init.gradle* 文件
```gradle
allprojects{
    repositories {
        def ALIYUN_REPOSITORY_URL = 'http://maven.aliyun.com/nexus/content/groups/public'
        def ALIYUN_JCENTER_URL = 'http://maven.aliyun.com/nexus/content/repositories/jcenter'
        all { ArtifactRepository repo ->
            if(repo instanceof MavenArtifactRepository){
                def url = repo.url.toString()
                if (url.startsWith('https://repo1.maven.org/maven2')) {
                    project.logger.lifecycle "Repository ${repo.url} replaced by $ALIYUN_REPOSITORY_URL."
                    remove repo
                }
                if (url.startsWith('https://jcenter.bintray.com/')) {
                    project.logger.lifecycle "Repository ${repo.url} replaced by $ALIYUN_JCENTER_URL."
                    remove repo
                }
            }
        }
        maven {
            url ALIYUN_REPOSITORY_URL
            url ALIYUN_JCENTER_URL
        }
    }
}
```

# IDEA配置
## 忽略.idea、.mvn和*.iml
- 路径`Files >> Setting >> Editor >> File Types`
- Ignore files and folders中添加`.idea;.mvn;*.iml`

# MySQL配置
mysql的配置碰到了不少的坑，特别是5.7+的版本，它不包含 *.ini* 的配置文件，也不包含 *data* 文件夹。经过一番折腾，整理出以下有效的配置。
- 下载mysql二进制压缩版，[mysql官方传送门](https://dev.mysql.com/downloads/mysql/)。
- 新建系统变量：
> `MYSQL_HOME` `D:\Program Files\mysql-5.7.20-winx64`

- 添加系统变量：
> `Path` `%MYSQL_HOME%\bin;`

- [官方配置文档传送门](https://dev.mysql.com/doc/refman/5.7/en/windows-installation.html)

## my.ini
新建配置文件 *my.ini*，内容如下：
```ini
[mysqld]
# 设置mysql的安装目录
basedir=D:\\Program Files\\mysql-5.7.18-winx64
# 设置mysql数据库的数据的存放目录，必须是data
datadir=D:\\Program Files\\mysql-5.7.18-winx64\\data
# mysql端口
port=3306
# 字符集
character_set_server=utf8
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
```
这地方要注意的是 *basedir* 和 *datadir* 的路径，要么使用 **/** 要么使用 **\\\\** ，否则会在下面的步骤出问题。

## cmd语句
以管理员身份启动cmd，按顺序执行下面的语句即可。
- 初始化mysql，这个过程中会自动 *data* 文件夹：
> `mysqld --initialize-insecure`

- 安装mysql：
> `mysqld –install`

- 启动服务：
> `net start mysql`

- 登陆mysql：
> `mysql -u root -p`

- 修改密码：
> `set password=password('123456')`
