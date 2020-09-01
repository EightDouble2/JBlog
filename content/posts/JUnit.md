---
title: "JUnit"
date: 2020-04-25T21:03:50+08:00
draft: false
tags: [ "Java" ]
categories: [ "技术文档" ]
---
# JUnit 简介

JUnit 是用于编写和运行可重复的自动化测试的开源测试框架，这样可以保证我们的代码按预期工作。JUnit 可广泛用于工业和作为支架(从命令行)或IDE(如 IDEA)内单独的 Java 程序。

JUnit 提供：

- 断言测试预期结果。
- 测试功能共享通用的测试数据。
- 测试套件轻松地组织和运行测试。
- 图形和文本测试运行。

JUnit 用于测试：

- 整个对象
- 对象的一部分 - 交互的方法或一些方法
- 几个对象之间的互动(交互)

### JUnit 特点

- JUnit 是用于编写和运行测试的开源框架。
- 提供了注释，以确定测试方法。
- 提供断言测试预期结果。
- 提供了测试运行的运行测试。
- JUnit 测试让您可以更快地编写代码，提高质量
- JUnit 是优雅简洁。它是不那么复杂以及不需要花费太多的时间。
- JUnit 测试可以自动运行，检查自己的结果，并提供即时反馈。没有必要通过测试结果报告来手动梳理。
- JUnit 测试可以组织成测试套件包含测试案例，甚至其他测试套件。
- Junit 显示测试进度的，如果测试是没有问题条形是绿色的，测试失败则会变成红色。

# 第一个 JUnit 单元测试

## POM

`pom.xml` 文件如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.johnny</groupId>
    <artifactId>hello-spring</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>4.3.17.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

主要增加了 `junit:junit` 依赖

## 创建测试类

在测试包下 `src/main/test` 创建一个名为 `MyTest` 的测试类，代码如下：

```java
package com.johnny.hello.spring.test;

import org.junit.After;
import org.junit.Before;
import org.junit.Test;

public class MyTest {

    /**
     * 执行测试方法前执行
     */
    @Before
    public void before() {
        System.out.println("执行 before() 方法");
    }

    /**
     * 执行测试方法后执行
     */
    @After
    public void after() {
        System.out.println("执行 after() 方法");
    }

    @Test
    public void testSayHi() {
        System.out.println("Hi Log4j");
    }

    @Test
    public void testSayHello() {
        System.out.println("Hello Log4j");
    }
}
```

# JUnit 注解

| 注解                                       | 描述                                                         |
| ------------------------------------------ | ------------------------------------------------------------ |
| `@Test` public void method()               | 测试注释指示该公共无效方法它所附着可以作为一个测试用例。     |
| `@Before` public void method()             | Before 注释表示，该方法必须在类中的每个测试之前执行，以便执行测试某些必要的先决条件。 |
| `@BeforeClass` public static void method() | BeforeClass 注释指出这是附着在静态方法必须执行一次并在类的所有测试之前。发生这种情况时一般是测试计算共享配置方法(如连接到数据库)。 |
| `@After` public void method()              | After 注释指示，该方法在执行每项测试后执行(如执行每一个测试后重置某些变量，删除临时变量等) |
| `@AfterClass` public static void method()  | 当需要执行所有的测试在 JUnit 测试用例类后执行，AfterClass 注解可以使用以清理建立方法，(从数据库如断开连接)。注意：附有此批注(类似于 BeforeClass)的方法必须定义为静态。 |
| `@Ignore` public static void method()      | 当想暂时禁用特定的测试执行可以使用忽略注释。每个被注解为 @Ignore 的方法将不被执行。 |

# JUnit 断言

## 什么是断言

> 断言是编程术语，表示为一些布尔表达式，程序员相信在程序中的某个特定点该表达式值为真，可以在任何时候启用和禁用断言验证，因此可以在测试时启用断言而在部署时禁用断言。同样，程序投入运行后，最终用户在遇到问题时可以重新启用断言。

使用断言可以创建更稳定、品质更好且 不易于出错的代码。当需要在一个值为 `false` 时中断当前操作的话，可以使用断言。单元测试必须使用断言（Junit/JunitX）。

## 常用断言方法

| 断言                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| void assertEquals([String message], expected value, actual value) | 断言两个值相等。值可能是类型有 int, short, long, byte, char or java.lang.Object. 第一个参数是一个可选的字符串消息 |
| void assertTrue([String message], boolean condition)         | 断言一个条件为真                                             |
| void assertFalse([String message],boolean condition)         | 断言一个条件为假                                             |
| void assertNotNull([String message], java.lang.Object object) | 断言一个对象不为空(null)                                     |
| void assertNull([String message], java.lang.Object object)   | 断言一个对象为空(null)                                       |
| void assertSame([String message], java.lang.Object expected, java.lang.Object actual) | 断言，两个对象引用相同的对象                                 |
| void assertNotSame([String message], java.lang.Object unexpected, java.lang.Object actual) | 断言，两个对象不是引用同一个对象                             |
| void assertArrayEquals([String message], expectedArray, resultArray) | 断言预期数组和结果数组相等。数组的类型可能是 int, long, short, char, byte or java.lang.Object. |

## 测试断言效果

在之前的单元测试类中创建一个名为 `testAssert` 方法来查看断言效果

```java
/**
 * 测试断言
 */
@Test
public void testAssert() {
    String obj1 = "junit";
    String obj2 = "junit";
    String obj3 = "test";
    String obj4 = "test";
    String obj5 = null;
    int var1 = 1;
    int var2 = 2;
    int[] arithmetic1 = {1, 2, 3};
    int[] arithmetic2 = {1, 2, 3};

    assertEquals(obj1, obj2);

    assertSame(obj3, obj4);

    assertNotSame(obj2, obj4);

    assertNotNull(obj1);

    assertNull(obj5);

    assertTrue("为真", var1 == var2);

    assertArrayEquals(arithmetic1, arithmetic2);
}
```