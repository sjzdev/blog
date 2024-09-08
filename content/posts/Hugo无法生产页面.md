---
blog: true
tags:
  - hugo
categories:
  - hugo
collections: 
title: Hugo无法生产页面
date: 2024-03-31 00:32:28
update: 2024-09-08 22:32:55
---

## 问题描述

当博客的 `date` 属性设置为当天（2023-11-19）时，发布时无法生产对应的页面，把时间需改为前一天（2023-11-18）则可以正常发布：

```yml
share: true
tags:
  - alist
title: window挂载webdav
date: 2023-11-19 12:54:37
update: 2023-11-19 13:07:37
```

## 原因

Hugo 在生成静态页面的时候，不会生成超过当前时间的文章，默认情况下 Hugo 采用的是格林尼治时间（UTC），比北京时间 (UTC+8) 晚了 8 个小时，所以当 `date` 为 `2023-11-19 12:54:37` 时，在 `2023-11-19 20:54:37` 之后才能被发布。

## 解决方案

有两种方式解决该问题：

- 为自己文章/页面的 date 加上时区
	```yml
	date: 2019-05-12
	timezone: UTC+8
	```
- 配置 Hugo 使其输出将来的页面
	```yaml
	buildFuture: true
	```
