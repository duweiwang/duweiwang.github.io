---
title: Jetpack Compose State Management
date: 2022-05-18 14:34:49
tags:
- Android
categories:
- Android
description:
---

--

<center>
    <img src="../images/ui-equals-function-of-state.png" width="100%"/>
</center>

#### 一、什么是状态
> 状态是一些被View、Widget订阅或观察的对象，它包含数据。任何状态（数据）的变化都会通知给观察它的View、Widget。

+ 网络断开时展示的Toast
+ 发布的博客及其评论
+ 点击按钮时的水波纹动画

#### 二、Compose中的状态

##### 2.1 记住状态
Composable函数可以使用`remember`在初始化时存储一个对象到内存中。每次recomposition时这个对象都会被返回。

##### 2.2 可观察状态
`mutableStateOf`函数可以创建一个可观察的可变数据，此数据的变化将触发recomposition

```kotlin
@Composable
fun MySwitch() {
   // I have a State in Compose's MutableState
   val checked = remember { mutableStateOf(false) }
   Switch(
       // Observe the State by accessing the value property
       checked = checked.value,
       onCheckedChange = {
           checked.value = it
       }
   )
}
```

##### 2.3 其他状态类型
除了使用`MutableState<T>`持有状态外，你还可以使用以下这些可观察数据类型来维护状态。
+ LiveData
+ Flow
+ RxJava2

但是，在使用时需要将其转成`State<T>`类型
+ `LiveData<T>.observeAsState()`
+ `Flow<T>.collectAsState()`


##### 2.4 stateless
在Composable函数中保存状态将降低它的可复用性，通过将状态向外抽离来增加其复用性。

```kotlin
@Composable
fun HelloScreen() {
    var name by rememberSaveable { mutableStateOf("") }

    HelloContent(name = name, onNameChange = { name = it })
}

@Composable
fun HelloContent(name: String, onNameChange: (String) -> Unit) {
    Column(modifier = Modifier.padding(16.dp)) {
        Text(
            text = "Hello, $name",
            modifier = Modifier.padding(bottom = 8.dp),
            style = MaterialTheme.typography.h5
        )
        OutlinedTextField(
            value = name,
            onValueChange = onNameChange,
            label = { Text("Name") }
        )
    }
}
```

##### 2.5 状态恢复
当Activity重建时，想要保留状态，并在重建后恢复状态，可以使用`rememberSaveable`将状态保存到Bundle中。

+ Parcelize
+ MapSaver
+ ListSaver

#### 三、参考
[State and Jetpack Compose](https://developer.android.com/jetpack/compose/state#state-in-composables)
[Jetpack Compose State Guideline](https://medium.com/@takahirom/jetpack-compose-state-guideline-494d467b6e76)
[Managing State in Android](https://levelup.gitconnected.com/managing-state-in-android-f4d042646521)
