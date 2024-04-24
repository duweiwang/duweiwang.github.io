---
title: Android性能优化
tags:
---

#### 性能介绍：


#### 性能分析：


#### 性能优化开源项目：

+ KOOM：
  + Java Heap泄露监控（周期性查询Java堆内存、线程数、文件描述符数等资源占用情况）
  + Native Heap泄露监控
  + Thread泄露监控（hook pthread_create/pthread_exit）

+ Matrix
  + APK Checker
  + Resource Canary
  + Trace Canary
  + SQLite Lint
  + Battery Canary
  + Memory Hook：Native内存泄漏检测
  + Pthread Hook：检测 Android Java 和 native 线程泄漏及缩减 native 线程栈空间的工具
    + 对 native 线程的默认栈大小进行减半降低线程带来的虚拟内存开销，在 32 位环境下可缓解虚拟内存不足导致的崩溃问题
  + WVPreAllocHook：
    + 一个用于安全释放 WebView 预分配内存以在不加载 WebView 时节省虚拟内存的工具，在 32 位环境下可缓解虚拟内存不足导致的崩溃问题
  + MemGuard：
  + IO Canary：


+ BlockCanary
+ [ArgusApm](https://github.com/Qihoo360/ArgusAPM)
    交互分析：分析Activity生命周期耗时，帮助提升页面打开速度，优化用户UI体验
    网络请求分析：监控流量使用情况，发现并定位各种网络问题
    内存分析：全面监控内存使用情况，降低内存占用
    进程监控：针对多进程应用，统计进程启动情况，发现启动异常（耗电、存活率等）
    文件监控：监控APP私有文件大小/变化，避免私有文件过大导致的卡顿、存储空间占用等问题
    卡顿分析：监控并发现卡顿原因，代码堆栈精准定位问题，解决明显的卡顿体验
    ANR分析：捕获ANR异常，解决APP的“未响应”问题
