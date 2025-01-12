---
title: 排序算法之交换排序
date: 2025-01-08 16:19:28
tags:
categories:
description:
---

--
# 交换类排序算法：冒泡排序与快速排序

交换类排序算法是一类通过交换元素位置来实现排序的算法。本篇博客将介绍两种经典的交换类排序算法：**冒泡排序**和**快速排序**，并通过 Java 代码实现具体的排序过程。

---

## 1. 冒泡排序

### 算法思想
冒泡排序通过多次比较相邻元素，将较大的元素逐步“冒泡”到序列的末尾。其核心步骤包括：

1. 从第一个元素开始，依次比较相邻元素；
2. 如果当前元素大于后一个元素，则交换两者的位置；
3. 重复上述过程，每轮确定一个最大元素的位置；
4. 重复 n-1 轮后，数组完全有序。

### Java 实现
以下是冒泡排序的 Java 实现：

```java
public class BubbleSort {
    public static void bubbleSort(int[] arr) {
        int n = arr.length;
        for (int i = 0; i < n - 1; i++) {
            // 提前退出标志位
            boolean swapped = false;
            for (int j = 0; j < n - 1 - i; j++) {
                if (arr[j] > arr[j + 1]) {
                    // 交换相邻元素
                    int temp = arr[j];
                    arr[j] = arr[j + 1];
                    arr[j + 1] = temp;
                    swapped = true;
                }
            }
            // 如果没有发生交换，提前结束
            if (!swapped) break;
        }
    }

    public static void main(String[] args) {
        int[] arr = {64, 34, 25, 12, 22, 11, 90};
        bubbleSort(arr);
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
- **最坏情况：** \(O(n^2)\)（完全逆序）
- **最佳情况：** \(O(n)\)（已排序，提前退出）
- **平均情况：** \(O(n^2)\)

### 空间复杂度
- 空间复杂度：\(O(1)\)（原地排序）
- 排序稳定性：稳定

---

## 2. 快速排序

### 算法思想
快速排序是基于分治思想的高效排序算法。其核心步骤包括：

1. 从序列中选择一个基准元素（通常为第一个或最后一个）；
2. 将序列分为两部分：一部分小于基准，一部分大于基准；
3. 分别对两部分递归进行快速排序；
4. 合并结果，最终得到有序序列。

### Java 实现
以下是快速排序的 Java 实现：

```java
public class QuickSort {
    public static void quickSort(int[] arr, int low, int high) {
        if (low < high) {
            // 获取分区索引
            int pi = partition(arr, low, high);

            // 对左右子数组递归排序
            quickSort(arr, low, pi - 1);
            quickSort(arr, pi + 1, high);
        }
    }

    private static int partition(int[] arr, int low, int high) {
        // 选择最后一个元素作为基准
        int pivot = arr[high];
        int i = low - 1;

        for (int j = low; j < high; j++) {
            // 如果当前元素小于或等于基准
            if (arr[j] <= pivot) {
                i++;
                // 交换元素
                int temp = arr[i];
                arr[i] = arr[j];
                arr[j] = temp;
            }
        }

        // 将基准元素放到正确位置
        int temp = arr[i + 1];
        arr[i + 1] = arr[high];
        arr[high] = temp;

        return i + 1;
    }

    public static void main(String[] args) {
        int[] arr = {64, 34, 25, 12, 22, 11, 90};
        quickSort(arr, 0, arr.length - 1);
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
- **最坏情况：** \(O(n^2)\)（每次选择的基准导致极端不平衡）
- **最佳情况：** \(O(n \log n)\)（每次平衡划分）
- **平均情况：** \(O(n \log n)\)

### 空间复杂度
- 空间复杂度：\(O(\log n)\)（递归栈的深度）
- 排序稳定性：不稳定

---

## 3. 冒泡排序与快速排序的对比

| 特性               | 冒泡排序              | 快速排序             |
|--------------------|----------------------|---------------------|
| **时间复杂度**      | \(O(n^2)\)          | \(O(n \log n)\)    |
| **空间复杂度**      | \(O(1)\)            | \(O(\log n)\)       |
| **排序稳定性**      | 稳定                 | 不稳定              |
| **适用场景**        | 小规模数据，简单场景  | 大规模数据，高效排序 |

---

## 总结

- **冒泡排序** 简单易懂，但效率较低，适合学习排序思想或处理少量数据。
- **快速排序** 利用分治思想，是实际应用中常用的高效排序算法，尤其适合大规模数据。

通过理解这两种交换类排序算法，我们可以更深入地了解排序算法的多样性及其在不同场景中的应用。
