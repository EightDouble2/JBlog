---
title: "Spring 事务管理"
date: 2020-04-27T17:27:58+08:00
draft: false
tags: [ "Java" ]
categories: [ "技术文档" ]
---
# Spring 事务管理

事务原本是数据库中的概念，用于数据访问层。但一般情况下，需要将事务提升到业务层，即 Service 层。这样做是为了能够使用事务的特性来管理具体的业务。

在 Spring 中通常可以通过以下三种方式来实现对事务的管理：

- 使用 Spring 的事务代理工厂管理事务（已过时）
- 使用 Spring 的事务注解管理事务
- 使用 AspectJ 的 AOP 配置管理事务

## Spring 事务管理 API

Spring 的事务管理，主要用到两个事务相关的接口。

### 事务管理器接口

事务管理器是 `PlatformTransactionManager` 接口对象。其主要用于完成事务的提交、回滚，及获取事务的状态信息。

该接口定义了 3 个事务方法：

- `void commit(TransactionStatus status)`：事务的提交
- `TransactionStatus getTransaction(TransactionDefinition definition)`：获取事务的状态
- `void rollback(TranscationStatus status)`：事务的回滚

#### 常用的两个实现类

`PlatformTransactionManager` 接口有两个常用的实现类：

- `DataSourceTransactionManager`：使用 JDBC 或 MyBatis 进行持久化数据时使用。
- `HibernateTransactionManager`：使用 Hibernate 进行持久化数据时使用。

#### Spring 的回滚方式

Spring 事务的默认回滚方式是：发生运行时异常回滚

### 事务定义接口

事务定义接口 `TransactionDefinition` 中定义了事务描述相关的三类常量：**事务隔离级别**、**事务传播行为**、**事务默认超时时限**，及对它们的操作。

#### 事务的四种隔离级别

- DEFAULT：采用 DB 默认的事务隔离级别。MySql 默认为 REPEATABLE_READ；Oracle 默认为：READ_COMMITTED；
- READ_UNCOMMITTED：读未提交。未解决任何并发问题。
- READ_COMMITTED：读已提交。解决脏读，存在不可重复读与幻读。
- REPEATABLE_READ：可重复读。解决脏读、不可重复读。存在幻读。
- SERIALIZABLE：串行化。不存在并发问题。

#### 事务的七种传播行为

所谓事务传播行为是指，处于不同事务中的方法在相互调用时，执行期间事务的维护情况。如，A 事务中的方法 `a()` 调用 B 事务中的方法 `b()`，在调用执行期间事务的维护情况，就称为事务传播行为。事务传播行为是加在方法上的。

- REQUIRED：指定的方法必须在事务内执行。若当前存在事务，就加入到当前事务中；若当前没有事务，则创建一个新事务。这种传播行为是最常见的选择，也是 Spring 默认的事务传播行为。
- SUPPORTS：指定的方法支持当前事务，但若当前没有事务，也可以以非事务方式执行。
- MANDATORY：指定的方法必须在当前事务内执行，若当前没有事务，则直接抛出异常。
- REQUIRES_NEW：总是新建一个事务，若当前存在事务，就将当前事务挂起，直到新事务执行完毕。
- NOT_SUPPORTED：指定的方法不能在事务环境中执行，若当前存在事务，就将当前事务挂起。
- NEVER：指定的方法不能在事务环境下执行，若当前存在事务，就直接抛出异常。
- NESTED：指定的方法必须在事务内执行。若当前存在事务，则在嵌套事务内执行；若当前没有事务，则创建一个新事务。

# 使用 AspectJ 的 AOP 配置管理事务

注：本案例所使用的数据库为 `项目实战` 中的数据库，有需要的请加群获取

AspectJ 主要是使用 XML 配置顾问方式自动为每个符合切入点表达式的类生成事务代理。创建测试操作步骤如下：

## 创建测试项目

