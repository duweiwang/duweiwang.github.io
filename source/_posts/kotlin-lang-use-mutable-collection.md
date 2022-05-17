---
title: Effective Kotlin-56：考虑使用可变集合
tags: Kotlin
date: 2021-11-20 08:47:24
categories: Kotlin
description:
---


与不可变集合相比，使用可变集合最大的优点是性能更快。向不可变集合添加元素需要新建集合再拷贝所有元素，下面是kotlin标准库现在的实现：
```kotlin
operator fun <T> Iterable<T>.plus(element: T): List<T> {
    if (this is Collection) return this.plus(element)
    val result = ArrayList<T>()
    result.addAll(this)
    result.add(element)
    return result
}
```

当集合很大时，拷贝操作就比较耗性能了。所以对于需要添加元素的集合使用可变集合性能更好。我们知道，不可变集合是线程安全的。但是对于不需要同步的局部变量来说则不适用。这也是为什么对于局部处理，使用可变集合通常更有意义。这个事实可以在标准库中反映出来，在标准库中，所有的集合处理函数都是使用可变集合内部实现的:

```kotlin
inline fun <T, R> Iterable<T>.map(
    transform: (T) -> R
): List<R> {
    val size =
        if (this is Collection<*>) this.size else 10
    val destination = ArrayList<R>(size)
    for (item in this)
        destination.add(transform(item))
    return destination
}
```

而不是不可变集合：
```kotlin
// This is not how map is implemented
inline fun <T, R> Iterable<T>.map(
    transform: (T) -> R
): List<R> {
    var destination = listOf<R>()
    for (item in this)
        destination += transform(item)
    return destination
}
```


总结：
如果是增加元素，可变集合是很快的，而不可变集合则给我们给多的控制，知道集合是如何变化的。但对于本地作用域来讲，我们不需要这种控制，所以使用可变集合更好。尤其在一些工具类，插入操作可能很频繁。


