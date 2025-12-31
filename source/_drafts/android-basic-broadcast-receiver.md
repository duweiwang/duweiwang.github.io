---
title: Android四大组件之广播接收器
tags:
---

#### 大纲：

+ 动态注册vs静态注册

+ 普通广播（Normal Broadcast）
  系统广播（System Broadcast）
  有序广播（Ordered Broadcast）
  粘性广播（Sticky Broadcast）
  App应用内广播（Local Broadcast）




#### 版本变更

| 版本（API Level）                            | 主要更改点             | 影响 / 行为说明                                                                                                                                       |
| ---------------------------------------- | ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| **Android 5.0 Lollipop<br/>(API 21)**    | 无针对广播接收器的专门行为变化列表 | 基础广播机制与以往一致；此版本不在官方行为变化页中单独说明广播行为变化，主要系统层框架变更如 ServiceIntent 明确 Intent 要求等（非广播本身）。([52im Documentation][1])                                     |
| **Android 6.0 Marshmallow<br/>(API 23)** | 引入权限动态授权          | 广播接收器接收需要权限的广播时，必须 **运行时请求权限** 才能成功（如接收某些权限受限的系统广播）。                                                                                            |
| **Android 7.0 Nougat<br/>(API 24)**      | 隐式广播优化            | 不再对 manifest 声明的 BroadcastReceiver 分发某些隐式广播（如 CONNECTIVITY_ACTION、ACTION_NEW_PICTURE/VIDEO）以节省资源；需动态注册或用 JobScheduler。([Android Developers][2]) |
| **Android 8.0 Oreo<br/>(API 26)**        | 隐式广播限制大幅加强        | 针对应用 **targetSdk ≥ 26**，禁止在 manifest 中接收大多数隐式广播（如大部分系统事件），除非是显式广播或列入白名单。([Stack Overflow][3])                                                   |
| **Android 9 / 10 / 11<br/>(API 28–30)**  | 广播安全与系统优化加强       | 系统逐步增强安全策略和 broadcast 过滤，不鼓励滥用广播；非系统受保护广播难以被截获（官方并未明确列出针对 BroadcastReceiver 的行为页，但安全限制逐步加强）。([i.blackhat.com][4])                               |
| **Android 12 / 13<br/>(API 31–33)**      | 后台优化影响广播          | 系统对后台应用广播的调度更严格，某些广播仅在前台或活跃生命周期才能送达；鼓励用 WorkManager / JobScheduler 替代。                                                                          |
| **Android 14<br/>(API 34)**              | 广播投递与隐式 Intent 限制 | - **隐式 broadcast 仅送达已导出（exported）组件**（未声明 export 将导致投递失败）。                                                                                      |
|                                          |                   | - 应用处于 cached 状态时，非关键广播可能 **延迟投递**；需显式处理生命周期激活。([Android Developers][5])                                                                        |
| **Android 15<br/>(API 35)**              | 起始限制影响广播相关服务      | - 限制应用在收到 BOOT_COMPLETED 等广播时 **启动前台服务**（Foreground Service）行为；需结合 JobScheduler/WorkManager。([Android Developers][6])                           |
| **Android 16<br/>(API 36)**              | 广播优先级行为变更         | - 跨进程广播优先级不再保证；priority 仅在同一进程有效。                                                                                                               |
|                                          |                   | - Priority 值范围被限制，系统低/高优先级由系统组件控制。([Android Developers][7])                                                                                     |

[1]: https://docs.52im.net/extend/docs/api/android-50/about/versions/android-5.0-changes.html?utm_source=chatgpt.com "Android 5.0 Behavior Changes"
[2]: https://developer.android.com/about/versions/nougat/android-7.0-changes?utm_source=chatgpt.com "Android 7.0 Behavior Changes  |  Android Developers"
[3]: https://stackoverflow.com/questions/51221680/broadcast-receiver-not-working-in-android-oreo?utm_source=chatgpt.com "broadcastreceiver - Broadcast receiver not working in android oreo - Stack Overflow"
[4]: https://i.blackhat.com/asia-21/Thursday-Handouts/as-21-Johnson-Unprotected-Broadcasts-In-Android-9-and-10-wp.pdf?utm_source=chatgpt.com "[PDF] (Un)protected Broadcasts in Android 9 and 10 - Black Hat"
[5]: https://developer.android.com/develop/background-work/background-tasks/broadcasts?utm_source=chatgpt.com "Broadcasts overview | Background work | Android Developers"
[6]: https://developer.android.com/about/versions/15/behavior-changes-15?utm_source=chatgpt.com "Behavior changes: Apps targeting Android 15 or higher"
[7]: https://developer.android.com/about/versions/16/behavior-changes-all?utm_source=chatgpt.com "Behavior changes: all apps  |  Android Developers"






