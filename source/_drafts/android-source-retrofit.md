---
title: Retrofit源码阅读
tags:
---

一、基本原理概述

1、声明接口方法

```kotlin
interface Service {
    @GET("/{a}/{b}/{c}")
    suspend fun params(
        @Path("a") a: String,
        @Path("b") b: String,
        @Path("c") c: String
    ): String
  }
```
2、生成代理类

```kotlin
public <T> T create(final Class<T> service) {
    Utils.validateServiceInterface(service);
    if (validateEagerly) {
      eagerlyValidateMethods(service);
    }
    return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[] { service },
        new InvocationHandler() {
          private final Platform platform = Platform.get();
          private final Object[] emptyArgs = new Object[0];

          @Override public @Nullable Object invoke(Object proxy, Method method,
              @Nullable Object[] args) throws Throwable {
            // If the method is a method from Object then defer to normal invocation.
            if (method.getDeclaringClass() == Object.class) {
              return method.invoke(this, args);
            }
            if (platform.isDefaultMethod(method)) {
              return platform.invokeDefaultMethod(method, service, proxy, args);
            }
            return loadServiceMethod(method).invoke(args != null ? args : emptyArgs);
          }
        });
  }
```

3、参数解析

loadServiceMethod
->


4、发起请求

ServiceMethod.invoke  
->  
ServiceMethod.adapt()  
->   
SuspendForBody/SuspendForResponse/CallAdapted.adapt()  
->  
Call.enqueue  
->  
OkhttpCall.enqueue  

二、
