---
title: 使用SDKMAN管理JDK
date: 2024-10-27 11:27:45
tags:
categories:
description:
---



在电脑安装多个版本的JDK，并自由切换，具有一定的灵活性。因为下载新JDK再配新环境变量真的很麻烦。SDKMAN可以帮我们解决这些烦恼。


以下是在Mac电脑安装SDKMAN并使用：

```shell
curl -s "https://get.sdkman.io" | bash
```
访问：https://get.sdkman.io/ 网站上的脚本，并使用bash执行

```shell
source $HOME/.sdkman/bin/sdkman-init.sh
```
执行sdkman初始化脚本


```shell
sdk list java
```
列出所有可用的jdk

```shell
sdk install java 19.0.2-oracle
```
安装指定的jdk

```shell
sdk use java 19.0.2-oracle
```
使用指定的jdk