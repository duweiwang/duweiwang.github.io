---
title: 排序算法之插入排序
date: 2025-01-08 14:35:38
tags:
categories:
description:
---


# 插入类排序算法：直接插入排序与希尔排序

插入类排序算法是一类基于插入操作的排序算法，其核心思想是：逐步构建一个有序序列，将每个新元素插入到已排序部分的适当位置。本篇博客将介绍两种常见的插入类排序算法：**直接插入排序**和**希尔排序**。

---

## 1. 直接插入排序

### 算法思想
直接插入排序通过将每个新元素插入到已排序部分的正确位置，逐步扩大已排序部分的范围。其核心步骤包括：

1. 从第二个元素开始，依次将当前元素插入到前面已排序部分的适当位置；
2. 重复此过程，直到所有元素有序。

### Java 实现
以下是直接插入排序的 Java 实现：

```java
public class InsertionSort {
    public static void insertionSort(int[] arr) {
        int n = arr.length;
        for (int i = 1; i < n; i++) {
            int key = arr[i];
            int j = i - 1;

            // 将大于 key 的元素向后移动
            while (j >= 0 && arr[j] > key) {
                arr[j + 1] = arr[j];
                j--;
            }

            // 插入 key 到正确位置
            arr[j + 1] = key;
        }
    }

    public static void main(String[] args) {
        int[] arr = {64, 34, 25, 12, 22, 11, 90};
        insertionSort(arr);
        System.out.println("Sorted array:");
        for (int num : arr) {
            System.out.print(num + " ");
        }
    }
}
```

### 输出结果
```
Sorted array:
11 12 22 25 34 64 90
```

### 时间复杂度
- **最坏情况：** \(O(n^2)\)（逆序输入）
- **最佳情况：** \(O(n)\)（已排序输入）
- **平均情况：** \(O(n^2)\)

### 空间复杂度
- 空间复杂度：\(O(1)\)（原地排序）
- 排序稳定性：稳定

---

## 2. 希尔排序

### 算法思想
希尔排序是直接插入排序的一种改进版本，通过引入**增量分组**的思想，使元素在初始阶段能够跨越较大距离移动，以加快收敛速度。其核心步骤包括：

1. 按照某个增量（gap）将数组分为若干组，每组分别进行直接插入排序；
2. 缩小增量（gap），重复分组和排序；
3. 当增量缩小到 1 时，整个数组进行一次直接插入排序。

### Java 实现
以下是希尔排序的 Java 实现：

```java
public class ShellSort {
    public static void shellSort(int[] arr) {
        int n = arr.length;

        // 初始化增量 gap
        for (int gap = n / 2; gap > 0; gap /= 2) {
            // 对每组元素进行直接插入排序
            for (int i = gap; i < n; i++) {
                int temp = arr[i];
                int j = i;

                // 在当前分组中寻找插入位置
                while (j >= gap && arr[j - gap] > temp) {
                    arr[j] = arr[j - gap];
                    j -= gap;
                }

                arr[j] = temp;
            }
        }
    }

    public static void main(String[] args) {
        int[] arr = {64, 34, 25, 12, 22, 11, 90};
        shellSort(arr);
        System.out.println("Sorted array:");
        for (int num : arr) {
            System.out.print(num + " ");
        }
    }
}
```

### 输出结果
```
Sorted array:
11 12 22 25 34 64 90
```

### 时间复杂度
- **最坏情况：** \(O(n^{2})\)（具体与增量序列选择有关）
- **最佳情况：** \(O(n \log n)\)
- **平均情况：** \(O(n \log n)\)

### 空间复杂度
- 空间复杂度：\(O(1)\)
- 排序稳定性：不稳定（分组时可能破坏相同元素的相对顺序）

---

## 3. 直接插入排序与希尔排序的对比

| 特性               | 直接插入排序          | 希尔排序            |
|--------------------|----------------------|---------------------|
| **时间复杂度**      | \(O(n^2)\)          | \(O(n \log n)\)    |
| **空间复杂度**      | \(O(1)\)            | \(O(1)\)           |
| **排序稳定性**      | 稳定                 | 不稳定              |
| **适用场景**        | 小规模数据，简单场景  | 中到大规模数据，效率优先 |

---

## 总结

- **直接插入排序** 简单直观，适合小规模数据集或已接近有序的数据。
- **希尔排序** 引入分组思想，显著提升了排序效率，更适合处理中到大规模的数据。

通过学习这两种插入类排序算法，我们可以更加深入地理解插入排序的思想及其优化方向。
