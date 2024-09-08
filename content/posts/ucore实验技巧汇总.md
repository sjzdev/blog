---
blog: true
tags:
  - gdb
  - 调试
  - ucore实验
categories:
  - ucore实验
collections: 
title: ucore实验技巧汇总
date: 2022-04-25 00:32:28
update: 2024-09-08 21:41:02
---

### gdb tui 界面

使用 ide 工具进行 debug 的时候可以很明显的看到当前执行的是源码的哪个文件中的哪行代码，`gdb` 提供的 `tui` 界面可以实现类型的功能，只需要执行命令的时候带上 `-tui` 参数。

![](/blog/img/IMG-20240908212558380.png)

下面是做 `ucore` 实验时，执行 `gdb -x gdbinit -tui` 命令后打开的一个 `gdb` 的命令行界面：

![](/blog/img/IMG-20240908212558936.png)