---
title: 209. 长度最小的子数组
date: 2023-10-30 14:15:31
tags: [Leetcode题解]
---

# 209.长度最小的子数组

力扣题目链接：[https://leetcode.cn/problems/minimum-size-subarray-sum/](https://leetcode.cn/problems/minimum-size-subarray-sum/)

<p>给定一个含有&nbsp;<code>n</code><strong>&nbsp;</strong>个正整数的数组和一个正整数 <code>target</code><strong> 。</strong></p>

<p>找出该数组中满足其总和大于等于<strong> </strong><code>target</code><strong> </strong>的长度最小的 <strong>连续子数组</strong>&nbsp;<code>[nums<sub>l</sub>, nums<sub>l+1</sub>, ..., nums<sub>r-1</sub>, nums<sub>r</sub>]</code> ，并返回其长度<strong>。</strong>如果不存在符合条件的子数组，返回 <code>0</code> 。</p>

<p>&nbsp;</p>

<p><strong>示例 1：</strong></p>

<pre>
<strong>输入：</strong>target = 7, nums = [2,3,1,2,4,3]
<strong>输出：</strong>2
<strong>解释：</strong>子数组&nbsp;<code>[4,3]</code>&nbsp;是该条件下的长度最小的子数组。
</pre>

<p><strong>示例 2：</strong></p>

<pre>
<strong>输入：</strong>target = 4, nums = [1,4,4]
<strong>输出：</strong>1
</pre>

<p><strong>示例 3：</strong></p>

<pre>
<strong>输入：</strong>target = 11, nums = [1,1,1,1,1,1,1,1]
<strong>输出：</strong>0
</pre>

<p>&nbsp;</p>

<p><strong>提示：</strong></p>

<ul>
	<li><code>1 &lt;= target &lt;= 10<sup>9</sup></code></li>
	<li><code>1 &lt;= nums.length &lt;= 10<sup>5</sup></code></li>
	<li><code>1 &lt;= nums[i] &lt;= 10<sup>5</sup></code></li>
</ul>

<p>&nbsp;</p>

<p><strong>进阶：</strong></p>

<ul>
	<li>如果你已经实现<em> </em><code>O(n)</code> 时间复杂度的解法, 请尝试设计一个 <code>O(n log(n))</code> 时间复杂度的解法。</li>
</ul>

## 方法一：暴力搜索

连续子数组等同于连续子序列，一个长为 n 的数组/字符串的连续子序列个数是$n+C^2_n$，因此我们可以直接两个 for 循环或者递归回溯来暴力搜索。另外看到连续子序列的和，也应该想到用两个前缀和相减求解的可能性。

- 时间复杂度$O(n^2)$
- 空间复杂度$O(1)$

## 方法二：滑动窗口

对于字符串/数组中涉及“连续”的问题，都可以考虑是否能用滑动窗口求解。对于本题，我们的 right 指针一直向右走直到连续子序列的和大于等于 target（如果走到头说明结束了），这时找到的就是所有以第一个数作为起始的连续子序列中的最短，然后 left 指针右移一位，然后再去判断并右移 right 指针。最终的出口有两个，一个是中途 left 超过 right，这时说明存在一个大于等于 target 的数，另一个是 right 走到尽头。

- 时间复杂度$O(n)$
- 空间复杂度$O(1)$

### AC 代码

```java
class Solution {
    public int minSubArrayLen(int target, int[] nums) {
        int left = 0, right = 0;
        int sum = 0;
        int res = Integer.MAX_VALUE;
        while (right < nums.length) {
            sum += nums[right];
            if (sum >= target) {
                while (sum >= target) {
                    res = Math.min(res, right - left + 1);
                    sum -= nums[left];
                    left++;
                }
            }
            right++;
        }
        return res == Integer.MAX_VALUE ? 0 : res;
    }
}
```
