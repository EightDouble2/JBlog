---
title: "Docker环境搭建"
date: 2020-06-25T18:51:17+08:00
draft: false
tags: [ "Docker" ]
categories: [ "环境搭建" ]
---
# 配置虚拟机

## 使用 ROOT 账户

在实际生产操作中，我们基本上都是使用超级管理员账户操作 Linux 系统，也就是 Root 用户，Linux 系统默认是关闭 Root 账户的，我们需要为 Root 用户设置一个初始密码以方便我们使用。

- 设置 Root 账户密码

```
sudo passwd root
```

- 切换到 Root

```
su
```

- 设置允许远程登录 Root

```
vi /etc/ssh/sshd_config

# 注释下面这句
# PermitRootLogin without-password
# 加入下面这句
PermitRootLogin yes
```

- 重启服务

```
service ssh restart
```

## 同步时间

- 设置时区

```
dpkg-reconfigure tzdata
```

- 选择 **Asia（亚洲）**

- 选择 **Shanghai（上海）**

- **时间同步**

```
# 安装 ntpdate
apt-get install ntpdate

# 设置系统时间与网络时间同步（cn.pool.ntp.org 位于中国的公共 NTP 服务器）
ntpdate cn.pool.ntp.org

# 将系统时间写入硬件时间
hwclock --systohc
```

- **确认时间**

```
date

# 输出如下（自行对照与系统时间是否一致）
Wed May 20 09:02:29 CST 2020
```

## 修改 cloud.cfg

主要作用是防止重启后主机名还原

```
vi /etc/cloud/cloud.cfg
# 第 15 行
# 该配置默认为 false，修改为 true 即可
preserve_hostname: true
```

## 修改 DNS

```
vi /etc/systemd/resolved.conf
# 第 15 行
# 取消 DNS 行注释，并增加 DNS 配置如：114.114.114.114，修改后重启下计算机
DNS=114.114.114.114
```

# 安装 Docker

## 卸载旧版本

- 清理资源

```
docker system prune --all --volumes
```

- 卸载

```
apt remove docker.io
```

## 安装新版本

```
apt install docker.io
```

## 验证安装

```
docker version
```

## 配置加速器

> **注意：** 国内镜像加速器可能会很卡，请替换成你自己阿里云镜像加速器，地址如：`https://yourself.mirror.aliyuncs.com`，在阿里云控制台的 **容器镜像服务 -> 镜像加速器** 菜单中可以找到

在 `/etc/docker/daemon.json` 中写入如下内容（以下配置修改 `cgroup` 驱动为 `systemd`，满足 K8S 建议）

```
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "registry-mirrors": [
    "https://qrkiae47.mirror.aliyuncs.com/",
    "https://dockerhub.azk8s.cn",
    "https://registry.docker-cn.com"
  ],
  "storage-driver": "overlay2"
}
```

```
# 重启 Docker
systemctl daemon-reload
systemctl restart docker
```

- 验证配置

```
docker info
```

## 安装Docker Compose

```
curl -L https://github.com/docker/compose/releases/download/1.26.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

# 配置容器服务

## 修改主机名

```
# 修改主机名
hostnamectl set-hostname docker-base

# 配置 hosts
cat >> /etc/hosts << EOF
192.168.50.99 docker-base
EOF
```

## 修改 IP 地址

编辑 `vi /etc/netplan/00-installer-config.yaml` 配置文件，修改内容如下

```
network:
  ethernets:
    ens160:
      addresses: [192.168.50.99/24]
      gateway4: 192.168.50.1
      nameservers:
        addresses: [192.168.50.1]
  version: 2
```

使用 `netplan apply` 命令让配置生效

## MySQL

```
version: '3.1'
services:
  mysql:
    image: mysql
    restart: always
    container_name: mysql
    environment:
      MYSQL_ROOT_PASSWORD: # 密码
    command:
      --default-authentication-plugin=mysql_native_password
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_general_ci
      --explicit_defaults_for_timestamp=true
      --lower_case_table_names=1
    ports:
      - 3306:3306
    volumes:
      - ./data:/var/lib/mysql
