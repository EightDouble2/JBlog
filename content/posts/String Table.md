---
title: "String Table"
date: 2020-09-13T20:06:28+08:00
draft: false
tags: [ "JVM" ]
categories: [ "技术文档" ]
---
# String Table

## String 的基本特性

- String ：字符串，使用一对 "" 引起来表示。

  - String s1 = "atgigu"；//字面量的定义方式
  - String s2 = new String("hello");

- String 声明为 final，不可被继承

- String 实现了 Serializable 接口：表示字符串是支持序列化的。实现了 Comparable 接口：表示 String 可以比较大小

- String ：代表不可变的字符序列。简称：

  不可变性

  - 当对字符串重新赋值时，需要重写指定内存区域赋值，不能使用原有的 value 进行赋值。当对现有的字符串进行连接操作时，也需要重新指定内存区域赋值，不能使用原有的 value 进行赋值
  - 当调用 String 的 `replace()` 方法修改指定字符或字符串时，也需要重新指定内存区域赋值，不能使用原有的 value 进行赋值。

- 通过字面量的方式（区别于 new ）给一个字符串赋值，此时的字符串值声明在字符串常量池中

- **字符串常量池中是不会存储相同内容的字符串的**

- String 的 String Pool 是一个固定大小的 Hashtable ，默认值大小长度是 1009 。如果放进 String pool 的 String 非常多，就会造成 Hash 冲突严重，从而导致链表会很长，而链表长了后直接会造成的影响就是当调用`String.intern` 时性能会大幅下降。

- 使用 `-XX:StringTableSize` 可设置 StringTable 的长度

- 在 JDK 6 中 StringTable 是固定的，就是 1009 的长度，所以如果常量池中的字符串过多就会导致效率下降很快。 StringTableSize 设置没有要求

- 在 JDK 7 中， StringTable 的长度默认值是 60013 。 StringTableSize 设置没有要求

- 从 JDK 8 开始，1009 是可设置的最小值。

### String存储结构变更

- **String 在 jdk8 及以前内部定义了 `final char[] value` 用于存储字符串数据。 jdk9 时改为 `byte[]`**
  - 改变的原因：char 占用两个字节，而 byte 占用一个字节，认为大多数 String 对象只包含 Latin-1 字符，因此只需要一个字节的空间，两个字节会造成浪费
- String 不再用 `char[]` 来存储，**改成了 byte[] 加上编码标记**，节约了一些空间。
- 基于 String 的类，例如 StringBuilder，StringBuffer 都进行了相应的更新

## String 的内存分配

- 在 Java 语言中有 8 种基本数据类型和一种比较特殊的类型 String 。这些类型为了使它们在运行过程中速度更快、更节省内存，都提供了一种**常量池**的概念。
- 常量池就类似一个 Java 系统级别提供的缓存。 8 种基本数据类型的常量池都是系统协调的， **String 类型的常量池比较特殊。它的主要使用方法有两种**。
  - 直接使用双引号声明出来的 String 对象会直接存储在常量池中。比如： `String info ="atgulgu";`。
  - 如果不是用双引号声明的 String 对象，可以使用 String 提供的 `intern()` 方法。这个后面重点谈。
- **Java 6及以前，字符串常量池存放在永久代**。
- **Java 7 中 Oracle 的工程师对字符串池的逻辑做了很大的改变，即将字符串常量池的位置调整到 Java 堆内**。
  - 所有的字符串都保存在堆（ Heap ）中，和其他普通对象一样，这样可以让你在进行调优应用时仅需要调整堆大小就可以了。
  - 字符串常量池概念原本使用得比较多，但是这个改动使得我们有足够的理由让我们重新考虑在 Java 7 中使用 `String.intern()`。
- **Java 8 元空间替代永久代，字符串常量池仍然在堆**。
- StringTable 调整的原因：永久代默认空间比较小；永久代垃圾回收频率低。

## String 的基本操作

Java 语言规范里要求完全相同的字符串字面量，应该包含同样的 Unicode 字符序列（包含同一份码点序列的常量），并且必须是指向同一个 String 类实例。

