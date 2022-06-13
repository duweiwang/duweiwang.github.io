---
title: 安卓上的大量数据库查询
tags: Android
-------------
SQLite是安卓平台存储成千上万数据的最佳方案，但是如此巨大的数据量，也可能导致性能问题。在Paging库之前，我们调研了很多现存的分页方案，尤其对SQLiteCursor的潜在缺陷进行了研究。

这篇文章我们谈一下这个问题，并阐述是什么激励我们在架构组件Room和Paging库中使用小的查询。


##### SQLiteCursor 和 CursorAdapter
SQLiteCursor是安卓SQLite数据库查询的返回值。它允许你以固定初始加载消耗来可视化大量查询结果集。


首次读操作初始化一个2MB的`CursorWindow`，并和查询结果一起返回。每次你请求一个不存在的数据行时，SQLiteCursor都会刷新这个window。因此，SQLiteCursor以固定大小页实现分页。


CursorAdapter在API-1.0就存在，它提供了一种简单的方式把来自游标的数据绑定在ListView的item上。虽然它能很好的完成这中功能场景，但每当需要进行新的加载时，它就直接在UI线程上查询数据库。这对于现代的响应性应用来说是不可接受的。所以有人可能会问:我们不能有一个基于游标的适配器，在后台线程上加载吗?毕竟，SQLiteCursor内置了分页功能。


#### SQLiteCursor分页导致的问题




