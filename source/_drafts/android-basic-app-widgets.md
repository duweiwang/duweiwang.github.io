---
title: Android的小部件
tags:
- Android
---------


####  一、概览：
1.1 分类
+ 信息展示小部件：天气小部件主要展示天气信息
+ 集合小部件：多个相同类型元素的集合，相册预览小部件、文章预览小部件，可垂直滚动。
+ 控制类小部件：常用功能的开关等
+ 混合类型小部件：音乐播放小部件既有控制操作也能展示信息


1.2 和谷歌助手整合
    通过相关集成，上述小部件可以通过语音指令添加。

1.3 小部件的局限性
+ 手势
    由于小部件在主屏幕展示，需要和导航共存，所以左右滑动只能交给导航来处理。小部件只能处理触摸和垂直滑动。
+ 元素
    由于上述手势交互的约束，一些UI元素就不能使用了。

1.4 设计指引
+ 小部件内容
    小部件是吸引用户进入app的绝佳机制。


+ 小部件导航
    将用户导航到常用功能，快速完成用户的意图。

+ 小部件的尺寸
    长按并放手进入大小调节模式。通过尺寸变化，控制信息的展示量。

+ 布局注意事项
    根据设备网格的分辨率来摆放布局。记住以下几点：

- Planning your widget resizing strategy across "size buckets" rather than variable grid dimensions gives you the most reliable results
- The number, size, and spacing of cells can vary widely from device to device. Hence, it is very important that your widget is flexible and can accommodate more or less space than anticipated
- In fact, as the user resizes a widget, the system responds with a dp size range in which your widget can redraw itself.
- Android-12后，你能更精确的指定尺寸和灵活布局：
  - 略
  - 略
  - 略
  - 略


1.5 小部件设计清单
+ 瞄一眼的信息放在小部件上，详细信息放进app
+ 根据需求选择合适小部件类型
+ 内容和尺寸的适配
+ 确保布局能够伸展来保证设备无关
+ 思考是否需要额外的配置


####  二、创建简单的小部件

2.1 小部件元器件
+ `AppWidgetProviderInfo`
    描述小部件元信息，布局、更新频率等
+ `AppWidgetProvider`
    定义交互的编程接口。


<center>
    <img src="../images/android-appwidget-overview.jpeg" width="100%"/>
</center>

除了基本的元器件，如果你的小部件需要用户配置信息，你需要提供配置修改界面让用户修改配置，如钟表小部件的时区修改。
+ Android-12起，提供默认配置，之后用户再修改。
+ Android-11及以前，每次添加小部件都启动配置页。


2.2 声明`AppWidgetProviderInfo`

在res/xml/声明：
```xml
<appwidget-provider xmlns:android="http://schemas.android.com/apk/res/android"
    android:minWidth="40dp"
    android:minHeight="40dp"
    android:targetCellWidth="1"
    android:targetCellHeight="1"
    android:maxResizeWidth="250dp"
    android:maxResizeHeight="120dp"
    android:updatePeriodMillis="86400000"
    android:description="@string/example_appwidget_description"
    android:previewLayout="@layout/example_appwidget_preview"
    android:initialLayout="@layout/example_loading_appwidget"
    android:configure="com.example.android.ExampleAppWidgetConfigurationActivity"
    android:resizeMode="horizontal|vertical"
    android:widgetCategory="home_screen"
    android:widgetFeatures="reconfigurable|configuration_optional">
</appwidget-provider>

```

尺寸属性:













