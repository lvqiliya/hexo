---
title: Java 学习笔记
date: 2019-06-09 21:13:25
tags: [Java]
categories: 白科技
---

## 面向对象

### 什么是面向对象

### 平台无关性

### 值传递

值传递和引用传递概述
> 值传递（pass by value）：是指在调用函数时将实际参数**复制**一份传递到函数中，这样在函数中如果对参数进行修改，将不会影响实际参数。  
> 引用传递（pass by reference）：是指在调用函数时将实际参数的地址**直接**传递到函数中，那么在函数中对参数进行修改，将会影响到实际参数。

验证基础类型是值传递

```java
public static void main(String[] args) {
    Demo demo = new Demo();
    int a = 10;
    demo.pass(a);
    System.out.println("print in main, a is " + a);
}

public void pass(int b) {
    b = 20;
    System.out.println("print in pass, b is " + b);
}
```

输出如下

```java
print in pass, b is 20
print in main, a is 10
```

验证对象是值传递

```java
// 仅仅是 String 对象，此例也可以用 BigDecimal 来代替
public static void main(String[] args) {
    Demo demo = new Demo();
    String a = "hello world";
    demo.pass(a);
    System.out.println("print in main, a is " + a);
}

public void pass(String b) {
    b = b.toUpperCase();
    System.out.println("print in pass, b is " + b);
}
```

输出如下

```java
print in pass, b is HELLO WORLD
print in main, a is hello world
```

```java
// 现在要用一个 Student 对象——包含 name、age 两个属性——来进行验证
public static void main(String[] args) {
    Demo demo = new Demo();
    Student s1 = new Student("xiaoming", 8);
    demo.pass(s1);
    System.out.println("print in main, s1.name = " + s1.getName() + " and s1.age = " + s1.getAge());
}

public void pass(Student s2) {
    s2.setName(s2.getName().toUpperCase());
    s2.setAge(10);
    System.out.println("print in pass, s2.name = " + s2.getName() + " and s2.age = " + s2.getAge());
}
```

此时惊讶的发现输出竟然相同

```java
print in main, s1.name = XIAOMING and s1.age = 10
print in main, s1.name = XIAOMING and s1.age = 10
```

> 疑问：String 和 Student 同为对象，但是 String 看起来是值传递，而 Student 看起来是引用，那么Java到底是值传递还是引用传递？

||值传递|引用传递|
|-|-|-|
|根本区别|会创建副本（copy）|不创建副本|
|所以|函数无法修改原始对象|函数可以修改原始对象|

```java
// 再次验证
public static void main(String[] args) {
    Demo demo = new Demo();
    Student s1 = new Student("xiaoming", 8);
    Student s2 = new Student("dahua", 10);
    demo.pass(s1, s2);
    System.out.println("print in main, s1.name = " + s1.getName() + " and s1.age = " + s1.getAge());
    System.out.println("print in main, s2.name = " + s2.getName() + " and s2.age = " + s2.getAge());
}

public void pass(Student x, Student y) {
    Student temp = x;
    x = y;
    y = temp;
    System.out.println("print in pass, x.name = " + x.getName() + " and x.age = " + x.getAge());
    System.out.println("print in pass, y.name = " + y.getName() + " and y.age = " + y.getAge());
}
```

输出如下

```java
print in pass, x.name = dahua and x.age = 10
print in pass, y.name = xiaoming and y.age = 8
print in main, s1.name = xiaoming and s1.age = 8
print in main, s2.name = dahua and s2.age = 10
```

从输出可以看出 x、y 是 s1、s2 的拷贝或者称为副本，它们都是对象引用。当进行传参操作之后，x 和 s1 指向同一个对象，y 和 s2 指向同一个对象，此时执行 pass() 方法中的逻辑之后，y 指向了 s1 指向的对象，x 指向了 s2 指向的对象。方法结束后 x、y 不再使用，s1 和 s2 指向的对象并没有变化。以上证明了 Java 是值传递。

> 疑问续，为什么第一次验证的时候，对象中的值进行了改变呢？  
> 不同的对象引用指向了同一块内存地址

### 封装、继承、多态

## Java 基础知识

### 基本数据类型

### 自动拆装箱

### String

### Java 关键字

### 集合类

### 枚举

### IO

### 动态代理

### 序列化

### 注解

### JMS

### JMX

### 泛型

### 单元测试

### 正则表达式

### 常用的 Java 工具库

### API & SPI

### 异常

### 时间处理

### 编码方式

### 语法糖

## 阅读源代码

1. String
2. Integer
3. Long
4. Enum
5. BigDecimal
6. ThreadLoacal
7. ClassLoader & URLClassLoader
8. HashMap & LinkedHashMap & TreeMap & CounCurrentHashMap
9. HashSet
10. LinkedHashSet & TreeSet

## Java 并发编程

### 并发与并行

### 什么是线程，与进程的区别

### 线程池

### 线程安全

### 锁

### 死锁

### synchronized

### volatile

### sleep 和 wait

### wait 和 notify

### notify 和 notifyAll

### TreadLocal

### 写一个死锁程序

### 写代码解决生产者消费者问题

### 并方包

#### Thread

#### Runnable

#### Callable

#### ReentrantLock

#### ReentrantReadWriteLoack

#### Atomic*

#### Semaphore

#### CountDownLatch

#### ConcurrentHashMap

#### Executors

## JVM

### JVM 内存结构

### Java 内存模型

### 垃圾回收

### JVM 参数及调优

### Java 对象模型

### HotSpot

### 虚拟机性能监控与故障处理工具

## 类加载机制

1. classLoader
2. 类加载过程
3. 双亲委派（破坏双亲委派）
4. 模块化（jboss moudules、osgi、jigsaw）

## 编译与反编译

1. 什么是编译（前端编译、后端编译）
2. 什么是反编译
3. JIT
4. JIT 优化（逃逸分析、栈上分配、标量替换、锁优化）
5. 编译工具：javac
6. 反编译工具：javap、jad、CRF
  
## Java 底层只是

### 字节码、class文件格式

### CPU缓存，L1、L2、L3 和伪共享

### 尾递归

### 位运算

## 设计模式

### 了解23种设计模式

### 会使用常用设计模式

单例的七种写法：懒汉——线程不安全、懒汉——线程安全、饿汉、饿汉——变种、静态内部类、枚举、双重校验锁

工厂模式、适配器模式、策略模式、模板方法模式、观察者模式、外观模式、代理模式

### 不用 synchronized 和 lock，实现线程安全的单例模式

### 实现 AOP

### 实现 IOC

### nio 和 reactor 设计模式

## 网络编程知识

### tcp、udp、http、https

### http/1.0、http/1.1、http/2之前的区别
