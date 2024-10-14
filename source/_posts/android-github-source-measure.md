---
title: Measure源码分析
date: 2024-09-24 15:42:26
tags:
  - Android
  - 安卓框架
categories: 计算机
description:
---


##### 概述

[Measure](https://github.com/measure-sh/measure.git)是一个开源的性能监控项目，它包含：Android、iOS、前端、后端。这里分析一下Android端的逻辑实现。


##### 分析
项目的基本结构：
+ measure-android
  + measure
  + measure-android-gradle
  + sample

+ measure模块的内容：
  + ANR捕获：Native实现，注册信号处理器，捕获SIGQUIT信号
  + AppLaunch:区分冷起、热起、温启动，并对外回调。
  + AppExit:ActivityManager#getHistoricalProcessExitReasons的使用
  + performance:
    + cpu usage
    + memory usage
  + NetworkChange:网络变化的监控
  + ScreenShot：生成屏幕截图，PixelCopy的使用，或者Canvas画上去
  + 其他模块：周期性的心跳包（Heartbeat）去执行任务。Okhttp拦截器。

+ measure-android-gradle
  + Manifest数据读取，AppId、VersionCode、VersionName
  + Mapping文件上传
  + Apk、Aab大小信息上传


##### 总结：
1、冷热起的判断逻辑，这部分代码参考自Square的[PaPa](https://github.com/square/papa/)有参考价值
2、各功能的对外调用都是接口化的，无侵入性
3、项目简单，阅读起来很顺。
4、包含前后端代码，看板，值得学习。