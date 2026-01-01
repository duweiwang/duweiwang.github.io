---
title: AGP transform 变更
tags: Android
date: 2026-01-01 09:46:32
categories:
description:
---


# AGP 8.0 重要 API 更新

## Transform API 被移除

从 **AGP 8.0** 开始，**Transform API** 被正式移除。这意味着 `com.android.build.api.transform` 包下的所有类都已被删除。

移除 Transform API 的主要原因是为了提升构建性能。使用 Transform API 的项目会迫使 AGP 走一条较低效的构建流程，可能导致构建时间出现明显回退。此外，Transform API 很难与其他 Gradle 特性良好协作；新的替代 API 旨在让开发者更容易扩展 AGP，同时避免性能问题或构建正确性问题。

---

## 替代 API 说明

Transform API 并没有一个“一对一”的替代方案，而是针对不同使用场景提供了**更精细化的 API**。  
所有替代 API 都位于 `androidComponents {}` 代码块中，并且 **从 AGP 7.2 起即可使用**。

---

## 字节码转换支持

如果需要对字节码进行转换，请使用 **Instrumentation API**。

- **Library 模块**：只能对本地项目中的类进行字节码插桩
- **Application / Test 模块**：可以选择仅对本地类插桩，或对包括远程依赖在内的所有类插桩

该 API 的特点是：
- 插桩逻辑**按单个 class 独立运行**
- 对 classpath 中其他类的访问能力受限（详见 `createClassVisitor()`）
- 有利于提升 **全量构建** 和 **增量构建** 的性能
- API 更简单，设计更安全

每个 library 会在其编译完成后立即并行进行插桩，而不是等待所有编译完成后再统一处理。在增量构建中，如果只修改了某一个类，那么只会重新插桩受影响的类。

[示例代码](https://github.com/android/gradle-recipes/blob/agp-7.2/BuildSrc/testAsmTransformApi/buildSrc/src/main/kotlin/ExamplePlugin.kt)

---

## 向 App 中添加生成的 Class

如果你的插件需要向 App 中额外添加生成的 class，请使用 **Artifacts API**，并结合  
`MultipleArtifact.ALL_CLASSES_DIRS`。

示例用法如下：

```kotlin
artifacts.use(taskProvider)
    .wiredWith(...)
    .toAppend(Artifact.Multiple)
```

通过 `MultipleArtifact.ALL_CLASSES_DIRS`，可以将自定义 task 生成的 class 目录追加到项目的 class 输出中。

Artifacts API 会自动为你的 task 分配一个唯一的输出目录。  
示例可参考官方文档中的 **addToAllClasses** 示例。

---

## 基于全程序分析（Whole Program Analysis）的转换支持

如果你的转换逻辑需要基于**整个程序的分析结果**，可以选择在**单个 task 中对所有 class 一起进行处理**。

⚠️ **注意事项**：
- 这种方式的构建性能开销明显高于 Instrumentation API
- 强烈建议将此能力设计为 **按 BuildType 可选启用**
- 方便开发者在 Debug / 开发构建中关闭该功能

---

## 使用 Artifacts.forScope API

从 **AGP 7.4** 开始，引入了 `Artifacts.forScope` API，用于注册对所有 class 统一处理的 task。

可选作用范围：
- `ScopedArtifacts.Scope.PROJECT`：仅当前项目
- `ScopedArtifacts.Scope.ALL`：当前项目 + 引入的项目 + 所有外部依赖

示例如下（对所有 class 进行统一转换）：

```kotlin
variant.artifacts.forScope(ScopedArtifacts.Scope.ALL)
    .use(taskProvider)
    .toTransform(
        ScopedArtifact.CLASSES,
        ModifyClassesTask::allJars,
        ModifyClassesTask::allDirectories,
        ModifyClassesTask::output,
    )
```

示例参考：
- [**modifyProjectClasses**](https://github.com/android/gradle-recipes/blob/agp-7.4/Kotlin/modifyProjectClasses/app/build.gradle.kts)：如何修改项目中的 class
- [**customizeAgpDsl**](https://github.com/android/gradle-recipes/tree/agp-7.2/BuildSrc/customizeAgpDsl)：如何为 BuildType 注册自定义 DSL 扩展

---

## 其他说明

- 如果你的使用场景未被 AndroidComponents API 覆盖，请向官方提交 issue
- 一些常用插件已经完成迁移并兼容 AGP 8.0，例如：
    - **Firebase Performance Monitoring**：1.4.1 起支持 AGP 8.0
    - **Hilt Gradle Plugin**：2.40.1 起支持 AGP 8.0
- **AGP Upgrade Assistant** 可以帮助项目升级常用插件
- 如果你通过第三方插件间接使用了 Transform API，请联系插件作者尽快适配新的 API

---

**总结**：  
AGP 8.0 彻底移除了 Transform API，推荐使用 Instrumentation API 与 Artifacts API 进行字节码处理和构建扩展。这些新 API 在性能、可维护性和构建正确性方面都有显著提升。


[原文](https://developer.android.com/build/releases/gradle-plugin-api-updates#transform-removed)