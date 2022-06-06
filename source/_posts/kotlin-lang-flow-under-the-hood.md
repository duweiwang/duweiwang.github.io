---
title: Kotlin Flow的工作原理
tags: Kotlin
date: 2022-05-25 11:08:23
categories: 计算机
description:
---

---------

我们有一个会被重复调用的lambda表达式：

```kotlin
import kotlin.*

fun main() {
    val f: () -> Unit = {
        print("A")
        print("B")
        print("C")
    }
    f() // ABC
    f() // ABC
}
```

我们给这个lambda加个参数：`(String) -> Unit` ,因为参数是个函数，为了能调用到函数，我们把它叫做`emit`吧：

```kotlin
import kotlin.*

fun main() {
    val f: ((String) -> Unit) -> Unit = { emit -> // 1
        emit("A")
        emit("B")
        emit("C")
    }
    f { print(it) } // ABC
    f { print(it) } // ABC
}
```

添加的参数看起来有点乱🤔，稍微修改一下，抽出来吧：

```kotlin
import kotlin.*

fun interface FlowCollector {
    fun emit(value: String)
}

fun main() {
    val f: (FlowCollector) -> Unit = {
        it.emit("A")
        it.emit("B")
        it.emit("C")
    }
    f { print(it) } // ABC
    f { print(it) } // ABC
}
```

好像能把`it`的调用逻辑去掉，这样更简洁：

```kotlin
import kotlin.*

fun interface FlowCollector {
    fun emit(value: String)
}

fun main() {
    val f: FlowCollector.() -> Unit = {
        emit("A")
        emit("B")
        emit("C")
    }
    f { print(it) } // ABC
    f { print(it) } // ABC
}
```

调用lambda表达式不是很方便。如果把它抽成接口🤔

```kotlin
import kotlin.*

fun interface FlowCollector {
    fun emit(value: String)
}

interface Flow {
    fun collect(collector: FlowCollector)
}

fun main() {
    val builder: FlowCollector.() -> Unit = {
        emit("A")
        emit("B")
        emit("C")
    }
    val flow: Flow = object : Flow {
        override fun collect(collector: FlowCollector) {
            collector.builder()
        }
    }
    flow.collect { print(it) } // ABC
    flow.collect { print(it) } // ABC
}
```

抽成可以重复调用的构建器：

```kotlin
import kotlin.*

fun interface FlowCollector {
    fun emit(value: String)
}

interface Flow {
    fun collect(collector: FlowCollector)
}

fun flow(builder: FlowCollector.() -> Unit) = object : Flow {
    override fun collect(collector: FlowCollector) {
        collector.builder()
    }
}

fun main() {
    val f: Flow = flow {
        emit("A")
        emit("B")
        emit("C")
    }
    f.collect { print(it) } // ABC
    f.collect { print(it) } // ABC
}
```

泛型化参数更通用：

```kotlin
import kotlin.*

fun interface FlowCollector<T> {
    suspend fun emit(value: T)
}

interface Flow<T> {
    suspend fun collect(collector: FlowCollector<T>)
}

fun <T> flow(builder: suspend FlowCollector<T>.() -> Unit) = object : Flow<T> {
    override suspend fun collect(collector: FlowCollector<T>) {
        collector.builder()
    }
}

suspend fun main() {
    val f: Flow<String> = flow {
        emit("A")
        emit("B")
        emit("C")
    }
    f.collect { print(it) } // ABC
    f.collect { print(it) } // ABC
}
```


[参考](https://kt.academy/article/how-flow-works)