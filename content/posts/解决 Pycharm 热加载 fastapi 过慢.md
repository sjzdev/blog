---
blog: true
tags:
  - 编程中的那些问题
  - python
categories:
  - 编程中的那些问题
collections: 
title: 解决 Pycharm 热加载 fastapi 过慢
date: 2024-07-13 19:23:14
update: 2024-08-25 16:00:02
---

## 问题描述

使用 pycharm 启动 fastapi 后，热加载时卡在如下位置很久：

![](img/IMG-20240816224818361.png)

## 解决方案

根据网络上的资料 [^1][^2] 可知，这是由于 Pycharm 的 bug 导致的，可通过降低 uvicorn 的版本解决，目前测试 0.20.0 可行。

[^1]: [fastapi使用uvicorn重载较慢问题_fastapi 热重载-CSDN博客](https://blog.csdn.net/qq_25894535/article/details/135763895)
[^2]: [pycharm使用fastapi/uvicorn无法reload的问题 - aminor - 博客园 (cnblogs.com)](https://www.cnblogs.com/aminor/p/17764109.html)
