---
blog: true
tags: 
categories: 
collections: 
title: Flutter开发环境搭建
date: 2024-09-21 20:46:04
update: 2024-11-01 21:44:55
---

> [!note]
> 所有涉及环境变量的配置，如果没有立即生效的话，需重启操作系统

## IDEA 配置

### 配置 Android sdk
1. 在设置 -->语言和框架 -->Android SDK 中配置安装位置，选择安装 SDK；
	![](/blog/img/IMG-20241101214529351.png)
	安装成功后，可以继续选择安装其他的平台和工具：
	![](/blog/img/IMG-20241101214529454.png)
3. 安装 Command-line 和 platform-tools：
	![](/blog/img/IMG-20241101214529507.png)
4. 将 Android SDK 的安装路径配置到 PATH 环境变量中：
	![](/blog/img/IMG-20241101214529570.png)
	![](/blog/img/IMG-20241101214529624.png)

### 插件安装

设置 -->插件中搜索安装 Dart 和 Flutter 两个插件：

![](/blog/img/IMG-20241101214529693.png)

## Flutter SDK 安装

### 前置条件

- 64-bit version of Microsoft Windows 10 或更高版本
-  Windows PowerShell 5.0 或更高版本
- Git for Windows 2.27 或更高的版本
### 下载 Flutter SDK

1. 下载以下 Flutter SDK 最新 stable 版本的压缩包：`https://mirrors.tuna.tsinghua.edu.cn/flutter/flutter_infra/releases/stable/windows/`；
2. 将压缩包解压并移动至想要安装的位置，需要注意的是安装路径不要包含特殊字符或空格或需要较高的权限（如 `C:\Program Files`）；
### 配置环境变量

1. 配置 Flutter 的 PATH 环境变量，目的是能够在终端中使用 `flutter` 命令；
2. Flutter 开发依赖于 SDK 的升级和 Dart Package 生态，因为中国政府设置了防火墙，在使用时可能会遇到阻碍，所以需要设置两个环境变量：
	```
	FLUTTER_STORAGE_BASE_URL="https://mirrors.tuna.tsinghua.edu.cn/flutter"
	PUB_HOSTED_URL="https://mirrors.tuna.tsinghua.edu.cn/dart-pub"
	```
3. 之后执行 `flutter doctor` 命令检查我们的环境是否配置成功；
	![](/blog/img/IMG-20241101214529755.png)
### flutter doctor 问题解决

执行 `flutter doctor` 命令的检查结果中可能会出现一些问题，接下来我们解决这些问题。

**问题 1：Unable to locate Android SDK**

这表示找不到 SDK，大概率是环境变量配置有误，可以重新配置环境变量（需要重启电脑配置的环境变量才会生效）。

**问题 2：Android SDK file not found: adb**

这表示找不到 adb 命令，通上一个问题一样，大概率是环境变量配置有误，重新配置环境变量即可。

**问题 3：Visual Studio not installed**

这表示没有安装 Visual Studio，解决这个问题只要安装上 Visual Studio 就可以了，如果不开发 Window 桌面应用的话，可以不关注这个问题。

**问题 4：A network error occurred while checking "https://maven.google.com/"**

这表是当前网络无法连接 `https://maven.google.com/`，这是由于伟大的防火墙造成的，可以翻墙解决。

## 创建项目

1. IDEA 新建 Flutter 项目：
	![](/blog/img/IMG-20241101214529810.png)
	![](/blog/img/IMG-20241101214529854.png)
2. 启动运行可以看到结果：
	![](/blog/img/IMG-20241101214529905.png)


## 连接真机

### 连接流程

1. 使用数据线连接手机和电脑；
2. 开始手机 USB 调试功能，并且允许 USB 安装；
	![](/blog/img/IMG-20241101214529960.jpg)
	![](/blog/img/IMG-20241101214530009.jpg)
3. 之后 IDEA 会自动连接到手机，可以在选项中看到；
	![](/blog/img/IMG-20241101214530098.png)
4. 启动程序，过一段时间就会在手机上看到允许安装的请求，允许安装后就可以在手机运行程序了：
	![](/blog/img/IMG-20241101214530153.jpg)

### 问题解决

**问题 1：启动程序后卡在如下位置**

![](/blog/img/IMG-20241101214530195.png)

这个问题大概率是 Gradle 下载不下来引起的，可以在启动配置中添加 `-v` 参数，打印出来的更详细的日志：

![](/blog/img/IMG-20241101214530238.png)

![](/blog/img/IMG-20241101214530287.png)

解决方案是替换 gradle 的下载地址，可以使用腾讯提供的服务 (`https://mirrors.cloud.tencent.com/gradle`)，选择对应的版本替换链接，将如下文件里的 `distributionUrl` 配置替换掉即可。

![](/blog/img/IMG-20241101214530334.png)

![](/blog/img/IMG-20241101214530371.png)