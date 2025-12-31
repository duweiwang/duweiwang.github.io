---
title: 【翻译】Kotlin 协程 模式与 反模式（Patterns & Anti‑Patterns）
tags: Kotlin
date: 2025-12-31 17:57:01
categories:
description:
---

---------

## 📌 前言

在开始使用 Kotlin 协程之前，我们需要意识到一些应当做和不应当做的事项，特别是在异常处理、作用域管理和并发模式方面。本文总结了多种协程使用中的模式与反模式，帮助你写出更安全、清晰、可维护的协程代码。

---

## 🧠 1. 把 `async` 调用包裹在 `coroutineScope` 或使用 `SupervisorJob` 来处理异常

### ❌ 反模式：在 async 中抛出异常但只用 try/catch 捕获

```kotlin
val job: Job = Job()
val scope = CoroutineScope(Dispatchers.Default + job)

// may throw Exception
fun doWork(): Deferred<String> = scope.async { … }   // (1)

fun loadData() = scope.launch {
    try {
        doWork().await()                               // (2)
    } catch (e: Exception) { … }
}
```
[运行这个代码看效果](https://github.com/duweiwang/kotlin-playground/blob/master/src/main/kotlin/com/wangduwei/kotlin/coroutines/ScopeDemo.kt)

这种写法看起来像在 try/catch 捕获异常，但它**不会如你预期那样工作**：如果 async 抛出异常，异常最终仍会导致父作用域失败。  
这是因为协程遵循 *结构化并发* 的原则：子协程失败会传播给父协程。

### ✅ 模式 1：使用 `SupervisorJob`

```kotlin
val scope = CoroutineScope(SupervisorJob() + Dispatchers.Default)
scope.async { … } // 异常不会取消同级协程
```

`SupervisorJob` 能让同一层级的子协程失败不会波及其他协程。  
**失败不会导致上层作用域取消**。

### ✅ 模式 2（更推荐）：使用 `coroutineScope`

```kotlin
suspend fun loadData() = coroutineScope {
    val deferred = async { … }
    deferred.await()
}
```

使用挂起的 `coroutineScope` 把 async 协程包起来：
- 使异常能被捕获；
- 限制失败只影响当前协程块，不传播到外部。([onlyfor-me-blog.tistory.com](https://onlyfor-me-blog.tistory.com/506))

此外，你也可以在 `async` 内部自己处理异常。

---

## 🎯 2. 根协程应优先使用 Main Dispatcher

### ❌ 反模式：在根协程中使用非 Main Dispatcher 去执行 UI 调用

这样写需要频繁手动切换到 Main 去更新 UI：

```kotlin
val scope = CoroutineScope(Dispatchers.Default)
scope.launch {
    // …
    withContext(Dispatchers.Main) { /* update UI */ }
    // …
}
```

### ✅ 推荐做法：根协程使用 Main Dispatcher

```kotlin
val scope = CoroutineScope(Dispatchers.Main)
scope.launch {
    // 不需要手动切换 dispatcher 就能更新 UI
}
```

这样可以让代码更简洁，避免大量显式切换上下文。([onlyfor-me-blog.tistory.com](https://onlyfor-me-blog.tistory.com/506))

---

## 🚫 3. 避免无意义的 `async` + `await`

### ❌ 不推荐：

```kotlin
val result = async { doWork() }.await()
```

这种写法的效果等同于直接调用挂起函数，并不会带来并发优势。([onlyfor-me-blog.tistory.com](https://onlyfor-me-blog.tistory.com/506))

### ✅ 推荐：

如果你只是想切换执行上下文，使用 `withContext`：

```kotlin
val result = withContext(Dispatchers.IO) {
    doWork()
}
```

`async` 更适合用于多个协程并行启动，再一起 await。

---

## ⚠️ 4. 避免取消整个 scope 的 Job

### ❌ 反模式：

如果你调用整个 scope 的 `job.cancel()`, 那个 scope 会进入 completed 状态。  
此后在该 scope 中启动的协程将不会执行。([onlyfor-me-blog.tistory.com](https://onlyfor-me-blog.tistory.com/506))

### ✅ 推荐做法：

使用 `cancelChildren()` 来取消当前 scope 下的所有子协程：

```kotlin
scope.coroutineContext.cancelChildren()
```

并适当为每个协程单独控制取消逻辑。

---

## 🛑 5. 不要在 suspend 函数中隐式依赖调度器

### ❌ 反模式：

如果 suspend 函数内部直接访问 UI，会导致在非 Main dispatcher 抛出异常（如 Android 的 `CalledFromWrongThreadException`）。([onlyfor-me-blog.tistory.com](https://onlyfor-me-blog.tistory.com/506))

### ✅ 推荐做法：

让 suspend 函数对调度器透明，在内部使用 `withContext` 做必要的调度：

```kotlin
suspend fun login() = withContext(Dispatchers.Main) {
    // UI 操作安全
}
```

这样该函数就能在任意 dispatcher 调用，而不会出错。([onlyfor-me-blog.tistory.com](https://onlyfor-me-blog.tistory.com/506))

---

## 🌍 6. 避免使用 `GlobalScope`

### ❌ 反模式：

到处使用 `GlobalScope` 启动协程会让协程脱离生命周期管理。  
这会导致资源泄露与不可预测行为，尤其在 Android 中很容易造成内存泄露、未取消协程等问题。([discuss.kotlinlang.org](https://discuss.kotlinlang.org/t/unavoidable-memory-leak-when-using-coroutines/11603/4))

### ✅ 推荐做法：

在 Android 中，协程应该绑定到 Activity/Fragment/ViewModel 生命周期，例如使用：

- `viewModelScope` / `lifecycleScope`
- 自定义带有 `SupervisorJob` 的 CoroutineScope

---

## 📌 总结

| 主题 | 建议 |
|------|------|
| 异常处理 | 用 `coroutineScope` 或 `SupervisorJob`，不要简单 try/catch |
| 根协程 Dispatcher | 优先使用 Main |
| async/await | 避免 immediate await，用于并发场景 |
| 取消协程 | 不要取消整个 scope Job，用 `cancelChildren()` |
| suspend 函数 | 不要依赖隐式 dispatcher |
| GlobalScope | 避免使用，绑定生命周期 |

---

## 🏁 参考

[原文](https://proandroiddev.com/kotlin-coroutines-patterns-anti-patterns-f9d12984c68e)