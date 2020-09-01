---
title: "树莓派搭建TimeMachine"
date: 2020-05-17T11:43:20+08:00
draft: false
tags: [ "树莓派", "TimeMachine" ]
categories: [ "环境搭建" ]
---
# 树莓派搭建TimeMachine
## 挂载硬盘

```cmd
# 创建挂载目录
mkdir /mnt/NetworkDisk/

# 查看硬盘
sudo fdisk -l

Device     Boot Start       End   Sectors   Size Id Type
/dev/sda1  *     2048 976773119 976771072 465.8G  7 HPFS/NTFS/exFAT

# 安装NTFS支持
sudo apt-get install hfsprogs hfsplus

# 格式化硬盘
sudo mkfs -t ntfs /dev/sda1

# 挂载本地硬盘
mount -t ntfs-3g /dev/sda1 /mnt/TimeMachine/

# 开机自动挂载
sudo nano /etc/fstab

/dev/sda1 /mnt/TimeMachine ntfs defaults,nofail,x-systemd.device-timeout=1,noatime 0 0
```

## 安装netatalk

```cmd
# 解压netatalk
tar -xf netatalk-3.1.12.tar.gz netatalk-3.1.12/
cd netatalk-3.1.12/

# 安装依赖
sudo apt install build-essential libevent-dev libssl-dev libgcrypt-dev libkrb5-dev libpam0g-dev libwrap0-dev libdb-dev libtdb-dev avahi-daemon libavahi-client-dev libacl1-dev libldap2-dev libcrack2-dev libdbus-1-dev libdbus-glib-1-dev libglib2.0-dev

# 安装netatalk
sudo ./configure --with-init-style=debian-systemd --without-libevent --without-tdb --with-cracklib --enable-krbV-uam --with-pam-confdir=/etc/pam.d --with-dbus-daemon=/usr/bin/dbus-daemon --with-dbus-sysconf-dir=/etc/dbus-1/system.d --with-tracker-pkgconfig-version=1.0
sudo make
sudo make install

# 检查netatalk版本
netatalk -V
```

## 配置netatalk

`sudo nano /etc/nsswitch.conf`

```cmd
# 添加 mdns4 mdns
hosts: files mdns4_minimal [NOTFOUND=return] dns mdns4 mdns
```

`sudo nano /etc/avahi/services/afpd.service`

```xml
<?xml version="1.0" standalone='no'?><!--*-nxml-*-->
<!DOCTYPE service-group SYSTEM "avahi-service.dtd">
<service-group>
    <name replace-wildcards="yes">%h</name>
    <service>
        <type>_afpovertcp._tcp</type>
        <port>548</port>
    </service>
    <service>
        <type>_device-info._tcp</type>
        <port>0</port>
        <txt-record>model=TimeCapsule</txt-record>
    </service>
</service-group>
```

`sudo nano /usr/local/etc/afp.conf`

```xml
[Global]
      mimic model = TimeCapsule6,106
      hostname = TimeCapsule

[Time Machine]
      path = /media/TimeMachine
      time machine = yes
```

```cmd
# 重启netatalk
sudo systemctl restart avahi-daemon 
sudo systemctl unmask netatalk
sudo systemctl restart netatalk
```

