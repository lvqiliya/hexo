---
title: Java 学习笔记
tags:
  - Java
categories: 白科技
abbrlink: c057c77b
date: 2019-06-09 21:13:25
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

- Java的继承与实现

“is-a”关系是继承的一个明显特征。

子类不能直接访问超类的私有域。如果超类没有定义无参构造函数，子类必须显示地调用超类的构造函数。

设计继承的技巧：

1. 将公共操作和域放在超类；
2. 不要使用受保护的域；
3. 使用继承实现 is-a 关系；
4. 除非所有继承的方法都有意义，否则不要使用继承；
5. 在重写方法时，不要改变预期行为；
6. 使用多态，而非类型信息；
7. 不要过多的使用反射。

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

- 数据类型之间的转换

> 实线表示无信息丢失的转换，虚线表示可能有精度损失的转换。
> byte ——> short ——> int ——> long
> char ——> int ——> double
> int --> float
> long --> float, long --> double
> 计算原则如下：
> 有 double 都转成 double 计算
> 否则，有 float 都转成 float 计算
> 否则，有 long 都转成 long 计算
> 否则，都转成 int 类型

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

值得注意的是装箱和拆箱是编译器认可的，而不是虚拟机。编译器在生成类的字节码时，插入必要的方法调用。虚拟机只是执行这些字节码。

```java
public class Demo {
    public static void main(String[] args) {
        ArrayList<Integer> list = new ArrayList<>();
        list.add(1);
        int i = list.get(0);
    }
}
```

以上代码先后实现了装箱操作和拆箱操作，我们来看看类文件：

```class
public class com.qly.Demo {
  public com.qly.Demo();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: new           #2                  // class java/util/ArrayList
       3: dup
       4: invokespecial #3                  // Method java/util/ArrayList."<init>":()V
       7: astore_1
       8: aload_1
       9: iconst_1
      10: invokestatic  #4                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
      13: invokevirtual #5                  // Method java/util/ArrayList.add:(Ljava/lang/Object;)Z
      16: pop
      17: aload_1
      18: iconst_0
      19: invokevirtual #6                  // Method java/util/ArrayList.get:(I)Ljava/lang/Object;
      22: checkcast     #7                  // class java/lang/Integer
      25: invokevirtual #8                  // Method java/lang/Integer.intValue:()I
      28: istore_2
      29: return
}
```

显然，在第 10 行编译器调用了 `Integer.valueOf()`，在 25 行编译器调用了 `Integer.intValue()`。

- Integer 的缓存机制

> 首先需要说明一下 `==` 在使用中的区别。
>
> 对于基础数据类型来讲，它比较的是基础数据类型变量的**值**是否相等。相等返回 true，否则返回 false；  
> 对于引用类型来讲，它比较的是引用类型变量的**地址值**是否相等。相等返回 true，否则返回 false。

补充说明一下 `equals()` 方法。它的作用是判断两个两个对象是否相等，但是分为两种情况。

```java
// 未重写时，调用 Object 类中的方法。它使用的是 == 来判断两个对象的地址是否相同。
public boolean equals(Object obj) { return (this == obj); }

// 已重写时，例如 String 类中的方法。它首先判断地址，然后判断类型，最后比较内容。
@Override
public boolean equals(Object obj) {
    if (this == obj) {
        return true;
    }
    if (obj instanceof String) {
        String objString = (String) obj;
        int n = value.length;
        if (n == objString.length()) {
            char[] v1 = value;
            char[] v2 = objString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i]) {
                    return false;
                }
                i++;
            }
            return true;
        }
    }
    return false;
}
```

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

- 字符串的不可变性

从概念上将，Java 字符串就是 Unicode 字符序列。String 类没有提供用于修改字符串的方法，如果想要修改字符串的内容，用拼接的方式实现。

由于无法修改 Java 字符串中的字符，所以在 Java 文档中将 String 类对象称为**不可变字符串**。它有一个优点，编译器可以让字符串共享。可以想象将各种字符串存放在公共存储池中，字符串变量指向存储池相应的位置。如果复制一个字符串变量，原始字符串与复制的字符串共享相同的字符。关于共享问题还有补充，只有字符串常量是共享的，而 + 或者 substring 等操作产生的结果并不是共享的。

> 值得注意的是，如果执行代码 `String sub = s.substring(0);` 和 `Sting sub = s.subtring(0, s.length());`，sub 实际上指向的是 s 指向的内存地址，也即是它们是共享的。具体原因见 substring 原理分析。

以上为概念。接下来看源码：

```java
// JDK 1.8

/** The value is used for character storage. */
private final char value[];

/** Cache the hash code for the string */
private int hash; // Default to 0
```

首先 String 类中声明了一个**不可变的字符数组**用于存储字符串，同时声明一个整型用来作为 hash code 的缓存，具体分析见源码分析部分。可以看到 value[] 是私有变量，且 String 类中没有提供类似 setValue() 修改字符串的方法，所以一旦字符串被定义，它本身是无法被修改的。

- JDK 6 和 JDK 7 中 substring 的原理及区别

从 substring 的用法切入。无论是 JDK 6 或者 JDK 7+，substring 的方法只有两种:

```java
// 给定起始偏移量
public String substring(int beginIndex) { …… }

// 给定起始偏移量和终止偏移量
public String substring(int beginIndex, int endIndex) { …… }
```

基于 JDK 1.8 中 String 的构造函数，对以上两个方法进行设想，那么应该如何实现呢？

```java
public String(char value[], int offset, int count) {
    if (offset < 0) {
        throw new StringIndexOutOfBoundsException(offset);
    }
    if (count <= 0) {
        if (count < 0) {
            throw new StringIndexOutOfBoundsException(count);
        }
        if (offset <= value.length) {
            this.value = "".value;
            return;
        }
    }
    // Note: offset or count might be near -1>>>1.
    if (offset > value.length - count) {
        throw new StringIndexOutOfBoundsException(offset + count);
    }
    this.value = Arrays.copyOfRange(value, offset, offset+count);
}
```

```java
/**
 * 实现前提
 * 显然，参数的取值范围是 0 <= beginIndex <= value.length
 * <p>
 * 针对它进行判断：
 * 若小于 0 ，则抛出 StringIndexOutOfBoundsException
 * 若大于 value.lenth，同样抛出 StringIndexOutOfBoundsException
 * 所有异常情况已经考虑完全，调用 String 的构造方法完成本方法的实现
 *
 * @param beginIndex
 * @return
 */
public String substring(int beginIndex) {
    if (beginIndex < 0) {
        throw new StringIndexOutOfBoundsException(beginIndex);
    }
    if (beginIndex > value.length) {
        throw new StringIndexOutOfBoundsException(value.length - beginIndex);
    }
    return new String(value, beginIndex, value.length - beginIndex);
}

// 代码进行优化后，最终如下
public String substring(int beginIndex) {
    if (beginIndex < 0) {
        throw new StringIndexOutOfBoundsException(beginIndex);
    }
    int len = value.length - beginIndex;
    if (len < 0) {
        throw new StringIndexOutOfBoundsException(len);
    }
    return beginIndex == 0 ? this : new String(value, beginIndex, len);
}
```

