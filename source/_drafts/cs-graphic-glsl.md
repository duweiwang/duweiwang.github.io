---
title: cs-graphic-glsl
tags:
---



step函数：
smoothstep函数：


### 处理后的日志数据及时间差计算

| 起始时间                    | 日志内容                        | 结束时间                    | 日志内容                                   | 时间差    |
|-------------------------|-----------------------------|-------------------------|----------------------------------------|--------|
| 2024-07-06 09:20:34.223 | InitAndLoad D call init.... | 2024-07-06 09:20:40.782 | InitAndLoad D onAdLoaded...cost = 5575 | 6.559秒 |
| 2024-07-06 09:22:02.900 | InitAndLoad D call init.... | 2024-07-06 09:22:09.924 | InitAndLoad D onAdLoaded...cost = 6095 | 7.024秒 |
| 2024-07-06 09:22:42.610 | InitAndLoad D call init.... | 2024-07-06 09:22:48.685 | InitAndLoad D onAdLoaded...cost = 5303 | 6.075秒 |
| 2024-07-06 09:36:08.648 | InitAndLoad D call init.... | 2024-07-06 09:36:10.988 | InitAndLoad D onAdLoaded...cost = 2038 | 2.340秒 |
| 2024-07-06 09:48:04.733 | InitAndLoad D call init.... | 2024-07-06 09:48:07.057 | InitAndLoad D onAdLoaded...cost = 2096 | 2.324秒 |
| 2024-07-06 09:48:53.495 | InitAndLoad D call init.... | 2024-07-06 09:48:55.658 | InitAndLoad D onAdLoaded...cost = 1910 | 2.163秒 |
| 2024-07-06 09:49:49.170 | InitAndLoad D call init.... | 2024-07-06 09:49:52.030 | InitAndLoad D onAdLoaded...cost = 1905 | 2.860秒 |
| 2024-07-06 09:50:05.846 | InitAndLoad D call init.... | 2024-07-06 09:50:07.949 | InitAndLoad D onAdLoaded...cost = 1882 | 2.103秒 |
| 2024-07-06 10:08:17.243 | InitAndLoad D call init.... | 2024-07-06 10:08:19.441 | InitAndLoad D onAdLoaded...cost = 2042 | 2.198秒 |
| 2024-07-06 10:08:58.036 | InitAndLoad D call init.... | 2024-07-06 10:09:03.229 | InitAndLoad D onAdLoaded...cost = 4808 | 5.193秒 |
| 2024-07-06 10:09:21.243 | InitAndLoad D call init.... | 2024-07-06 10:09:23.016 | InitAndLoad D onAdLoaded...cost = 1601 | 1.773秒 |
| 2024-07-06 10:10:06.061 | InitAndLoad D call init.... | 2024-07-06 10:10:08.416 | InitAndLoad D onAdLoaded...cost = 2192 | 2.355秒 |




---------初始化同时加载
2024-07-06 09:20:34.223  9428-9428  InitAndLoad              D  call init....
2024-07-06 09:20:35.207  9428-9428  InitAndLoad              D  start load....
2024-07-06 09:20:38.459  9428-9428  InitAndLoad              D  after init = 4236
2024-07-06 09:20:40.782  9428-9428  InitAndLoad              D  onAdLoaded...cost = 5575


2024-07-06 09:22:02.900 10164-10164 InitAndLoad              D  call init....
2024-07-06 09:22:03.829 10164-10164 InitAndLoad              D  start load....
2024-07-06 09:22:06.907 10164-10164 InitAndLoad              D  after init = 4007
2024-07-06 09:22:09.924 10164-10164 InitAndLoad              D  onAdLoaded...cost = 6095


2024-07-06 09:22:42.610 10801-10801 InitAndLoad              D  call init....
2024-07-06 09:22:43.382 10801-10801 InitAndLoad              D  start load....
2024-07-06 09:22:45.886 10801-10801 InitAndLoad              D  after init = 3276
2024-07-06 09:22:48.685 10801-10801 InitAndLoad              D  onAdLoaded...cost = 5303


2024-07-06 09:36:08.648 16621-16621 InitAndLoad              D  call init....
2024-07-06 09:36:08.950 16621-16621 InitAndLoad              D  start load....
2024-07-06 09:36:09.861 16621-16621 InitAndLoad              D  after init = 1213
2024-07-06 09:36:10.988 16621-16621 InitAndLoad              D  onAdLoaded...cost = 2038


