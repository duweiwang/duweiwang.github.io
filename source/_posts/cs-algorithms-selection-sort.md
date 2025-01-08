---
title: 排序算法之选择排序
date: 2025-01-08 14:35:52
tags:
categories:
description:
---


# 选择类排序算法：选择排序与堆排序

在本篇博客中，我们将探讨两种常见的选择类排序算法：**选择排序**和**堆排序**。它们都基于“每次选择最小（或最大）元素并将其放到正确位置”的思想，但实现方式和效率有所不同。

## 1. 选择排序

### 算法思想
选择排序是一种简单直观的排序算法。其核心思想是：

1. 在未排序部分中找到最小元素；
2. 将该元素与未排序部分的第一个元素交换；
3. 重复以上过程，直到所有元素有序。

### Java 实现
以下是选择排序的 Java 实现：

```java
public class SelectionSort {
    public static void selectionSort(int[] arr) {
        int n = arr.length;
        for (int i = 0; i < n - 1; i++) {
            // 找到未排序部分的最小值
            int minIndex = i;
            for (int j = i + 1; j < n; j++) {
                if (arr[j] < arr[minIndex]) {
                    minIndex = j;
                }
            }
            // 交换最小值与未排序部分的第一个元素
            int temp = arr[minIndex];
            arr[minIndex] = arr[i];
            arr[i] = temp;
        }
    }

    public static void main(String[] args) {
        int[] arr = {64, 25, 12, 22, 11};
        selectionSort(arr);
        System.out.println("Sorted array: ");
        for (int num : arr) {
            System.out.print(num + " ");
        }
    }
}
```

### 输出结果
```
Sorted array: 
11 12 22 25 64
```

### 时间复杂度
- 最佳情况：\(O(n^2)\)
- 最坏情况：\(O(n^2)\)
- 平均情况：\(O(n^2)\)

### 空间复杂度
- 空间复杂度：\(O(1)\)
- 是否稳定：不稳定

---

## 2. 堆排序

### 算法思想
堆排序是一种基于堆（Heap）这种数据结构的选择排序算法。其核心步骤如下：

1. **构建堆：** 将无序数组构建为一个最大堆；
2. **选择最大值：** 堆顶元素即为未排序部分的最大值；
3. **移除堆顶：** 将堆顶元素与堆的最后一个元素交换，并对剩余部分重新堆化；
4. **重复：** 不断缩小堆的范围，直到堆为空。

### Java 实现
以下是堆排序的 Java 实现：

```java
public class HeapSort {
    public static void heapSort(int[] arr) {
        int n = arr.length;

        // 构建最大堆
        for (int i = n / 2 - 1; i >= 0; i--) {
            heapify(arr, n, i);
        }

        // 提取元素，调整堆
        for (int i = n - 1; i > 0; i--) {
            // 将堆顶与当前未排序部分的最后一个元素交换
            int temp = arr[0];
            arr[0] = arr[i];
            arr[i] = temp;

            // 对剩余部分重新堆化
            heapify(arr, i, 0);
        }
    }

    private static void heapify(int[] arr, int n, int i) {
        int largest = i;  // 初始化最大值为根节点
        int left = 2 * i + 1;
        int right = 2 * i + 2;

        // 左子节点是否大于根节点
        if (left < n && arr[left] > arr[largest]) {
            largest = left;
        }

        // 右子节点是否大于当前最大值
        if (right < n && arr[right] > arr[largest]) {
            largest = right;
        }

        // 如果最大值不是根节点，交换并继续堆化
        if (largest != i) {
            int temp = arr[i];
            arr[i] = arr[largest];
            arr[largest] = temp;

            heapify(arr, n, largest);
        }
    }

    public static void main(String[] args) {
        int[] arr = {64, 25, 12, 22, 11};
        heapSort(arr);
        System.out.println("Sorted array: ");
        for (int num : arr) {
            System.out.print(num + " ");
        }
    }
}
```

### 输出结果
```
Sorted array: 
11 12 22 25 64
```

### 时间复杂度
- 最佳情况：\(O(n \log n)\)
- 最坏情况：\(O(n \log n)\)
- 平均情况：\(O(n \log n)\)

### 空间复杂度
- 空间复杂度：\(O(1)\)
- 是否稳定：不稳定

---

## 3. 选择排序与堆排序的对比

| 特性               | 选择排序              | 堆排序                |
|--------------------|----------------------|----------------------|
| **时间复杂度**      | \(O(n^2)\)          | \(O(n \log n)\)      |
| **空间复杂度**      | \(O(1)\)            | \(O(1)\)            |
| **排序稳定性**      | 不稳定               | 不稳定               |
| **适用场景**        | 小规模数据，教学演示  | 大规模数据，工程应用  |

---

## 总结

- **选择排序** 适合用来学习排序算法的基本思想，简单易懂，但性能较低；
- **堆排序** 通过堆数据结构优化了选择过程，适合大规模数据的排序。

通过理解这两种排序算法，我们可以更深入地了解选择类排序的思想以及算法优化的重要性！



