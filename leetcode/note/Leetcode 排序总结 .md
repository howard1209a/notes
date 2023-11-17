---
title: 排序总结
date: 2023-11-17 11:07:11
tags: [Leetcode, 排序, 快速排序, 归并排序, 堆排序, 桶排序]
---

# 排序总结

## 冒泡排序

- 时间复杂度$O(n^2)$
- 空间复杂度$O(1)$
- 稳定

### 算法原理

以从小到大为例，每次把最大数放到后面。

### 模板代码

```java
public void bubbleSort(int[] nums) {
    for (int i = nums.length - 1; i >= 0; i--) {
        for (int j = 0; j < i; j++) {
            if (nums[j] > nums[j + 1]) {
                swap(nums, j, j + 1);
            }
        }
    }
}
```

## 选择排序

- 时间复杂度$O(n^2)$
- 空间复杂度$O(1)$
- 不稳定

### 算法原理

以从小到大为例，遍历一遍，找到最大的数和最后一个数交换，再遍历一遍找到最大的数和倒数第二个数交换。

### 模板代码

```java
public void selectionSort(int[] nums) {
    for (int i = nums.length - 1; i >= 0; i--) {
        int maxIndex = 0;
        for (int j = 0; j <= i; j++) {
            if (nums[j] > nums[maxIndex]) {
                maxIndex = j;
            }
        }
        swap(nums, maxIndex, i);
    }
}
```

## 插入排序

- 时间复杂度$O(n^2)$
- 空间复杂度$O(1)$
- 稳定

### 算法原理

以从小到大为例，假设前 k 个数已经排好序，对于第 k+1 个数，我们可以查看第 k+1 个数是否大于等于第 k 个数，如果成立则不需要变，如果不成立，我们把第 k 个数后移，再比较第 k+1 个数和第 k-1 个数，直到找到相应位置。

### 模板代码

```java
public void insertionSort(int[] nums) {
    for (int i = 0; i < nums.length; i++) {
        int num = nums[i];
        int j = i - 1;
        while (j >= 0 && nums[j] > num) {
            nums[j + 1] = nums[j];
            j--;
        }
        nums[j + 1] = num;
    }
}
```

## 快速排序

- 时间复杂度$O(nlogn)$
- 空间复杂度$O(logn)$
- 不稳定

### 算法原理

partition 方法的作用是，在一定区间内选定一个数，调整这个区间使得所有比这个数小的数都在它的左边，所有比这个数大的数都在它的右边。关于这个数的选择一般有三种方式：

- 选区间第一个数：简单直观，缺点是有可能带来最坏的情况
- 选区间随机一个数：不会带来最坏的情况，但是随机选取本身存在消耗
- 选区间最左边数、最右边数和中间数的中值：比较好的方式

partition 方法如何进行调整呢？我们把中间值换到 left，从右边开始遍历（right--），直到找到第一个小于中间值的数，交换中间值和这个小于中间值的数，然后从左边开始遍历（left++），直到找到第一个大于中间值的数，交换中间值和这个大于中间值的数...直到 left==right，此时中间值必然在 left 或 right 中，而中间值的左右都是小于/大于它的数。

quickSort 方法的含义是，对 left 到 right 的区间进行排序，首先进行 partition，根据 partition 返回的中间值的位置，将区间切分成两份向下递归。partition 使得左边的数都小于等于中值，右边的数都大于等于中值，这时只要再满足左边有序且右边有序就可以保证整体区间有序了。

快排的空间复杂度是由递归带来的，快排的时间复杂度考虑的是平均的情况，快排的最坏时间复杂度是$O(n^2)$，最好时间复杂度也是$O(nlogn)$。

### 模板代码