```java
/**
 * 实现前提
 * 因为有两个参数，除开自身的取值范围之外还有一个条件
 * 0 <= beginIndex <= value.length
 * 0 <= endIndex <= value.length
 * beginIndex <= endIndex
 * 在进行判断的时候要善于利用第三个条件
 * <p>
 * 针对它们分别进行判断：
 * 若 beginIndex 小于 0 ，则抛出 StringIndexOutOfBoundsException
 * 若 endIndex 大于 value.length ，则抛出 StringIndexOutOfBoundsException
 * 若 int len = endIndex - begin 小于 0，抛出异常 StringIndexOutOfBoundsException
 * 所有异常情况已经考虑完全，调用 String 的构造方法完成本方法的实现
 *
 * @param beginIndex
 * @param endIndex
 * @return
 */
public String substring(int beginIndex, int endIndex) {
    if (beginIndex < 0) {
        throw new StringIndexOutOfBoundsException(beginIndex);
    }
    if (endIndex > value.length) {
        throw new StringIndexOutOfBoundsException(endIndex);
    }
    int len = endIndex - beginIndex;
    if (len < 0) {
        throw new StringIndexOutOfBoundsException(len);
    }
    return ((beginIndex == 0) && (endIndex == value.length)) ? this : new String(value, beginIndex, len);
}
```

- replaceFirst、replaceAll、replace 区别

```java
/**
 * 使用给定 replacement 替换此字符串匹配给定正则表达式 regex 的第一个子字符串
 * Replaces the first substring of this string that matches the given <a
 * href="../util/regex/Pattern.html#sum">regular expression</a> with the
 * given replacement.
 * ......
 * @param regex
 * @param replacement
 * @return
 */
public String replaceFirst(String regex, String replacement) {
    return Pattern.compile(regex).matcher(this).replaceFirst(replacement);
}
```

```java
/**
 * 使用给定 replacement 替换此字符串匹配给定正则表达式 regex 的所有子字符串
 * Replaces each substring of this string that matches the given <a
 * href="../util/regex/Pattern.html#sum">regular expression</a> with the
 * given replacement.
 * ......
 * Note that backslashes ({@code \}) and dollar signs ({@code $}) in the
 * replacement string may cause the results to be different than if it were
 * being treated as a literal replacement string;
 * ......
 * @param regex
 * @param replacement
 * @return
 */
public String replaceAll(String regex, String replacement) {
    return Pattern.compile(regex).matcher(this).replaceAll(replacement);
}
```

```java
/**
 * 使用给定字符序列 replacement 替换此字符串匹配给定字符序列 target 的所有子字符串
 * Replaces each substring of this string that matches the literal target
 * sequence with the specified literal replacement sequence.
 * ......
 * @param target
 * @param replacement
 * @return
 */
public String replace(CharSequence target, CharSequence replacement) {
    return Pattern.compile(target.toString(), Pattern.LITERAL).matcher(this)
            .replaceAll(Matcher.quoteReplacement(replacement.toString()));
}

public static String quoteReplacement(String s) {
    if ((s.indexOf('\\') == -1) && (s.indexOf('$') == -1))
        return s;
    StringBuilder sb = new StringBuilder();
    for (int i=0; i<s.length(); i++) {
        char c = s.charAt(i);
        if (c == '\\' || c == '$') {
            sb.append('\\');
        }
        sb.append(c);
    }
    return sb.toString();
}

/**
 * 用 newChar 替换字符串中出现的所有 oldChar 字符，并返回替换后的新字符串
 * Returns a string resulting from replacing all occurrences of
 * {@code oldChar} in this string with {@code newChar}.
 * ......
 * @param   oldChar   the old character.
 * @param   newChar   the new character.
 * @return
 */
public String replace(char oldChar, char newChar) { ... }
```

|              | 参数 1              | 内容说明   | 参数 2                   | 内容说明 |
| ------------ | ------------------- | ---------- | ------------------------ | -------- |
| replaceFirst | String regex        | 正则表达式 | String replacement       | 字符串   |
| replaceAll   | String regex        | 正则表达式 | String replacement       | 字符串   |
| replace      | CharSequence target | 字符序列   | CharSequence replacement | 字符序列 |
| replace      | char oldChar        | 字符       | char newChar             | 字符     |

由上表可知，replaceFirst 和 replaceAll 应该接收正则表达式然后进行匹配，然后将 replacement 替换被匹配的子串，返回一个新的字符串；而 replace 则是接收字符或者字符串，然后将后者替换被匹配的子串。

To be continued...

- String 对“+”的重载、字符串拼接的几种方式和区别

首先利用 `+` 完成字符串拼接，编译并执行成功：

```java
// example 1
System.out.println(("a" + "b") == "ab");

// example 2
String s1 = "1";
String s2 = "2";
String s = "12";
System.out.println((s1 + s2) == s);
```

打印输出：

```java
true
false
```

如果跟你预想中的结果不一样，可能你有一个知识点欠缺了。example 1 中是对**字符常量**进行 `+` 运算，example 2 中是对**字符串**进行运算。我们接着看 `.class` 文件结构：

```class
public class com.qly.Demo {
  public com.qly.Demo();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: ldc           #2                  // String ab
       2: astore_1
       3: ldc           #2                  // String ab
       5: astore_2
       6: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
       9: aload_1
      10: aload_2
      11: if_acmpne     18
      14: iconst_1
      15: goto          19
      18: iconst_0
      19: invokevirtual #4                  // Method java/io/PrintStream.println:(Z)V
      22: ldc           #5                  // String 1
      24: astore_3
      25: ldc           #6                  // String 2
      27: astore        4
      29: ldc           #7                  // String 12
      31: astore        5
      33: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
      36: new           #8                  // class java/lang/StringBuilder
      39: dup
      40: invokespecial #9                  // Method java/lang/StringBuilder."<init>":()V
      43: aload_3
      44: invokevirtual #10                 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      47: aload         4
      49: invokevirtual #10                 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      52: invokevirtual #11                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      55: aload         5
      57: if_acmpne     64
      60: iconst_1
      61: goto          65
      64: iconst_0
      65: invokevirtual #4                  // Method java/io/PrintStream.println:(Z)V
      68: return
}
```

第 0 行并未进行相加操作，而是通过编译器优化成一个 `String ab`；第 22、25、29 行分别初始化了三个字符串，在调用 `+` 的时候，通过 36 行可以看到是 new 了一个 **StringBuilder** 对象，然后在 44 行调用了 `append()` 方法。

总结来说，两个字符常量用 `+` 操作的时候，编译器会预编译成一个字符常量；两个字符串用 `+` 操作的时候，会先 new 一个 StringBuilder，然后调用 `append()` 方法完成字符串的拼接。

除开直接使用 `+` 对字符串进行拼接之外，还可以通过 concat、StringBuffer、StringBuilder 来达到效果。下面的代码分别展示其用法和效率：

