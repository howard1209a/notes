---
title: 线段树总结
date: 2023-11-17 09:51:25
tags: [Leetcode, 线段树]
---

# 线段树总结

## 适用场景

线段树是著名的用于高效求解 「区间问题」 的数据结构。只需要维护一个数组，可以以$O(logn)$的时间复杂度，完成数组单点查询、单点修改、区间求和、区间求最大值、区间求最小值的操作。

例题：力扣题目链接：[https://leetcode.cn/problems/range-sum-query-mutable/](https://leetcode.cn/problems/range-sum-query-mutable/)

## 算法原理

我们通过数组维护一个完全二叉树，每个节点保存了一个区间的和/最大值/最小值，数组长度一般粗略的设置为 4n，n 为元素个数。根节点保存了整个区间的和/最大值/最小值，左子节点保存了[left,(left+right)/2]区间的和/最大值/最小值，右子节点保存了[(left+right)/2+1,right]区间的和/最大值/最小值。所有叶子节点都只保存了单个点的值。上面的切分方法保证了最终是一个完全二叉树。

如果我们需要$O(logn)$的区间求和操作，则节点就保存区间的和，需要$O(logn)$的区间求最大值，则节点就保存区间的最大值，需要$O(logn)$的区间求最小值，则节点就保存区间的最小值。如果三个操作都需要同时$O(logn)$，则我们需要同时维护 3 个数组。

如果我们只维护一个区间和数组，那么求区间和的操作是$O(logn)$，求区间最大值/最小值操作是$O(n)$。

观察模板代码中的 sum、min、max 方法，明明是二叉树的递归后序遍历，为什么时间复杂度是$O(logn)$呢？原因在于$if (start == left \&\& end == right)$这一条剪枝，模拟一下就可以知道，这条剪枝使得遍历时每一层最多遍历四个节点。

query 方法和 update 方法的时间复杂度为$O(logn)$是显然的。

## 模板代码

模板代码维护了三个数组，同时支持 O(logn)$的数组单点查询、单点修改、区间求和、区间求最大值、区间求最小值。

```java
class SegmentTreeBasic {
    private int[] treeSum, treeMax, treeMin;
    private int n;

    public SegmentTreeBasic(int[] nums) {
        n = nums.length;
        treeSum = new int[n * 4];
        treeMax = new int[n * 4];
        treeMin = new int[n * 4];
        build(nums, 0, nums.length - 1, 1);
    }

    private void build(int[] nums, int left, int right, int index) {
        if (left == right) {
            treeSum[index] = nums[left];
            treeMax[index] = nums[left];
            treeMin[index] = nums[left];
            return;
        }
        int mid = (left + right) / 2;
        build(nums, left, mid, index * 2);
        build(nums, mid + 1, right, index * 2 + 1);
        updateSumNode(index);
        updateMinNode(index);
        updateMaxNode(index);
    }

    public int query(int id) {
        return query(id, 0, n - 1, 1);
    }

    private int query(int id, int left, int right, int index) {
        if (left == right) {
            return treeSum[index];
        }
        int mid = (left + right) / 2;
        if (id <= mid) {
            return query(id, left, mid, index * 2);
        } else {
            return query(id, mid + 1, right, index * 2 + 1);
        }
    }

    public void update(int id, int num) {
        update(id, num, 0, n - 1, 1);
    }

    private void update(int id, int num, int left, int right, int index) {
        if (left == right) {
            treeSum[index] = num;
            treeMax[index] = num;
            treeMin[index] = num;
            return;
        }
        int mid = (left + right) / 2;
        if (id <= mid) {
            update(id, num, left, mid, index * 2);
        } else {
            update(id, num, mid + 1, right, index * 2 + 1);
        }
        updateSumNode(index);
        updateMinNode(index);
        updateMaxNode(index);
    }

    public int sum(int left, int right) {
        return sum(0, n - 1, left, right, 1);
    }

    private int sum(int start, int end, int left, int right, int index) {
        if (start == left && end == right) {
            return treeSum[index];
        }
        int mid = (start + end) / 2;
        int sum = 0;
        if (left <= mid) {
            sum += sum(start, mid, left, Math.min(mid, right), index * 2);
        }
        if (right > mid) {
            sum += sum(mid + 1, end, Math.max(mid + 1, left), right, index * 2 + 1);
        }
        return sum;
    }

    public int max(int left, int right) {
        return max(0, n - 1, left, right, 1);
    }

    private int max(int start, int end, int left, int right, int index) {
        if (start == left && end == right) {
            return treeMax[index];
        }
        int mid = (start + end) / 2;
        int leftValue = Integer.MIN_VALUE, rightValue = Integer.MIN_VALUE;
        if (left <= mid) {
            leftValue = max(start, mid, left, Math.min(mid, right), index * 2);
        }
        if (right > mid) {
            rightValue = max(mid + 1, end, Math.max(mid + 1, left), right, index * 2 + 1);
        }
        return Math.max(leftValue, rightValue);
    }

    public int min(int left, int right) {
        return min(0, n - 1, left, right, 1);
    }

    private int min(int start, int end, int left, int right, int index) {
        if (start == left && end == right) {
            return treeMin[index];
        }
        int mid = (start + end) / 2;
        int leftValue = Integer.MAX_VALUE, rightValue = Integer.MAX_VALUE;
        if (left <= mid) {
            leftValue = min(start, mid, left, Math.min(mid, right), index * 2);
        }
        if (right > mid) {
            rightValue = min(mid + 1, end, Math.max(mid + 1, left), right, index * 2 + 1);
        }
        return Math.min(leftValue, rightValue);
    }

    private void updateSumNode(int index) {
        treeSum[index] = treeSum[index * 2] + treeSum[index * 2 + 1];
    }

    private void updateMaxNode(int index) {
        treeMax[index] = Math.max(treeMax[index * 2], treeMax[index * 2 + 1]);
    }

    private void updateMinNode(int index) {
        treeMin[index] = Math.min(treeMin[index * 2], treeMin[index * 2 + 1]);
    }
}
```

## 参考资料

https://leetcode.cn/circle/discuss/H4aMOn/
