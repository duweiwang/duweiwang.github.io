---
title: Jetpack Compose Side Effects
date: 2022-05-17 16:11:13
tags:
- Android
- Compose
categories: 计算机
description:
---


#### 一、什么是Side-Effect

> [wiki](https://en.wikipedia.org/wiki/Side_effect_(computer_science)):在计算机科学中，当函数、表达式、操作会修改它作用域之外的状态变量值时，我们就说它具有副作用。也就是说除返回值之外其他的对外修改都叫副作用。
> 常见的例子包括：修改非本地变量、修改参数传进来的可变引用、执行I/O等。

####  二、Compose中的Side-Effect

因为Jetpack Compose是由一系列的`@Composable`函数组成的，`@Composable`函数在执行时可能会被跳过（出于优化的角度）或执行多次（recomposition）。  
那对于函数内部的网络请求、对外部状态的修改怎么办？有可能执行多次？有可能不执行？这就产生了某种不可预期的错误状态，或者泄露。
为了解决此类问题，Compose提供了相应的API，这些API主要聚焦于对这些副作用的生命周期进行管理。

这些API可以分成两大类：
+ SuspendedEffect
  + rememberCoroutineScope
  + launchedEffect
+ Non-Suspended Side Effects
  + DisposableEffect
  + SideEffect


#### 三、SuspendedEffect


#####  3.1 LaunchedEffect

在首次composition的时候被调用。recomposition时不会再次调用。可以通过改变Key让其重新调用。  
它是一个协程作用域，我们可以执行一些挂起函数，当composable函数退出时，协程会被取消。

下面的例子中，启动即执行循环逻辑，每秒修改一次状态值并触发recomposition，但recomposition并不影响LaunchedEffect内的逻辑。

```kotlin
@Composable
fun LaunchedEffect() {
    var timer by remember { mutableStateOf(0) }
    Box(modifier = Modifier.fillMaxSize(), 
        contentAlignment = Alignment.Center) {
        Text("Time $timer")
    }

    LaunchedEffect(key1 = Unit) {
        while (true) {
            delay(1000)
            timer++
        }
    }
}
```

##### 3.2 RememberCoroutineScope
`LaunchedEffect`的启动和取消跟随Composable函数的生命周期。如果想自己控制生命周期可以使用`RememberCoroutineScope`

下面的例子中，协程的控制权转移到我们自己手中：

```kotlin
@Composable
fun JustRememberCoroutineScope() {
    val scope = rememberCoroutineScope()
    var timer by remember { mutableStateOf(0) }
    var timerStartStop by remember { mutableStateOf(false) }
    var job: Job? by remember { mutableStateOf(null) }

    Box(modifier = Modifier.fillMaxSize(),
        contentAlignment = Alignment.Center) {
        Column(horizontalAlignment = Alignment.CenterHorizontally) {
            Text("Time $timer")
            Button(onClick = {
                timerStartStop = !timerStartStop

                if (timerStartStop) {
                    job?.cancel()
                    job = scope.launch {
                        while (true) {
                            delay(1000)
                            timer++
                        }
                    }
                } else {
                    job?.cancel()
                }

            }) {
                Text(if (timerStartStop) "Stop" else "Start")
            }
        }
    }
}
```

#### 四、Non-Suspended Side Effects

##### 4.1 DisposableEffect
与`LaunchedEffect`一样，立即启动，通过修改key可以再次启动。它提供了一个销毁时的回调，当再次启动时，前一个的`onDispose`方法会被回调。


下面的例子每次点击按钮改变状态进行recompose，同时会改变`DisposableEffect`的key，销毁上一个Effect并收到回调。
```kotlin
@Composable
fun JustDisposableEffect() {
    var timerStartStop by remember { mutableStateOf(false) }
    Box(modifier = Modifier.fillMaxSize(),
        contentAlignment = Alignment.Center) {
        Column(horizontalAlignment = Alignment.CenterHorizontally) {
            Button(onClick = {
                timerStartStop = !timerStartStop
            }) {
                Text(if (timerStartStop) "Stop" else "Start")
            }
        }
    }

    val context = LocalContext.current

    DisposableEffect(key1 = timerStartStop) {
        val x = (1..10).random()
        Toast.makeText(context, "Start $x", LENGTH_SHORT).show()

        onDispose {
            Toast.makeText(context, "Stop $x", LENGTH_SHORT).show()
        }
    }
}
```

##### 4.2 SideEffect

使用场景：
+ 每次composition / recomposition成功后被调用
+ 用于对外部状态的更新
+ 无需做清理回收操作时

例1：对外部状态的更新
```kotlin
@Composable
fun MyScreen(drawerTouchHandler: TouchHandler) {
  val drawerState = rememberDrawerState(DrawerValue.Closed)

  SideEffect {
    drawerTouchHandler.enabled = drawerState.isOpen
  }

  // ...
}
```

例2：
```kotlin
var i = 0
@Composable
fun MyComposable(){
    SideEffect { //this will handle the side effect that may occur
        i++
    }
    Button(onClick = {}){
        Text(text = "Click")
    }
}
```


#### 五、参考

[Jetpack Compose Effect Handlers](https://jorgecastillo.dev/jetpack-compose-effect-handlers)
[SideEffects and Effects Handling in Jetpack Compose](https://www.section.io/engineering-education/side-effects-and-effects-handling-in-jetpack-compose/)
[Jetpack Compose Side Effects Made Easy](https://medium.com/mobile-app-development-publication/jetpack-compose-side-effects-made-easy-a4867f876928)