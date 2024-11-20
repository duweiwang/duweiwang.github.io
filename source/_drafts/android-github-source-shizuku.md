---
title: android-github-source-shizuku
tags:
---





项目地址：https://github.com/RikkaApps/Shizuku


运行此项目后，执行下列命令开启Shizuku服务：
```shell
adb shell sh /storage/emulated/0/Android/data/moe.shizuku.privileged.api/start.sh
```

执行命令查看和Shizuku相关的进程：
```shell
adb shell ps | grep shizuku
```

输出：
```shell
u0_a1858       815  1289   14842600 218968 0                   0 S moe.shizuku.privileged.api
shell         2049     1   13778256 132868 do_epoll_wait       0 S shizuku_server
```

