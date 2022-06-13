---
title: coroutine-pattern-antipattern
tags: Kotlin
------------
https://proandroiddev.com/kotlin-coroutines-patterns-anti-patterns-f9d12984c68e

简介

我决定写一些在使用协程时你需要了解的东西：

#### 使用coroutineScope包裹异步调用或使用SupervisorJob来处理异常

× 如果异步代码块可能抛出异常，不要依赖try-catch块