2024-07-06 09:48:04.733 19812-19812 InitAndLoad              D  call init....
2024-07-06 09:48:04.961 19812-19812 InitAndLoad              D  start load....
2024-07-06 09:48:05.799 19812-19812 InitAndLoad              D  after init = 1065
2024-07-06 09:48:07.057 19812-19812 InitAndLoad              D  onAdLoaded...cost = 2096


2024-07-06 09:48:53.495 20163-20163 InitAndLoad              D  call init....
2024-07-06 09:48:53.748 20163-20163 InitAndLoad              D  start load....
2024-07-06 09:48:54.797 20163-20163 InitAndLoad              D  after init = 1302
2024-07-06 09:48:55.658 20163-20163 InitAndLoad              D  onAdLoaded...cost = 1910

2024-07-06 09:49:49.170 20650-20650 InitAndLoad              D  call init....
2024-07-06 09:49:49.986 20650-20650 InitAndLoad              D  after init = 816
2024-07-06 09:49:50.123 20650-20650 InitAndLoad              D  start load....
2024-07-06 09:49:52.030 20650-20650 InitAndLoad              D  onAdLoaded...cost = 1905

2024-07-06 09:50:05.846 20847-20847 InitAndLoad              D  call init....
2024-07-06 09:50:06.066 20847-20847 InitAndLoad              D  start load....
2024-07-06 09:50:06.871 20847-20847 InitAndLoad              D  after init = 1025
2024-07-06 09:50:07.949 20847-20847 InitAndLoad              D  onAdLoaded...cost = 1882



2024-07-06 10:08:17.243 22687-22687 InitAndLoad               D  call init....
2024-07-06 10:08:17.399 22687-22687 InitAndLoad               D  start load....
2024-07-06 10:08:18.341 22687-22687 InitAndLoad               D  after init = 1098
2024-07-06 10:08:19.441 22687-22687 InitAndLoad               D  onAdLoaded...cost = 2042

2024-07-06 10:08:58.036 22961-22961 InitAndLoad               D  call init....
2024-07-06 10:08:58.421 22961-22961 InitAndLoad               D  start load....
2024-07-06 10:09:01.251 22961-22961 InitAndLoad               D  after init = 3215
2024-07-06 10:09:03.229 22961-22961 InitAndLoad               D  onAdLoaded...cost = 4808

2024-07-06 10:09:21.243 23315-23315 InitAndLoad               D  call init....
2024-07-06 10:09:21.415 23315-23315 InitAndLoad               D  start load....
2024-07-06 10:09:22.589 23315-23315 InitAndLoad               D  after init = 1345
2024-07-06 10:09:23.016 23315-23315 InitAndLoad               D  onAdLoaded...cost = 1601

2024-07-06 10:10:06.061 23555-23555 InitAndLoad               D  call init....
2024-07-06 10:10:06.224 23555-23555 InitAndLoad               D  start load....
2024-07-06 10:10:07.401 23555-23555 InitAndLoad               D  after init = 1340
2024-07-06 10:10:08.416 23555-23555 InitAndLoad               D  onAdLoaded...cost = 2192


-------不初始化直接加载

2024-07-06 09:24:24.195 11468-11468 DontInit              D  start load....
2024-07-06 09:24:27.222 11468-11468 DontInit              D  onAdLoaded...cost = 3025

2024-07-06 09:25:23.075 12509-12509 DontInit              D  start load....
2024-07-06 09:25:37.792 12509-12509 DontInit              D  onAdLoaded...cost = 14716

2024-07-06 09:26:23.261 13358-13358 DontInit              D  start load....
2024-07-06 09:26:27.299 13358-13358 DontInit              D  onAdLoaded...cost = 4038

2024-07-06 09:27:14.389 14036-14036 DontInit              D  start load....
2024-07-06 09:27:18.001 14036-14036 DontInit              D  onAdLoaded...cost = 3612

2024-07-06 09:27:44.170 14457-14457 DontInit              D  start load....
2024-07-06 09:27:48.394 14457-14457 DontInit              D  onAdLoaded...cost = 4224

