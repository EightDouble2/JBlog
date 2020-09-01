---
title: "树莓派搭建samba"
date: 2020-05-17T12:12:58+08:00
draft: false
tags: [ "树莓派", "samba" ]
categories: [ "环境搭建" ]
---
# 树莓派搭建samba
## 挂载硬盘
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

## 安装samba

`sudo apt-get install samba samba-common-bin`

## 配置
`sudo nano /etc/samba/smb.conf`

```cmd
[NetworkDisk]
    # 说明信息
    comment = NetworkDisk
    # 共享文件的路径
    path = /mnt/NetworkDisk
    # 可被其他人看到资源名称（非内容）
    browseable = yes
    # 可写
    writable = yes
    # 新建文件的权限为 664
    create mask = 0664
    # 新建目录的权限为 775
    directory mask = 0775
```

## 添加用户
`sudo smbpasswd -a pi`

## 启动
`sudo systemctl restart smbd`