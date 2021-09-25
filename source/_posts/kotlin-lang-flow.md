---
title: Kotlin Flow 基础
date: 2021-09-25 15:27:31
tags: Kotlin
categories: Kotlin
description:
---

#### 一、什么是Flow

##### 1、如何通过同步和异步方式返回多个值？

1.1 同步方式返回多个值

```kotlin
fun simple(): List<Int> = listOf(1, 2, 3)
 
fun main() {
    simple().forEach { value -> println(value) } 
}
```
1.2 同步方式**依次**返回多个值

```kotlin
fun simple(): Sequence<Int> = sequence { // sequence builder
    for (i in 1..3) {
        Thread.sleep(1000) // pretend we are computing it
        yield(i) // yield next value
    }
}

fun main() {
    simple().forEach { value -> println("time = ${System.currentTimeMillis()} ,$value") } 
}
```


1.3 异步方式返回多个值

```kotlin
suspend fun simple(): List<Int> {
    delay(1000) // pretend we are doing something asynchronous here
    return listOf(1, 2, 3)
}

fun main() = runBlocking<Unit> {
    simple().forEach { value -> println(value) } 
}
```

1.4 如何通过异步的方式**依次**返回多个值呢？

```kotlin
fun simple(): Flow<Int> = flow { // flow builder
    for (i in 1..3) {
        delay(100) // pretend we are doing something useful here
        emit(i) // emit next value
    }
}

fun main() = runBlocking<Unit> {
    // Launch a concurrent coroutine to check if the main thread is blocked
    launch {
        for (k in 1..3) {
            println("I'm not blocked $k")
            delay(100)
        }
    }
    // Collect the flow
    simple().collect { value -> println(value) } 
}
```

**通过flow，我们可以实现异步的数据流。**


##### 2、Flow是冷数据流

冷数据流：每次调用`collect`函数时都会触发执行

```kotlin
fun simple(): Flow<Int> = flow { 
    println("Flow started")
    for (i in 1..3) {
        delay(100)
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
    println("Calling simple function...")
    val flow = simple()
    println("Calling collect...")
    flow.collect { value -> println(value) } 
    println("Calling collect again...")
    flow.collect { value -> println(value) } 
}
```

##### 3、Flow的取消
```kotlin
fun simple(): Flow<Int> = flow { 
    for (i in 1..3) {
        delay(100)          
        println("Emitting $i")
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
    withTimeoutOrNull(250) { // Timeout after 250ms 
        simple().collect { value -> println(value) } 
    }
    println("Done")
}
```


##### 4、如何构建Flow？

+ `flow{ ... }`
+ `flowOf`
+ `.asFlow()`


##### 5、Flow操作符

5.1 中间操作符
+ map
+ filter
+ transform
+ take

5.2 终止操作符
+ collect
+ toList / toSet
+ first / single
+ reduce / fold

5.3 `buffer`当流的处理速度慢于发射速度时，通过buffer提前缓存可以提高效率： 

优化前：耗时1200ms
```kotlin
fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100) // pretend we are asynchronously waiting 100 ms
        emit(i) // emit next value
    }
}

fun main() = runBlocking<Unit> { 
    val time = measureTimeMillis {
        simple().collect { value -> 
            delay(300) // pretend we are processing it for 300 ms
            println(value) 
        } 
    }   
    println("Collected in $time ms")
}
```
优化后：耗时1000ms
````kotlin
val time = measureTimeMillis {
    simple()
        .buffer() // buffer emissions, don't wait
        .collect { value -> 
            delay(300) // pretend we are processing it for 300 ms
            println(value) 
        } 
}   
println("Collected in $time ms")
````

5.4 `conflate`当生产速度大于消费速度，忽略未来得及处理的值

```kotlin
val time = measureTimeMillis {
    simple()
        .conflate() // conflate emissions, don't process each one
        .collect { value -> 
            delay(300) // pretend we are processing it for 300 ms
            println(value) 
        } 
}   
println("Collected in $time ms")
//输出：
//1
//3
//Collected in 758 ms
```

5.5 `collectLatest` 如果消费者很慢，取消它，新值过来时再重新启动。
```kotlin
val time = measureTimeMillis {
    simple()
        .collectLatest { value -> // cancel & restart on the latest value
            println("Collecting $value") 
            delay(300) // pretend we are processing it for 300 ms
            println("Done $value") 
        } 
}   
println("Collected in $time ms")
//output:
//Collecting 1
//Collecting 2
//Collecting 3
//Done 3
//Collected in 741 ms
```

5.6 多个flow合并：zip / combine  

使用`zip`操作符
```kotlin
val nums = (1..3).asFlow() // numbers 1..3
val strs = flowOf("one", "two", "three") // strings 
nums.zip(strs) { a, b -> "$a -> $b" } // compose a single string
    .collect { println(it) } // collect and print
//output:
//1 -> one
//2 -> two
//3 -> three
```

当两个flow不同步时：

```kotlin
//速度不同步时，使用zip，到达同步点才输出
val nums = (1..3).asFlow().onEach { delay(300) } // numbers 1..3 every 300 ms
val strs = flowOf("one", "two", "three").onEach { delay(400) } // strings every 400 ms
val startTime = System.currentTimeMillis() // remember the start time 
nums.zip(strs) { a, b -> "$a -> $b" } // compose a single string with "zip"
    .collect { value -> // collect and print 
        println("$value at ${System.currentTimeMillis() - startTime} ms from start") 
    } 
//output:
//1 -> one at 428 ms from start
//2 -> two at 828 ms from start
//3 -> three at 1230 ms from start
```
```kotlin
////速度不同步时，使用combine，有值到达即输出，无视同步
val nums = (1..3).asFlow().onEach { delay(300) } // numbers 1..3 every 300 ms
val strs = flowOf("one", "two", "three").onEach { delay(400) } // strings every 400 ms          
val startTime = System.currentTimeMillis() // remember the start time 
nums.combine(strs) { a, b -> "$a -> $b" } // compose a single string with "combine"
    .collect { value -> // collect and print 
        println("$value at ${System.currentTimeMillis() - startTime} ms from start") 
    } 
//output:
//1 -> one at 452 ms from start
//2 -> one at 651 ms from start
//2 -> two at 854 ms from start
//3 -> two at 952 ms from start
//3 -> three at 1256 ms from start
```


参考文档：  
[ 1.官方文档](https://kotlinlang.org/docs/flow.html)