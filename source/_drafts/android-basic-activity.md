---
title: Android四大组件之活动
tags:
---


#### 生命周期



#### 启动模式

Standard：  
这是默认的启动模式。每次启动Activity时，系统都会创建一个新的实例，不管这个实例是否已经存在。


SingleTop：  
如果在任务栈的栈顶已经有这个Activity的实例，就重用这个实例(会调用其onNewIntent方法)，否则就会创建新的实例。


SingleTask：  
如果在任务栈中已经有这个Activity的实例，就重用这个实例(会调用其onNewIntent方法)，并且会清除它上面的所有Activity。如果没有，则创建新的实例。


SingleInstance：  
这种模式下的Activity会单独位于一个任务栈中，系统不会启动其他任何Activity到这个任务栈中，这个模式的Activity只能单独存在。