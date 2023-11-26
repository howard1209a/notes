---
title: 1049.最后一块石头的重量 II
date: 2023-11-11 10:08:33
tags: [Leetcode题解]
---

# 1049.最后一块石头的重量 II

力扣题目链接：[https://leetcode.cn/problems/last-stone-weight-ii/](https://leetcode.cn/problems/last-stone-weight-ii/)

<p>有一堆石头，用整数数组&nbsp;<code>stones</code> 表示。其中&nbsp;<code>stones[i]</code> 表示第 <code>i</code> 块石头的重量。</p>

<p>每一回合，从中选出<strong>任意两块石头</strong>，然后将它们一起粉碎。假设石头的重量分别为&nbsp;<code>x</code> 和&nbsp;<code>y</code>，且&nbsp;<code>x &lt;= y</code>。那么粉碎的可能结果如下：</p>

<ul>
	<li>如果&nbsp;<code>x == y</code>，那么两块石头都会被完全粉碎；</li>
	<li>如果&nbsp;<code>x != y</code>，那么重量为&nbsp;<code>x</code>&nbsp;的石头将会完全粉碎，而重量为&nbsp;<code>y</code>&nbsp;的石头新重量为&nbsp;<code>y-x</code>。</li>
</ul>

<p>最后，<strong>最多只会剩下一块 </strong>石头。返回此石头 <strong>最小的可能重量 </strong>。如果没有石头剩下，就返回 <code>0</code>。</p>

<p>&nbsp;</p>

<p><strong>示例 1：</strong></p>

<pre>
<strong>输入：</strong>stones = [2,7,4,1,8,1]
<strong>输出：</strong>1
<strong>解释：</strong>
组合 2 和 4，得到 2，所以数组转化为 [2,7,1,8,1]，
组合 7 和 8，得到 1，所以数组转化为 [2,1,1,1]，
组合 2 和 1，得到 1，所以数组转化为 [1,1,1]，
组合 1 和 1，得到 0，所以数组转化为 [1]，这就是最优值。
</pre>

<p><strong>示例 2：</strong></p>

<pre>
<strong>输入：</strong>stones = [31,26,33,21,40]
<strong>输出：</strong>5
</pre>

<p>&nbsp;</p>

<p><strong>提示：</strong></p>

<ul>
	<li><code>1 &lt;= stones.length &lt;= 30</code></li>
	<li><code>1 &lt;= stones[i] &lt;= 100</code></li>
</ul>

## 方法一：01 背包

本题的难点在于如何将问题转化为 01 背包，我们可以将石头分成两个非空子集，从每个子集中分别取出一个石头来做消除，直到其中至少一个子集为空，这时候剩下的石头重量其实就等于两个非空子集石头重量总和的差值，也就是问题转化为了如何划分子集让它们的总和之差最小，即找到总和最接近 sum/2 的子集，这就是典型的 01 背包问题了。显然的是，划分子集互相消除的方式和不断任选两个消除的方式其实是等价的。

- 时间复杂度$O(n*sum)$
- 空间复杂度$O(sum)$

### AC 代码

```java
class Solution {
    public int lastStoneWeightII(int[] stones) {
        if (stones.length == 1) {
            return stones[0];
        }
        int sum = 0;
        for (int num : stones) {
            sum += num;
        }
        int length = sum / 2;
        int[] dp = new int[length + 1];
        for (int i = 0; i < dp.length; i++) {
            if (stones[0] <= i) {
                dp[i] = stones[0];
            }
        }
        for (int i = 1; i < stones.length; i++) {
            for (int j = dp.length - 1; j >= 0; j--) {
                if (j >= stones[i]) {
                    dp[j] = Math.max(dp[j], stones[i] + dp[j - stones[i]]);
                }
            }
        }
        return sum - 2 * dp[dp.length - 1];
    }
}
```
