---
title: "Maven 私服配置"
date: 2020-06-26T22:01:45+08:00
draft: false
tags: [ "Maven", "Nexus" ]
categories: [ "环境搭建" ]
---
# Nexus配置

实际企业开发过程中所有依赖都会走私服下载，需要在 Nexus 上配置相对应的 Maven 代理

## 创建代理仓库

目前咱们只需要配置三个必须的代理仓库（分别为 `aliyun-nexus`、`spring-milestone`、`spring-snapshot`），如果你有其它的代理仓库配置流程同下

- 登录 Nexus 服务器
- 点击 `设置按钮` -> `Repository` -> `Repositories`

- 点击 `Create Repository` -> 选择 `maven2 (proxy)` 创建 Maven 代理仓库

- 配置阿里云仓库代理（版本策略为Release）
  - **Name：** `aliyun-nexus`
  - **Version pollcy：** `Release`
  - **Remote storate：** `http://maven.aliyun.com/nexus/content/groups/public/`

- 配置 Spring 仓库代理（版本策略为Release）
  - **Name：** `spring-milestone`
  - **Version pollcy：** `Release`
  - **Remote storate：** `https://repo.spring.io/milestone`

- 配置 Spring 仓库代理（版本策略为Snapshot）
  - **Name：** `spring-snapshot`
  - **Version pollcy：** `Snapshot`
  - **Remote storate：** `https://repo.spring.io/snapshot`

- 三个代理仓库创建成功后如下图所示

## 配置代理仓库

三个代理仓库创建完成后还无法直接使用，需要进一步配置

- 点击 `设置按钮` -> `Repository` -> `Repositories`
- 选择 `maven-public`，修改 `Group`如下图所示（注意先后顺序）

## 配置计划任务

实际开发过程中可能每天都会产生大量的快照版本，每个快照都会占用相应的空间，历史快照版本就没有什么意义了应该定时清理以释放多占用的空间资源，我们可以通过 **Tasks** 计划任务选项定期清理旧的快照版本。

- 点击`设置按钮`->`System`->`Tasks`
  - **Task name：** `Delete SNAPSHOT`
  - **Repository：** `(All Repositories)`
  - **Minimum snapshot count：** `1`
  - **Snapshot retention (days)：** `0`
  - **Task frequency：** `Manual`

# Maven配置

Nexus 配置完成后还需要配置 Maven，如果第一次启动 Nexus 时选择了 **禁止匿名访问** （修改密码之后的操作）拉取依赖时是需要权限验证的还包括部署等其它配置。

## 配置服务认证

- 修改 `{你的 Maven 目录}/conf/settings.xml` 配置文件
- 修改`<servers>`元素
  - **id：** 唯一标识（**POM 和 mirror 元素需要与之匹配**）
  - **username：** Nexus 登录账号
  - **password：** Nexus 登录密码

```xml
<servers>
    <server>
        <id>nexus-public</id>
        <username>johnnyhao</username>
        <password>密码</password>
    </server>
    <server>
        <id>nexus-releases</id>
        <username>johnnyhao</username>
        <password>密码</password>
    </server>
    <server>
        <id>nexus-snapshots</id>
        <username>johnnyhao</username>
        <password>密码</password>
    </server>
</servers>
```

## 配置镜像仓库

- 修改 `<mirrors>`元素
  - **id：** 需要与 `server` 元素中的 `id` 匹配
  - **mirrorOf：** 可以填入 `central` 或 `*`（所有依赖均通过私服下载）
  - **name：** 随便
  - **url：** 仓库地址

```xml
<mirrors>
    <mirror>
        <id>nexus-public</id>
        <mirrorOf>*</mirrorOf>
        <name>Nexus Public</name>
        <url>http://nexus.johnny.com/repository/maven-public/</url>
    </mirror>
</mirrors>
```
