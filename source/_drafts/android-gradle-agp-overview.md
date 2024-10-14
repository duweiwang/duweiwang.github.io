---
title: android-gradle-agp-overview
tags:
---


一、Gradle阶段划分
1. 初始化阶段 (Initialization Phase)
   在这一阶段，Gradle 会创建一个项目对象模型（POM），并确定要构建的项目（或多个项目）。它会解析构建脚本，识别所有的项目及其依赖关系。
2. 配置阶段 (Configuration Phase)
   在此阶段，Gradle 会执行所有的构建脚本，配置所有的项目。具体来说，它会：
   评估 build.gradle 文件。
   创建任务并设置其依赖关系。
   在这一阶段，Gradle 还会生成一个任务图，该图描述了任务的执行顺序。
3. 执行阶段 (Execution Phase)
   在执行阶段，Gradle 会根据任务图的顺序执行任务。这一阶段包括：
   运行实际的构建任务，如编译代码、打包 APK、运行测试等。
   任务的执行可以是增量的，只处理更改的文件，以提高构建效率。
4. 结束阶段 (Finalization Phase)
   构建完成后，Gradle 会进行一些清理和审计工作，例如：
   输出构建结果。
   记录构建日志。


二、AGP插件
