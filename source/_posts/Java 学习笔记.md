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

||+|concat|StringBuffer|StringBuilder|
|-|-|-|-|-|
|循环次数|十万|十万|百万|百万|
|耗时|5685|1190|49|22|

显然，`+` 的效率是最低的，StringBuilder 的效率是最高的。因为 StringBuffer 是线程安全的，同步会耗费一部分时间。

concat 的原理是，在拼接字符串的时候，根据原字符串和字符串参数的长度申请一个字符数组用以存储新的字符串，也就是说每拼接一次就需要申请新的字符数组，十万次的拼接中 concat 需要十万次的扩容。

StringBuilder 的原理是，在拼接字符串的时候对现有长度进行翻倍，十万次的拼接中只需要 log100000 < 17 次扩容。

- String.valueOf 和 Integer.toString 的区别

```java
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
