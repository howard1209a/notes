---
title: 274.H 指数
date: 2023-10-30 12:42:50
tags: [Leetcode题解]
---

# 274.H 指数

力扣题目链接：[https://leetcode.cn/problems/h-index/](https://leetcode.cn/problems/h-index/)

<p>给你一个整数数组 <code>citations</code> ，其中 <code>citations[i]</code> 表示研究者的第 <code>i</code> 篇论文被引用的次数。计算并返回该研究者的 <strong><code>h</code><em>&nbsp;</em>指数</strong>。</p>

<p>根据维基百科上&nbsp;<a href="https://baike.baidu.com/item/h-index/3991452?fr=aladdin" target="_blank">h 指数的定义</a>：<code>h</code> 代表“高引用次数” ，一名科研人员的 <code>h</code><strong> 指数 </strong>是指他（她）至少发表了 <code>h</code> 篇论文，并且每篇论文<strong> 至少</strong> 被引用 <code>h</code> 次。如果 <code>h</code><em> </em>有多种可能的值，<strong><code>h</code> 指数 </strong>是其中最大的那个。</p>

<p>&nbsp;</p>

<p><strong>示例 1：</strong></p>

<pre>
<strong>输入：</strong><code>citations = [3,0,6,1,5]</code>
<strong>输出：</strong>3 
<strong>解释：</strong>给定数组表示研究者总共有 <code>5</code> 篇论文，每篇论文相应的被引用了 <code>3, 0, 6, 1, 5</code> 次。
&nbsp;    由于研究者有 <code>3 </code>篇论文每篇 <strong>至少 </strong>被引用了 <code>3</code> 次，其余两篇论文每篇被引用 <strong>不多于</strong> <code>3</code> 次，所以她的 <em>h </em>指数是 <code>3</code>。</pre>

<p><strong>示例 2：</strong></p>

<pre>
<strong>输入：</strong>citations = [1,3,1]
<strong>输出：</strong>1
</pre>

<p>&nbsp;</p>

<p><strong>提示：</strong></p>

<ul>
	<li><code>n == citations.length</code></li>
	<li><code>1 &lt;= n &lt;= 5000</code></li>
	<li><code>0 &lt;= citations[i] &lt;= 1000</code></li>
</ul>

## 方法一：排序

从大到小排序，排序之后从前向后遍历。判断第一个数是否$\geq1$，如果$\geq1$说明至少有 1 个数大于等于 1，继续向下看，如果第二个数$\geq2$，则说明至少有 2 个数大于等于 2，如果第二个数$<2$，则说明不够 2 个数大于等于 2，说明 h 指数是 1，不是 2 更不可能是 3,4...

- 时间复杂度$O(n\log n)$，其中$n = len(citations)$
- 空间复杂度$O(\log n)$

### AC 代码

```java
class Solution {
    public int hIndex(int[] citations) {
        Arrays.sort(citations);
        for (int i = citations.length - 1; i >= 0; i--) {
            if (citations[i] < citations.length - i) {
                return citations.length - i - 1;
            }
        }
        return citations.length;
    }
}
```

## 方法二：动态规划

数组下标定义为截至目前（包括当前这个数），h 指数的值以及大于等于 h 的数的个数，递归公式为，当我们新增下一个数时，如果下一个数大于等于 h+1 且前面至少有 h 个大于等于 h+1 的数，则 h 指数可以提高 1，否则不变。前面有多少个大于等于 h+1 的数我们是通过大于等于 h 的数的个数减去等于 h 的数的个数求得的，因此我们维护了一个 map。最后初始化 dp[0]并从左到右遍历即可。

- 时间复杂度$O(n)$，其中$n = len(citations)$
- 空间复杂度$O(n)$

### AC 代码

```java
class Solution {
    public int hIndex(int[] citations) {
        Map<Integer, Integer> map = new HashMap();
        int[][] dp = new int[citations.length][2];
        dp[0][0] = (citations[0] >= 1) ? 1 : 0;
        dp[0][1] = 1;
        map.put(citations[0], 1);
        for (int i = 1; i < citations.length; i++) {
            Integer num = map.getOrDefault(dp[i - 1][0], 0);
            if (citations[i] >= dp[i - 1][0] + 1 && (dp[i - 1][1] - num >= dp[i - 1][0])) {
                dp[i][0] = dp[i - 1][0] + 1;
                dp[i][1] = dp[i - 1][1] - num + 1;
            } else {
                dp[i][0] = dp[i - 1][0];
                dp[i][1] = dp[i - 1][1] + (citations[i] >= dp[i - 1][0] ? 1 : 0);
            }
            map.put(citations[i], map.getOrDefault(citations[i], 0) + 1);
        }
        return dp[citations.length - 1][0];
    }
}
```