```java
public void quickSort(int[] nums) {
    quickSort(nums, 0, nums.length - 1);
}

private void quickSort(int[] nums, int left, int right) {
    if (left >= right) {
        return;
    }
    int partition = partition(nums, left, right);
    quickSort(nums, left, partition - 1);
    quickSort(nums, partition + 1, right);
}

private int partition(int[] nums, int left, int right) {
    int state = 0;
    while (left < right) {
        if (state == 0) {
            if (nums[right] < nums[left]) {
                swap(nums, left, right);
                state = 1;
            } else {
                right--;
            }
        } else if (state == 1) {
            if (nums[left] > nums[right]) {
                swap(nums, left, right);
                state = 0;
            } else {
                left++;
            }
        }
    }
    return left;
}
```

## 归并排序

- 时间复杂度$O(nlogn)$
- 空间复杂度$O(n)$
- 稳定

### 算法原理

对于一个[left,right]区间，我们对它们排序等价于：将整个区间分成[left,(left+right)/2]和[(left+right)/2+1,right]两个区间，对两个区间分别排序，然后利用额外的数组空间，对两个已经排好序的数组进行合并，最后将合并好的数组写回原数组。递归的最小条件是$left==right$，这时候相当于已经排好序了。

关于时间复杂度的分析，因为我们总是从中间切分区间，因此递归二叉树一共有$logn$层，而每层加起来有$O(n)$的操作，总体就是$O(nlogn)$的时间复杂度，空间复杂度是由于我们需要另一个数组用于合并操作。稳定是因为我们的判断条件是$if (nums[start1] <= nums[start2])$。

### 模板代码

```java
public void mergeSort(int[] nums) {
    mergeSort(nums, new int[nums.length], 0, nums.length - 1);
}

private void mergeSort(int[] nums, int[] tmpNums, int left, int right) {
    if (left == right) {
        return;
    }
    int mid = (left + right) / 2;
    mergeSort(nums, tmpNums, left, mid);
    mergeSort(nums, tmpNums, mid + 1, right);
    merge(nums, tmpNums, left, mid, mid + 1, right);
}
private void merge(int[] nums, int[] tmpNums, int start1, int end1, int start2, int end2) {
    int index = start1;
    while (start1 <= end1 && start2 <= end2) {
        if (nums[start1] <= nums[start2]) {
            tmpNums[index] = nums[start1];
            start1++;
        } else {
            tmpNums[index] = nums[start2];
            start2++;
        }
        index++;
    }
    if (start1 <= end1) {
        for (int i = start1; i <= end1; i++) {
            tmpNums[index] = nums[i];
            index++;
        }
    } else {
        for (int i = start2; i <= end2; i++) {
            tmpNums[index] = nums[i];
            index++;
        }
    }
    for (int i = start1; i <= end2; i++) {
        nums[i] = tmpNums[i];
    }
}
```

## 堆排序

- 时间复杂度$O(nlogn)$
- 空间复杂度$O(1)$ 原地调整
- 不稳定

### 算法原理

以大根堆为例，大根堆的定义为一个完全二叉树，其根节点大于等于左右子树所有节点，且左右子树均为大根堆。（注意，完全二叉树的左右子树一定是完全二叉树），堆排序的关键操作是 heapUp 和 heapDown。

heapUp 主要用于对于一个现有堆，我们在完全二叉树的末尾添加一个节点，然后自底向上进行调整的过程。对于添加节点的父节点，如果父节点大于等于添加节点，那么不用做任何事。如果父节点小于添加节点，两者交换，然后继续探查添加节点的父节点，直到父节点大于等于添加节点或添加节点被交换到根节点。

heapDown 主要用于当我们将一个堆的根节点替换为一个其他节点后，自顶向下进行调整的过程。如果根节点有两个子节点（如果根节点有一个子节点，则这个子节点就是更大者），取更大者与根节点相比较，如果更大者小于等于根节点，那么不用做任何事，反之交换根节点与更大者，重复这个过程直到根节点大于等于更大者或根节点延伸到了叶节点。

heapify 是建堆过程，我们首先初始化一个只有一个元素的堆，然后不断地将元素添加到完全二叉树的末尾并 heapUp 调整。

heapSort 排序过程我们采用原地调整的方式，交换堆顶的数和完全二叉树的最后一个数，然后 heapDown 调整（大根堆中少一个数，完全二叉树少一个数）。

