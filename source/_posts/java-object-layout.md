---
title: 对象的内存布局
date: 2021-10-03 15:59:09
tags:
- Java
- JVM
categories: Java
description:
---
#### 一、概述  
很多时候我们都有一个疑问：一个对象在内存中占用多大的空间呢？

```shell script
$ jdk8-64/java -jar jol-cli.jar internals java.lang.Object
# Running 64-bit HotSpot VM.
# Using compressed oop with 3-bit shift.
# Using compressed klass with 3-bit shift.

Instantiated the sample instance via default constructor.

java.lang.Object object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                  VALUE
      0     4        (object header)              05 00 00 00 # Mark word
      4     4        (object header)              00 00 00 00 # Mark word
      8     4        (object header)              00 10 00 00 # (not mark word)
     12     4        (loss due to the next object alignment)
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total
```

通过OpenJDK的Java-Object-Layout我们看到`java.lang.Object`的一个实例占用16 bytes。

同样的`java.lang.Boolean`类型占用的字节数：
```shell script
java.lang.Boolean object internals:
 OFFSET  SIZE      TYPE DESCRIPTION                               VALUE
      0    12           (object header)                           N/A
     12     1   boolean Boolean.value                             N/A
     13     3           (loss due to the next object alignment)
Instance size: 16 bytes
Space losses: 0 bytes internal + 3 bytes external = 3 bytes total
```

```shell script
[HEADER: 12 bytes]  12 
[value:   1 byte ]  13
[padding: 3 bytes]  16
```

其实一个对象通常由三块组成，分别是：对象头、实例数据和对齐填充。

#### 二、对象头（object header）  
在前面通过JOL打印输出的关于java.lang.Object信息中，我们看到object header占用12字节，但是输出并没有包含详细的结构信息，我们可以通过[Hotspot的源码](http://hg.openjdk.java.net/jdk/jdk/file/19afeaa0fdbe/src/hotspot/share/oops/oop.hpp#l52)了解到对象头包含两个部分：**mark word**和**class word**。

##### 2.1 [mark word](http://hg.openjdk.java.net/jdk/jdk/file/19afeaa0fdbe/src/hotspot/share/oops/markWord.hpp#l33)：
mark word在32位和64位机分别占32位和64位，当其中锁标志位的值不同时，它前面的bit存储不同的含义。
+ 存储对象的gc年龄信息
+ 存储Hashcode
+ 存储锁信息

<center>
    <img src="../images/mark-word-64bit.png" width="600"/>
</center>

##### 2.2 class word:
代码运行的时候，对象只是一串字节，我们可以通过class-word获取一些对象的元信息，它存储指向方法区中表示对象类型的指针，比如以下使用场景：
+ 运行时类型检查
+ 决定对象大小
+ 计算接口调用的目标类

##### 2.3 数组长度：
如果是数组类型，对象头会额外存储数组的长度信息。
+ 快速计算对象的大小
+ 数组边界检查


#### 三、实例数据和对齐填充
实例数据即我们在代码中声明的变量等信息，它的存储受到一些规则的约束以及虚拟机参数的控制。

##### 3.1 没有属性的类的内存布局

>规则一：每个对象都是八字节对齐。

从前面Object的输出中，我们看到，当一个对象只有头部信息时占用16byte，刚好是8的整数倍。

##### 3.2 Object子类的内存布局
跟在对象头后面的类属性按照它们的大小在内存中排序，例如：int是4字节、long是8字节。采用字节对齐可以提高性能，因为从内存读取4字节到4字节的寄存器性能更好。

为了节约内存，Sun的虚拟机在分配对象字段的时候和它们声明的顺序不同，有如下顺序规则：

1、double和long类型
2、int和float
3、short和char
4、boolean和byte
5、reference

为什么可以优化内存呢？我们看一下这个例子：
```java
class MyClass {
    byte a;
    int c;
    boolean d;
    long e;
    Object f;        
}
```

它的对象布局如下：
```shell script
[HEADER:  8 bytes]  8
[a:       1 byte ]  9
[padding: 3 bytes] 12
[c:       4 bytes] 16
[d:       1 byte ] 17
[padding: 7 bytes] 24
[e:       8 bytes] 32
[f:       4 bytes] 36
[padding: 4 bytes] 40
```
总共使用了40字节内存，其中14个用于内存对齐而浪费掉。如果重排顺序则：

```shell script
[HEADER:  8 bytes]  8
[e:       8 bytes] 16
[c:       4 bytes] 20
[a:       1 byte ] 21
[d:       1 byte ] 22
[padding: 2 bytes] 24
[f:       4 bytes] 28
[padding: 4 bytes] 32
```

经过优化后只有6个字节用于对齐填充，总内存也只有32byte。

##### 3.3 子类的内存布局

>规则三：继承结构中属于不同类的字段不混合在一起。父类优先，子类其次。

```java
class A {
   long a;
   int b;
   int c;
}

class B extends A {
   long d;
}
```
B类的布局如下：
```shell script
[HEADER:  8 bytes]  8
[a:       8 bytes] 16
[b:       4 bytes] 20
[c:       4 bytes] 24
[d:       8 bytes] 32
```

如果父类的字段不符合4字节对齐。

>规则四：父类的最后一个字段和子类第一个字段之间必须4字节对齐。

```java
class A {
   byte a;
}

class B {
   byte b;
}
```

```shell script
[HEADER:  8 bytes]  8
[a:       1 byte ]  9
[padding: 3 bytes] 12
[b:       1 byte ] 13
[padding: 3 bytes] 16
```
a后面的3个字节就是为了使其4字节对齐。这3个字节只能浪费不能给B使用。

最后一个规则可以用于节约一些内存空间：当子类的第一个属性是long或者double类型且父类没有以8字节边界结束。

>规则五：子类的第一个字段是doubel或long且父类没有以8字节边界结束，JVM打破2的规则，先存储int、short、byte、reference来填补空缺。

例如：
```java
class A {
  byte a;
}

class B {
  long b;
  short c;  
  byte d;
}
```

内存布局如下：
```shell script
[HEADER:  8 bytes]  8
[a:       1 byte ]  9
[padding: 3 bytes] 12
[c:       2 bytes] 14
[d:       1 byte ] 15
[padding: 1 byte ] 16
[b:       8 bytes] 24
```

在字节12的位置，A类结束了。JVM打破2的规则放入short和byte类型，节约了4字节中的3个字节，否则将浪费掉。


##### 3.4 数组的内存布局
数组类型有一个额外的头部字段保存数组的长度，接下来是数组的元素，数组作为普通对象也是8字节对齐的。

这是byte[3]的布局：
```shell script
[HEADER:  12 bytes] 12
[[0]:      1 byte ] 13
[[1]:      1 byte ] 14
[[2]:      1 byte ] 15
[padding:  1 byte ] 16
```

这是long[3]的布局:
```shell script
[HEADER:  12 bytes] 12
[padding:  4 bytes] 16
[[0]:      8 bytes] 24
[[1]:      8 bytes] 32
[[2]:      8 bytes] 40
```

##### 3.5 内部类的内存布局
非静态内部类有一个额外的隐藏字段，持有外部类的引用。这个字段是一个常规的引用,它符合对象在内存布局的规则。内部类因此有4字节额外的开销。