---
title: 排序算法之交换排序
tags:
-----

#### 简介


#### 冒泡

冒泡是一种双指针算法：
在一趟排序中，会有一个最大或最小的元素被放到有序的位置。
那么这个元素就不应该参与下一次的排序。
所以需要一个指针标识此分界线。另一个指针作为游动指针做两两对比。

```java
   public static void sort(int[] numbers) {
        int size = numbers.length;
        for (int i = 0; i < size - 1; i++) {
            for (int j = 0; j < size - 1 - i; j++) {
                if (numbers[j] > numbers[j + 1]) {  //交换两数位置
                    swap(numbers,j,j+1);
                }
            }
        }
    }
```

#### 快排



#### 比较