2024-07-06 09:28:08.598 14689-14689 DontInit              D  start load....
2024-07-06 09:28:10.697 14689-14689 DontInit              D  onAdLoaded...cost = 2099

2024-07-06 09:28:47.926 15067-15067 DontInit              D  start load....
2024-07-06 09:29:10.420 15067-15067 DontInit              D  onAdLoaded...cost = 22494

2024-07-06 09:29:36.843 15490-15490 DontInit              D  start load....
2024-07-06 09:29:39.009 15490-15490 DontInit              D  onAdLoaded...cost = 2166

2024-07-06 09:30:25.297 15732-15732 DontInit              D  start load....
2024-07-06 09:30:27.380 15732-15732 DontInit              D  onAdLoaded...cost = 2083

2024-07-06 10:04:49.856 21650-21650 DontInit                  D  start load....
2024-07-06 10:04:55.452 21650-21650 DontInit                  D  onAdLoaded...cost = 5596

2024-07-06 10:05:49.207 22055-22055 DontInit                  D  start load....
2024-07-06 10:05:50.927 22055-22055 DontInit                  D  onAdLoaded...cost = 1720

### 处理后的日志数据及时间差计算

| 起始时间                    | 日志内容           | 结束时间                    | 日志内容                      | 时间差     |
|-------------------------|----------------|-------------------------|---------------------------|---------|
| 2024-07-06 09:24:24.195 | start load.... | 2024-07-06 09:24:27.222 | onAdLoaded...cost = 3025  | 3.027秒  |
| 2024-07-06 09:25:23.075 | start load.... | 2024-07-06 09:25:37.792 | onAdLoaded...cost = 14716 | 14.717秒 |
| 2024-07-06 09:26:23.261 | start load.... | 2024-07-06 09:26:27.299 | onAdLoaded...cost = 4038  | 4.038秒  |
| 2024-07-06 09:27:14.389 | start load.... | 2024-07-06 09:27:18.001 | onAdLoaded...cost = 3612  | 3.612秒  |
| 2024-07-06 09:27:44.170 | start load.... | 2024-07-06 09:27:48.394 | onAdLoaded...cost = 4224  | 4.224秒  |
| 2024-07-06 09:28:08.598 | start load.... | 2024-07-06 09:28:10.697 | onAdLoaded...cost = 2099  | 2.099秒  |
| 2024-07-06 09:28:47.926 | start load.... | 2024-07-06 09:29:10.420 | onAdLoaded...cost = 22494 | 22.494秒 |
| 2024-07-06 09:29:36.843 | start load.... | 2024-07-06 09:29:39.009 | onAdLoaded...cost = 2166  | 2.166秒  |
| 2024-07-06 09:30:25.297 | start load.... | 2024-07-06 09:30:27.380 | onAdLoaded...cost = 2083  | 2.083秒  |
| 2024-07-06 10:04:49.856 | start load.... | 2024-07-06 10:04:55.452 | onAdLoaded...cost = 5596  | 5.596秒  |
| 2024-07-06 10:05:49.207 | start load.... | 2024-07-06 10:05:50.927 | onAdLoaded...cost = 1720  | 1.720秒  |

### 导入Excel的步骤



//初始化完成再加载

2024-07-06 09:42:21.889 17499-17499 LoadAfterInit              D  call init....
2024-07-06 09:42:22.971 17499-17499 LoadAfterInit              D  after init = 1082
2024-07-06 09:42:22.973 17499-17499 LoadAfterInit              D  start load....
2024-07-06 09:42:24.957 17499-17499 LoadAfterInit              D  onAdLoaded...cost = 1984


2024-07-06 09:43:02.339 17735-17735 LoadAfterInit              D  call init....
2024-07-06 09:43:03.555 17735-17735 LoadAfterInit              D  after init = 1215
2024-07-06 09:43:03.558 17735-17735 LoadAfterInit              D  start load....
2024-07-06 09:43:05.284 17735-17735 LoadAfterInit              D  onAdLoaded...cost = 1725


2024-07-06 09:43:25.237 17984-17984 LoadAfterInit              D  call init....
2024-07-06 09:43:26.516 17984-17984 LoadAfterInit              D  after init = 1278
2024-07-06 09:43:26.518 17984-17984 LoadAfterInit              D  start load....
2024-07-06 09:43:28.908 17984-17984 LoadAfterInit              D  onAdLoaded...cost = 2390