创建一个名为 `aspectj-aop` 项目，`pom.xml` 文件如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>hello-spring-transaction</groupId>
    <artifactId>aspectj-aop</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <properties>
        <!-- 环境配置 -->
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>

        <!-- 统一的依赖管理 -->
        <log4j.version>1.2.17</log4j.version>
        <slf4j.version>1.7.25</slf4j.version>
        <spring.version>4.3.17.RELEASE</spring.version>
        <alibaba-druid.version>1.1.6</alibaba-druid.version>
        <mysql.version>5.1.46</mysql.version>
        <mybatis.version>3.2.8</mybatis.version>
        <mybaits-spring.version>1.3.1</mybaits-spring.version>
        <junit.version>4.12</junit.version>
        <lombok.version>1.16.18</lombok.version>
    </properties>

    <dependencies>
        <!-- Test Begin -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>${junit.version}</version>
        </dependency>
        <!-- Test Begin -->

        <!-- Spring Begin -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <!-- Spring End -->

        <!-- Log Begin -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>jcl-over-slf4j</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>jul-to-slf4j</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>${log4j.version}</version>
        </dependency>
        <!-- Log End -->

        <!-- Database Begin -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>${alibaba-druid.version}</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${mysql.version}</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>${mybatis.version}</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>${mybaits-spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <!-- Database End -->

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>${lombok.version}</version>
        </dependency>
    </dependencies>
</project>
```

相较于 `项目实战` 中的 POM，主要是增加了 `org.springframework:spring-aspects` 依赖

## 创建实体类

### TbContentCategory

```java
package com.hello.spring.transaction.aspectsj.aop.domain;

import lombok.Data;

import java.util.Date;

/**
 * 分类管理
 * <p>Title: TbContentCategory</p>
 * <p>Description: </p>
 *
 * @author JohnnyHao
 * @version 1.0.0
 * @date 2018/6/25 9:14
 */
@Data
public class TbContentCategory {
    private Long id;
    private String name;
    private Integer status;
    private Integer sortOrder;
    private Boolean isParent;
    private Date created;
    private Date updated;
    private TbContentCategory parent;
}
```

### TbContent

```java
package com.hello.spring.transaction.aspectsj.aop.domain;

import lombok.Data;

import java.util.Date;

/**
 * 内容管理
 * <p>Title: TbContent</p>
 * <p>Description: </p>
 *
 * @author JohnnyHao
 * @version 1.0.0
 * @date 2018/6/25 14:02
 */
@Data
public class TbContent {
    private Long id;
    private String title;
    private String subTitle;
    private String titleDesc;
    private String url;
    private String pic;
    private String pic2;
    private String content;
    private Date created;
    private Date updated;
    private TbContentCategory tbContentCategory;
}
```

## 创建数据访问层

### TbContentCategoryDao

```java
package com.hello.spring.transaction.aspectsj.aop.dao;

import com.hello.spring.transaction.aspectsj.aop.domain.TbContentCategory;
import org.springframework.stereotype.Repository;

@Repository
public interface TbContentCategoryDao {
    void insert(TbContentCategory tbContentCategory);
}
```

### TbContentDao

```java
package com.hello.spring.transaction.aspectsj.aop.dao;

import com.hello.spring.transaction.aspectsj.aop.domain.TbContent;
import org.springframework.stereotype.Repository;

@Repository
public interface TbContentDao {
    void insert(TbContent tbContent);
}
```

## 创建业务逻辑层

### TbContentCategoryService

接口

```java
package com.hello.spring.transaction.aspectsj.aop.service;

import com.hello.spring.transaction.aspectsj.aop.domain.TbContent;
import com.hello.spring.transaction.aspectsj.aop.domain.TbContentCategory;

public interface TbContentCategoryService {
    void save(TbContentCategory tbContentCategory, TbContent tbContent);
}
```

实现

```java
package com.hello.spring.transaction.aspectsj.aop.service.impl;

import com.hello.spring.transaction.aspectsj.aop.dao.TbContentCategoryDao;
import com.hello.spring.transaction.aspectsj.aop.domain.TbContent;
import com.hello.spring.transaction.aspectsj.aop.domain.TbContentCategory;
import com.hello.spring.transaction.aspectsj.aop.service.TbContentCategoryService;
import com.hello.spring.transaction.aspectsj.aop.service.TbContentService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class TbContentCategoryServiceImpl implements TbContentCategoryService {

    @Autowired
    private TbContentCategoryDao tbContentCategoryDao;

    @Autowired
    private TbContentService tbContentService;

    public void save(TbContentCategory tbContentCategory, TbContent tbContent) {
        tbContentCategoryDao.insert(tbContentCategory);
        tbContentService.save(tbContent);
    }
}
```

### TbContentService

接口

```java
package com.hello.spring.transaction.aspectsj.aop.service;

import com.hello.spring.transaction.aspectsj.aop.domain.TbContent;

public interface TbContentService {
    void save(TbContent tbContent);
}
```

实现

```java
package com.hello.spring.transaction.aspectsj.aop.service.impl;

import com.hello.spring.transaction.aspectsj.aop.dao.TbContentDao;
import com.hello.spring.transaction.aspectsj.aop.domain.TbContent;
import com.hello.spring.transaction.aspectsj.aop.service.TbContentService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class TbContentServiceImpl implements TbContentService {

    @Autowired
    private TbContentDao tbContentDao;

    public void save(TbContent tbContent) {
        tbContentDao.insert(tbContent);
    }
}
```

## 创建 Spring 配置

### `spring-context.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <context:annotation-config/>
    <context:component-scan base-package="com.hello.spring.transaction.aspectsj.aop">
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

    <!-- 配置事务管理器 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 配置事务通知 -->
    <tx:advice id="myAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="save*" propagation="REQUIRED"/>
        </tx:attributes>
    </tx:advice>

    <!-- 配置顾问和切入点 -->
    <aop:config>
        <aop:pointcut id="myPointcut" expression="execution(* com.hello.spring.transaction.aspectsj.aop.service.*.*(..))" />
        <aop:advisor advice-ref="myAdvice" pointcut-ref="myPointcut" />
    </aop:config>
</beans>
```

### `spring-context-druid.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 加载配置属性文件 -->
    <context:property-placeholder ignore-unresolvable="true" location="classpath:jdbc.properties"/>

    <!-- 数据源配置, 使用 Druid 数据库连接池 -->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
        <!-- 数据源驱动类可不写，Druid默认会自动根据URL识别DriverClass -->
        <property name="driverClassName" value="${jdbc.driverClass}"/>

        <!-- 基本属性 url、user、password -->
        <property name="url" value="${jdbc.connectionURL}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>

        <!-- 配置初始化大小、最小、最大 -->
        <property name="initialSize" value="${jdbc.pool.init}"/>
        <property name="minIdle" value="${jdbc.pool.minIdle}"/>
        <property name="maxActive" value="${jdbc.pool.maxActive}"/>

        <!-- 配置获取连接等待超时的时间 -->
        <property name="maxWait" value="60000"/>

        <!-- 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒 -->
        <property name="timeBetweenEvictionRunsMillis" value="60000"/>

        <!-- 配置一个连接在池中最小生存的时间，单位是毫秒 -->
        <property name="minEvictableIdleTimeMillis" value="300000"/>

        <property name="validationQuery" value="${jdbc.testSql}"/>
        <property name="testWhileIdle" value="true"/>
        <property name="testOnBorrow" value="false"/>
        <property name="testOnReturn" value="false"/>

        <!-- 配置监控统计拦截的filters -->
        <property name="filters" value="stat"/>
    </bean>
</beans>
```

### `spring-context-mybatis.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">

    <!-- 配置 SqlSession -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <!-- 用于配置对应实体类所在的包，多个 package 之间可以用 ',' 号分割 -->
        <property name="typeAliasesPackage" value="com.hello.spring.transaction.aspectsj.aop.domain"/>
        <!-- 用于配置对象关系映射配置文件所在目录 -->
        <property name="mapperLocations" value="classpath:/mapper/**/*.xml"/>
        <property name="configLocation" value="classpath:/mybatis-config.xml"></property>
    </bean>

    <!-- 扫描 Mapper -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.hello.spring.transaction.aspectsj.aop.dao" />
    </bean>
</beans>
```

## 创建 MyBatis 配置和映射文件

### `mybatis-config.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 全局参数 -->
    <settings>
        <!-- 打印 SQL 语句 -->
        <setting name="logImpl" value="STDOUT_LOGGING" />

        <!-- 使全局的映射器启用或禁用缓存。 -->
        <setting name="cacheEnabled" value="false"/>

        <!-- 全局启用或禁用延迟加载。当禁用时，所有关联对象都会即时加载。 -->
        <setting name="lazyLoadingEnabled" value="true"/>

        <!-- 当启用时，有延迟加载属性的对象在被调用时将会完全加载任意属性。否则，每种属性将会按需要加载。 -->
        <setting name="aggressiveLazyLoading" value="true"/>

        <!-- 是否允许单条 SQL 返回多个数据集 (取决于驱动的兼容性) default:true -->
        <setting name="multipleResultSetsEnabled" value="true"/>

        <!-- 是否可以使用列的别名 (取决于驱动的兼容性) default:true -->
        <setting name="useColumnLabel" value="true"/>

        <!-- 允许 JDBC 生成主键。需要驱动器支持。如果设为了 true，这个设置将强制使用被生成的主键，有一些驱动器不兼容不过仍然可以执行。 default:false  -->
        <setting name="useGeneratedKeys" value="false"/>

        <!-- 指定 MyBatis 如何自动映射 数据基表的列 NONE：不映射 PARTIAL：部分 FULL:全部  -->
        <setting name="autoMappingBehavior" value="PARTIAL"/>

        <!-- 这是默认的执行类型 （SIMPLE: 简单； REUSE: 执行器可能重复使用prepared statements语句；BATCH: 执行器可以重复执行语句和批量更新） -->
        <setting name="defaultExecutorType" value="SIMPLE"/>

        <!-- 使用驼峰命名法转换字段。 -->
        <setting name="mapUnderscoreToCamelCase" value="true"/>

        <!-- 设置本地缓存范围 session:就会有数据的共享 statement:语句范围 (这样就不会有数据的共享 ) defalut:session -->
        <setting name="localCacheScope" value="SESSION"/>

        <!-- 设置 JDBC 类型为空时,某些驱动程序 要指定值, default:OTHER，插入空值时不需要指定类型 -->
        <setting name="jdbcTypeForNull" value="NULL"/>
    </settings>
</configuration>
```

### `TbContentCategoryMapper.xml`

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.hello.spring.transaction.aspectsj.aop.dao.TbContentCategoryDao">
    <insert id="insert">
        INSERT INTO tb_content_category (
          `parent_id`,
          `name`,
          `status`,
          `sort_order`,
          `is_parent`,
          `created`,
          `updated`
        )
        VALUES
          (
            #{parent.id},
            #{name},
            #{status},
            #{sortOrder},
            #{isParent},
            #{created},
            #{updated}
          );
    </insert>
</mapper>
```

### `TbContentMapper.xml`

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.hello.spring.transaction.aspectsj.aop.dao.TbContentDao">
    <insert id="insert">
        INSERT INTO tb_content (
          `category_id`,
          `title`,
          `sub_title`,
          `title_desc`,
          `url`,
          `pic`,
          `pic2`,
          `content`,
          `created`,
          `updated`
        )
        VALUES
          (
            #{tbContentCategory.id},
            #{title},
            #{subTitle},
            #{titleDesc},
            #{url},
            #{pic},
            #{pic2},
            #{content},
            #{created},
            #{updated}
          )
    </insert>
</mapper>
```

## 创建系统配置文件

### `log4j.properties`

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

### `jdbc.properties`

```properties
#============================#
#==== Database settings ====#
#============================#

# JDBC
# MySQL 8.x: com.mysql.cj.jdbc.Driver
jdbc.driverClass=com.mysql.jdbc.Driver
jdbc.connectionURL=jdbc:mysql://192.168.75.134:3306/myshop?useUnicode=true&characterEncoding=utf-8&useSSL=false
jdbc.username=root
jdbc.password=123456

# JDBC Pool
jdbc.pool.init=1
jdbc.pool.minIdle=3
jdbc.pool.maxActive=20

# JDBC Test
jdbc.testSql=SELECT 'x' FROM DUAL
```

## 创建测试类

```java
package com.hello.spring.transaction.aspectsj.aop.service.test;

import com.hello.spring.transaction.aspectsj.aop.domain.TbContent;
import com.hello.spring.transaction.aspectsj.aop.domain.TbContentCategory;
import com.hello.spring.transaction.aspectsj.aop.service.TbContentCategoryService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration({"classpath:spring-context.xml", "classpath:spring-context-druid.xml", "classpath:spring-context-mybatis.xml"})
public class TestSpringTransaction {

    @Autowired
    private TbContentCategoryService tbContentCategoryService;

    @Test
    public void test() {
        TbContentCategory tbContentCategory = new TbContentCategory();
        tbContentCategory.setId(1L);
        tbContentCategory.setName("测试事务分类");

        TbContent tbContent = new TbContent();
        tbContent.setTbContentCategory(tbContentCategory);
        // 在这里你可以将内容设置为超出数据库字段的存储范围来验证事务是否开启
        tbContent.setTitle("测试事务内容");

        tbContentCategoryService.save(tbContentCategory, tbContent);
    }
}
```

运行观察事务效果：

- 有事务：数据插入成功则两张表都存在数据，只要出现异常则两张表都没有数据
- 无事务：如果第一张表数据插入成功，但第二张表报错则第一张表的数据不会回滚

# 使用 Spring 注解管理事务

通过 `@Transactional` 注解方式，也可将事务织入到相应方法中。而使用注解方式，只需在配置文件中加入一个 `tx` 标签，以告诉 Spring 使用注解来完成事务的织入。该标签只需指定一个属性，事务管理器。

```xml
<!-- 开启事务注解驱动 -->
<tx:annotation-driven transaction-manager="transactionManager" />
```

## `@Transactional` 注解简介

`@Transactional` 的所有可选属性：

- `propagation`：用于设置事务传播属性。该属性类型为 Propagation 枚举，默认值为 `Propagation.REQUIRED`。
- `isolation`：用于设置事务的隔离级别。该属性类型为 Isolation 枚举 ，默认值为 `Isolation.DEFAULT`。
- `readOnly`：用于设置该方法对数据库的操作是否是只读的。该属性为 boolean，默认值为 `false`。
- `timeout`：用于设置本操作与数据库连接的超时时限。单位为秒，类型为 int，默认值为 -1，即没有时限。
- `rollbackFor`：指定需要回滚的异常类。类型为 `Class[]`，默认值为空数组。当然，若只有一个异常类时，可以不使用数组。
- `rollbackForClassName`：指定需要回滚的异常类类名。类型为 `String[]`，默认值为空数组。当然，若只有一个异常类时，可以不使用数组。
- `noRollbackFor`：指定不需要回滚的异常类。类型为 `Class[]`，默认值为空数组。当然，若只有一个异常类时，可以不使用数组。
- `noRollbackForClassName`： 指定不需要回滚的异常类类名。类型为 `String[]`，默认值为空数组。当然，若只有一个异常类时，可以不使用数组。

需要注意的是，`@Transactional` 若用在方法上，只能用于 `public` 方法上。对于其他非 `public` 方法，如果加上了注解 `@Transactional`，虽然 Spring 不会报错，但不会将指定事务织入到该方法中。因为 Spring 会忽略掉所有非 `public` 方法上的 `@Transaction` 注解。

若 `@Transaction` 注解在类上，则表示该类上所有的方法均将在执行时织入事务。

## 使用 `@Transaction` 注解

使用起来很简单，只需要在需要增加事务的业务类上增加 `@Transaction` 注解即可，案例代码如下：

```java
package com.hello.spring.transaction.aspectsj.aop.service.impl;

import com.hello.spring.transaction.aspectsj.aop.dao.TbContentCategoryDao;
import com.hello.spring.transaction.aspectsj.aop.domain.TbContent;
import com.hello.spring.transaction.aspectsj.aop.domain.TbContentCategory;
import com.hello.spring.transaction.aspectsj.aop.service.TbContentCategoryService;
import com.hello.spring.transaction.aspectsj.aop.service.TbContentService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Transactional
@Service(value = "tbContentCategoryService")
public class TbContentCategoryServiceImpl implements TbContentCategoryService {

    @Autowired
    private TbContentCategoryDao tbContentCategoryDao;

    @Autowired
    private TbContentService tbContentService;

    public void save(TbContentCategory tbContentCategory, TbContent tbContent) {
        tbContentCategoryDao.insert(tbContentCategory);
        tbContentService.save(tbContent);
    }
}
```