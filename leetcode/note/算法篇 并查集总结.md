---
title: 并查集总结
date: 2023-11-16 17:41:46
tags: [Leetcode总结]
---

# 并查集总结

## 应用场景

并查集只用一个数组就可以维护所有元素的集合关系，并查集提供两个方法 join 和 isSame，join 方法可以合并任意两个元素所处集合，isSame 方法可以判断任意两个元素是否处于同一个集合。当我们需要判断两个元素是否在同一个集合里的时候，我们就要想到用并查集。并查集常常用于连通性问题，不同的连通分量是不同的集合，两点不在同一个集合等同于两点不在同一个连通分量，也就是不连通。

## 算法原理

我们定义一个长度为元素个数的 father 数组，father[x]=y 代表着 x 元素的父亲是 y 元素，我们定义根元素为 father[t]=t，即根元素的父亲是它自己。显然对于任何元素，我们可以不断找父亲从而一直追溯到根元素，我们定义如果两个元素的根元素相同等价于两个元素在同一个集合。

在初始化 father 数组时，我们令所有元素的父亲都是它自己，即 father[i]=i，这意味着最开始每个元素各自处于一个集合。

路径压缩方法可以显著提高并查集的性能，思路也很简单，就是在 find 过程中，一旦找到了根元素，就把寻找路径上的所有节点的 father 都更新为根元素，这样操作不会影响并查集表示的集合关系，但是可以让之后的 find 函数走一步就找到根元素，显著提高效率。

- 三种操作的时间复杂度都是$O(logn)$
- 空间复杂度$O(n)$

路径压缩后的并查集时间复杂度在 $O(logn)$与 $O(1)$之间，且随着查询或者合并操作的增加，时间复杂度会越来越趋于 O(1)。在第一次查询的时候，相当于是 n 叉树上从叶子节点到根节点的查询过程，时间复杂度是 $O(logn)$，但路径压缩后，后面的查询操作都是 $O(1)$。

## 模板代码

```java
class Solution {
    private int[] father;

    private void init() {
        for (int i = 0; i < father.length; i++) {
            father[i] = i;
        }
    }

    private int find(int u) {
        if (father[u] == u) {
            return u;
        }
        father[u] = find(father[u]);
        return father[u];
    }

    private boolean isSame(int x, int y) {
        return find(x) == find(y);
    }

    private void join(int x, int y) {
        x = find(x);
        y = find(y);
        if (x != y) {
            father[y] = x;
        }
    }
}
```

## 相关题目

力扣题目链接：[https://leetcode.cn/problems/find-if-path-exists-in-graph/](https://leetcode.cn/problems/find-if-path-exists-in-graph/)

力扣题目链接：[https://leetcode.cn/problems/redundant-connection/](https://leetcode.cn/problems/redundant-connection/)
