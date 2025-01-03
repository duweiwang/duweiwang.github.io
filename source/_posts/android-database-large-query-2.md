---
title: android-database-large-query-2
date: 2025-01-03 15:38:16
tags:
categories:
description:
---


SQLite是安卓平台存储成千上万数据的最佳方案，但是如此巨大的数据量，也可能导致性能问题。
在Paging库之前，我们调研了很多现存的分页方案，尤其对SQLiteCursor的潜在缺陷进行了研究。
这篇文章我们谈一下这个问题，并阐述是什么激励我们在架构组件Room和Paging库中使用小的查询。


##### SQLiteCursor 和 CursorAdapter
SQLiteCursor是安卓SQLite数据库查询的返回值。它允许你以固定初始加载消耗来可视化大量查询结果集。


首次读操作初始化一个2MB的`CursorWindow`，并和查询结果一起返回。
每次你请求一个不存在的数据行时，SQLiteCursor都会刷新这个window。
因此，SQLiteCursor以固定大小页实现分页。


CursorAdapter在Android API-1.0就存在，它提供了一种简单的方式把来自游标的数据绑定在ListView的item上。
虽然它能很好的完成这中功能场景，但每当需要进行新的加载时，它就直接在UI线程上查询数据库。
这对于现代的响应性应用来说是不可接受的。
所以有人可能会问:我们不能有一个基于游标的适配器，在后台线程上加载吗?毕竟，SQLiteCursor内置了分页功能。


#### SQLiteCursor分页导致的问题

##### SQLiteCursor 不会保持任何数据库事务处于打开状态

当我开始研究分页时，我对 SQLite 以及 Android 上的 Cursors 还不熟悉。
我只是假设 SQLiteCursor 在加载窗口后会暂停查询，并在需要下一个窗口时恢复。
这样，访问第 10 个窗口的效率就和访问第 1 个窗口一样高。
这是不正确的。每次读取新窗口时，查询都会从位置 0 重新开始，并一次跳过不需要填充窗口的行。
这是因为 SQLiteCursor 无法恢复查询。

