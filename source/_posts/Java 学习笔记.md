---
title: Java 学习笔记
date: 2019-06-09 21:13:25
tags: [Java]
categories: 白科技
---

## 面向对象

### 什么是面向对象

对象的行为
对象的状态
对象标识

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

结尾补充

> 在使用 IDEA 进行代码验证的时候，发现无论是基础类型 int 或者是对象 String，当在调用 pass() 方法的时候一旦对传入的参数进行了修改，那么 pass() 方法中的参数则变成了灰色且提示 *Parameter can be converted to a local variable*（参数可以转换为局部变量），说明此时传入的参数并没有被使用到。
>
> 一个方法不能修改一个基本数据类型的参数（即数值型和布尔型）。  
> 一个方法可以改变一个对象参数的状态。  
> 一个方法不能让对象参数引用一个新的对象。

### 封装、继承、多态

- 封装

从形式上看，封装是将数据和行为组合组合在一个包中，并对对象的使用者隐藏了数据的实现方式。

对象的数据称为实例域（instance field），操作数据的过程称为方法（method）。对于每个特定的类实例（对象）都有一组特定的实例域值。这些值的集合就是这个对象的当前状态（state）。当向对象发送一个消息时，它的状态就有可能发生改变。

实现封装的关键在于绝对不能让类中的方法直接访问其他类的实例域。程序仅通过对象的方法来与对象数据进行交互。封装赋予对象“黑盒”特征，这是提高重用性和可靠性的关键。

什么是多态、方法重写与重载

- 重载

有相同的名字、不同的参数，便产生了重载。编译器必须挑选出具体执行哪个方法，它通过各个方法给出的**参数类型**（被调用的方法）与特定方法调用所使用的**值类型**（调用者传参类型）进行匹配来挑选出相应的方法。这个过程可以称为重载解析（overloading resolution）。

需要注意的是，要完整地描述一个方法，需要指出**方法名**以及**参数类型**。这叫做方法的签名（signature）。例如，String 类有4个称为 indexOf 的公有方法。它们的签名是

```java
indexOf(int)
indexOf(int, int)
indexOf(String)
indexOf(String, int)
```

**返回类型不是方法签名的一部分**。也就是说，不能有两个名字相同、参数也相同却返回不同类型值的方法。

Java的继承与实现

“is-a”关系是继承的一个明显特征

构造函数与默认构造函数

构造函数与类同名，在构造某个类的对象时，构造函数便会运行，一遍将实例域初始化为所希望的状态。它总是伴随着 **new** 操作符的执行被调用。每个类都有至少一个构造函数，它的参数可以是 n >= 0 个，它没有返回值。

当且仅当类没有提供任何构造函数的时候，系统才会提供一个默认的构造函数。此时类中的域会被自动地赋为默认值：数值-0、布尔值-false、对象引用-null。

如果在编写类的时候，给出了一个构造函数，哪怕是很简单的，要想使用 `new ClassName()` 来构造实例就必须提供一个默认构造函数（即不带参数的构造函数），例如 `public ClassName() {}`。

类变量、成员变量和局部变量

成员变量和方法作用域

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
