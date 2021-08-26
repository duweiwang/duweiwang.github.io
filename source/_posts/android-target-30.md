---
title: Android-11适配
date: 2021-08-26 20:27:25
tags:
- Android
categories: Android
description: Android-11,target-sdk 30 变更及适配指南
---

#### 一、变更概览

##### 1、隐私
+ 分区存储
+ 自动释放权限
+ 后台定位
+ 包可见性

##### 2、安全
+ [堆指针添加标记](https://source.android.com/devices/tech/debug/tagged-pointers)
+ Toast
    - 🚫禁止后台弹出含自定义View的Toast
    - Toast支持显示/隐藏回调
    - 部分View相关API调整

##### 3、连接
+ 限制对APN数据库（Access Point Names）的读写

##### 4、Accessibility
+ 文本转语音（TTS）和语音识别 需要声明<queries>
+ 在res/raw下声明对accessibility的使用

##### 5、相机
+ 以下Action默认只能调起系统相机，除非指定包名
  - android.media.action.VIDEO_CAPTURE
  - android.media.action.IMAGE_CAPTURE
  - android.media.action.IMAGE_CAPTURE_SECURE
  
##### 6、应用打包和安装
+ 🚫禁止压缩resources.arsc
+ 必须使用V2及以上版本进行apk签名
##### 7、Firebase
+ target-30的apk在6.0及以上设备，不能使用Firebase JobDispatcher 和 GCMNetworkManager

##### 8、allowBackup属性调整

##### 9、修改SP的OnSharedPreferenceChangeListener回调

##### 10、非SDK接口调用黑名单

#### 二、变更详情
##### 1、分区存储
+ 针对特殊类型应用(文件管理器、文件备份、文件加密)<br>
这种类型的应用需要大规模的进行文件操作，迁移成本较高，推荐使用新的存储管理权限 MANAGE_EXTERNAL_STORAGE （参考1:此权限的授权范围）

    - 优点：<br>
        1、开启后直接拥有所有读写权限（包括拔插的SD卡，USB等）<br>
        2、清缓存不会丢失授权<br>
        3、处理成本较低，只需兼容旧的动态权限申请部分的逻辑。<br>
    - 缺点：<br>
        1、需要跳转设置页面开启权限，有损产品数据（轮询权限，强制拉回）<br>
        2、需要提交GP申请表单，审核通过才能上架<br>
    - 实践：<br>
        1、判断权限 Environment.isExternalStorageManager() <br>
        2、跳转设置页面引导用户开启权限的方式：注意下面两个**Action**的区别<br>
            Settings.ACTION_MANAGE_APP_ALL_FILES_ACCESS_PERMISSION<br>
            Settings.ACTION_MANAGE_ALL_FILES_ACCESS_PERMISSION


+ 针对普通应用  
无法申请新的存储管理权限 MANAGE_EXTERNAL_STORAGE 。只能对旧的API进行改造兼容，将其迁移到存储访问框架SAF（Storage Access Framework）包括Document-API 或 MediaStore-API。
    - 优点： <br>
        1、交互逻辑，和代码逻辑结构比较统一 <br>
        2、提高隐私与安全性，避免杂乱无章的文件系统 <br>
    - 缺点： <br>
        1、迁移成本较高，需要做大量的API兼容，可能影响产品逻辑 <br>
    - 实践： <br>
        1、分区存储文档 <br>
    - 注意事项： <br>
        1、做好数据迁移，避免升级后，原来的数据在不可访问区域 <br>
        2、保留 [requestLegacyExternalStorage=true] 属性 <br>

> 我的应用1.0版本之前数据库存储在外部，为了适配target-30，我在1.1版本，写了数据迁移代码。1.2版本，我做了target-30升级。某用户直接跳过1.1版本升级到了1.2版本。无法运行迁移代码。怎么办！

preserveLegacyExternalStorage 属性：针对升级用户，暂时维持兼容属性，卸载后失效。

```xml
<application
  	android:preserveLegacyExternalStorage="true"/>
```


##### 2、电话号码 <br>
以下两个方法的调用需要申请新的权限： READ_PHONE_NUMBERS <br>
getLine1Number()  
getMsisdn()

##### 3、定位 <br>
+ 增加Only this time 选项
+ 不能一次性申请前后台定位权限，要分两次单独申请
+ 后台位置访问需要跳转设置页面

##### 4、包可见性 <br>
概念： <br>
在Android-10及以前的版本通过 queryIntentActivities、getPackageInfo、getInstalledApplications可以获取所有安装的App。但是在Android-11之后只能获取到一部分已安装应用列表。
解决方案： <br>
+ 通过 <queries> 标签指定需要交互的应用
+ 申请查看所有安装列表
```xml
<manifest package="com.example.game">
  <queries>
    <!-- 需要和你做交互的app -->
    <package android:name="com.example.store" />
    <package android:name="com.example.service" />
    <!-- 你需要查询的Intent -->
    <intent>
      <action android:name="android.intent.action.SEND" />
      <data android:mimeType="image/jpeg" />
    </intent>
  </queries>
  ...
</manifest>
```

```xml
<uses-permission android:name="android.permission.QUERY_ALL_PACKAGES"/>
```

#### 三、参考文献

1、[官方文档-behavior-changes-11](https://developer.android.com/about/versions/11/behavior-changes-11#change-details) <br>
2、[GP政策-MANAGE_EXTERNAL_STORAGE](https://support.google.com/googleplay/android-developer/answer/10467955) <br>
3、[存储权限-管理所有文件](https://developer.android.com/training/data-storage/manage-all-files) <br>
4、[GP政策-all-packages-permission](https://support.google.com/googleplay/android-developer/answer/10158779?hl=en#zippy=%2Cpermitted-uses-of-the-query-all-packages-permission) <br> 
