---
title: Android中的日志库
date: 2024-07-09 15:54:19
tags:
categories:
description:
---


日志库是一个项目的基础功能之一，它应该包括但不限于以下功能：
+ 扩展性：支持把日志记录到多个地方，如：本地文件、控制台、崩溃系统。
+ 高性能：日志记录较为频繁时，如何写文件，提高性能。
+ 稳定性：崩溃前一刻的日志不丢失。

以下是一些开源的日志库，做一个记录：


+ [Log4a](https://github.com/pqpo/Log4a) 0.9k star

基于 mmap, 高性能、高可用的 Android 日志收集框架

```shell
    Log4a.i(TAG, "Hello，Log4a!");
    Log4a.flush();
    Log4a.release();
```


+ [xLog](https://github.com/elvishew/xLog) 3.1k star

```java
class Test {
    LogConfiguration config = new LogConfiguration.Builder()
        .logLevel(BuildConfig.DEBUG ? LogLevel.ALL             // Specify log level, logs below this level won't be printed, default: LogLevel.ALL
            : LogLevel.NONE)
        .tag("MY_TAG")                                         // Specify TAG, default: "X-LOG"
        .enableThreadInfo()                                    // Enable thread info, disabled by default
        .enableStackTrace(2)                                   // Enable stack trace info with depth 2, disabled by default
        .enableBorder()                                        // Enable border, disabled by default
        .jsonFormatter(new MyJsonFormatter())                  // Default: DefaultJsonFormatter
        .xmlFormatter(new MyXmlFormatter())                    // Default: DefaultXmlFormatter
        .throwableFormatter(new MyThrowableFormatter())        // Default: DefaultThrowableFormatter
        .threadFormatter(new MyThreadFormatter())              // Default: DefaultThreadFormatter
        .stackTraceFormatter(new MyStackTraceFormatter())      // Default: DefaultStackTraceFormatter
        .borderFormatter(new MyBoardFormatter())               // Default: DefaultBorderFormatter
        .addObjectFormatter(AnyClass.class,                    // Add formatter for specific class of object
            new AnyClassObjectFormatter())                     // Use Object.toString() by default
        .addInterceptor(new BlacklistTagsFilterInterceptor(    // Add blacklist tags filter
            "blacklist1", "blacklist2", "blacklist3"))
        .addInterceptor(new MyInterceptor())                   // Add other log interceptor
        .build();
    
    Printer androidPrinter = new AndroidPrinter(true);         // Printer that print the log using android.util.Log
    Printer consolePrinter = new ConsolePrinter();             // Printer that print the log to console using System.out
    Printer filePrinter = new FilePrinter                      // Printer that print(save) the log to file
        .Builder("<path-to-logs-dir>")                         // Specify the directory path of log file(s)
        .fileNameGenerator(new DateFileNameGenerator())        // Default: ChangelessFileNameGenerator("log")
        .backupStrategy(new NeverBackupStrategy())             // Default: FileSizeBackupStrategy(1024 * 1024)
        .cleanStrategy(new FileLastModifiedCleanStrategy(MAX_TIME))     // Default: NeverCleanStrategy()
        .flattener(new MyFlattener())                          // Default: DefaultFlattener
        .writer(new MyWriter())                                // Default: SimpleWriter
        .build();
    
    XLog.init(                                                 // Initialize XLog
        config,                                                // Specify the log configuration, if not specified, will use new LogConfiguration.Builder().build()
        androidPrinter,                                        // Specify printers, if no printer is specified, AndroidPrinter(for Android)/ConsolePrinter(for java) will be used.
        consolePrinter,
        filePrinter);
}
````

+ [logger](https://github.com/orhanobut/logger) 13.7k star


```java
class Test { 
    FormatStrategy formatStrategy = PrettyFormatStrategy.newBuilder()
      .showThreadInfo(false)  // (Optional) Whether to show thread info or not. Default true
      .methodCount(0)         // (Optional) How many method line to show. Default 2
      .methodOffset(7)        // (Optional) Hides internal method calls up to offset. Default 5
      .logStrategy(customLog) // (Optional) Changes the log strategy to print out. Default LogCat
      .tag("My custom tag")   // (Optional) Global tag for every log. Default PRETTY_LOGGER
      .build();

    Logger.addLogAdapter(new AndroidLogAdapter(formatStrategy));
}

```