```java
public static void main(String[] args) {
    plus();
    concat();
    stringBuffer();
    stringBuilder();
}

public static void plus() {
    String result = "";
    long start = System.currentTimeMillis();
    for (int i = 0; i < 100000; i++) {
        result += "a";
    }
    long end = System.currentTimeMillis();
    System.out.println(end - start);
}

public static void concat() {
    String result = "";
    long start = System.currentTimeMillis();
    for (int i = 0; i < 100000; i++) {
        result = result.concat("a");
    }
    long end = System.currentTimeMillis();
    System.out.println(end - start);
}

public static void stringBuffer() {
    StringBuffer buffer = new StringBuffer();
    long start = System.currentTimeMillis();
    for (int i = 0; i < 1000000; i++) {
        buffer.append("a");
    }
    long end = System.currentTimeMillis();
    System.out.println(end - start);

}

public static void stringBuilder() {
    StringBuilder builder = new StringBuilder();
    long start = System.currentTimeMillis();
    for (int i = 0; i < 1000000; i++) {
        builder.append("a");
    }
    long end = System.currentTimeMillis();
    System.out.println(end - start);
}
```

输出结果：

```java
5685
1190
49
22
```

针对以上内容做如下汇总：

|          | +    | concat | StringBuffer | StringBuilder |
| -------- | ---- | ------ | ------------ | ------------- |
| 循环次数 | 十万 | 十万   | 百万         | 百万          |
| 耗时     | 5685 | 1190   | 49           | 22            |

显然，`+` 的效率是最低的，StringBuilder 的效率是最高的。因为 StringBuffer 是线程安全的，同步会耗费一部分时间。

concat 的原理是，在拼接字符串的时候，根据原字符串和字符串参数的长度申请一个字符数组用以存储新的字符串，也就是说每拼接一次就需要申请新的字符数组，十万次的拼接中 concat 需要十万次的扩容。

StringBuilder 的原理是，在拼接字符串的时候对现有长度进行翻倍，十万次的拼接中只需要 log100000 < 17 次扩容。

- String.valueOf 和 Integer.toString 的区别

```java
/**
 * Returns a {@code String} object representing the
 * specified integer. The argument is converted to signed decimal
 * representation and returned as a string, exactly as if the
 * argument and radix 10 were given as arguments to the {@link
 * #toString(int, int)} method.
 *
 * 返回一个表示指定整数的字符串。
 * 参数被转换成带符号的十进制表示，并以字符串返回。
 * 准确来说就像是这个参数和进制 10 同时作为 Integer.toString(int i, int radix) 方法的参数一样。
 *
 * @param   i   an integer to be converted.
 * @return  a string representation of the argument in base&nbsp;10.
 */
public static String toString(int i) {
    if (i == Integer.MIN_VALUE)
        return "-2147483648";
    int size = (i < 0) ? stringSize(-i) + 1 : stringSize(i);
    char[] buf = new char[size];
    getChars(i, size, buf);
    return new String(buf, true);
}
```

这是 Integer 中的 toString 方法。首先会根据入参 i 来取一个 size 作为字符数组的长度，然后把 i 放到字符数组里面，再调用构造函数 String(char[] value, boolean share) 实例化 i 并返回。

String 类中有大量的 valueOf() 方法进行重载，参数包括了基本数据类型和对象，基本数据类型分为 char、int 和其他。对于 char 来说大多数都是字符数组，而 byte 和 short 作为参数时会默认转化为 int 类型，其他类型就调用其本身。

- switch 对 String 的支持

写一个简单的 switch 判断：

```java
public static void main(String[] args) {
    String s = "hello";
    switch (s) {
        case "hello":
            System.out.println("world");
            break;
        default:
            break;
    }
}
```

输出结果：

```java
world
```

在 terminal 中执行 `javap -c Demo.class` 并观察：

```class
public class com.qly.Demo {
  public com.qly.Demo();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: ldc           #2                  // String hello
       2: astore_1
       3: aload_1
       4: astore_2
       5: iconst_m1
       6: istore_3
       7: aload_2
       8: invokevirtual #3                  // Method java/lang/String.hashCode:()I
      11: lookupswitch  { // 1
              99162322: 28
               default: 39
          }
      28: aload_2
      29: ldc           #2                  // String hello
      31: invokevirtual #4                  // Method java/lang/String.equals:(Ljava/lang/Object;)Z
      34: ifeq          39
      37: iconst_0
      38: istore_3
      39: iload_3
      40: lookupswitch  { // 1
                     0: 60
               default: 71
          }
      60: getstatic     #5                  // Field java/lang/System.out:Ljava/io/PrintStream;
      63: ldc           #6                  // String world
      65: invokevirtual #7                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      68: goto          71
      71: return
}
```

推测，在第 8 行调用 String.hashCode() 方法。接下来反编译 Demo.class：

```java
public static void main(String[] args) {
    String s = "hello";
    byte var3 = -1;
    switch(s.hashCode()) {
    case 99162322:
        if (s.equals("hello")) {
            var3 = 0;
        }
    default:
        switch(var3) {
        case 0:
            System.out.println("world");
        default:
        }
    }
}
```

显然，switch 调用了String.hashCode()，当 hashCode 一致时，对字符串进行 equals 比较。之所以这么做是因为不同的字符串通过执行 String.hashCode() 方法可能会计算出相同的值。最后进入switch-byte这个结构中，则完成整个逻辑。

总的说来，switch 支持字符串是利用了字符串的 hashCode，本质上还是 switch-int，经过 equals 方法处理 hashCode 可能会重复的问题之后，利用 switch-byte 完成精确匹配。

- 字符串池、常量池（运行时常量池、Class 常量池）、intern

Class（Java类）文件中除了有类的版本、字段、方法、接口描述等信息外，还有一项信息是常量池（Constant Pool Table），用于存放编译期生成的**各种字面量**和**符号引用**，这个池子就是 Class 常量池。而这一部分内容将在类加载后存放到方法区中的运行时常量池中。运行期间可以将新的常量放入池中，即 String 类中的 intern() 方法。

```java
/**
 * Returns a canonical representation for the string object.
 * <p>
 * A pool of strings, initially empty, is maintained privately by the
 * class {@code String}.
 * <p>
 * When the intern method is invoked, if the pool already contains a
 * string equal to this {@code String} object as determined by
 * the {@link #equals(Object)} method, then the string from the pool is
 * returned. Otherwise, this {@code String} object is added to the
 * pool and a reference to this {@code String} object is returned.
 * <p>
 * It follows that for any two strings {@code s} and {@code t},
 * {@code s.intern() == t.intern()} is {@code true}
 * if and only if {@code s.equals(t)} is {@code true}.
 * <p>
 * All literal strings and string-valued constant expressions are
 * interned. String literals are defined in section 3.10.5 of the
 * <cite>The Java&trade; Language Specification</cite>.
 *
 * @return  a string that has the same contents as this string, but is
 *          guaranteed to be from a pool of unique strings.
 */
public native String intern();
```