2024-07-06 09:43:43.801 18202-18202 LoadAfterInit              D  call init....
2024-07-06 09:43:44.868 18202-18202 LoadAfterInit              D  after init = 1067
2024-07-06 09:43:44.871 18202-18202 LoadAfterInit              D  start load....
2024-07-06 09:43:46.488 18202-18202 LoadAfterInit              D  onAdLoaded...cost = 1617


2024-07-06 09:44:02.949 18434-18434 LoadAfterInit              D  call init....
2024-07-06 09:44:03.911 18434-18434 LoadAfterInit              D  after init = 962
2024-07-06 09:44:03.914 18434-18434 LoadAfterInit              D  start load....
2024-07-06 09:44:05.560 18434-18434 LoadAfterInit              D  onAdLoaded...cost = 1646


2024-07-06 09:44:20.341 18677-18677 LoadAfterInit              D  call init....
2024-07-06 09:44:21.265 18677-18677 LoadAfterInit              D  after init = 924
2024-07-06 09:44:21.268 18677-18677 LoadAfterInit              D  start load....
2024-07-06 09:44:22.996 18677-18677 LoadAfterInit              D  onAdLoaded...cost = 1728


2024-07-06 09:45:23.237 18907-18907 LoadAfterInit              D  call init....
2024-07-06 09:45:24.179 18907-18907 LoadAfterInit              D  after init = 942
2024-07-06 09:45:24.181 18907-18907 LoadAfterInit              D  start load....
2024-07-06 09:45:25.862 18907-18907 LoadAfterInit              D  onAdLoaded...cost = 1681

2024-07-06 09:46:19.173 19405-19405 LoadAfterInit              D  call init....
2024-07-06 09:46:20.330 19405-19405 LoadAfterInit              D  after init = 1157
2024-07-06 09:46:20.336 19405-19405 LoadAfterInit              D  start load....
2024-07-06 09:46:22.038 19405-19405 LoadAfterInit              D  onAdLoaded...cost = 1702

2024-07-06 10:06:09.519 22300-22300 LoadAfterInit             D  call init....
2024-07-06 10:06:10.438 22300-22300 LoadAfterInit             D  after init = 918
2024-07-06 10:06:10.439 22300-22300 LoadAfterInit             D  start load....
2024-07-06 10:06:11.733 22300-22300 LoadAfterInit             D  onAdLoaded...cost = 1293




### 处理后的日志数据及时间差计算

| 起始时间                    | 日志内容          | 结束时间                    | 日志内容                     | 时间差    |
|-------------------------|---------------|-------------------------|--------------------------|--------|
| 2024-07-06 09:42:21.889 | call init.... | 2024-07-06 09:42:24.957 | onAdLoaded...cost = 1984 | 3.068秒 |
| 2024-07-06 09:43:02.339 | call init.... | 2024-07-06 09:43:05.284 | onAdLoaded...cost = 1725 | 2.945秒 |
| 2024-07-06 09:43:25.237 | call init.... | 2024-07-06 09:43:28.908 | onAdLoaded...cost = 2390 | 3.671秒 |
| 2024-07-06 09:43:43.801 | call init.... | 2024-07-06 09:43:46.488 | onAdLoaded...cost = 1617 | 2.687秒 |
| 2024-07-06 09:44:02.949 | call init.... | 2024-07-06 09:44:05.560 | onAdLoaded...cost = 1646 | 2.611秒 |
| 2024-07-06 09:44:20.341 | call init.... | 2024-07-06 09:44:22.996 | onAdLoaded...cost = 1728 | 2.655秒 |
| 2024-07-06 09:45:23.237 | call init.... | 2024-07-06 09:45:25.862 | onAdLoaded...cost = 1681 | 2.625秒 |
| 2024-07-06 09:46:19.173 | call init.... | 2024-07-06 09:46:22.038 | onAdLoaded...cost = 1702 | 2.865秒 |
| 2024-07-06 10:06:09.519 | call init.... | 2024-07-06 10:06:11.733 | onAdLoaded...cost = 1293 | 2.214秒 |

### 导入Excel的步骤











