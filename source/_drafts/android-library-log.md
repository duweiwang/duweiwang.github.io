---
title: Android中的日志库
tags:
---



+ [Log4a](https://github.com/pqpo/Log4a)
900 star

    基于 mmap, 高性能、高可用的 Android 日志收集框架

```shell
    Log4a.i(TAG, "Hello，Log4a!");
    Log4a.flush();
    Log4a.release();
```


+ [xLog](https://github.com/elvishew/xLog)

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

+[logger](https://github.com/orhanobut/logger)
13.7K star


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