字符串池是由类私下维护的，它最开始是空的。

当 intern() 被调用时，如果字符串池中已经包含一个字符串与对象引用所指的字符串相等（通过 equals() 方法比较），那么从字符串池返回已存在的字符串。否则对象引用的字符串应该被添加到字符串池中，并且返回这个字符串的引用。

```java
String s = "abc";
String s1 = "abc01";
String s2 = s + "01";
String s3 = new String("abc01");
String s4 = s3.intern();
System.out.println(s1 == s2);
System.out.println(s1 == s3);
System.out.println(s1 == s4);
System.out.println(s2 == s3);
System.out.println(s2 == s4);
System.out.println(s3 == s4);
System.out.println(System.identityHashCode(s1));
System.out.println(System.identityHashCode(s2));
System.out.println(System.identityHashCode(s3));
System.out.println(System.identityHashCode(s4));
```

输出结果：

```java
false
false
true
false
false
false
460141958
1163157884
1956725890
460141958
```

分析：s1 是字符串常量，保存至字符串池中；s2 实质上是调用了 StringBuilder，实例化的对象保存在堆中；s3 是直接 new 了一个 String 对象，同理保存在堆中。至此三个变量的地址都不相同。s4 等于 s3.intern()，根据前面的说明做推理。字符串池目前已经保存了一个字符串 s1，当 s3 调用 intern() 方法时，s3 所指的字符串 “abc01” ——它目前存在堆中——与字符串池中的 s1 进行 equals，发现相等，那么返回 s1 给 s4，即 s1 和 s4 指向同一个对象。

### 反射

能够分析类能力的程序称为反射（reflective）。反射可以用来做以下事情：

1. 在运行中分析类的能力。
2. 在运行中查看对象，例如，编写一个 toString 方法供所有类使用。
3. 实现通用的数组操作代码。
4. 利用 Method 对象。

在程序运行期间，Java 运行时系统始终为所有的对象维护一个被称为运行时的类型标识。这个信息跟踪着每一个对象所属的类。虚拟机利用运行时类型信息选择相应的方法执行。

我们可以通过专门的 Java 类访问这些信息。保存这些信息的类被称为 Class。一共有三种方法来获取 Class 类：

```java
// 调用 Object 类中的 getClass() 方法
Pet pet;
Class c1 = pet.getClass();

// 调用 Class 类中的静态方法 forName()
String className = "java.lang.String";
Class c2 = Class.forName(className);

// 直接 .class
Class c3 = Date.class;
Class c4 = int.class;
// 请注意，一个 Class 对象实际上表示的是一个类型，而这个类型未必是一个类。虽然 int 是基础类型，但是 c4 是一个 Class 类型的对象。
```

还有一个很有用的方法 newInstance()，可以用来快速地创建一个类的实例。例如：

```java
String className = "java.lang.String";
Class c2 = Class.forName(className);
Object o = c2.newInstance();
// 这一步操作是调用默认构造函数，如果没有默认构造函数则会报错。
```

- 利用反射分析类的能力

在 java.lang.reflect 包中有三个类 Field、Method 和 Constructor 分别描述类的域、方法和构造器。

|                     | 功能         | Field | Method | Constructor |
| ------------------- | ------------ | ----- | ------ | ----------- |
| getName()           | 获取名字     | V     | V      | V           |
| getType()           | 获取类型     | V     | -      | -           |
| getParameterTypes() | 获取参数类型 | -     | V      | V           |
| getModifiers()      | 获取修饰符   | V     | V      | V           |

Class 类中的 getFieds、getMethods 和 getConstructors 方法将分别返回类提供的 public 域、方法和构造器数组，这其中包括父类的公有成员。

Class 类中的 getDeclaredFields、getDeclaredMethods 和 getDeclaredConstructors 方法分别返回类中声明的全部域、方法和构造器，这其中包括私有和受保护的成员，但不包括超类的成员。

具体应用见下面代码：

```java
public class Demo {
    public static void main(String[] args) {
        String className = "java.lang.String";
        try {
            Class clazz = Class.forName(className);
            Class superClazz = clazz.getSuperclass();
            String modifier = Modifier.toString(clazz.getModifiers());
            if (modifier.length() > 0) {
                System.out.print(modifier + " ");
            }
            System.out.print("class " + className);
            if (superClazz != null && superClazz != Object.class) {
                System.out.print(" extends " + superClazz.getName());
            }
            Class[] impls = clazz.getInterfaces();
            if (impls != null && impls.length > 0) {
                System.out.print(" implements");
                StringBuilder sb = new StringBuilder();
                for (Class impl : impls) {
                    sb.append(" ").append(impl).append(",");
                }
                System.out.print(sb.toString().substring(0, sb.length() - 1));
            }
            System.out.println(" {");
            printConstructors(clazz);
            System.out.println();
            printMethod(clazz);
            System.out.println();
            printFields(clazz);
            System.out.println("}");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }

    private static void printConstructors(Class clazz) {
        Constructor[] constructors = clazz.getDeclaredConstructors();
        for (Constructor constructor : constructors) {
            System.out.print("    ");
            String modifier = Modifier.toString(constructor.getModifiers());
            if (modifier.length() > 0) {
                System.out.print(modifier + " ");
            }
            String name = constructor.getName();
            System.out.print(name + "(");
            Class[] parameters = constructor.getParameterTypes();
            for (int i = 0; i < parameters.length; i++) {
                if (i>0) {
                    System.out.print(", ");
                }
                System.out.print(parameters[i].getName());
            }
            System.out.println(");");
        }
    }

    private static void printMethod(Class clazz) {
        Method[] methods = clazz.getDeclaredMethods();
        for (Method method : methods) {
            System.out.print("    ");
            String modifier = Modifier.toString(method.getModifiers());
            if (modifier.length() > 0) {
                System.out.print(modifier + " ");
            }
            Class returnType = method.getReturnType();
            String name = method.getName();
            System.out.print(returnType + " " + name + "(");
            Class[] parameterTypes = method.getParameterTypes();
            for (int i = 0; i < parameterTypes.length; i++) {
                if (i > 0) {
                    System.out.print(", ");
                }
                System.out.print(parameterTypes[i].getName());
            }
            System.out.println(");");
        }
    }

    private static void printFields(Class clazz) {
        Field[] fields = clazz.getDeclaredFields();
        for (Field field : fields) {
            System.out.print("    ");
            String modifier = Modifier.toString(field.getModifiers());
            Class type = field.getType();
            String name = field.getName();
            if (modifier.length() > 0) {
                System.out.println(modifier + " " + type.getName() + " " + name + ";");
            }
        }
    }
}
```

输出结果：

