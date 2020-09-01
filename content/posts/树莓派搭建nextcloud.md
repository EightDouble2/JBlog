---

title: "树莓派搭建nextcloud"
date: 2020-05-15T22:00:36+08:00
draft: false
tags: [ "树莓派", "nextcloud" ]
categories: [ "环境搭建" ]
---

# 树莓派搭建nextcloud

## 准备系统

### 安装raspbain系统

[raspbain官网](https://www.raspberrypi.org/downloads/raspbian/)下载官方raspbain系统系统  

创建文件名为ssh的空文件放入系统卡根目录默认开启ssh  

创建文件名为wpa_supplicant.conf的文件放入系统卡根目录默认开启wifi

```cmd
## To use this file, you should run command "systemctl disable network-manager" and reboot system. 
## (Do not uncomment this line and above!) ##
## 除第一行外，第一行可以删除，去掉以下每行只有单个“#”的注释符号，两个“#”注释符号的行位说明内容，请不要修改
## 中文内容是注释，删除或不要取消前面的“#”符号

## country是设置无线的国家地区，CN是中国
# country=CN
# ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
# update_config=1

## 下面的 "WIFI 1" 、"WIFI 2" 代表多个无线网络的设置
## 除非你要设置多个无线网络，否则只需要设置 "WIFI 1" 这部分的设置即可
## WIFI 1 (Do not uncomment this line!)

## 除了取消需要生效的内容注释以外，以下仅需要修改 "ssid" 和 "psk" 后面引号内的内容即可
## ssid是你的无线Wifi名称，psk是你无线Wifi的密码
# network={
#     ssid="Wifi名称"
#     psk="Wifi密码"
#     priority=1
#     id_str="wifi-1"
# }


## WIFI 2 (Do not uncomment this line!)

# network={
#     ssid="Wifi名称"
#     psk="Wifi密码"
#     priority=2
#     id_str="wifi-2"
# }
```

### 开启ssh远程root访问权限

```cmd
# 编辑sshd_config文件
sudo nano /etc/ssh/sshd_config

# 注释此行 
# PermitRootLogin without-password
# 加入此行 
PermitRootLogin yes

# 重启ssh服务
sudo service ssh restart
```

### 设置时区

```cmd
sudo raspi-config

# 设置time zone
Asia/Shanghai
```

### 配置国内数据源

备份源文件

```cmd
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
sudo cp /etc/apt/sources.list.d/raspi.list /etc/apt/sources.list.d/raspi.list.bak
```

配置sources.list

`sudo nano /etc/apt/sources.list`

```cmd
deb http://mirrors.ustc.edu.cn/raspbian/raspbian/ buster main contrib non-free rpi
```

配置raspi.list

`sudo nano /etc/apt/sources.list.d/raspi.list`

```cmd
deb http://mirrors.ustc.edu.cn/archive.raspberrypi.org/debian/ buster main
```

### 更新系统软件

```cmd
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get dist-upgrade -y
sudo rpi-update
```

## 安装MariaDB

### 安装MariaDB

```cmd
sudo apt-get install -y mariadb-server mariadb-client
```

### 配置MariaDB

设置root密码，禁用远程root登录和删除测试数据库来保护数据库安装

 `sudo mysql_secure_installation`

```cmd
Enter current password for root (enter for none): 
Set root password? [Y/n] y
New password: 
Re-enter new password: 
Remove anonymous users? [Y/n] y
Disallow root login remotely? [Y/n] y
Remove test database and access to it? [Y/n] y
Reload privilege tables now? [Y/n] y
```

进入MariaDB控制台 

`sudo mysql -u root -p`

```cmd
# 创建nextcloud用户
CREATE USER 'nextcloud'@'localhost' IDENTIFIED BY '密码';

# 创建nextcloud数据库
CREATE DATABASE nextcloud;

# 授权
GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'localhost' IDENTIFIED BY '密码';

# 刷新配置
FLUSH PRIVILEGES;

# 退出
exit;
```

## 安装Redis

### 安装Redis

```cmd
sudo apt-get install -y redis-server
```

### 配置Redis

`sudo nano /etc/redis/redis.conf`

```cmd
# 开启远程访问，注释下行
# bind 127.0.0.1 ::1
# bind 127.0.0.1

# 开启后台运行
daemonize yes

# 开启密码登录
requirepass 密码
```

```cmd
# 重启Redis
sudo systemctl restart redis-server

# 查看Redis状态
sudo systemctl status redis-server
```

## 安装php环境

### 安装php

```cmd
sudo apt-get install -y php php-{cli,xml,zip,curl,gd,cgi,mysql,mbstring,fpm,intl,imagick,redis}
```

### 配置php

`sudo nano /etc/php/7.3/fpm/php.ini`

```cmd
date.timezone = Asia/Shanghai

memory_limit = 512M

upload_max_filesize = 500M

post_max_size = 500M

max_execution_time = 300
```

`sudo nano /etc/php/7.3/fpm/pool.d/www.conf`

```cmd
# 取消注释
clear_env = no

env[HOSTNAME] = $HOSTNAME
env[PATH] = /usr/local/bin:/usr/bin:/bin
env[TMP] = /tmp
env[TMPDIR] = /tmp
env[TEMP] = /tmp
```

```cmd
# 重启Redis
sudo systemctl restart php7.3-fpm

# 查看Redis状态
sudo systemctl status php7.3-fpm
```

## 安装apache2

### 安装apache2

```cmd
sudo apt-get install -y apache2 libapache2-mod-php
```

### 修改apache2端口

`sudo nano /etc/apache2/ports.conf`

```cmd
Listen 8080

<IfModule ssl_module>
	Listen 8443
</IfModule>

<IfModule mod_gnutls.c>
	Listen 8443
</IfModule>
```

`sudo nano /etc/apache2/sites-enabled/000-default.conf`

```cmd
<VirtualHost *:8080>
```

### 配置apache2

`sudo nano /etc/php/7.3/apache2/php.ini`

```cmd
date.timezone = Asia/Shanghai

memory_limit = 512M

upload_max_filesize = 500M

post_max_size = 500M

max_execution_time = 300
```

```cmd
# 重启apache2
sudo systemctl restart apache2

# 查看Redis状态
sudo systemctl status apache2
```

## 安装nginx

### 安装nginx

```cmd
sudo apt-get install -y nginx
```

### 生成ssl证书

```cmd
# 创建证书存放目录
mkdir /etc/nginx/cert

# 使用openssl生成证书
openssl req -new -x509 -days 365 -nodes -out /etc/nginx/cert/nextcloud.crt -keyout /etc/nginx/cert/nextcloud.key

Country Name (2 letter code) [AU]:CN
State or Province Name (full name) [Some-State]:ChongQing
Locality Name (eg, city) []:ChongQing
Organization Name (eg, company) [Internet Widgits Pty Ltd]:JProject
Organizational Unit Name (eg, section) []:Technical
Common Name (e.g. server FQDN or YOUR name) []:JohnnyHao
Email Address []:johnny173862903@gmail.com

# 配置证书目录权限
chmod 700 /etc/nginx/cert
chmod 600 /etc/nginx/cert/*
```

### 配置nginx

`sudo nano /etc/nginx/sites-enabled/default`

```cmd
upstream php-handler {
    server 127.0.0.1:8080;
    #server unix:/var/run/php/php7.3-fpm.sock;
}

server {
    listen 80;
    listen [::]:80;
    server_name nextcloud.johnny.com;
    # enforce https
    return 301 https://$server_name:443$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name nextcloud.johnny.com;

    # Use Mozilla's guidelines for SSL/TLS settings
    # https://mozilla.github.io/server-side-tls/ssl-config-generator/
    # NOTE: some settings below might be redundant
    ssl_certificate /etc/nginx/cert/nextcloud.crt;
    ssl_certificate_key /etc/nginx/cert/nextcloud.key;

    # Add headers to serve security related headers
    # Before enabling Strict-Transport-Security headers please read into this
    # topic first.
    add_header Strict-Transport-Security "max-age=15768000; includeSubDomains; preload;" always;
    #
    # WARNING: Only add the preload option once you read about
    # the consequences in https://hstspreload.org/. This option
    # will add the domain to a hardcoded list that is shipped
    # in all major browsers and getting removed from this list
    # could take several months.
    add_header Referrer-Policy "no-referrer" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Download-Options "noopen" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Permitted-Cross-Domain-Policies "none" always;
    add_header X-Robots-Tag "none" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # Remove X-Powered-By, which is an information leak
    fastcgi_hide_header X-Powered-By;

    # Path to the root of your installation
    root /var/www/html/nextcloud;

    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }

    # The following 2 rules are only needed for the user_webfinger app.
    # Uncomment it if you're planning to use this app.
    #rewrite ^/.well-known/host-meta /public.php?service=host-meta last;
    #rewrite ^/.well-known/host-meta.json /public.php?service=host-meta-json last;

    # The following rule is only needed for the Social app.
    # Uncomment it if you're planning to use this app.
    #rewrite ^/.well-known/webfinger /public.php?service=webfinger last;

    location = /.well-known/carddav {
      return 301 $scheme://$host:$server_port/remote.php/dav;
    }
    location = /.well-known/caldav {
      return 301 $scheme://$host:$server_port/remote.php/dav;
    }

    # set max upload size
    client_max_body_size 512M;
    fastcgi_buffers 64 4K;

    # Enable gzip but do not remove ETag headers
    gzip on;
    gzip_vary on;
    gzip_comp_level 4;
    gzip_min_length 256;
    gzip_proxied expired no-cache no-store private no_last_modified no_etag auth;
    gzip_types application/atom+xml application/javascript application/json application/ld+json application/manifest+json application/rss+xml application/vnd.geo+json application/vnd.ms-fontobject application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml application/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy;

    # Uncomment if your server is build with the ngx_pagespeed module
    # This module is currently not supported.
    #pagespeed off;

    location / {
        rewrite ^ /index.php;
    }

    location ~ ^\/(?:build|tests|config|lib|3rdparty|templates|data)\/ {
        deny all;
    }
    location ~ ^\/(?:\.|autotest|occ|issue|indie|db_|console) {
        deny all;
    }

    location ~ ^\/(?:index|remote|public|cron|core\/ajax\/update|status|ocs\/v[12]|updater\/.+|oc[ms]-provider\/.+)\.php(?:$|\/) {
        fastcgi_split_path_info ^(.+?\.php)(\/.*|)$;
        set $path_info $fastcgi_path_info;
        try_files $fastcgi_script_name =404;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $path_info;
        fastcgi_param HTTPS on;
        # Avoid sending the security headers twice
        fastcgi_param modHeadersAvailable true;
        # Enable pretty urls
        fastcgi_param front_controller_active true;
        fastcgi_pass php-handler;
        fastcgi_intercept_errors on;
        fastcgi_request_buffering off;
    }

    location ~ ^\/(?:updater|oc[ms]-provider)(?:$|\/) {
        try_files $uri/ =404;
        index index.php;
    }

    # Adding the cache control header for js, css and map files
    # Make sure it is BELOW the PHP block
    location ~ \.(?:css|js|woff2?|svg|gif|map)$ {
        try_files $uri /index.php$request_uri;
        add_header Cache-Control "public, max-age=15778463";
        # Add headers to serve security related headers (It is intended to
        # have those duplicated to the ones above)
        # Before enabling Strict-Transport-Security headers please read into
        # this topic first.
        add_header Strict-Transport-Security "max-age=15768000; includeSubDomains; preload;" always;
        #
        # WARNING: Only add the preload option once you read about
        # the consequences in https://hstspreload.org/. This option
        # will add the domain to a hardcoded list that is shipped
        # in all major browsers and getting removed from this list
        # could take several months.
        add_header Referrer-Policy "no-referrer" always;
        add_header X-Content-Type-Options "nosniff" always;
        add_header X-Download-Options "noopen" always;
        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-Permitted-Cross-Domain-Policies "none" always;
        add_header X-Robots-Tag "none" always;
        add_header X-XSS-Protection "1; mode=block" always;

        # Optional: Don't log access to assets
        access_log off;
    }

    location ~ \.(?:png|html|ttf|ico|jpg|jpeg|bcmap)$ {
        try_files $uri /index.php$request_uri;
        # Optional: Don't log access to other assets
        access_log off;
    }
}
```
```cmd
# 重启nginx
sudo systemctl restart nginx

# 查看nginx状态
sudo systemctl status nginx
```

## 安装nextcloud

### 下载nextcloud

[nextcloud官网](https://nextcloud.com/)下载软件包

### 安装unzip解压软件包

```cmd
sudo apt-get install -y unzip
unzip nextcloud-18.0.4.zip
```

### 配置目录权限

```cmd
# 移动软件包到apache2项目目录
sudo mv nextcloud /var/www/html/

# 设置nextcloud目录权限
sudo chown -R www-data:www-data /var/www/html/nextcloud
sudo sudo chmod -R 755 /var/www/html/nextcloud

# 设置nextcloud数据目录权限
sudo mkdir /var/www/html/nextcloud-data
sudo chown -R www-data:www-data /var/www/html/nextcloud-data
sudo sudo chmod -R 755 /var/www/html/nextcloud-data
```

### 配置nextcloud

`sudo nano /var/www/html/nextcloud/config/config.php`

```cmd
# 挂载外部磁盘
  'check_data_directory_permissions' => false,
  
# 设置域名权限  
  'trusted_domains' => 
  array (
    0 => '192.168.50.164',
    1 => 'johnnyhao.vicp.io',
    2 => 'nextcloud.johnny.com',
  ),

# 配置redis
  'filelocking.enabled' => 'true',
  'memcache.local' => '\OC\Memcache\Redis',
  'memcache.distributed' => '\OC\Memcache\Redis',
  'memcache.locking' => '\OC\Memcache\Redis',
  'redis' =>
  array (
    'host' => 'localhost',
    'port' => 6379,
    'timeout' => 0,
    'password' => '密码',
    'dbindex' => 0,
  ),
```

解决设置警告

```cmd
cd /var/www/html/nextcloud

# 数据库丢失了一些索引
sudo -u www-data php occ db:convert-filecache-bigint

# 数据库中的一些列由于进行长整型转换而缺失
sudo -u www-data php occ db:add-missing-indices
```

## 挂载本地硬盘

### 挂载本地硬盘

```cmd
# 创建挂载目录
mkdir /mnt/NetworkDisk/

# 查看硬盘
sudo fdisk -l

Device     Boot Start       End   Sectors   Size Id Type
/dev/sdb1  *     2048 976773119 976771072 465.8G  7 HPFS/NTFS/exFAT

# 挂载本地硬盘
mount -t ntfs-3g /dev/sdb1 /mnt/NetworkDisk/

# 开机自动挂载
sudo nano /etc/fstab

/dev/sdb1 /mnt/NetworkDisk ntfs defaults,nofail,x-systemd.device-timeout=1,noatime 0 0

```

### 配置nextcloud

```cmd
# 开启挂载外部磁盘应用
External storage support

# 设置中设置外部存储
```

## 使用rclone挂载OneDrive

### 下载rclone

`curl https://rclone.org/install.sh | sudo bash`

### 配置rclone

```cmd
n/s/q> n
name> OneDrive5T
Storage> 23
client_id> 
client_secret> 
Edit advanced config? (y/n)
y) Yes
n) No (default)
y/n> 
Remote config
Use auto config?
 * Say Y if not sure
 * Say N if you are working on a remote or headless machine
   y) Yes (default)
   n) No
   y/n> n
   # windows或mac上使用rclone获取授权 rclone authorize "onedrive"
   result> 
   Choose a number from below, or type in an existing value
    1 / OneDrive Personal or Business
   \ "onedrive"
   Your choice> 1
   Found 1 drives, please select the one you want to use:
   0: OneDrive (business) 
   Chose drive to use:> 0
   Is that okay?
   y) Yes (default)
   n) No
   y/n> 
y) Yes this is OK (default)
e) Edit this remote
d) Delete this remote
y/e/d> 
Current remotes:

Name                 Type
====                 ====
OneDrive5T           onedrive

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> q
```

### 挂载OneDrive

`rclone mount OneDrive5T:/ /mnt/NetworkDisk/OneDrive5T --copy-links --no-gzip-encoding --no-check-certificate --allow-other --allow-non-empty --umask 000 --daemon`