---
title: 通过Maven构建Java项目
tags:
  - Spring
  - Translation
categories: 译文二三事
abbrlink: 2a519e30
date: 2017-09-29 13:50:47
---
本指南将指导你利用Maven构建一个简单的Java项目。
<!--more-->
# 你所构建
你将使用Maven来构建一个应用程序，它将提供一天中的时间。
# 你所需要
- 大约15min
- 一个你喜欢的代码编译器
- JDK6或者更高版本

# 如何完成本指南
想大多数的Spring[入门指南](https://spring.io/guides)一样，你可以从头开始然后完成每一步，或者你可以绕过已经熟悉的基本设置步骤。无论哪种方法，你都会得到工作代码。

想从头开始？请移步***设置项目***

跳过了基础设置？继续看以下内容：
- 下载并解压本指南的源储存库，或者通过Git clone本指南：`git clone https://github.com/spring-guides/gs-maven.git`
- cd 进入`gs-maven/initial`
- 跳转到***初始化***

当你完成之后，你可以根据`gs-maven/complete`中的代码检验结果。

# 设置项目
首先你需要通过Maven来构建一个Java项目，只需要注意Maven，至于这个项目可以尽量的简单

**创建目录结构**

在你选中的项目目录中，创建以下子目录结构；例如，在Unix/Linux系统中使用命令 `mkdir -p src/main/java/hello`
```
└── src
    └── main
        └── java
            └── hello
```
在`src/main/java/hello`这个目录中，你可以创建任何你想要的Java class文件。为了保持与本指南其余部分的一致，创建以下两个class文件：`HelloWorld.java`、`Greeter.java`。

`src/main/java/hello/HelloWorld.java`
```java
package hello;

public class HelloWorld {
    public static void main(String[] args) {
        Greeter greeter = new Greeter();
        System.out.println(greeter.sayHello());
    }
}
```
`src/main/java/hello/Greeter.java`
```java
package hello;

public class Greeter {
    public String sayHello() {
        return "Hello world!";
    }
}
```
至此你已经有了一个可以用Maven来构造的项目，接下来就是安装Maven。

你可以通过[http://maven.apache.org/download.cgi](https://maven.apache.org/download.cgi)下载作为压缩文件的Maven。只需要二进制文件，所以只需要找到`apache-maven-{version}-bin.zip `和`apache-maven-{version}-bin.tar.gz`的连接即可。

下载压缩文件并解压它，之后将它`bin`文件夹的路径添加到环境变量`path`中。

若想测试Maven的安装情况，在命令行执行以下命令：
**`mvn - v`**

如果一切正常，你可以看到一些关于Maven的安装信息，例如下面所展示的内容（可能有些轻微的区别）：
```
Apache Maven 3.0.5 (r01de14724cdef164cd33c7c8c2fe155faf9602da; 2013-02-19 07:51:28-0600)
Maven home: /usr/share/maven
Java version: 1.7.0_09, vendor: Oracle Corporation
Java home: /Library/Java/JavaVirtualMachines/jdk1.7.0_09.jdk/Contents/Home/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "mac os x", version: "10.8.3", arch: "x86_64", family: "mac"
```
**恭喜你！Maven已经成功安装啦！**

# 定义一个简单的Maven构造

Maven已经安装，现在你需要做的就是创建一个Maven项目定义。Maven项目有一个名叫*pom.xml*的XML文件，它将定义整个项目。除此之外，它还定义了项目的名称、版本和对于外部库的依赖关系。

在项目的根目录中创建一个*pom.xml*，并给出一下内容：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.springframework</groupId>
    <artifactId>gs-maven</artifactId>
    <packaging>jar</packaging>
    <version>0.1.0</version>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.1</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <transformers>
                                <transformer
                                    implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>hello.HelloWorld</mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```
除了可选元素`<packaging>`，以上就是一个Java项目需要的最简单的*pom.xml*文件。它包括以下项目配置的细节：
- `<modelVersion>`，POM版本（通常是4.0.0）；
- `<gropId>`，本项目属于哪一个项目组，通常用它的反转域名作为名称；
- `<artifactId>`，给与项目artifact的名称（例如，那些名字中包含JAR和WAR的文件）
- `<version>`，正在构造的项目的版本
- `<packaging>`，项目应该以哪种形式被打包，默认用“jar”打包JAR文件，用“war”打包JAR文件。
> 当谈到选择一个版本控制方案，Spring推荐使用[semantic versioning](http://semver.org)。

在这一点上，你已经定义了一个最小但是又有能力的Maven项目！
# 构建Java代码
Maven现在已经准备好构建你的项目了！你可以使用Maven执行几个构建生命周期目标，其中包括编译你的项目代码，创建库包（如JAR文件 ），并将库安装到本地Maven依赖关系库中。
若想尝试构建，请在命令行输入以下命令：
`mav compile`
它会运行Maven，通知Maven运行编译这个功能。当编译完成，你可以在*target/classes*目录中找到那个被编译的*.class*文件
由于你不可能直接分发或使用*.class*文件，你可能跟你想运行包命令：
`mvn package`
打包命令将编译你的Java代码，运行所有的测试用例，然后把代码导报成一个JAR文件，放在*target*文件夹中。JAR文件的命名基于项目的`<artifactId>`和`<version>`。举个栗子，在前面提到的最基本的*pom.xml*，它的JAR文件将被命名为*gs-maven-0.1.0.jar*。
> 如果你改变了`<packaging>`中的值，从“jar”变成“war”，那么结果将是*target*中的文件是WAR，而不是JAR文件。

为了快速访问项目的依赖，Maven在本机维护了一个依赖关系库（通常是在*.m2/repository*这个目录下）。如果你想安装项目的JAR文件到本地库，只需要执行*install*命令
`mvn install`
这个命令将编译、测试和打包你项目的代码，然后把它拷贝到依赖关系库中，为其他项目
同一个依赖参考。
说了这么久的依赖，是时候在Maven构建中声明依赖关系了。
# 声明依赖
Hello World这个简单的实示例完全的独立的，并不需要依赖其他的外部库。但是对于大多数应用来说，都需要依赖外部库来处理一些常见且复杂的功能。
举个栗子，假设除了say“Hello World！”，你还想让这个应用打印出当前的日期和时间，虽然你可以使用Java库中的日期和时间工具，但如果使用Joda Time库来操作，会变得非常有趣。
首先，修改你的Hello World.java文件，就像以下这样：
`src/main/java/hello/HelloWorld.java`
```java
package hello;

import org.joda.time.LocalTime;

public class HelloWorld {
	public static void main(String[] args) {
		LocalTime currentTime = new LocalTime();
		System.out.println("The current local time is: " + currentTime);
		Greeter greeter = new Greeter();
		System.out.println(greeter.sayHello());
	}
}
```
这里的`HelloWorld`用了Joda Time的`LocalTime`类来获取当前的打印的时间。
如果你现在运行`mvn compile`来构建项目，这个构建必将失败，其原因是你还没有将Joda Time声明成一个编译依赖。你可以通过增加以下的代码到*pom.xml*文件（增加到`<project>`这个元素里）来修复这个问题：
```xml
<dependencies>
		<dependency>
			<groupId>joda-time</groupId>
			<artifactId>joda-time</artifactId>
			<version>2.9.2</version>
		</dependency>
</dependencies>
```
这个XMl快声明了项目的依赖关系列表，具体来说，它为Joda Time库声明了一个依赖关系。在`<dependency>`元素中，依赖坐标由三个子元素定义：
- `<groupId>` - 这个依赖关系属于什么组织
- `<artifactId>` - 被要求的库
- `<version>` - 这个库被要求的具体版本

默认的，所有的依赖关系都作为编译依赖关系，意思是，他们将在编译时候被利用到（如果你正在构造一个WAR文件，包括在WAR的*/WEB-INF/libs*文件夹中）。另外，你可以指定`<scope>`元素来指定以下范围之一：
- `<provided>` - 编译项目代码所需要的依赖关系，但运行时将由运行代码容器提供（例如Java Servlet API）
- `<test>` - 依赖关系在编译和运行测试的时候将被用到，但是在构建或运行项目的运行代码不是必需的

那么现在，如果你再次运行`mvn compile`或者`mvn package`，Maven将从Maven中心库中解析Joda Time依赖，然后就构建成功了。
# 写一个测试
首先添加一个JUnit依赖到你的*pom.xml*文件中，作用域为`test`：
```xml
<dependency>
		<groupId>junit</groupId>
		<artifactId>junit</artifactId>
		<version>4.12</version>
		<scope>test</scope>
</dependency>
```
接下来创建一个测试用例像以下所示：
`src/test/java/hello/GreeterTest.java`
```java
package hello;

import static org.hamcrest.CoreMatchers.containsString;
import static org.junit.Assert.*;

import org.junit.Test;

public class GreeterTest {

	private Greeter greeter = new Greeter();

	@Test
	public void greeterSaysHello() {
		assertThat(greeter.sayHello(), containsString("Hello"));
	}

}
```
Maven使用一个名叫“surefire”的插件来运行单元测试，此插件默认配置是编译并运行在`src/test/java`中名称与*Test匹配的类。你可以通过以下的命令行来运行这些测试
`mvn test`
或者用`mvn install`，就像我们在上面已经提到过的那样（有一个生命周期定义，“test”被包括在“install”阶段）。
这里有一个完整的`pom.xml`文件：
`pom.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>org.springframework</groupId>
	<artifactId>gs-maven</artifactId>
	<packaging>jar</packaging>
	<version>0.1.0</version>

	<dependencies>
		<!-- tag::joda[] -->
		<dependency>
			<groupId>joda-time</groupId>
			<artifactId>joda-time</artifactId>
			<version>2.9.2</version>
		</dependency>
		<!-- end::joda[] -->
		<!-- tag::junit[] -->
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.12</version>
			<scope>test</scope>
		</dependency>
		<!-- end::junit[] -->
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-shade-plugin</artifactId>
				<version>2.1</version>
				<executions>
					<execution>
						<phase>package</phase>
						<goals>
							<goal>shade</goal>
						</goals>
						<configuration>
							<transformers>
								<transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
									<mainClass>hello.HelloWorld</mainClass>
								</transformer>
							</transformers>
						</configuration>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>

</project>
```
> 这个完整的**pom.xml**文件使用Maven Shade Plugin来简化JAR文件的可执行性。本指南专注于开始使用Maven，而不是使用这个插件。
# 摘要
恭喜你！你已经创造了一个简单的有用的通过Maven定义并构造的Java项目
# 扩展浏览
下面的这个指南或许能更好的帮助到你
- [Building Java Projects with Gradle](https://spring.io/guides/gs/gradle/)

想要写一个全新的入门指南或者为现有的入门指南做贡献么？点击[贡献指南](https://github.com/spring-guides/getting-started-guides/wiki)！

> 所有的指南都将发布ASLv2许可证的代码、[Attribution](https://creativecommons.org/licenses/by-nd/3.0/)还有为写作颁发的[NoDerivatives creative commons license](https://creativecommons.org/licenses/by-nd/3.0/)

> 原文地址：[Building Java Projects with Maven](https://spring.io/guides/gs/maven/)