```java
public final class java.lang.String implements interface java.io.Serializable, interface java.lang.Comparable, interface java.lang.CharSequence {
    public java.lang.String([B, int, int);
    public java.lang.String([B, java.nio.charset.Charset);
    public java.lang.String([B, java.lang.String);
    public java.lang.String([B, int, int, java.nio.charset.Charset);
    public java.lang.String([B, int, int, java.lang.String);
    java.lang.String([C, boolean);
    public java.lang.String(java.lang.StringBuilder);
    public java.lang.String(java.lang.StringBuffer);
    public java.lang.String([B);
    public java.lang.String([I, int, int);
    public java.lang.String();
    public java.lang.String([C);
    public java.lang.String(java.lang.String);
    public java.lang.String([C, int, int);
    public java.lang.String([B, int);
    public java.lang.String([B, int, int, int);

    public boolean equals(java.lang.Object);
    public class java.lang.String toString();
    public int hashCode();
    public int compareTo(java.lang.String);
    public volatile int compareTo(java.lang.Object);
    public int indexOf(java.lang.String, int);
    public int indexOf(java.lang.String);
    public int indexOf(int, int);
    public int indexOf(int);
    static int indexOf([C, int, int, [C, int, int, int);
    static int indexOf([C, int, int, java.lang.String, int);
    public static class java.lang.String valueOf(int);
    public static class java.lang.String valueOf(long);
    public static class java.lang.String valueOf(float);
    public static class java.lang.String valueOf(boolean);
    public static class java.lang.String valueOf([C);
    public static class java.lang.String valueOf([C, int, int);
    public static class java.lang.String valueOf(java.lang.Object);
    public static class java.lang.String valueOf(char);
    public static class java.lang.String valueOf(double);
    public char charAt(int);
    private static void checkBounds([B, int, int);
    public int codePointAt(int);
    public int codePointBefore(int);
    public int codePointCount(int, int);
    public int compareToIgnoreCase(java.lang.String);
    public class java.lang.String concat(java.lang.String);
    public boolean contains(java.lang.CharSequence);
    public boolean contentEquals(java.lang.CharSequence);
    public boolean contentEquals(java.lang.StringBuffer);
    public static class java.lang.String copyValueOf([C);
    public static class java.lang.String copyValueOf([C, int, int);
    public boolean endsWith(java.lang.String);
    public boolean equalsIgnoreCase(java.lang.String);
    public static transient class java.lang.String format(java.util.Locale, java.lang.String, [Ljava.lang.Object;);
    public static transient class java.lang.String format(java.lang.String, [Ljava.lang.Object;);
    public void getBytes(int, int, [B, int);
    public class [B getBytes(java.nio.charset.Charset);
    public class [B getBytes(java.lang.String);
    public class [B getBytes();
    public void getChars(int, int, [C, int);
    void getChars([C, int);
    private int indexOfSupplementary(int, int);
    public native class java.lang.String intern();
    public boolean isEmpty();
    public static transient class java.lang.String join(java.lang.CharSequence, [Ljava.lang.CharSequence;);
    public static class java.lang.String join(java.lang.CharSequence, java.lang.Iterable);
    public int lastIndexOf(int);
    public int lastIndexOf(java.lang.String);
    static int lastIndexOf([C, int, int, java.lang.String, int);
    public int lastIndexOf(java.lang.String, int);
    public int lastIndexOf(int, int);
    static int lastIndexOf([C, int, int, [C, int, int, int);
    private int lastIndexOfSupplementary(int, int);
    public int length();
    public boolean matches(java.lang.String);
    private boolean nonSyncContentEquals(java.lang.AbstractStringBuilder);
    public int offsetByCodePoints(int, int);
    public boolean regionMatches(int, java.lang.String, int, int);
    public boolean regionMatches(boolean, int, java.lang.String, int, int);
    public class java.lang.String replace(char, char);
    public class java.lang.String replace(java.lang.CharSequence, java.lang.CharSequence);
    public class java.lang.String replaceAll(java.lang.String, java.lang.String);
    public class java.lang.String replaceFirst(java.lang.String, java.lang.String);
    public class [Ljava.lang.String; split(java.lang.String);
    public class [Ljava.lang.String; split(java.lang.String, int);
    public boolean startsWith(java.lang.String, int);
    public boolean startsWith(java.lang.String);
    public interface java.lang.CharSequence subSequence(int, int);
    public class java.lang.String substring(int);
    public class java.lang.String substring(int, int);
    public class [C toCharArray();
    public class java.lang.String toLowerCase(java.util.Locale);
    public class java.lang.String toLowerCase();
    public class java.lang.String toUpperCase();
    public class java.lang.String toUpperCase(java.util.Locale);
    public class java.lang.String trim();

    private final [C value;
    private int hash;
    private static final long serialVersionUID;
    private static final [Ljava.io.ObjectStreamField; serialPersistentFields;
    public static final java.util.Comparator CASE_INSENSITIVE_ORDER;
}
```

- 在运行时使用反射分析对象

### 接口与内部类

本章分为三点：接口、内部类、代理。接口主要是用来描述类具有什么功能，而并不给出每个功能的具体实现。一个类可以实现多个接口。

- 接口

接口不是类，而是对类的一组需求描述，这些类要遵从接口描述的统一格式进行定义。接口中所有的方法自动地属于 public。接口中可以定义常量，但绝不能含有实例域，也不能在接口中实现方法。

让一个类实现一个接口，通常需要两个步骤：将类声明为实现给定的接口，即 implements 关键字；对接口中所有的方法进行定义。

接口具有这些特性：接口不是类，不能用 new 运算符实例化一个接口。接口可以声明接口变量，但接口变量必须引用实现了接口的类对象。接口可以继承接口，接口也可以被扩展。接口可以包含常量。

接口可以提供多重继承的大多数好处，同时还能避免多重继承的复杂性和低效性（To be continued ...）。

- 对象克隆

当拷贝一个变量时，原始变量与拷贝变量引用同一个对象，改变一个引用变量所引用对象的状态将会对另一个变量产生影响。如果创建一个对象的新的 copy，它的最初状态与 original 一样，但以后可以各自改变各自的状态，就使用 clone() 方法。但仅仅调用 Object 类中的 clone() 方法是**浅拷贝**，如果对象中只有数值和基本类型，这样的操作没有问题，但是如果在对象中包含子对象的引用，拷贝的结果会使得两个域引用同一个子对象。从结果来讲，浅拷贝并没有克隆包含在对象中的内部对象。为了实现**深拷贝**，必须克隆所有可变的实例域。clone() 默认时浅拷贝，下面时深拷贝的的示例：

```java
class Pet implements Cloneable {
    private String name;
    private Integer age;

    ......

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}

class Cat implements Cloneable {
    private Pet pet;
    private String type;

    ......

    @Override
    protected Object clone() throws CloneNotSupportedException {
        Cat copy = (Cat) super.clone();
        copy.pet = (Pet) pet.clone();
        return copy;
    }
}
```

> Cloneable 接口中没有方法，它是一个表能力的标识接口。
> 重写 clone() 方法，首先该类必须实现 Cloneable 接口，然后重写。

- 内部类

内部类时定义在另一个类中的类。为何使用内部类，有以下三个原因：

