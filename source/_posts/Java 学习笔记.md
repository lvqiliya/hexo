---
title: Java 学习笔记
date: 2019-06-09 21:13:25
tags: [Java]
categories: 白科技
---
Java 基础知识与进阶
<!--more-->
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

|          | 值传递               | 引用传递             |
| -------- | -------------------- | -------------------- |
| 根本区别 | 会创建副本（copy）   | 不创建副本           |
| 所以     | 函数无法修改原始对象 | 函数可以修改原始对象 |

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

- 多态

一个对象变量可以指示多种实际类型的现象被称为多态。在运行时能够自动地选择调用哪个方法的现象称为动态绑定。

多态存在的三个必要条件：

1. 继承
2. 重写
3. 父类引用指向子类对象

动态绑定过程描述：

1. 编译器查看对象的声明类型和方法名，获得所有可能被调用的候选方法
2. 编译器查看调用方法时提供的参数类型，获得需要调用的方法名和参数类型
3. 如果是 `private`、`static`、`final` 等方法或构造器，那么编译器可以准确的知道去调用哪一个方法，这种调用方式称为**静态绑定**
4. 动态绑定调用方法时，从子类到父类

它有一个非常重要的特性：无需对现存代码进行修改，就可以对程序进行扩展。需要实现新的功能？那么再派生一个新的类，**重写**父类方法即可。

- 重写

子类对父类的允许访问的方法的实现过程进行重新编写，返回值和参数都不变。外壳不变核心重写。

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

重写（override）与重载（overloading）的区别

| 区别点   | 重载     | 重写               |
| -------- | -------- | ------------------ |
| 参数列表 | 必须修改 | 不能修改           |
| 返回类型 | 可以修改 | 不能修改           |
| 异常     | 可以修改 | 可以减少，不能增加 |
| 访问     | 可以修改 | 可以降低，不能严格 |

重载和重写都是 Java 多态性的不同表现。重载是一个类的多态性表现，重写则是父类与子类之间多态性表现。

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

- 数据类型

4种整型、2种浮点型、1种布尔型、1种字符型

- 取值范围

| 类型  | 储存需求         | 取值范围                   |
| ----- | ---------------- | -------------------------- |
| int   | 4个字节 2^(8\*4) | -2^(8\*4-1) ~ 2^(8\*4-1)-1 |
| short | 2个字节 2^(8\*2) | -2^(8\*2-1) ~ 2^(8\*2-1)-1 |
| long  | 8个字节 2^(8\*8) | -2^(8\*8-1) ~ 2^(8\*8-1)-1 |
| byte  | 1个字节 2^(8\*1) | -2^(8\*1-1) ~ 2^(8\*1-1)-1 |

- 什么是浮点型

Java 中用于表示有小数部分的数值。有 float 和 double 两种浮点类型。在计算浮点数值时，用于表示溢出和出错的情况有三个特殊的浮点数值：正无穷大、负无穷大、NaN（不是一个数字）。正整数/0 = 正无穷大，负整数/0 = 负无穷大， 0/0 或 负数的平方根 = NaN。

- 什么是单/双精度

double 表示这种类型的数值精度是 float 类型的两倍，所以称 float 为单精度 double 为双精度。float 的有效位数是6 ~ 7位，double 的有效位数是15位。绝大多数情况下都用 double 类型。如果需要用 float 类型，那么数值需加一个后缀 F（例如，3.14F）。没有后缀 F 的浮点数（如 3.14）默认为 double 类型。当然也可以手动添加后缀 D（例如，3.14D）。

- 为什么不能用浮点型表示金额

```java
// 执行以下语句
System.out.println(2.0-1.1)

// 打印输出 0.8999999999999999
```

与我们想象的不同，并未打印出 0.9。主要原因是浮点数值采用二进制系统表示，而二进制系统中无法精确的表示分数 1/10。就好像在十进制系统中无法精确表示分数 1/3 一样。小数换算为二进制则是乘以 2，直到小数为 0 为止，每次结果整数位的值从上到下组合起来代表整个小数的二进制数。演算过程如下：

$$0.9*2 = 1.8$$
整数位：1
$$0.8*2 = 1.6$$
整数位：1
$$0.6*2 = 1.2$$
整数位：1
$$0.2*2 = 0.4$$
整数位：0
$$0.4*2 = 0.8$$
整数位：0
$$0.8*2 = 1.6$$
整数位：1

至此出现了无限循环。

### 自动拆装箱

- 包装类型和基本类型

有时，需要将 int 这样的基本类型转换为对象。所有的基本类型（int、short、long、byte、float、double、boolean、char 共计8种）都有一个与之对应的类，它们被称为包装器或包装类型。这些对象包装器拥有很鲜明的名字：Integer、Short、Long、Byte、Float、Doubl、Boolean 和 Character。**对象包装器是不可变的，一旦构造了包装器，就不允许改变其中的值。**

