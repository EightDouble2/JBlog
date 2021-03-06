---
title: "Maven"
date: 2020-04-24T17:22:06+08:00
draft: false
tags: [ "Java" ]
categories: [ "技术文档" ]
---
# Maven 简介

Maven 是一个**项目管理和综合工具**。Maven 提供了开发人员构建一个完整的生命周期框架。开发团队可以自动完成项目的基础工具建设，Maven 使用**标准的目录结构和默认构建生命周期**（使项目和开发环境无关）。

在多个开发团队环境时，Maven 可以设置按标准在非常短的时间里完成配置工作。由于大部分项目的设置都很简单，并且可重复使用，Maven 让开发人员的工作更轻松，同时创建报表，检查，构建和测试自动化设置。

Maven 提供了开发人员的方式来管理：

- 构建：Builds
- 文档管理：Documentation
- 报告：Reporting
- 依赖：Dependencies
- 管理：SCMs 
- 发行：Releases
- 分布式：Distribution 
- 邮件列表：Mailing List

概括地说，Maven **简化和标准化项目建设过程**（编译、打包、发布、部署、测试、生成报告）。处理编译，分配，文档，团队协作和其他任务的无缝连接。 Maven 增加可重用性并负责建立相关的任务。

# Maven 安装配置

想要安装 Apache Maven 在 Windows 系统上, 需要下载 Maven 的 zip 文件，并将其解压到你想安装的目录，并配置 Windows 环境变量。

注意：请尽量使用 JDK 1.8 及以上版本

## JDK 和 JAVA_HOME

确保已安装 JDK，并设置 `JAVA_HOME` 环境变量到 Windows 环境变量。

```properties
JAVA_HOME=D:\Java\jdk1.7;
JRE_HOME=D:\Java\jre7;
Path=%JAVA_HOME%\bin;%JRE_HOME%\bin;
CLASSPATH=.;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar;
```

## 下载 Apache Maven

