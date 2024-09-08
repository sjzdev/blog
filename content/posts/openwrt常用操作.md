---
blog: true
tags:
  - "#openwrt"
categories:
  - openwrt
collections: 
title: openwrt常用操作
date: 2022-06-03 00:32:28
update: 2024-09-08 22:11:05
---

## rclone 开机启动项

![](/blog/img/openwrt常用操作_image_1.png)

将挂载命令改成 `/usr/bin/rclone mount crypt:/ /mnt/disk/data/webdav --daemon --log-file=/root/rclone.log` 后，输出日志中的内容为：

```
2023/11/18 15:22:55 NOTICE: Config file "/.config/rclone/rclone.conf" not found - using defaults
2023/11/18 15:22:55 Failed to create file system for "crypt:/": didn't find section in config file
```

发现不成功的原因是没发现配置文件 `/.config/rclone/rclone.conf`，可能原因是启动脚本执行的时候环境变量还没创建，所以无法定位到默认的 `${HOME}/.config/rclone/rlone.conf` 文件，可以在命令中直接指定配置文件的位置。

```
/usr/bin/rclone mount crypt:/ /mnt/disk/data/webdav --daemon  --config "/etc/rclone/rclone.conf" --allow-non-empty
```

添加 `--allow-non-empty` 参数是为了当挂载目录不为空的时候可以正常挂载。

## 自动挂在硬盘

openwrt 中的挂载配置文件为 `/etc/config/fstab`，其中的内容大致为：

```txt
config global
	option anon_swap '0'
	option anon_mount '0'
	option auto_swap '1'
	option auto_mount '1'
	option delay_root '5'
	option check_fs '0'
	option port_mount '1'

config mount
	option uuid '84173db5-fa99-e35a-95c6-28613cc79ea9'
	option target '/boot'
	option enabled '0'

config mount
	option uuid 'd00dfaa3-a1ce8d9e-99754d0b-b3f80430'
	option target '/rom'
	option enabled '0'

config mount
	option uuid 'd4256811-4edf-42a2-aa2e-579cad74e9e3'
	option target '/overlay'
	option enabled '0'
```

使用 `lsblk -o name,uuid` 可以查看各个分区对应的 `uuid`：

![](/blog/img/openwrt常用操作_image_2.png)

之后可以在 `/etc/config/fstab` 文件中配置自动挂载，下面的示例对 `sdb1` 和 `sdb2` 两个分区进行了配置，之后重启路游器两个分区就可自动挂载上去了。

```
config mount
        option uuid 'cfe22461-7489-4d08-a47f-9421defb6044'
        option target '/mnt/disk/data'
        option enabled 1

config mount
        option uuid '31da9653-be98-4a43-805d-c95f233e4586'
        option target '/mnt/disk/app'
        option enabled 1
```
