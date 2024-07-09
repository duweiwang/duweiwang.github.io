---
title: android jetpack startup
date: 2024-07-09 15:04:39
tags:
categories:
description:
---



#### 概述
[源码](https://cs.android.com/androidx/platform/frameworks/support/+/androidx-main:startup/)

startup库是一个极简启动优化库，可以解决启动中的依赖顺序问题。

#### 基本使用

1.1 实现Initializer接口

```kotlin
class LoggerInitializer : Initializer<Logger> {
    override fun create(context: Context): Logger {
        Logger.init(context)
        return Logger
    }

    override fun dependencies(): List<Class<out Initializer<*>>> {
        return emptyList()
    }
}
```

1.2 AndroidManifest.xml中声明

```xml
<provider>
    android:name="androidx.startup.InitializationProvider"
    android:authorities="${applicationId}.androidx-startup"
    android:exported="false"
    tools:node="merge">

    <meta-data
        android:name="me.xxx.demo.LoggerInitializer"
        android:value="androidx.startup"/>
</provider>
```


#### 源码分析
1、默认由ContentProvider onCreate中获取AppInitializer**单例**
2、调用AppInitializer单例的discoverAndInitialize，**解析Manifest**里面的mate
3、解析完后放入Set<Class<? extends Initializer<?>>>中
4、遍历Set，调用doInitialize进行初始化
    4.1 检查循环依赖问题
    4.2 找到此项的所有依赖项，递归调用doInitialize
    4.3 此项依赖初始化完成


#### 缺点
+ 不支持线程控制和等待

#### 其他相关库
[android-startup](https://github.com/idisfkj/android-startup)