## 字符串拼接操作

- **常量与常量的拼接结果在常量池，原理是编译期优化**
- 常量池中不会存在相同内容的常量。
- 只要其中有一个是变量，结果就在堆中。变量拼接的原理是 StringBuilder。
   - StringBuilder 是 JDK 5 之后出现的类，之前使用的是 StringBuffer。
   - 字符串接作不一定使用的是 StringBuilder ，如果拼接符号左右两边部是字符串常量或常量引用，则仍然使用编译器优化，即 StringBuilder 的方式。
   - 针对于 final 修饰类、方法、基本数据类型、引用数据类型的结构时，能使用上 final 的时候建议使用上。
- 如果拼接的结果调用 `intern()` 方法，则主动将常量池中还没有的字符串对象放入池中，并返回此对象地址。

带变量的字符串拼接的 `s1 + s2` 的执行细节：
- `StringBuilder s = new StringBuilder();`
- `s.append("a");`
- `s.append("b");`
- `s.toString()` 约等于 `new String("ab");`

补充：在jdk5.0之后使用的是StringBuilder,在jdk5.0之前使用的是StringBuffer。

体会执行效率：通过StringBuilder的append()的方式添加字符串的效率要远高于使用String的字符串拼接方式！
- StringBuilder的append()的方式：自始至终中只创建过一个StringBuilder的对象。
- 使用String的字符串拼接方式：创建过多个StringBuilder和String的对象。
- 使用String的字符串拼接方式：内存中由于创建了较多的StringBuilder和String的对象，内存占用更大；如果进行GC，需要花费额外的时间。

改进的空间：在实际开发中，如果基本确定要前前后后添加的字符串长度不高于某个限定值highLevel的情况下,建议使用构造器实例化：`StringBuilder s = new StringBuilder(highLevel);//new char[highLevel]`。

## `intern()` 的使用

如果不是用双引号声明的 String 对象，可以使用 String 提供的 intern 方法：intern 方法会从字符串常量池中査询当前字符串是否存在，若不存在就会将当前字符串放入常量池中。

比如： `String myInfo = new String(" I love atguigu ").intern();`。

也就是说，如果在任意字符串上调用 `String.intern` 方法，那么其返回结果所指向的那个类实例，必须和直接以常量形式出现的字符串实例完全相同。因此，下列表达式的值必定是 true ：`(a + b + c).intern() == "abc "`。

通俗点讲， Interned String 就是确保字符串在内存里只有一份拷贝，这样可以节约内存空间，加快字符串操作任务的执行速度。注意，这个值会被存放在字符串内部池（String Intern Pool ）。

如何保证变量s指向的是字符串常量池中的数据呢？有两种方式：
- 方式一： `String s = "shkstart";//字面量定义的方式`
- 方式二： 调用 `intern()`
  - `String s = new String("shkstart").intern();`
  - `String s = new StringBuilder("shkstart").toString().intern();`

### 经典面试题

#### new String("ab") 会创建几个对象？

通过看字节码，知道是两个

- 一个对象是：new 关键字在堆空同创建的
- 另ー个对象是：字符常量池中的对象。字节码指令：`ldc`

#### new String("a") + new string("b") 会创建几个对象？

- 对象 1：`new StringBuilder()`
- 对象 2：`new String("a")`
- 对象 3：常量池中的 "a"
- 对象 4：`new String("b")`
- 对象 5：常量池中的 "b"
- 深入副析: `StringBuilder.toString()`：
  - 对象 6：`new String("ab")`
    - 强调一下，`toString()` 的调用，在字符串常量池中，没有生成 "ab"

### `intern()` 的使用：jdk6 vs jdk7/8

