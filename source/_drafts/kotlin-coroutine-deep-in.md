---
title: Kotlin协程的工作原理
tags: Kotlin
categories: 计算机
description: 
---

大纲：
一、异步编程

异步编程是：在不阻塞程序主执行流程的情况下，去执行一些子流程。

+ 当子流程结束以后可以通过消息、事件、回调等通知主流程
+ 多个子流程相互依赖执行结果时，将形成回调嵌套
+ 解决回调嵌套问题的方案包括：Rx-、JS的Promise、协程等

二、协程
    0、基本概念
    什么是协程，与线程的区别
    5、结构化并发
    1、挂起函数和cps转换
    2、协程上下文
    4、线程切换
    3、异常处理
    
    Job的父子关系  Job与Supervisorjob。
    SupervisorJob中，某个孩子失败不会导致其他孩子失败。Job则会导致其他孩子和自己都失败
    Children of a supervisor job can fail independently of each other.
    
    val job = Job
    
    val scope = CoroutineScope(job+Dispatchers.Main)
    
   val job2 = scope.launch{
   }
   
  job2是job的孩子。


supervisorScope的使用
```kotlin
private fun fetchUsers() {
        viewModelScope.launch {
            users.postValue(Resource.loading(null))
            try {
                // supervisorScope is needed, so that we can ignore error and continue
                // here, more than two child jobs are running in parallel under a supervisor,
                    // one child job gets failed, we can continue with other.
                supervisorScope {
                    val usersFromApiDeferred = async { apiHelper.getUsersWithError() }
                    val moreUsersFromApiDeferred = async { apiHelper.getMoreUsers() }

                    val usersFromApi = try {
                        usersFromApiDeferred.await()
                    } catch (e: Exception) {
                        emptyList()
                    }

                    val moreUsersFromApi = try {
                        moreUsersFromApiDeferred.await()
                    } catch (e: Exception) {
                        emptyList()
                    }

                    val allUsersFromApi = mutableListOf<ApiUser>()
                    allUsersFromApi.addAll(usersFromApi)
                    allUsersFromApi.addAll(moreUsersFromApi)

                    users.postValue(Resource.success(allUsersFromApi))
                }
            } catch (e: Exception) {
                users.postValue(Resource.error("Something Went Wrong", null))
            }
        }
    }

```
coroutineScope的使用：
```kotlin
  private fun fetchUsers() {
        viewModelScope.launch {
            users.postValue(Resource.loading(null))
            try {
                // coroutineScope is needed, else in case of any network error, it will crash
                coroutineScope {
                    val usersFromApiDeferred = async { apiHelper.getUsers() }
                    val moreUsersFromApiDeferred = async { apiHelper.getMoreUsers() }

                    val usersFromApi = usersFromApiDeferred.await()
                    val moreUsersFromApi = moreUsersFromApiDeferred.await()

                    val allUsersFromApi = mutableListOf<ApiUser>()
                    allUsersFromApi.addAll(usersFromApi)
                    allUsersFromApi.addAll(moreUsersFromApi)

                    users.postValue(Resource.success(allUsersFromApi))
                }
            } catch (e: Exception) {
                users.postValue(Resource.error("Something Went Wrong", null))
            }
        }
    }
```

    
三、协程与安卓
    0、Room与协程https://medium.com/androiddevelopers/room-coroutines-422b786dc4c5
    
```kotlin
fun postItem(item: Item) {             
    val token = requestToken() 
    val post = createPost(token, item) 
    processPost(post) 
}
```    
 
    
三、Channel和Flow


一、概述

+ 协程的线程模型（线程和协程）
+ 异步代码同步化处理的实现原理（cps）
+ 现场切换和拦截器
+ 协程之间的通信csp


二、CPS （Continuation-passing style）

是起源于1970年的一种编程风格，并在1980-1990年用于高级语言编译器的中间表示。现如今他被发展成一种非堵塞的编程风格

什么是cps呢？
如果一个语言支持continuation，程序员可以添加控制结构例如：异常，回溯，线程和生成器。