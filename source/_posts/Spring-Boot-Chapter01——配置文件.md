---
title: Spring Boot Chapter01——配置文件
tags:
  - Spring Boot
categories: 白科技
abbrlink: f9807a74
date: 2017-12-26 16:30:56
---
关于Spring Boot配置文件的一些整理
<!--more-->
# 前言
`src/main/resources`目录下的`application.properties`是Spring Boot的配置文件，为了少些代码北庐改用`application.yml`。
# 正文
### @Value(value = “*${config.name}*”)
此注解可以获得配置文件里面的属性。首先在`application.yml`中自定义配置name和age属性：
```yml
girl:
  cup: B
  age: 18
  mygirl: "${girl.age}的女孩,cup是${girl.cup}" # 参数间的引用
```
然后在controller类中获取并打印出来：
```java
HelloController.java

@RestController
public class HelloController {
    @Value(value = "${girl.cup}")
    private String cup;
    @Value(value = "${girl.age}")
    private Integer age;

    @GetMapping(value = "hello")
    public String say() {
        return "cup : " + cup + "age : " + age;
    }
}
```
访问`http://localhost:8080/hello`则能查看到打印出来的cup和age。但是实际上girl这个对象的属性除了cup和age还会有height、weight、hairstyle、lipstick等等，如果按照以上代码每一个属性的引入都要写一个`@Value`也太麻烦了，所以采用如下方法取得属性。
### @ConfigurationProperties(prefix = "girl")
先建一个girl的属性类：
```java
GirlProperties.java

@Component
@ConfigurationProperties(prefix = "girl") // prefix指前缀，值为需要获取到的属性前缀，本例中为[girl]
public class GirlProperties {
    private Integer age;
    private String cup;
    // 省略getter和setter
}
```
然后在`HelloController.java`中使用它
```java
HelloController.java

@RestController
public class HelloController {
    @Autowired
    GirlProperties girlProperties;

    @GetMapping(value = "hello")
    public String say() {
        return girlProperties.getCup();
    }
}
```
再次访问`http://localhost:8080/hello`，则能查看到cup值为B。
### 多环境配置——profiles
实际项目中会出现开发环境和生产环境不同的情况，如果只有一个配置文件，当你开发的时候需要配置开发环境的一些参数，在发布的时候又要换成生产环境的参数，很麻烦。故Spring Boot提供了一种很轻易的替换方式来解决这个问题。还是以girl为例，新建两个配置文件：
- application-dev.yml   开发环境
```yml
server:
  port: 8080
  context-path: /girl
girl:
  cup: B
  age: 18
```
- application-prod.yml  生产环境
```yml
server:
  port: 8081
  context-path: /girl
girl:
  cup: C
  age: 22
```
现在只要在`application.yml`中进行切换就行了：
```yml
spring:
  profiles:
    action: dev # 如果需要生产环境，dev改为prod即可
```
### 其他
使用随机数构建properties文件：
```yml
random:
  value: "${random.value}" # 随机字符串
  int: "${random.int}" # 随机int
  long: "${random.long}" # 随机long
  one: "${random.int(100)}" # 100以下随机数
  two: "${random.int(66,88)}" # 66到88之间的随机数
```