- 什么是自动拆装箱

```java
ArrayList<Integer> list = new ArrayList<>();
list.add(1);

// 它将会自动变换为
list.add(Integer.valueOf(1));
// 这便是自动装箱

// 接下来执行
int n = list.get(0);

// 此时将翻译成
int n = list.get(0).intValue();
// 这便是自动拆箱
```

- Integer 的缓存机制

> 首先需要说明一下 `==` 在使用中的区别。
>
> 对于基础数据类型来讲，它比较的是基础数据类型变量的**值**是否相等。相等返回 true，否则返回 false；  
> 对于引用类型来讲，它比较的是引用类型变量的**地址值**是否相等。相等返回 true，否则返回 false。

接下来说明 Integer 的缓存机制，直接看源码：

```java
/**
 * Cache to support the object identity semantics of autoboxing for values between
 * -128 and 127 (inclusive) as required by JLS.
 *
 * The cache is initialized on first usage.  The size of the cache
 * may be controlled by the {@code -XX:AutoBoxCacheMax=<size>} option.
 * During VM initialization, java.lang.Integer.IntegerCache.high property
 * may be set and saved in the private system properties in the
 * sun.misc.VM class.
 */
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer cache[];

    static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
            sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
                // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;

        cache = new Integer[(high - low) + 1];
        int j = low;
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);

        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert IntegerCache.high >= 127;
    }

    private IntegerCache() {}
}
```

注释第一段说明 IntegerCache 整个静态内部类的作用是，根据 JLS 要求，缓存 -128 和 127（包括）之间的值以支持自动装箱的对象标识语义。  
接下来阐述了在第一次使用的时候 cache 就已经被初始化了，并且它的大小是可以被控制的，通过 `-XX:AutoBoxCacheMax=<size>` 来达到改变大小的目的。

代码分析：

1. 定义并初始化最小值，同时定义最大值，和 cache[]；
2. 静态块中初始化最大值为 127，并获取外部修改的最大值定义为 i；
3. 取 i 和 127 的最大值并赋值给 i，然后取 i 和 （Integer.MAX_VALUE - (-low) -1）的最小值，防止外部定义的值超出了 int 的取值范围；
4. 这时候根据 i 来初始化数组 cache，它的长度就是 i；
5. 从最小值开始，进行自增循环，把值存到 cache 数组，完成缓存。

理解完以上内容后进行代码验证：

```java
Integer a = new Integer(100);
Integer b = new Integer(100);
System.out.println(System.identityHashCode(a));
System.out.println(System.identityHashCode(b));
System.out.println(a == b);

// 预期输出：地址不同，false

// 实际输出
460141958
1163157884
false
```

```java
Integer c = 100;
Integer d = 100;
System.out.println(System.identityHashCode(c));
System.out.println(System.identityHashCode(d));
System.out.println(c == d);

// 预期输出：地址相同，true

// 实际输出
460141958
460141958
true
```

```java
Integer e = 100;
Integer f = new Integer(100);
System.out.println(System.identityHashCode(e));
System.out.println(System.identityHashCode(f));
System.out.println(e == f);

// 预期输出：地址不相同，false，此例跟 a、b 没有区别

// 实际输出
460141958
1163157884
false
```

```java
Integer g = 128;
Integer h = 128;
System.out.println(System.identityHashCode(g));
System.out.println(System.identityHashCode(h));
System.out.println(g == h);

// 预期输出：因为超出了缓存范围 [-128, 127]，故地址不同，false

// 实际输出
460141958
1163157884
false
```

```java
int i = 100;
Integer j = new Integer(100);
Integer k = 100;
System.out.println(System.identityHashCode(i));
System.out.println(System.identityHashCode(j));
System.out.println(System.identityHashCode(k));
System.out.println(i == j);
System.out.println(i == k);
System.out.println(j == k);

// 预期输出：i、k 拥有相同地址；j 的地址不同；
// i == j -> true，i == k -> true，j == k -> false

// 实际输出
460141958
1163157884
460141958
true
true
false
```

> 最后一个例子值得注意！

因为是采用了 `==` 比较，对于包装类来说会先自动拆包为基础数据类型。  两个基础数据类型进行比较的是**值**。

> 思考！

注意到了吧！是 i 和 j 的地址是不同的，而 i 和 k 的地址是相同的。最后一个例子比较的时候是采用的 `==`。对于 j 和 k 来说都是对象，那应该也可以调用 Integer 对象的 equals 方法来比较。

```java
……

System.out.println(j.equals(i));
System.out.println(k.equals(i));

// 输出都为 true
```

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