1. 内部类方法可以访问该类定义所在的作用域中的数据，包括私有数据。
2. 内部类可以对同一个包中的其他类隐藏起来。
3. 当想要定义一个回调函数且不想编写大量代码时，使用匿名内部类比较便捷。

### Java 关键字原理及用法

- private、default、protected、public

| 修饰符    | 类内部 | 同一包 | 子类 | 任何地方 |
| --------- | ------ | ------ | ---- | -------- |
| private   | Yes    |        |      |          |
| default   | Yes    | Yes    |      |          |
| protected | Yes    | Yes    | Yes  |          |
| public    | Yes    | Yes    | Yes  | Yes      |

- const
- final
- instanceof
- static
- synchronized
- transient

默认情况下，执行了对象序列化之后，类中所有属性的内容将全部都被序列化。但是很多情况下，有一些属性并不需要序列化处理，此时在属性定义上使用 transient 关键字。此时该属性将不会被序列化，当对象被反序列化的时候，该属性为默认值（0 或者 null）。transient 只能修饰变量，不能修饰类和方法。

- volatile

### 集合类

常用集合类的使用、ArrayList 和 LinkedList 和 Vector 的区别 、SynchronizedList 和 Vector 的区别、HashMap、HashTable、ConcurrentHashMap 区别、

Set 和 List 区别？Set 如何保证元素不重复？

Java 8 中 stream 相关用法、apache 集合处理工具类的使用、不同版本的 JDK 中 HashMap 的实现的区别以及原因

Collection 和 Collections 区别

Arrays.asList 获得的 List 使用时需要注意什么

Enumeration 和 Iterator 区别

fail-fast 和 fail-safe

CopyOnWriteArrayList、ConcurrentSkipListMap

### 枚举

枚举的用法、枚举的实现、枚举与单例、Enum 类

Java 枚举如何比较

switch 对枚举的支持

枚举的序列化如何实现

枚举的线程安全性问题

### IO

### 动态代理

### 序列化

所谓对象序列化指的是将内存中保存的对象以二进制数据流的形式进行处理，可以实现对象的保存或网络传输。如果要序列化对象，那么对象所在的类必须实现 java.io.Serializable 接口，作为序列化的标记。该接口描述了类的能力，同 Cloneable。

- 序列化与反序列化

可以利用一下两个类完成序列化与反序列化操作：

| 类名称   | 序列化：ObjectOutputStream                                                                          | 反序列化：ObjectInputStream                                                                      |
| -------- | --------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| 类定义   | public class ObjectOutputStream extends OutputStream implements ObjectOutput, ObjectStreamConstants | public class ObjectInputStream extends InputStream implements ObjectInput, ObjectStreamConstants |
| 构造方法 | public ObjectOutputStream(OutputStream out) throws IOException                                      | public ObjectInputStream(InputStream in) throws IOException                                      |
| 操作方法 | public final void writeObject(Object obj) throws IOException                                        | public final Object readObject() throws IOException, ClassNotFoundException                      |

### 注解

### JMS

### JMX

### 泛型

### 单元测试

### 正则表达式

### 常用的 Java 工具库

### API & SPI

### 异常

- 异常分类

所有异常都是由 java.lang.Throwable 继承而来，在下一层立即分解为两个分支：Error 和 Exception。

Error 类层次结构描述了 Java 运行时系统的内部错误和资源耗尽错误。程序无法处理 Error。

Exception 类层次是程序可以处理的异常。这个层次结构分解为两个分支：RuntimeException 和另外的异常。划分两个分支的规则是：由程序错误导致的异常属于 RuntimeException，如果出现 RuntimeException 异常，一定是代码有问题。而程序本身没有问题，但由像 I/O 错误这类问题导致的异常属于其他异常。

> 派生于 RuntimeException 的异常包含：  
> 错误的类型转换；  
> 数组访问越界；  
> 访问空指针；  
> 算数异常。
>
> 不是派生于 RuntimeException 的异常包含：  
> 试图在文件尾部后面读取数据；  
> 试图打开一个不存在的文件；  
> 试图根据给定字符串查找 Class 对象，这个字符串表示的类并不存在。

Java 语言规范将派生于 Error 类或 RuntimeException 类的所有异常称为未检查异常。

- 异常处理

以下情况应该抛出异常：

> 调用一个抛出已检查异常的方法，例如，FileInputStream 构造器。  
> 程序运行过程中发现错误，并利用 throw 语句抛出一个已检查的异常。
> 程序错误，例如数组下标越界。
> Java 虚拟机和运行时库出现的内部错误。

- 自定义异常

定义一个派生于 Exception 的类，或者是派生于 Exception 子类的类。习惯上，定义的类应该包含两个构造器，一个是默认构造器，另一个是带有详细描述新的构造器（超类 Throwable 的 toString 方法将会打印出这些详细信息）。

- 捕获异常

try 块：用于捕获异常。其后可接零个或多个catch块，如果没有catch块，则必须跟一个finally块。

catch 块：用于处理try捕获到的异常。

finally 块：无论是否捕获或处理异常，finally块里的语句都会被执行。当在try块或catch块中遇到return语句时，finally语句块将在方法返回之前被执行。

- 异常链

异常链指将捕获的异常包装进一个新的异常中并重新抛出的异常处理方式。在 catch 子句中可以抛出一个异常，这样做的目的是改变异常的类型。如果开发了一个供其他程序员使用的子系统，那么，用于子系统故障的异常类型可能会产生多种解释。ServletException 就是这样的一个异常类型的例子。执行 servlet 的代码可能不想知道发生错误的细节原因，但是希望明确知道 servlet 是否出了问题。

```java
try {

} catch (SQLException e) {
    throw new ServletException("database error: " + e.getMessage);
}
```

这里，ServletException 用了带有异常信息文本的构造器来构造。不过，还有一种更好的处理方法，并且将原始异常设置为新异常的“原因”：

```java
try {

} catch (SQLException e) {
    Throwable se = new ServletException("database error");
    se.initCause(e);
    throw se;
}
```

当捕获到异常时，就可以使用 `Throwable e = se.getCause();` 重新得到原始异常信息。这种包装技术可以让用户抛出子系统中的高级异常，同时不会丢失原始异常的细节。

- try-with-resources

