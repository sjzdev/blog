---
blog: true
tags:
  - ubuntu
  - 远程桌面
  - 编程中的那些问题
categories: 
collections: 编程中的那些问题
title: Authentication required. System policy prevents WiFi scans 问题的解决
date: 2024-08-01 23:41:21
update: 2024-08-31 19:09:24
---

## 问题描述

通过远程桌面连接到 ubuntu 后，持续弹出窗口要求输入密码，窗口提示信息为：Authentication required. System policy prevents WiFi scans。

## 解决方案

编辑或新建文件：`/etc/polkit-1/localauthority/50-local.d/wifi.scan.pkla`，输入如下内容保存，重启操作系统即可。

```
[Allow Wifi Scan]
Identity=unix-user:*
Action=org.freedesktop.NetworkManager.wifi.scan;org.freedesktop.NetworkManager.enable-disable-wifi;org.freedesktop.NetworkManager.settings.modify.own;org.freedesktop.NetworkManager.settings.modify.system;org.freedesktop.NetworkManager.network-control
ResultAny=yes
ResultInactive=yes
ResultActive=yes
```