```

## GitLab

> **注意：** 视频里这段内容说错了，`gitlab_rails['gitlab_shell_ssh_port'] = 2222` 只是为了在网页端 SSH 地址显示为 2222，实际容器里的端口依然是 22，所以映射端口应该是 `2222:22`。下面的配置已修复，直接复制即可不用担心。

```
version: '3.1'
services:
  gitlab:
    image: 'gitlab/gitlab-ce'
    restart: always
    hostname: 'gitlab.johnny.com'
    container_name: 'gitlab'
    environment:
      TZ: 'Asia/Shanghai'
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://gitlab.johnny.com'         
        gitlab_rails['gitlab_shell_ssh_port'] = 2222
        unicorn['port'] = 8888
        nginx['listen_port'] = 80
    ports:
      - '80:80'
      - '443:443'
      - '2222:22'
    volumes:
      - ./config:/etc/gitlab
      - ./data:/var/opt/gitlab
      - ./logs:/var/log/gitlab
```

## Nexus

如果是做的本地数据卷映射，直接启动 Nexus 会有权限问题，我们通过如下命令手动创建数据卷目录

```
mkdir data && chown -R 200 data
```

```
version: '3.1'
services:
  nexus:
    restart: always
    image: sonatype/nexus3
    container_name: nexus
    environment:
      INSTALL4J_ADD_VM_PARAMS: -XX:ActiveProcessorCount=4
    ports:
      - 80:8081
    volumes:
      - ./data:/nexus-data
```

启动成功后访问 Nexus 服务器，默认账号密码如下：

- 默认账号：admin
- 默认密码：`cat /usr/local/docker/nexus/data/admin.password`

注意上面的 Compose 配置文件中我增加了环境变量的配置：

```
environment:
  # 这里是配置 Nexus 分配给应用程序的内核数，按照咱们的虚拟机配置给 4 就可以了
  INSTALL4J_ADD_VM_PARAMS: -XX:ActiveProcessorCount=4
```

## Harbor

下载地址：https://github.com/goharbor/harbor/releases/download/v2.0.0/harbor-offline-installer-v2.0.0.tgz

上传压缩包到服务器的 `/usr/local/docker` 目录下并解压

```
tar -zxvf harbor-offline-installer-v2.0.0.tgz
```

进入 `harbor` 目录并修改配置文件

```
cd harbor
cp harbor.yml.tmpl harbor.yml
vi harbor.yml
```

```
# 第 5 行，修改主机名为自定义域名
hostname: harbor.johnny.com

# 第 12 行，注释掉 HTTPS 的相关配置
# https related config
# https:
  # https port for harbor, default is 443
  # port: 443
  # The path of cert and key files for nginx
  # certificate: /your/certificate/path
  # private_key: /your/private/key/path
```

安装 Harbor

```
./install.sh
```

启动成功后访问 Harbor 服务器，默认账号密码如下：

- 默认账号：admin
- 默认密码：Harbor12345

> **注意：** 安装成功后会在 `harbor` 目录下自动生成 `docker-compose.yml` 配置文件，下一次就可以直接通过操作 Compose 启动和停止容器了

## Jenkins

```
mkdir data && chown -R 1000:1000 data
```

```
version: '3.1'
services:
  jenkins:
    restart: always
    image: jenkins/jenkins
    container_name: jenkins
    environment:
      TZ: Asia/Shanghai
    ports:
      - 80:8080
      - 50000:50000
    volumes:
      - ./data:/var/jenkins_home
```

> **注意：** 默认数据卷的位置在 `/var/lib/docker/volumes` 目录下

启动成功后访问 Jenkins 服务器

- 默认密码：`cat /var/lib/docker/volumes/jenkins_data/_data/secrets/initialAdminPassword`