下载地址：[http://maven.apache.org/download.cgi](http://maven.apache.org/download.cgi)

下载 Maven 的 zip 文件，例如：`apache-maven-3.6.3-bin.zip`，将它解压到你要安装 Maven 的文件夹。假设你解压缩到文件夹 `D:\apache-maven-3.6.3`

注意：在这一步，只是文件夹和文件，安装不是必需的。

## 添加 MAVEN 环境变量

添加 `MAVEN_HOME` 环境变量到 Windows 环境变量，并将其指向你的 Maven 文件夹。

```properties
MAVEN_HOME=D:\apache-maven-3.6.3；
Path=%MAVEN_HOME%\bin;
```

## 验证

使用命令：`mvn -version`

# Maven 本地仓库

Maven 的本地资源库是用来存储所有项目的依赖关系(插件 Jar 和其他文件，这些文件被 Maven 下载)到本地文件夹。很简单，当你建立一个 Maven 项目，所有相关文件将被存储在你的 Maven 本地仓库。

默认情况下，Maven 的本地资源库默认为 `.m2` 目录文件夹：

- Unix/Mac OS X：`~/.m2`
- Windows：`C:\Documents and Settings\{your-username}\.m2`

通常情况下，可改变默认的 `.m2` 目录下的默认本地存储库文件夹到其他更有意义的名称，例如， maven-repo 找到 `{M2_HOME}\conf\setting.xml`, 更新 `localRepository` 到其它名称。

```xml
<localRepository>D:\apache-maven-3.6.3\repo</localRepository>
```

执行之后，新的 Maven 本地存储库现在改为 `D:/apache-maven-3.6.3/repo`

国内使用阿里云镜像：

```xml
<mirrors>
    <mirror>
        <id>alimaven</id>
        <name>aliyun maven</name>
        <mirrorOf>central</mirrorOf>       
        <url>http://maven.aliyun.com/nexus/content/groups/public/</url> 
    </mirror>
</mirrors>
```

# Maven 中央仓库

当你建立一个 Maven 的项目，Maven 会检查你的 `pom.xml` 文件，以确定哪些依赖下载。首先，Maven 将从本地资源库获得 Maven 的本地资源库依赖资源，如果没有找到，然后把它会从默认的 Maven 中央存储库 [http://repo1.maven.org/maven2/](http://repo1.maven.org/maven2/) 查找下载。

使用 MVNrepository 搜索：[https://mvnrepository.com/](https://mvnrepository.com/)

# Maven 依赖机制

在 Maven 依赖机制的帮助下**自动下载所有必需的依赖库，并保持版本升级**。让我们看一个案例研究，以了解它是如何工作的。假设你想使用 Log4j 作为项目的日志。这里你要做什么？

## 传统方式

- 访问 [http://logging.apache.org/log4j/](http://logging.apache.org/log4j/)
- 下载 Log4j 的 jar 库
- 复制 jar 到项目类路径
- 手动将其包含到项目的依赖
- 所有的管理需要一切由自己做

如果有 Log4j 版本升级，则需要重复上述步骤一次。

## Maven 的方式

- 你需要知道 log4j 的 Maven 坐标，例如：

```xml
<groupId>log4j</groupId>
<artifactId>log4j</artifactId>
<version>1.2.17</version>
```

- 它会自动下载 log4j 的 1.2.17 版本库
- 声明 Maven 的坐标转换成 `pom.xml` 文件

```xml
<dependencies>
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>
</dependencies>
```

- 当 Maven 编译或构建，log4j 的 jar 会自动下载，并把它放到 Maven 本地存储库
- 所有由 Maven 管理

## 解释说明

看看有什么不同？那么到底在 Maven 发生了什么？当建立一个 Maven 的项目，pom.xml 文件将被解析，如果看到 log4j 的 Maven 坐标，然后 Maven 按此顺序搜索 log4j 库：

- 在 Maven 的本地仓库搜索 log4j
- 在 Maven 中央存储库搜索 log4j
- 在 Maven 远程仓库搜索 log4j(如果在 pom.xml 中定义)

Maven 依赖库管理是一个非常好的工具，为您节省了大量的工作

# Maven POM

POM 代表项目对象模型。它是 Maven 中工作的基本单位，这是一个 XML 文件。它始终保存在该项目基本目录中的 pom.xml 文件。

POM 包含的项目是使用 Maven 来构建的，它用来包含各种配置信息。

POM 也包含了目标和插件。在执行任务或目标时，Maven 会使用当前目录中的 POM。它读取POM得到所需要的配置信息，然后执行目标。部分的配置可以在 POM 使用如下：

- project dependencies
- plugins
- goals
- build profiles
- project version
- developers
- mailing list

创建一个POM之前，应该要先决定项目组(groupId)，它的名字(artifactId)和版本，因为这些属性在项目仓库是唯一标识的。

## POM 的例子

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
   http://maven.apache.org/xsd/maven-4.0.0.xsd">
   <modelVersion>4.0.0</modelVersion>
   <groupId>com.johnny</groupId>
   <artifactId>project</artifactId>
   <version>1.0</version>
<project>
```

要注意的是，每个项目只有一个 POM 文件

- 所有的 POM 文件要项目元素必须有三个必填字段: groupId，artifactId，version
- 在库中的项目符号是：`groupId:artifactId:version`
- `pom.xml` 的根元素是 project，它有三个主要的子节点。

| 节点       | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| groupId    | 这是项目组的编号，这在组织或项目中通常是独一无二的。 例如，一家银行集团 `com.company.bank` 拥有所有银行相关项目。 |
| artifactId | 这是项目的 ID。这通常是项目的名称。 例如，`consumer-banking`。 除了 groupId 之外，artifactId 还定义了 artifact 在存储库中的位置。 |
| version    | 这是项目的版本。与 groupId 一起使用，artifact 在存储库中用于将版本彼此分离。 例如：`com.company.bank:consumer-banking:1.0`，`com.company.bank:consumer-banking:1.1` |

# Maven 插件

Maven 是一个执行插件的框架，每一个任务实际上是由插件完成的。Maven 插件通常用于：

- 创建 `jar` 文件
- 创建 `war` 文件
- 编译代码文件
- 进行代码单元测试
- 创建项目文档
- 创建项目报告 一个插件通常提供了一组目标，可使用以下语法来执行：

`mvn [plugin-name]:[goal-name]`

例如，一个 Java 项目可以使用 Maven 编译器插件来编译目标，通过运行以下命令编译

`mvn compiler:compile`

## 插件类型

Maven 提供以下两种类型插件：

| 类型     | 描述                                               |
| -------- | -------------------------------------------------- |
| 构建插件 | 在生成过程中执行，并在 `pom.xml` 中的 元素进行配置 |
| 报告插件 | 在网站生成期间执行，在 `pom.xml` 中的,元素进行配置 |

以下是一些常见的插件列表：

| 插件     | 描述                                  |
| -------- | ------------------------------------- |
| clean    | 编译后的清理目标，删除目标目录        |
| compiler | 编译 Java 源文件                      |
| surefile | 运行JUnit单元测试，创建测试报告       |
| jar      | 从当前项目构建 JAR 文件               |
| war      | 从当前项目构建 WAR 文件               |
| javadoc  | 产生用于该项目的 Javadoc              |
| antrun   | 从构建所述的任何阶段运行一组 Ant 任务 |

# Maven 快照

大型应用软件一般由多个模块组成，一般它是多个团队开发同一个应用程序的不同模块，这是比较常见的场景。例如，一个团队正在对应用程序的应用程序，用户界面项目(`app-ui.jar:1.0`) 的前端进行开发，他们使用的是数据服务工程 (`data-service.jar:1.0`)。

现在，它可能会有这样的情况发生，工作在数据服务团队开发人员快速地开发 bug 修复或增强功能，他们几乎每隔一天就要释放出库到远程仓库。

现在，如果数据服务团队上传新版本后，会出现下面的问题：

- 数据服务团队应该发布更新时每次都告诉应用程序 UI 团队，他们已经发布更新了代码。
- UI 团队需要经常更新自己 `pom.xml` 以获得更新应用程序的版本。

为了处理这类情况，引入快照的概念，并发挥作用。

## 什么是快照？

快照（SNAPSHOT）是一个特殊版本，指出目前开发拷贝。不同于常规版本，Maven 每生成一个远程存储库都会检查新的快照版本。

现在，数据服务团队将在每次发布代码后更新快照存储库为：`data-service:1.0-SNAPSHOT` 替换旧的 SNAPSHOT jar。

## 快照与版本

在使用版本时，如果 Maven 下载所提到的版本为 `data-service:1.0`，那么它永远不会尝试在库中下载已经更新的版本 1.0。要下载更新的代码，data-service 的版本必须要升级到 1.1。

在使用快照（SNAPSHOT）时，Maven 会在每次应用程序 UI 团队建立自己的项目时自动获取最新的快照（`data-service:1.0-SNAPSHOT`）。

# Maven 常用命令

本章节只提供 Maven 使用时的一些基本命令

## 清除产生的项目

`mvn clean`

## 编译源代码

`mvn compile`

## 打包

`mvn package`

## 只打包不测试（跳过测试）

`mvn package`

## 安装到本地仓库

`mvn install`

## 源码打包

`mvn source:jar`
或
`mvn source:jar-no-fork`

# 第一个 Maven 应用程序

下面我们来学习如何使用 Maven 创建一个 Java Web 应用程序

## 创建 Maven 项目

选择 `File` -> `New` -> `Project...`

选择 `Maven` 项目

填写项目信息

```yaml
GroupId: com.johnny
Artifactld: hello-maven
Version: 1.0.0-SNAPSHOT
```

选择工作空间

```yaml
Project name: hello-maven
Project location: D:\IDEAProjects\hello-maven
```

## 目录结构

Java Web 的 Maven 基本结构如下：

```text
├─src
│  ├─main
│  │  ├─java
│  │  ├─resources
│  │  └─webapp
│  │      └─WEB-INF
│  └─test
│      └─java
```

结构说明：

- `src`

  ：源码目录

  - `src/main/java`：Java 源码目录
  - `src/main/resources`：资源文件目录
  - `src/main/webapp`：Web 相关目录
  - `src/test`：单元测试

## IDEA Maven 项目管理

在 IDEA 界面的右侧 `Maven Projects` 选项，可以管理 Maven 项目的整个生命周期、插件、依赖等

## 完善 Java Web 程序

### POM

修改 `pom.xml` 配置，内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.johnny</groupId>
    <artifactId>hello-maven</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>war</packaging>

    <dependencies>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>4.0.1</version>
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>log4j-over-slf4j</artifactId>
            <version>1.7.25</version>
        </dependency>
    </dependencies>
</project>
```

配置说明：

- `packaging`：打包方式，这里是 `war` 包，表示为 Java Web 应用程序
- `dependencies`：项目依赖配置，整个项目生命周期中所需的依赖都在这里配置

### 创建测试用 Servlet

创建一个 `Servlet` 用于测试请求

```java
package com.johnny.hello.maven.servlet;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class HelloServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        req.getRequestDispatcher("/index.jsp").forward(req, resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doPost(req, resp);
    }
}
```

### 创建测试用 JSP

创建一个 `JSP` 页面，用于测试请求

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
    <head>
        <title>Title</title>
    </head>
    <body>
        Hello Maven
    </body>
</html>
```

### 创建 Log4J 的配置文件

在 `src/main/resources` 目录下创建 `log4j.properties` 配置文件，内容如下：

```properties
log4j.rootLogger=INFO, console, file

log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=%d %p [%c] - %m%n

log4j.appender.file=org.apache.log4j.DailyRollingFileAppender
log4j.appender.file.File=logs/log.log
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.A3.MaxFileSize=1024KB
log4j.appender.A3.MaxBackupIndex=10
log4j.appender.file.layout.ConversionPattern=%d %p [%c] - %m%n
```

### 配置 `web.xml`

`web.xml` 配置文件如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <servlet>
        <servlet-name>HelloServlet</servlet-name>
        <servlet-class>com.johnny.hello.maven.servlet.HelloServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>HelloServlet</servlet-name>
        <url-pattern>/servlet/hello</url-pattern>
    </servlet-mapping>
</web-app>
```

### 测试运行

按照之前章节 `第一个 IDEA 应用程序` 配置完 `Tomcat` 后直接运行，打开浏览器访问 [http://localhost:8080](http://localhost:8080) 显示如下：

`Hello Maven`