```java
public void print(String[] args) {
    String filePath = "D:" + File.separator + "test.txt";
    try (Writer os = new FileWriter(filePath)) {
        os.write("hello\r\n你好\r\nこんにちは\r\n");
    } catch (IOException e) {
        e.printStackTrace();
    }
    try (Reader is = new FileReader(filePath)) {
        char[] cbuf = new char[1024];
        int n = 0;
        while ((n = is.read(cbuf)) != -1) {
            System.out.println(new String(cbuf));
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

- finally 和 return 的执行顺序

> 有三种情况下会执行 finally 子句：  
> 代码没有抛出异常。  
> 抛出一个在 catch 子句中捕获的异常。  
> 代码抛出了一个异常，但不是由 catch 子句捕获的。
>
> 在以下4种特殊情况下，finally块不会被执行：  
> 在finally语句块第一行发生了异常。  
> 在前面的代码中用了System.exit(int)已退出程序。 exit是带参函数 ；若该语句在异常语句之后，finally会执行。  
> 程序所在的线程死亡。  
> 关闭CPU。
>
> 如果try语句里有return，返回的是try语句块中变量值。 详细执行过程如下：  
> 如果有返回值，就把返回值保存到局部变量中。  
> 执行jsr指令跳到finally语句里执行。  
> 执行完finally语句后，返回之前保存在局部变量表里的值。  
> 如果try，finally语句里均有return，忽略try的return，而使用finally的return。

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

### ThreadLocal

```java
public class ThreadLocal<T> {
    // set值
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }

    // get值
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }

    // 删除值
    public void remove() {
        ThreadLocalMap m = getMap(Thread.currentThread());
        if (m != null)
            m.remove(this);
    }

    static class ThreadLocalMap {
        static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
        ......
    }
}
```

1. 当在线程 t 的某个类的方法中 `ThreadLocal tl = new ThreadLocal()` 时，会产生引用关系：tl 引用 ThreadLocal 对象；
2. 接着向 ThreadLocal 对象中 set 值，通过源码可知，是向 ThreadLocalMap 中放入 key（ThreadLocal） 和 value；
3. ThreadLocalMap 的引用是线程 t 的 threadLocals 属性；
4. ThreadLocalMap 中的最小单位是 Entry，它继承了弱引用 WeakReference；
5. 生成 Entry 时，将 key 设置为了 WeakReference，并指向 ThreadLocal，此时 key 和 ThreadLocal 之间是弱引用关系。value 还是正常的引用关系。
6. 当 tl 对 ThreadLocal 的引用结束时，threadLocals 对 ThreadLocalMap 的引用仍然成立，key 对 ThreadLocal 的弱引用仍然成立，value 对具体对象的引用仍然成立；
7. 如果 key 是强引用，则 ThreadLocal 不会被 GC 回收，会导致内存泄漏；
8. 如果每次使用完 ThreadLocal 忘记 remove，虽然 ThreadLocal 被回收掉了，但 ThreadLocalMap 仍然存在，那么 value 引用的对象仍然存在，也会导致内存泄露。

![ThreadLocal](/images/java/ThreadLocal.png)

### ArrayList

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    private static final long serialVersionUID = 8683452581122892189L;

    // 默认初始容量为 10，后面会用到这个值
    private static final int DEFAULT_CAPACITY = 10;

    // 用于空实例的共享空数组实例
    private static final Object[] EMPTY_ELEMENTDATA = {};

    // 用于默认大小的空实例的共享空数组实例
    // 设计师将其与 EMPTY_ELEMENTDATA 区分开，它可以在第一个元素添加的时候知道增加多少容量
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    // 存储 ArrayList 元素的数组缓冲区
    // 任何空 ArrayList 被定义为 elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA，那么当第一个元素添加的时候，默认扩容 DEFAULT_CAPACITY
    transient Object[] elementData;

    // 数组的大小
    private int size;

    // 构造函数 1：构造具有指定初始容量的空列表
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

    // 构造函数 2：构造一个容量为 10 的空列表
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    // 构造函数 3：构造一个指定集合的元素的列表，按集合的迭代器返回元素的顺序排列。
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }

    // 应用程序可以调用此方法来最小化 ArrayList 实例的存储。
    public void trimToSize() {
        modCount++;
        if (size < elementData.length) {
            elementData = (size == 0)
              ? EMPTY_ELEMENTDATA
              : Arrays.copyOf(elementData, size);
        }
    }

    // 增加 ArrayList 实例的容量，如果必要的话，确保它至少可以容纳由 minCapacity 参数指定数量的元素。
    public void ensureCapacity(int minCapacity) {
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            // any size if not default element table
            ? 0
            // larger than default for default empty table. It's already
            // supposed to be at default size.
            : DEFAULT_CAPACITY;

        if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
        }
    }

    private static int calculateCapacity(Object[] elementData, int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }

    private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }

    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
}
```

### HashMap

- 类的属性

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {

    // 默认初始容量大小 16 —— 必须为 2 的幂次方数，其原因是这样可以提高散列度
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
    // 最大容量
    static final int MAXIMUM_CAPACITY = 1 << 30;
    // 默认负载因子，当填充度达到 3/4 时，进行扩容操作
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    // 阈值，当链表节点个数大于它时，转化为红黑树
    static final int TREEIFY_THRESHOLD = 8;
    // 阈值，当红黑树节点个数小于它时，转化为链表
    static final int UNTREEIFY_THRESHOLD = 6;
    // 阈值，链表结构转化为红黑树对应的 table 的最小容量
    static final int MIN_TREEIFY_CAPACITY = 64;

    // 存储元素的数组，长度总是 2 的幂
    transient Node<K,V>[] table;
    // 存放具体元素的集
    transient Set<Map.Entry<K,V>> entrySet;
    // 存放元素的个数，非数组长度
    transient int size;
    // 每次扩容和更改map结构的计数器
    transient int modCount;
    // 阈值,当实际大小(容量*负载因子)超过临界值时，会进行扩容
    int threshold;
    // 负载因子
    final float loadFactor;

    // 计算哈希值，先算出 key 的 hashCode，然后与之高 16 位进行异或计算，这是用来降低 hash 冲突的方法
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
}
```

- Node 节点

```java
// 继承自 Map.Entry<K,V>
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash; // hash 值，存放元素到 hashmap 中时用来与其他元素 hash 值进行比较
    final K key; // 键
    V value; // 值
    Node<K,V> next; // 指向下一个节点

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }

    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }

    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```

- Tree 节点

```java
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;
    TreeNode(int hash, K key, V val, Node<K,V> next) {
        super(hash, key, val, next);
    }

    // 返回包含此节点的树的根
    final TreeNode<K,V> root() {
        for (TreeNode<K,V> r = this, p;;) {
            if ((p = r.parent) == null)
                return r;
            r = p;
        }
    }
}
```

- 构造方法

```java
// 默认构造函数，容量为默认初始值 16，负载因子默认 0.75f
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}

// 指定初始容量大小，负载因子默认 0.75f的构造函数
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

// 指定初始容量大小和负载因子的构造函数
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                            initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                            loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}

static final int tableSizeFor(int cap) {
    int n = cap - 1;  // 为防止 cap = 2^n 时，最后得到的值是 2^(n+1)，从而首先减去 1 以保证得到的是原值。
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}