这就像访问链接列表中的第 1000 到 1050 个项目一样 — 您必须跳过大量项目才能访问要加载的下一个页面。
在加载过程中，每个后续窗口都必须跳过越来越多的查询，这会减慢速度。
这相当于使用 SQL OFFSET 关键字跳过内容，这不是分页内容的最有效方法，但在依赖 SQLiteCursor 内分页时无法避免。
您可以在[此处](https://android.googlesource.com/platform/frameworks/base/+/fee4546fd648b519ad828ea1f950554c1054699d/core/jni/android_database_SQLiteConnection.cpp#695)查看 SQLiteCursor 如何在新窗口中分页。

##### SQLiteCursor.getCount() 是必需的，它会扫描整个查询
在读取第一行之前，SQLiteCursor 会调用 getCount() 进行边界检查。
由于 SQLite 必须扫描查询的整个结果集才能对其进行计数（再次强调，就像链接列表一样），因此这可能会增加大量开销。
如果您要逐步将大型查询分页到 UI，以响应用户的滚动，您可能不需要知道查询的整个大小，因此计数会增加不必要的前期工作。

##### SQLiteCursor.getCount() 始终加载第一个行窗口
作为计算计数的一部分，在扫描结果集时，SQLiteCursor 会主动从位置 0 填充其窗口，假设将需要查询中的第一个项目。

它会预加载这些项目，以便提前知道窗口可以容纳多少行（下面将详细介绍）。
如果您从查询的开头显示数据，则这种随机加载是合理的，但从保存的实例状态恢复的位置可能会使索引从列表的更下方开始，而初始窗口并不相关。
如果您想从第三个内容窗口显示数据，则必须先加载并丢弃 2MB 的数据。此计数行为的代码在[此处](https://android.googlesource.com/platform/frameworks/base/+/fee4546fd648b519ad828ea1f950554c1054699d/core/java/android/database/sqlite/SQLiteCursor.java#130)。

##### SQLiteCursor 可能会加载您未要求的数据
Cursor.moveToPosition() 保证请求的行在窗口中，但 SQLiteCursor 不会在请求的行开始填充。
由于 SQLiteCursor 不假设应用程序正在向前读取，因此当它距离目标位置约 ⅓ 个窗口时，它会开始填充窗口。
这意味着 CursorAdapter 在窗口加载后向后滚动几行不会触发另一个窗口加载。
这还意味着在第一次加载之后加载的每 2MB 数据都会加载您已经看到的 650KB 或更多数据。
您可以在[此处](https://android.googlesource.com/platform/frameworks/base/+/fee4546fd648b519ad828ea1f950554c1054699d/core/java/android/database/DatabaseUtils.java#742)查看此行为的代码和解释。


##### SQLiteCursor 加载位置可能无法预测
当 SQLiteCursor 尝试加载目标位置时，它会尝试提前启动窗口的 ⅓。这意味着它必须猜测一个窗口中可以容纳多少行。
为此，它使用在第一次窗口加载中填充的行数。不幸的是，这意味着如果您的行大小不一（例如，如果您嵌入任意长度的用户评论字符串），它的猜测可能是错误的。
SQLiteCursor 可能会低于目标位置，用内容填充窗口 - 然后丢弃所有内容，并重新开始填充。
例如，如果您正在扫描一个长查询并到达需要重新加载窗口的行，则加载可能只会捕获少量的新行。
清除窗口和重新启动代码在[这里](https://android.googlesource.com/platform/frameworks/base/+/fee4546fd648b519ad828ea1f950554c1054699d/core/jni/android_database_SQLiteConnection.cpp#709)。


##### 游标需要关闭
必须使用 close() 方法关闭游标，因此无论它们存储在哪里，都必须有代码在不再需要它们时清理它们。
CursorAdapter 显然对此无能为力，而是将这一责任转交给应用程序开发人员。
要存储和重用游标，开发人员需要编写代码来处理活动停止等事件。

##### SQLiteCursor 不知道数据已更改
SQLiteCursor 不会跟踪数据库在第一次窗口读取（和第一次计数）后是否已更改。
这意味着如果添加或删除了某些项目，SQLiteCursor 的缓存大小是不正确的——这对于边界检查和您希望加载的数据看起来一致都是一个问题。
这可能会导致在移动到不再存在的行时出现异常，或者在某些情况下导致数据不一致。
例如，如果您已经加载了第 N 行，并且在位置 0 处插入了一个新项目，然后您尝试读取第 N+1 行，那么您最终将再次加载旧的第 N 行。



#### 避免这些问题

上述问题告诉我们，SQLCursor 无法扩展到具有数千个结果的查询。
幸运的是，这些问题都有一个简单的解决方法：小查询。
适合单个 CursorWindow 的查询可以避免所有问题，这就是我们在 Paging 和 Room 中如此推崇它们的原因。
通常将页面大小配置为仅 10 到 20 个项目，并且一次只查询这么多项目。


不过，选择页面大小需要考虑很多因素 — 窗口内较大的查询通常可以提高性能，较小的查询可以改善延迟和内存。
对于数据库不是瓶颈的较高列表项，10 个项目可能有意义，而如果您的列表项是小图块，或者您的查询成本很高，则 300 个项目可能更好。

如果您依赖 SQLiteCursor 的内部分页来延迟加载更大的结果集，我们建议您切换到另一种方法。
要么使用新的分页库及其与 Room Persistence Library 的集成，要么使用自定义实现，
您自己处理分页并确保您的查询结果足够小以适合单个 CursorWindow。
要使用新的分页库在 Room 中使用小查询对大型 SQL 查询结果进行分页，您可以更改：

```java

@Dao
interface UserDao {
    // regular list query — falls over with too much data
    @Query("SELECT * FROM user ORDER BY age DESC")
    LiveData<List<User>> loadUsersByAgeDesc();
}
```

改成：

```java
@Dao
interface UserDao {
    // paged query — handles arbitrarily large queries
    @Query("SELECT * FROM user ORDER BY age DESC")
    DataSource.Factory<Integer, User> loadUsersByAgeDesc();
}
```

然后将其传递给 LivePagedListBuilder 以获取可处理任意大结果集的 LiveData<PagedList>：

```java
LiveData<PagedList<User>> users = new LivePagedListBuilder<>(
        userDao.loadUsersByAgeDesc(), /*page size*/ 20).build();
```

在上面的代码中，我们获取了分页查询结果的 LiveData 版本，当数据库发生变化时，这些结果还会更新任何已订阅的观察者。
要了解有关使用架构组件从 SQLite 进行分页的更多信息，请参阅分页简介和 Github 上的分页示例。

#### Android Platform — Android P Update!

自本文首次发布以来，我们在 Android P 开发者预览版中添加了新的 API 来改善上述行为。
该平台现在允许应用禁用 ⅓ 窗口启发式算法并配置 CursorWindow 大小。
小查询仍然是避免上述所有问题的好方法，但 P 的更改赋予了应用更多的控制权，我们很快将在 Room 中使用它们来尽可能提高查询效率。

#### 参考
[原文](https://medium.com/androiddevelopers/large-database-queries-on-android-cb043ae626e8)