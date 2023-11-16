---
title: 最小生成树总结
date: 2023-11-16 19:33:47
tags: [Leetcode, 最小生成树]
---

# 最小生成树总结

生成树是特殊的生成子图，生成子图的定义是点集不变，取边子集。树是连通无圈无向图，树的边数等于点数减 1。最小生成树指的是，边权之和最小的生成树。任意无向连通图必有生成树，因此也必有最小生成树。

## Kruskal 算法

首先为所有边从小到大排序，然后考虑包含所有点但不含任何边的初始图，依次取最小边尝试加入图中，如果出现圈就丢掉这条边，如果没有出现圈则加入图中，直到图中有点数减 1 条边，这时候我们就找到了一个最小生成树（最小生成树可能不止一个）。

### 1584.连接所有点的最小费用

力扣题目链接：[https://leetcode.cn/problems/min-cost-to-connect-all-points/](https://leetcode.cn/problems/min-cost-to-connect-all-points/)

<p>给你一个<code>points</code>&nbsp;数组，表示 2D 平面上的一些点，其中&nbsp;<code>points[i] = [x<sub>i</sub>, y<sub>i</sub>]</code>&nbsp;。</p>

<p>连接点&nbsp;<code>[x<sub>i</sub>, y<sub>i</sub>]</code> 和点&nbsp;<code>[x<sub>j</sub>, y<sub>j</sub>]</code>&nbsp;的费用为它们之间的 <strong>曼哈顿距离</strong>&nbsp;：<code>|x<sub>i</sub> - x<sub>j</sub>| + |y<sub>i</sub> - y<sub>j</sub>|</code>&nbsp;，其中&nbsp;<code>|val|</code>&nbsp;表示&nbsp;<code>val</code>&nbsp;的绝对值。</p>

<p>请你返回将所有点连接的最小总费用。只有任意两点之间 <strong>有且仅有</strong>&nbsp;一条简单路径时，才认为所有点都已连接。</p>

<p>&nbsp;</p>

<p><strong>示例 1：</strong></p>

<p><img alt="" src="https://assets.leetcode.com/uploads/2020/08/26/d.png" style="height:268px; width:214px; background:#e5e5e5" /></p>

<pre>
<strong>输入：</strong>points = [[0,0],[2,2],[3,10],[5,2],[7,0]]
<strong>输出：</strong>20
<strong>解释：
</strong><img alt="" src="https://assets.leetcode.com/uploads/2020/08/26/c.png" style="height:268px; width:214px; background:#e5e5e5" />
我们可以按照上图所示连接所有点得到最小总费用，总费用为 20 。
注意到任意两个点之间只有唯一一条路径互相到达。
</pre>

<p><strong>示例 2：</strong></p>

<pre>
<strong>输入：</strong>points = [[3,12],[-2,5],[-4,1]]
<strong>输出：</strong>18
</pre>

<p><strong>示例 3：</strong></p>

<pre>
<strong>输入：</strong>points = [[0,0],[1,1],[1,0],[-1,1]]
<strong>输出：</strong>4
</pre>

<p><strong>示例 4：</strong></p>

<pre>
<strong>输入：</strong>points = [[-1000000,-1000000],[1000000,1000000]]
<strong>输出：</strong>4000000
</pre>

<p><strong>示例 5：</strong></p>

<pre>
<strong>输入：</strong>points = [[0,0]]
<strong>输出：</strong>0
</pre>

<p>&nbsp;</p>

<p><strong>提示：</strong></p>

<ul>
	<li><code>1 &lt;= points.length &lt;= 1000</code></li>
	<li><code>-10<sup>6</sup>&nbsp;&lt;= x<sub>i</sub>, y<sub>i</sub> &lt;= 10<sup>6</sup></code></li>
	<li>所有点&nbsp;<code>(x<sub>i</sub>, y<sub>i</sub>)</code>&nbsp;两两不同。</li>
</ul>

#### 方法一：Kruskal 算法+并查集

本题要考虑任意两点间的所有边，我们首先生成所有边，然后从小到大排序，在尝试将边加入图中的时候，用并查集来检查是否会出现圈，因为并查集可以返回任意两点是否连通。

- 时间复杂度$O(n^2logn)$
- 空间复杂度$O(n^2)$

##### AC 代码

```java
class Solution {
    private static class Edge {
        public int source;
        public int destination;
        public int length;

        public Edge(int source, int destination, int length) {
            this.source = source;
            this.destination = destination;
            this.length = length;
        }
    }

    private static class UnionSet {
        private int[] father;

        public UnionSet(int n) {
            father = new int[n];
            for (int i = 0; i < n; i++) {
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

    public int minCostConnectPoints(int[][] points) {
        List<Edge> edges = new ArrayList<>();
        for (int i = 0; i < points.length; i++) {
            for (int j = i + 1; j < points.length; j++) {
                edges.add(new Edge(i, j, distance(points, i, j)));
            }
        }
        Collections.sort(edges, (edge1, edge2) -> {
            if (edge1.length == edge2.length) {
                return 0;
            }
            return edge1.length > edge2.length ? 1 : -1;
        });
        UnionSet unionSet = new UnionSet(points.length);
        int sum = 0;
        for (int i = 0; i < edges.size(); i++) {
            Edge edge = edges.get(i);
            if (unionSet.isSame(edge.source, edge.destination)) {
                continue;
            }
            sum += edge.length;
            unionSet.join(edge.source, edge.destination);
        }
        return sum;
    }

    private int distance(int[][] points, int x, int y) {
        return Math.abs(points[x][0] - points[y][0]) + Math.abs(points[x][1] - points[y][1]);
    }
}
```
