---
title: cs-algorithms-merge-sort
date: 2025-01-08 16:31:00
tags:
categories:
description:
---


# 归并排序算法详解

归并排序（Merge Sort）是一种高效的排序算法，采用分治策略（Divide and Conquer）。它将一个大问题分解为多个小问题，然后再将小问题的解合并成大问题的解。归并排序是一种稳定的排序算法，最坏时间复杂度为 \(O(n \log n)\)，因此在处理大数据时非常有效。

## 归并排序的基本思想

归并排序的核心思想是：

1. **分解（Divide）**：将待排序数组分成两半，分别对这两部分进行归并排序。
2. **合并（Conquer）**：将排序好的两部分合并成一个有序的数组。

## 归并排序的步骤

1. 将数组分成两部分，分别进行归并排序。
2. 对两个已排序的部分进行合并，合并过程通过比较两个部分的元素，生成一个有序数组。
3. 重复这个过程，直到所有的子数组都合并成一个大数组。

## 归并排序的Java实现

下面是归并排序算法的Java实现：

```java
public class MergeSort {

    // 主方法
    public static void main(String[] args) {
        int[] array = { 38, 27, 43, 3, 9, 82, 10 };
        System.out.println("原始数组: ");
        printArray(array);
        
        mergeSort(array);
        
        System.out.println("
排序后的数组: ");
        printArray(array);
    }

    // 归并排序的核心方法
    public static void mergeSort(int[] array) {
        if (array.length < 2) {
            return;
        }
        
        int mid = array.length / 2;
        int[] left = new int[mid];
        int[] right = new int[array.length - mid];

        // 将数组分成两部分
        System.arraycopy(array, 0, left, 0, mid);
        System.arraycopy(array, mid, right, 0, array.length - mid);

        // 对左右两部分分别进行归并排序
        mergeSort(left);
        mergeSort(right);

        // 合并两部分
        merge(array, left, right);
    }

    // 合并两部分
    public static void merge(int[] array, int[] left, int[] right) {
        int i = 0, j = 0, k = 0;

        // 合并过程：比较left和right的元素，按顺序放入array
        while (i < left.length && j < right.length) {
            if (left[i] <= right[j]) {
                array[k++] = left[i++];
            } else {
                array[k++] = right[j++];
            }
        }

        // 如果left数组还有剩余元素，直接放入array
        while (i < left.length) {
            array[k++] = left[i++];
        }

        // 如果right数组还有剩余元素，直接放入array
        while (j < right.length) {
            array[k++] = right[j++];
        }
    }

    // 打印数组的方法
    public static void printArray(int[] array) {
        for (int i = 0; i < array.length; i++) {
            System.out.print(array[i] + " ");
        }
        System.out.println();
    }
}
```

## 代码解析

1. **`mergeSort` 方法**：
    - 该方法首先检查数组的长度，如果数组长度小于2，则无需排序。
    - 将数组分成左右两部分，分别调用 `mergeSort` 方法进行递归排序。

2. **`merge` 方法**：
    - 该方法负责合并两个已排序的数组（`left` 和 `right`）为一个有序的数组。
    - 通过两个指针 `i` 和 `j`，分别遍历 `left` 和 `right` 数组，并将较小的元素放入原数组中。
    - 如果某一部分数组元素还未处理完，就直接将剩余的元素加入到原数组中。

3. **`printArray` 方法**：
    - 该方法用于打印数组的内容，帮助我们查看排序结果。

## 时间复杂度分析

- **时间复杂度**：归并排序的时间复杂度为 \(O(n \log n)\)，其中 \(n\) 是数组的长度。
- **空间复杂度**：归并排序需要额外的空间来存储临时数组，因此空间复杂度为 \(O(n)\)。

## 总结

归并排序是一种非常高效的排序算法，适合处理大规模数据。其优点在于稳定性和较为平衡的时间复杂度，缺点在于空间开销较大。通过分治策略，归并排序能够确保在最坏情况下仍能保持较高的效率。


## 参考
[cnblog](https://www.cnblogs.com/chengxiao/p/6194356.html)