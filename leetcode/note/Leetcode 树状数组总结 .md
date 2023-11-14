---
title: 树状数组总结
date: 2023-11-13 22:53:26
tags: [Leetcode, 树状数组]
---

# 树状数组总结

## 适用场景

对一个数组做单点修改以及求任意前缀和/区间和

直接操作数组的单点修改时间复杂度 O(1)，求前缀和/区间和时间复杂度 O(n)，平均操作时间复杂度 O(n)。额外记录前缀和数组的单点修改时间复杂度 O(n)，求前缀和/区间和时间复杂度 O(1)，平均操作时间复杂度 O(n)。

树状数组这种数据结构的单点修改时间复杂度 O(logn)，求前缀和/区间和时间复杂度 O(logn)，平均操作时间复杂度 O(logn)。

## 算法原理

记录一个 A 数组一个 C 数组，A 数组保存每个数，C 数组保存以当前数作为结尾的一定长度的区间和，这个区间的长度=$2^n$，n 为当前位置索引的二进制表示的末尾 0 的个数。当我们构建好 A 数组和 C 数组后，如何求任意位置的前缀和呢？假设这个位置索引为 k，那么 k 前缀和=C[k]+（k-C[k]区间长度）的前缀和，而 C[k]区间长度是多少呢？根据定义是$2^n$，n 为 k 的二进制表示的末尾 0 的个数。这里其实有一种简便算法，即 C[k]的区间长度=k&-k，在模板中我们定义这个函数为 lowbit。接下来我们只需要重复这个过程，直到 0。这里可以明显的看出来，求前缀和操作的时间复杂度是 O(logn)。

那么接下来如何做单点修改呢？假如说我们要修改位置 k 的数，对于 A 数组我们直接修改即可，对于 C 数组，我们首先要修改 C[k]，然后要修改树状数组中 C[k]的父亲，然后再修改父亲的父亲，直到某个 C 没有父亲为止。当我们把树状数组画出来后，可以很容易地看出 k 的父亲索引为 k+lowbit(k)。因此我们只需要不断地修改，直到超出数组范围为止。这里可以明显的看出来，单点修改操作的时间复杂度也是 O(logn)。

那么当最初给定一个 nums 数组，如何初始化树状数组呢？我们可以首先初始化 A 数组和 C 数组，里面的元素默认都是 0。根据树状数组的定义，此时其实 A 数组和 C 数组已经满足树状数组的关系了，只不过数组元素都是 0。这时我们可以遍历 nums 数组中的每一个元素，对树状数组依次做单点更新即可。整个初始化过程的时间复杂度是 O(nlogn)。

- 树状数组空间复杂度$O(n)$

- 构造树状数组时间复杂度$O(nlogn)$

- 单点更新时间复杂度$O(logn)$

- 求任意位置前缀和时间复杂度$O(logn)$

- 求任意区间和时间复杂度$O(logn)$

## 模板代码

力扣题目链接：[https://leetcode.cn/problems/range-sum-query-mutable/](https://leetcode.cn/problems/range-sum-query-mutable/)

```java
class NumArray {
    private int[] A;
    private int[] C;

    public NumArray(int[] nums) {
        A = new int[nums.length + 1];
        C = new int[nums.length + 1];
        for (int i = 0; i < nums.length; i++) {
            update(i, nums[i]);
        }
    }

    public void update(int index, int val) {
        index++;
        int change = val - A[index];
        A[index] = val;
        while (index < C.length) {
            C[index] += change;
            index += lowbit(index);
        }
    }

    public int sumRange(int left, int right) {
        return prefix(right) - prefix(left - 1);
    }

    private int prefix(int index) {
        index++;
        int sum = 0;
        while (index > 0) {
            sum += C[index];
            index -= lowbit(index);
        }
        return sum;
    }

    private int lowbit(int m) {
        return m & -m;
    }
}
```

## 参考博客

https://blog.csdn.net/Jxianxu/article/details/108214752
