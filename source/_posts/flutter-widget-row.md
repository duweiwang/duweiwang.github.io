---
title: Flutter Widget之Row、Column
date: 2023-03-07 16:54:08
tags: Flutter
categories: 计算机
description:
---

#### Row和Column分别在横向和纵向对子Widget进行布局。
+ 对于Row来讲，横向是主轴，纵向是交叉轴。
+ 对于Column来讲，横向是交叉轴，纵向是主轴。

针对主轴和交叉轴，不同大小的子Widget该如何对齐呢？Flutter提供了下列属性：


####  MainAxisAlignment：主轴对齐方式
| 属性值 | 解释 | 图示 |
| -- | -- | -- |
| start | 靠近主轴的开始 | <center><img src="../images/flutter/flutter_row_start.png"/></center> |
| end | 靠近主轴的末尾 | <center><img src="../images/flutter/flutter_row_end.png"/></center> |
| center | 靠近主轴中间 | <center><img src="../images/flutter/flutter_row_center.png"/></center> |
| spaceBetween | 剩余空间在孩子中间平分 | <center><img src="../images/flutter/flutter_row_space_between.png"/></center> |
| spaceAround | 剩余空间围绕孩子平分 | <center><img src="../images/flutter/flutter_row_space_around.png"/></center> |
| spaceEvenly | 剩余空间在孩子之间均等分配 | <center><img src="../images/flutter/flutter_row_space_evenly.png"/></center> |

####  CrossAxisAlignment:交叉轴对齐方式
| 属性值 | 解释 | 图示 |
| -- | -- | -- |
| start | 左对齐(Column)或上对齐(Row) | <center><img src="../images/flutter/cross_axis_start.jpeg" width="50%"/></center> |
| end | 右对齐(Column)或下对齐(Row) | <center><img src="../images/flutter/cross_axis_end.jpeg" width="50%"/></center> |
| center | 中间对齐 | <center><img src="../images/flutter/cross_axis_center.jpeg" width="50%"/></center> |
| stretch | 拉伸 | <center><img src="../images/flutter/cross_axis_stretch.jpeg" width="50%"/></center> |
| baseline | 基线对齐,需要配合textBaseline属性使用 | <center><img src="../images/flutter/cross_axis_baseline.jpg" width="50%"/></center> |