---
title: Android之Shizuku库
date: 2025-01-03 16:06:05
tags:
categories:
description:
---



#### Shizuku是什么

一个可以让其他App，通过Shizuku来使用具有 adb/root 权限的系统 API的服务。


#### 项目地址：

https://github.com/RikkaApps/Shizuku


#### 启动Shizuku的三种方式

1、 通过无线调试启动

+ 开启无线调试：开发者选项 -> 无线调试 -> 配对码配对 -> 将配对码输入通知栏
+ 启动服务

2、通过ADB启动

```shell
adb shell sh /storage/emulated/0/Android/data/moe.shizuku.privileged.api/start.sh
```
3、 Root设备直接启动


#### 启动后

+ 执行命令查看和Shizuku相关的进程：
```shell
adb shell ps | grep shizuku
```

```shell
u0_a1858       815  1289   14842600 218968 0                   0 S moe.shizuku.privileged.api
shell         2049     1   13778256 132868 do_epoll_wait       0 S shizuku_server
```


#### 实现原理

Shizuku的代码库包含3个部分
+ Shizuku API
+ Shizuku App（manager）
+ Server

其他App接入Shizuku API后访问提权的系统API。
Shizuku App引导用户启动Server。
Server作为中介帮其他应用提供服务API中专。


由于Server通过上面的三种方式启动，因此Server具有较高的权限。可以访问一些限制的系统API。