- heapUp 操作的时间复杂度$O(logn)$
- heapDown 操作的时间复杂度$O(logn)$
- heapify 建堆的时间复杂度$O(nlogn)$
- heapSort 排序的时间复杂度$O(nlogn)$

### 模板代码

```java
public void heapSort(int[] nums) {
    heapify(nums);
    for (int i = nums.length - 1; i > 1; i--) {
        swap(nums, 1, i);
        heapDown(nums, i - 1);
    }
    for (int i = 1; i < nums.length; i++) {
        if (nums[i - 1] <= nums[i]) {
            return;
        } else {
            swap(nums, i - 1, i);
        }
    }
}

private void heapify(int[] nums) {
    for (int i = 1; i < nums.length; i++) {
        heapUp(nums, i);
    }
}

private void heapDown(int[] nums, int lastIndex) {
    int index = 1;
    while (index * 2 + 1 <= lastIndex) {
        int max = Math.max(nums[index * 2], nums[index * 2 + 1]);
        if (nums[index] >= max) {
            return;
        } else {
            swap(nums, index, nums[index * 2] > nums[index * 2 + 1] ? index * 2 : index * 2 + 1);
            index = (nums[index * 2] > nums[index * 2 + 1] ? index * 2 : index * 2 + 1);
        }
    }
    if (index * 2 == lastIndex) {
        if (nums[index * 2] > nums[index]) {
            swap(nums, index, index * 2);
        }
    }
}

private void heapUp(int[] nums, int index) {
    while (index / 2 != 0) {
        if (nums[index / 2] >= nums[index]) {
            return;
        } else {
            swap(nums, index / 2, index);
            index /= 2;
        }
    }
}
```

## 桶排序

- 最好情况时间复杂度$O(n)$
- 空间复杂度$O(n)$

适用于数据均匀分布的情况，即当我们切分多个区间时，每个区间的元素数目大致相等。

### 算法原理

桶排序将原数组划分到称为 「桶」 的多个区间中，然后对每个桶单独进行排序，之后再按桶序和桶内序输出结果。其中对于每个桶进行排序的排序算法可以自己选择。

找最大最小值和分配桶的过程耗费$O(n)$的时间复杂度和$O(n)$的空间复杂度，假设有 k 个桶，且数据分布均匀，若每个桶都采用$O(n^2)$的排序算法，那么总时间复杂度为$O(n^2/k)$，若每个桶都采用$O(nlogn)$的排序算法，总时间复杂度为$O(k*(n/k)*log(n/k))$，即$O(n*log(n/k))$，这时如果 $k=n/p$，p 是一个常数，那么理论上时间复杂度可以降到$O(n)$。

### 模板代码

```java
public int[] bucketSort(int[] arr) {
    if(arr.length < 2) return arr;
    int n = arr.length, k = n / 3, min = arr[0], max = arr[0]; // k=n/3 个桶
    for (int i = 1; i < n; i++) { // 确定 min 和 max
        min = Math.min(min, arr[i]);
        max = Math.max(max, arr[i]);
    }
    List<ArrayList<Integer>> buckets = new ArrayList<>(k);
    for (int i = 0; i < k; i++) { // k 个桶
        buckets.add(new ArrayList<>()); // 每个桶是一个ArrayList<Integer>
    }
    double interval = (max - min) * 1.0 / (k - 1); // 桶间隔
    for (int num : arr) { // 遍历arr，根据元素值将所有元素装入对应值区间的桶中
        int bucketIdx = (int) ((num - min) / interval); // arr[i] (num) 元素应该装入的桶的下标
        buckets.get(bucketIdx).add(num); // 装入对应桶中
    }
    for (ArrayList<Integer> bucket : buckets) {
        Collections.sort(bucket); // 桶内排序(调用库函数，从小到大)
    }
    int index = 0;
    for (ArrayList<Integer> bucket : buckets) {
        for (int sortedNum : bucket) {
            arr[index] = sortedNum; // 复用输入数组arr
            index++;
        }
    }
    return arr;
}
```

## 参考资料

https://leetcode.cn/circle/discuss/eBo9UB/
