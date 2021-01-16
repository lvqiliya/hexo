---
title: Spring Boot 学习笔记
tags:
  - Spring Boot
categories: 白科技
abbrlink: ed61e28f
date: 2019-05-26 19:10:44
---
本文是关于 Spring Boot 学习的笔记
<!--more-->

## 入门

### Hello World 探究

#### POM文件

##### 父项目

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.5.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
它的父项目是：

<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.1.5.RELEASE</version>
    <relativePath>../../spring-boot-dependencies</relativePath>
</parent>
它来真正管理Spring Boot应用里面的所有依赖版本，我们可以称其为Spring Boot版本仲裁中心，故而导入依赖默认无需设定版本号，当然也可以指定版本号。
```

##### 启动器

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

**spring-boot-starter**-web:

spring-boot-starter是 Spring Boot 场景启动器；spring-boot-starter-web 帮我们自动导入Web应用所需要的依赖

Spring Boot 将所有的功能场景抽取出来，做成一个个 starter，只需要在项目中引入这些 starter，相关场景的依赖都会导入进来。

#### 主程序类/主入口类

```java
@SpringBootApplication
public class SpringBoot01Application {

    public static void main(String[] args) {
        SpringApplication.run(SpringBoot01Application.class, args);
    }

}
```

**@SpringBootApplication**：它标注的类是 Spring Boot 的主配置类，Spring Boot 运行此类中的 main 方法来启动 Spring Boot 应用。它是一个组合注解，有两个很重要的子注解：

> @SpringBootConfiguration

把它标注在类上，以表示该类是一个 Spring Boot 的配置类。它有一个子注解是 **@Configuration**，也就是 Spring 的注解。

> @EnableAutoConfiguration

开启自动配置。通过标注它，让 Spring Boot 来自动配置。

它有一个子注解 **@AutoConfigurationPackage**——自动配置包，它的作用是将主配置类——**@SpringBootApplication** 标注的类——所在包及其子包里面所有组件扫描到 Spring 容器。

它另外一个子注解 **@Import(AutoConfigurationImportSelector.class)**，给容器中导入组件。将所需要导入的组件以全类名的方式返回，这些组件就会被添加到容器中，这些组件都是一些*自动配置类*。

需要深入研究 **spring-boot-autoconfigure-2.1.5.RELEASE.jar**

## 配置

### 配置文件

Spring Boot 使用一个全局的配置文件，有固定名字：

- application.properties
- application.yml

其作用主要是用来修改 Spring Boot 自动配置的默认值。

### YAML语法

#### 基本语法

k:(空格)v：表示一对键值对（空格必须有），以空格的缩减来控制层级关系，左对齐则同一级。

##### 字面量：普通值（数字、字符串、布尔）

k: v：字面值直接写
字符串默认不用加单引号和双引号
双引号：不会转义特殊字符
单引号：会转义特殊字符，输出的是字符串

##### 对象、Map

k: v：

```yml
friends:
    lastName: zhangsan
    age: 20
```

行内写法：

```yml
friends: {lastName: zhangsan, age: 20}
```

##### 数组（List、Set）

用`- 值`表示数组中的元素

```yml
pets:
 - cat
 - dog
 - pig
```

行内写法：

```yml
pets: [cat, dog, pig]
```

### 配置文件注入

配置文件application.yml

```yml
person:
  name: zhangsan
  age: 18
  boss: false
  birthday: 2019/05/28
  maps: {k1: 11, k2: 12}
  lists:
    - lisi
    - wangwu
    - zhaoliu
  dog:
    dogName: xiaogou
    dogAge: 3

```

javaBean

```java
@Component
@ConfigurationProperties(prefix = "person")
public class Person {

    private String name;
    private Integer age;
    private Boolean boss;
    private Date birthday;

    private Map<String, Object> maps;
    private List<Object> lists;
    private Dog dog;
```

值得注意的是需要声明 **@Component** 和 **@ConfigurationProperties** 两个注解，后者默认是从全局配置文件中获取。

#### @ConfigurationProperties 和 @ Value 获取值的比较

| | @ConfigurationProperties | @ Value |
|-|-|-|
|功能|批量注入（作用于类）|单个注入（作用于属性）|
|SpEL（${11*2}）|不支持|支持|
|复杂类型封装（map）|支持|不支持|

#### @PropertySource 和 @ImportResource

**@PropertySource** 加载指定的配置文件，例如 person.properties:

```java
@PropertySource(value = {"classpath:person.properties"})
public class Person {}
```

**@ImportResource** 导入Spring的配置文件，使其生效：

```java
@ImportResource(locations = {"classpath:beans.xml"})
public class Person {}
```

Spring Boot 推荐使用全注解的方式给容器添加组件：

```java
@Configuration
public class Config {

    @Bean
    public Person helloPerson() {
        return new Person();
    }
}
```

#### 配置文件的占位符

随机数

```properties
${random.int}
```

占位符获取之前配置的值

```properties
# person.hello并未被定义，所以返回冒号后的值
person.name = ${person.hello:hello}
```

#### Profile

- 多 Profile 文件

配置文件可以定义为 application-**{profile}**.properties/yml，用于实现动态切换。主配置文件名依然是 application.properties，与此同时定义开发环境和生产环境两个配置文件 application-**dev**.properties 和 application-**prod**.properties。

默认情况下 Spring Boot 使用主配置文件，此时需要指定 profile：

```properties
# 激活 profile 为 dev 的配置文件
spring.profiles.active = dev
```

- yml支持多文档块方式

```yml
server:
  port: 8080
spring:
  profiles:
    active: prod  # 激活 profile 为 prod 的配置文件
---
# dev 环境的配置文档块
server:
  port: 8081
spring:
  profiles: dev
---
# prod 环境的配置文档块
server:
  port: 8082
spring:
  profiles: prod
```

- 激活指定 profile

除开以上两种方式激活指定 profile 之外，还有以下方法：

1. -命令行 --spring.profiles.active=prod
2. -jvm参数 -Dspring.profiles.activ=dev
