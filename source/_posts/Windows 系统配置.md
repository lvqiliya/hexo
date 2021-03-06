---
title: Windows 系统配置
tags:
  - Java
  - Maven
  - Tomcat
  - Gradle
  - Git
  - TortoiseGit
  - IDEA
  - MySQL
  - Sublime Text3
categories: 白科技
abbrlink: a40258e8
date: 2017-12-26 22:53:26
---
整理的环境配置
<!--more-->

## Java开发环境相关配置

### Java

- 安装java，跳过不说。
- 新建系统变量：

> `JAVA_HOME` `C:\Program Files\Java\jdk1.8.0_101`  
> `CLASSPATH` `.;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar;`

- 添加系统变量：

> `Path` `%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;`

- 校验是否成功：

> `java -version`  
> `javac`

### Tomcat

- 下载tomcat二进制压缩版，[tomcat官方传送门](http://tomcat.apache.org/)。
- 新建系统变量：

> `CATALINA_HOME` `D:\Program Files\apache-tomcat-8.5.24`

- 添加系统变量：

> `Path` `%CATALINA_HOME%\bin;`

- 校验是否成功：

> 启动 *D:\Program_Files\apache-tomcat-8.5.24\bin\startup.bat*  
> 打开 [http://localhost:8080](http://localhost:8080)

### Maven

- 下载maven二进制压缩版，[maven官方传送门](http://maven.apache.org/)。
- 新建系统变量：

> `MAVEN_HOME` `D:\Program_Files\apache-maven-3.5.2`

- 添加系统变量：

> `Path` `%MAVEN_HOME%\bin;`

- 校验是否成功：

> `mvn -v`

- 配置setting.xml：

> *C:\Users\lvqiliya* 中新建文件夹，命名`.m2.`，copy一份setting.xml到.m2文件夹里面  
> 修改本地仓库 & 添加阿里云镜像

```xml
<localRepository>D:\Program_Files\apache-maven-3.5.2\repository</localRepository>

<mirror>
      <id>alimaven</id>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <mirrorOf>central</mirrorOf>
</mirror>
```

### Gradle

- 下载gradle二进制压缩版，[gradle官方传送门](https://gradle.org/install/#manually)。
- 新建系统变量：

> `GRADLE_HOME` `D:\Program_Files\gradle-4.4.1`

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

## IDEA配置

### License

```license
ThisCrackLicenseId-{
"licenseId":"ThisCrackLicenseId",
"licenseeName":"Liam Lv",
"assigneeName":"",
"assigneeEmail":"idea@gmail.com",
"licenseRestriction":"For This Crack, Only Test! Please support genuine!!!",
"checkConcurrentUse":false,
"products":[
{"code":"II","paidUpTo":"2099-12-31"},
{"code":"DM","paidUpTo":"2099-12-31"},
{"code":"AC","paidUpTo":"2099-12-31"},
{"code":"RS0","paidUpTo":"2099-12-31"},
{"code":"WS","paidUpTo":"2099-12-31"},
{"code":"DPN","paidUpTo":"2099-12-31"},
{"code":"RC","paidUpTo":"2099-12-31"},
{"code":"PS","paidUpTo":"2099-12-31"},
{"code":"DC","paidUpTo":"2099-12-31"},
{"code":"RM","paidUpTo":"2099-12-31"},
{"code":"CL","paidUpTo":"2099-12-31"},
{"code":"PC","paidUpTo":"2099-12-31"}
],
"hash":"2911276/0",
"gracePeriodDays":7,
"autoProlongated":false}
```

### Ignore the suffix

- 路径`Files >> Setting >> Editor >> File Types`
- Ignore files and folders中添加`.idea;.mvn;*.iml`

### Before start-up

- 进入到解压后的IntelliJ IDEA文件夹中的 *bin* 文件夹，找到 *idea64.exe.vmoptions* 文件
![File location](/images/Development environment configuration/vmoptions.png)
- 修改参数
![Change parameter](/images/Development environment configuration/vmoptionsconfig.png)

*`-Xms1024m`，`-Xmx1536m`，具体情况可根据电脑配置自行适当修改，本教程只提供参考值，字段含义请自行百度。*
添加`-javaagent:*.jar`

### Default setting

- System Settings
![System Settings](/images/Development environment configuration/systemsettings.png)

*说明：取消勾选Reopen last project on startup，这样可以在打开IntelliJ IDEA选择自己想选择的项目Synchronization操作框内，第三项表示自动保存频率。*

- 打开内存监控
Appearance & Behavior >> Appearance:
Window Options - [check]Show memory indicator;
- 快捷键修改：
Keymap：
查找关键字“d”，修改Duplicate Entire Lines的快捷键为Ctrl+D;
- 不区分大小写：
Editor >> General >> Code Completion：
Case sensitive compeltion - None;
- Tab显示格式和数目：
Editor >> General >> Editor Tabs >> Tab Closing Policy:
Tab Appearance - [uncheck]Show Tabs In Single Row;
Tab Limit — 30;
- 引入包完整显示不用*替代：
Editor >> Code Style >> Java >> Imports：
Class count to use import with * - 50;
Names count to use static import with * - 30;
- 代码注释：
Editor >> Code Style >> Java >> Code Generation：
Comment Code - [uncheck]Line comment at first column;
- 修改类注释模板:
Editor >> File and Code Templates:
Class Interface Enum - 自第二行加一下注释，前后各空一行

```java
/**
  * FileName: ${NAME}
  * Author:   ${USER}
  * Date:     ${DATE} ${TIME}
  * Description: ${DESCRIPTION}
  * Version:  ${VERSION}
  */
```

## Git相关配置

### Git配置

- 初始化

```git
git config --global user.name "lvqiliya"
git config --global user.email "lvqiliya@gmail.com"
git config --global http.proxy socks5://127.0.0.1:1080
git config --global https.proxy socks5://127.0.0.1:1080
如果想要取消代理则执行以下命令：
git config --global --unset http.proxy
git config --global --unset https.proxy
```

- SSH检查

```git
ls -al ~/.ssh
```

- SSH生成

```git
ssh-keygen -t rsa -b 4096 -C "lvqiliya@gmail.com"
```

- 添加SSH到ssh-agent

```git
eval $(ssh-agent -s)
ssh-add ~/.ssh/id_rsa
```

- SSH验证

```git
ssh -T git@github.com
```

### TortoiseGit配置

- 打开PuTTYgen
Firstly, loading an existing private kiy file,then save private key to .ssh
![PuTTYgen设置](/images/Development environment configuration/PuTTYgen.png)

- 打开Pageant
Add Key the private key
![PuTTYgen设置](/images/Development environment configuration/Pageant.png)

## MySQL配置

mysql的配置碰到了不少的坑，特别是5.7+的版本，它不包含 *.ini* 的配置文件，也不包含 *data* 文件夹。经过一番折腾，整理出以下有效的配置。

- 下载mysql二进制压缩版，[mysql官方传送门](https://dev.mysql.com/downloads/mysql/)。
- 新建系统变量：

> `MYSQL_HOME` `D:\Program_Files\mysql-5.7.20-winx64`

- 添加系统变量：

> `Path` `%MYSQL_HOME%\bin;`

- [官方配置文档传送门](https://dev.mysql.com/doc/refman/5.7/en/windows-installation.html)

### my.ini

新建配置文件 *my.ini*，内容如下：

```ini
[mysqld]
# 设置3306端口
port=3306
# 设置mysql的安装目录
basedir=D:/Program_Files/mysql-8.0.12-winx64
# 设置mysql数据库的数据的存放目录
datadir=D:/Program_Files/mysql-8.0.12-winx64/data
# 允许最大连接数
max_connections=200
# 允许连接失败的次数。这是为了防止有人从该主机试图攻击数据库系统
max_connect_errors=10
# 服务端使用的字符集默认为UTF8
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
# 默认使用“mysql_native_password”插件认证
# default_authentication_plugin=mysql_native_password
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8
[client]
# 设置mysql客户端连接服务端时默认使用的端口
port=3306
default-character-set=utf8
```

这地方要注意的是 *basedir* 和 *datadir* 的路径，要么使用 **/** 要么使用 **\\\\** ，否则会在下面的步骤出问题。

### cmd语句

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

```sql
use mysql;
alter user 'root'@'localhost' identified with mysql_native_password by '123456';
flush privileges;
```

## Sublime Text3

- 注册码

```license
----- BEGIN LICENSE -----
sgbteam
Single User License
EA7E-1153259
8891CBB9 F1513E4F 1A3405C1 A865D53F
115F202E 7B91AB2D 0D2A40ED 352B269B
76E84F0B CD69BFC7 59F2DFEF E267328F
215652A3 E88F9D8F 4C38E3BA 5B2DAAE4
969624E7 DC9CD4D5 717FB40C 1B9738CF
20B3C4F1 E917B5B3 87C38D9C ACCE7DD8
5F7EF854 86B9743C FADC04AA FB0DA5C0
F913BE58 42FEA319 F954EFDD AE881E0B
------ END LICENSE ------
```

## Node.js

- 解压完成后新建两个文件夹 `node_cache`和`node_global`到根目录
- 新增系统环境变量：

> `NODE_HOME` `E:\Program_Files\node-v10.15.3-win-x64`

- 在CMD中执行命令

> `npm config set prefix "E:\Program_Files\node-v10.15.3-win-x64\node_global"`  
> `npm config set cache "E:\Program_Files\node-v10.15.3-win-x64\node_cache"`

- 新增以下内容到 `PATH`:

> `%NODE_HOME%`  
> `%NODE_HOME%\node_global`  
> `%NODE_HOME%\node_modules`

## 系统杂项

### 照片查看器

```shell
#进入注册表
计算机\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Photo Viewer\Capabilities\FileAssociations

# 新建字符串
jpg PhotoViewer.FileAssoc.Tiff
png PhotoViewer.FileAssoc.Tiff
```

### U盘无法识别解决方法

```shell
# 在打开的cmd运行窗口里输入
diskpart

# 回车，然后输入
list disk

# 回车，查看目前电脑上所有的磁盘，如果U盘为2，那么输入以下命令将焦点指定到该磁盘
select disk 1

# 回车，然后再输入
clean

# 最后启动设备管理器进行格式化
```