// 包含另一个 “Map” 的构造函数
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}
```

- putMapEntries

```java
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
    int s = m.size();
    if (s > 0) {
        if (table == null) { // pre-size
            float ft = ((float)s / loadFactor) + 1.0F;
            int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                        (int)ft : MAXIMUM_CAPACITY);
            if (t > threshold)
                threshold = tableSizeFor(t);
        }
        else if (s > threshold)
            resize();
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
            K key = e.getKey();
            V value = e.getValue();
            putVal(hash(key), key, value, false, evict);
        }
    }
}
```

- put

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&  // 判断此节点的 key 是否等于待插入节点的 key
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode) // 判断是否为树节点
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else { // 链表
            for (int binCount = 0; ; ++binCount) { // 遍历链表
                if ((e = p.next) == null) { // 此节点的下一节点为 null
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st // 判断是否需要树化
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash && // 判断此节点的 key 是否等于待插入节点的 key
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

- resize

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                    oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr; // 指定初始容量大小和负载因子的构造函数在此处会起到作用
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                    (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do { // 循环里所做的事情即是将链表拆分为两个，一个的位置一定是原位置，另外一个的位置加上 oldCap
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

- get

hash 函数降低冲突举例

| 原值     | 10010001 10010101 10110000 11110001 | 00010001 10010101 10110000 11110001 |
| -------- | ----------------------------------- | ----------------------------------- |
| 右移     | 00000000 00000000 10010001 10010101 | 00000000 00000000 00010001 10010101 |
| 异或后   | 10010001 10010101 00100001 01100100 | 00010001 10010101 10100001 01100100 |
| 数组大小 | 00000000 00000001 00000000 00000000 | 00000000 00000001 00000000 00000000 |
| n - 1    | 00000000 00000000 11111111 11111111 | 00000000 00000000 11111111 11111111 |
| 原值与   | 00000000 00000000 10110000 11110001 | 00000000 00000000 10110000 11110001 |
| 异或后与 | 00000000 00000000 00100001 01100100 | 00000000 00000000 10100001 01100100 |

很明显，两个原值不同，但是原值与却相同，所以用异或后进行与计算大大降低了 hash 冲突。

## Java 并发编程

### 并发与并行

### 什么是线程，与进程的区别

#### Thread 和 Runnable 两者的关系

Runnable 接口中只有一个方法 run()

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

Thread 类中实现了 Runnable 接口

```java
public class Thread implements Runnable {
    private Runnable target;

    // 重点在于 this.target = target
    public Thread(Runnable target) {
        init(null, target, "Thread-" + nextThreadNum(), 0);
    }

    /**
     * Causes this thread to begin execution;
     * the Java Virtual Machine calls the run method of this thread.
     */
    public void start()

    @Override
    public void run() {
        if (target != null) {
            target.run();
        }
    }
}
```

现在给定一个类 MyThread，使其实现 Runnable 接口并重写 run() 方法，那么它们之间的联系就如同下图一样:
![Thread & Runnable](/images/java/Thread-Runnable1.png)

根据上图及代码思考，实现多线程的过程如下：

1. 初始化一个 Thread 类，并传入一个参数为 Runnable 的对象（带参数的构造方法）；
2. 这个 Runnable 对象被 Thread 类中的 target 参数保存；
3. 调用 start() 方法，让线程开始执行，JVM 会调用该线程的 run() ；
4. 线程的中的 run() 方法实际上调用的是 Runnable 子类对象重写的 run() 方法。

多线程开发的本质实际上是**多个线程**可以进行对**同一资源**的抢占。Thread 主要描述多个线程，Runnable 描述资源的处理。

![Thread & Runnable](/images/java/Thread-Runnable2.png)

#### Thread、Runnable 和 Callable 三者的关系

Runnable 有一个缺点——没有返回值。java.util.concurrent.Callable 解决了这个问题。

```java
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```

实现 Callable 接口的时候可以设置一个泛型，该泛型即是返回数据类型，它可以避免向下转型所带来的隐患。但是目前有一个问题：多线程必须使用 Thread.start()，这意味着传入的参数必须是 Runnable 的实现类，而 Callable 接口并不是 Runnable 接口的子类！所以需要另外一个类——FutureTask——把这几个接口联系起来。

```java
public interface RunnableFuture<V> extends Runnable, Future<V> {
    void run();
}

public class FutureTask<V> implements RunnableFuture<V> {
    public FutureTask(Callable<V> callable) {
        if (callable == null)
            throw new NullPointerException();
        this.callable = callable;
        this.state = NEW;       // ensure visibility of callable
    }
}
```

很明显：FutureTask 接口实现了 RunnableFuture 接口，而 RunnableFuture 接口继承了 Runnable 和 Future，同时 FutureTask 的构造方法可以传入 Callable 对象作为参数。那么要用 Callable 来实现多线程只需要 `new Thread(new Future<V>(Callable<V>)).start()` 即可。它们之间的关系见下图：

![Thread & Runnable & Callable](/images/java/Thread-Runnable-Callable1.png)

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

位运算直接对二进制数进行操作：包括按位与、按位或、按位异或、取反、左移、右移、无符号右移以及补码。

- 补码

正整数的补码是其二进制表示，与原码相同；求负整数的补码，将其原码除符号位外的所有位取反（0变1，1变0，符号位为1不变）后加1。

```java
// +11 的原码
00001001
// +11 的补码
00001001

// -11 的原码
10001011
// -11 的补码
11110101
```

- 按位与（&）

```java
// 两个二进制数对应位都为 1，则取 1，否则取 0
将 10 与 -10 进行按位与(&)运算：
0000 0000 0000 1010
1111 1111 1111 0110
--------------------
0000 0000 0000 0010
```

- 按位或（|）

```java
// 两个二进制数对应位都为 0，则取 0，否则取 1
将 10 与 -10 进行按位或(|)运算：
0000 0000 0000 1010
1111 1111 1111 0110
--------------------
1111 1111 1111 1110
```

- 按位异或（^）

```java
// 两个二进制数对应位数字不同，则取 1，否则取 0
将 10 与 -10 进行按位异或(^)运算：
0000 0000 0000 1010
1111 1111 1111 0110
--------------------
1111 1111 1111 1100
```

- 取反（~）

```java
// 一个二进制数，每个位都取反值
对 10 进行取反(~)运算：
0000 0000 0000 1010
--------------------
1111 1111 1111 0101
```

- 左移（<<）

```java
// 一个二进制数，向左移动若干位；相当于乘以 2
对 10 左移 2 位(就相当于在右边加 2 个 0)：
0000 0000 0000 1010
--------------------
0000 0000 0010 1000
10 << 2 = 40
```

- 右移（>>）

```java
// 一个二进制数，向右移动若干位；相当于除以 2
对 10 右移 2 位(就相当于在左边加 2 个 0)：
0000 0000 0000 1010
--------------------
0000 0000 0000 0010
10 >> 2 = 2
```

- 无符号右移（>>>）

```java
对 -10 无符号右移 1 位
1111 1111 1111 0110
--------------------
0111 1111 1111 1011
--------------------
7ffb

// 陷阱题
byte b = (byte)(-8 >>> 2);
System.out.println(b);

-8 的原码：
1000 0000 0000 0000 0000 0000 0000 1000
-8 的补码：
1111 1111 1111 1111 1111 1111 1111 1000
>>> 2:
0011 1111 1111 1111 1111 1111 1111 1110
(byte):
1111 1110
转为原码：
1000 0010
结果等于 -2
```

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
