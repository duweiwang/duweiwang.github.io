---
title: Jetpack Compose 概念
tags: Android
-------------


#####  一、Thinking in Compose


#####  二、状态管理
> 状态：任何可能引起界面变化的变量、参数、事件

要想改变Compose的界面，只能通过修改控制界面的状态（变量、参数），将新状态传给它，让它重新布局绘制。  
这个重新布局绘制的过程称为`recompose`

2.1 State in composables

1、可以使用`remember`在来保存一些和状态相关的数据，`recompose`时这些变量不会丢失。

2、可以使用`mutableStateOf`函数，创建一个可观察的数据集。此数据的变化，会触发`recompose`。

（1 + 2） = 数据变化 -> recompose -> 读取新数据构建界面

2.2 其他可观察数据
+ LiveData
+ Flow
+ ExJava2

2.3 提取状态


2.4 状态恢复


#####  三、生命周期

#####  四、Modifiers
4.1 modifiers事件顺序
4.2 内建modifiers

#####  五、Side-effects
> Side-effects：状态的改变超出作用域范围。

+ LaunchedEffect: run suspend functions in the scope of a composable

`LaunchedEffect`生命周期同外层Composable函数，只在首次启动执行，recomposition不执行。通过修改provided key也能重新执行。


+ rememberCoroutineScope: obtain a composition-aware scope to launch a coroutine outside a composable

`rememberCoroutineScope`也是一个Composable函数，他会返回一个`CoroutineScope`，它也和调用自己的Composable函数绑定。

+ rememberUpdatedState: reference a value in an effect that shouldn't restart if the value changes


+ DisposableEffect: effects that require cleanup

+ SideEffect: publish Compose state to non-compose code

+ produceState: convert non-Compose state into Compose state
+ derivedStateOf: convert one or multiple state objects into another state
+ snapshotFlow: convert Compose's State into Flows

#####  六、Phases

#####  七、架构分层

#####  八、性能

#####  九、语义



参考资料

Jetpack-Compose资料

1、[展示jetpack compose中使用mvvm mvi redux等框架](https://github.com/feiyin0719/compose_architecture)

2、[jetpack compose 开发架构选择探讨（一）](https://juejin.cn/post/6994398933334097934)
[jetpack compose 开发架构选择探讨（二）]()
[jetpack compose 开发架构选择探讨（三）]()

3、[Under the hood of Jetpack Compose — part 2 of 2](https://medium.com/androiddevelopers/under-the-hood-of-jetpack-compose-part-2-of-2-37b2c20c6cdd)

Inside Jetpack Compose：
https://medium.com/@takahirom/inside-jetpack-compose-2e971675e55e
Jetpack Compose · snapshot system：
https://programmer.ink/think/jetpack-compose-snapshot-system.html

https://jorgecastillo.dev/book/

一文看懂 Jetpack Compose 快照系统
https://juejin.cn/post/7095544677515919367#heading-17


flutter：
https://book.flutterchina.club/chapter4/layoutbuilder.html#_4-8-3-flutter-%E7%9A%84-build-%E5%92%8C-layout