```java
public class StringIntern {
    public static void main(String[] args) {

        String s = new String("1");
        s.intern();//调用此方法之前，字符串常量池中已经存在了"1"
        String s2 = "1";
        System.out.println(s == s2);//jdk6：false   jdk7/8：false


        String s3 = new String("1") + new String("1");//s3变量记录的地址为：new String("11")
        //执行完上一行代码以后，字符串常量池中，是否存在"11"呢？答案：不存在！！
        String sIntern = s3.intern();//在字符串常量池中生成"11"。如何理解：jdk6:创建了一个新的对象"11",也就有新的地址。
        //         jdk7:此时常量中并没有创建"11",而是创建一个指向堆空间中new String("11")的地址
        System.out.println(s3 == sIntern); // true
        String s4 = "11";//s4变量记录的地址：使用的是上一行代码代码执行时，在常量池中生成的"11"的地址
        System.out.println(s3 == s4);//jdk6：false  jdk7/8：true
    }
}
```

#### 拓展

```java
public class StringIntern1 {
    public static void main(String[] args) {
        String s3 = new String("1") + new String("1");//new String("11")
        //执行完上一行代码以后，字符串常量池中，是否存在"11"呢？答案：不存在！！
        String s4 = "11";//在字符串常量池中生成对象"11"
        String s5 = s3.intern();
        System.out.println(s3 == s4);//false
        System.out.println(s5 == s4);//true
    }
}
```

#### 总结 String 的 `intern()` 的使用

- JDK 1.6 中，将这个字符串对象尝试放入串池。
  - 如果串池中有，则并不会放入。返回已有的串池中的对象的地址
  - 如果没有，会把**此对象复制一份**，放入串池，并返回串池中的对象地址
- JDK 1.7 起，将这个字符串对象尝试放入串池。
  - 如果串池中有，则并不会放入。返回己有的串池中的对象的地址
  - 如果没有，则会把**对象的引用地址复制一份**，放入串池，并返回串池中的引用地址

### `intern()` 的效率测试

大的网站平台，需要内存中存储大量的字符串。比如社交网站，很多都存储：北京市、海淀区等信息。这时候如果字符串都调用 `intern()` 方法，就会明显降低内存的大小。

## StringTable 的垃圾回收

```
-XX:+PrintStringTableStatistics
```

## G1 中的 String 去重操作

- 背景：对许多 Java 应用（有大的也有小的）做的测试得出以下结果
  - 堆存活数据集合里面 String 对象占了 25%
  - 堆存活数据集合里面重复的 String 对象有 13.5%
  - String 对象的平均长度是 45
- 许多大规模的 Java 应用的瓶颈在于内存，测试表明，在这些类型的应用里面， **Java 堆中存活的数据集合差不多 25% 是 String 对象**。更进一步这里面差不多一半 String 对象是重复的，重复的意思是说 `string1.equals(string2) = true` 。**堆上存在重复的 String 对象必然是一种内存的浪费**。这个项目将在 G1 垃圾收集器中实现自动持续对重复的 String 对象进行去重，这样就能避免浪费内存。
- 实现
  - 当垃圾收集器工作的时候，会访问堆上存活的对象。对每一个访问的对象都会检查是否是候选的要去重的 String 对象。
  - 如果是，把这个对象的一个引用插入到队列中等待后续的处理。一个去重的线程在后台运行，处理这个队列。处理队列的一个元素意味着从队列删除这个元素，然后尝试去重它引用的 String 对象。
  - 使用一个 hashtable 来记录所有的被 String 对象使用的不重复的 char 数组。当去重的时候，会查这个 hashtable ，来看堆上是否已经存在一个一模一样的 char 数组。
  - 如果存在， String 对象会被调整引用那个数组，释放对原来的数组的引用，最终会被垃圾收集器回收掉。
  - 如果查找失败， char 数组会被插入到 hashtable ，这样以后的时候就可以共享这个数组了。
- 命令行选项
  - UseStringDeduplication(bool) ：开启 String 去重，**默认是不开启的，需要手动开启。**
  - PrintStringDeduplicationStatistics(bool) ：打印详细的去重统计信息
  - StringDeduplicationAgeThreshold(uintx) ：达到这个年龄的 String 对象被认为是去重的候选对象