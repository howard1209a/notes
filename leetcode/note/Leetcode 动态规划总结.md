---
title: 动态规划总结
date: 2023-11-11 22:07:48
tags: [Leetcode, 动态规划, 子序列]
---

# 动态规划总结

## 标准解题步骤

1. 确定 dp 数组下标的含义
2. 确定递推公式
3. 确定 dp 数组如何初始化
4. 确定遍历顺序
5. dp 数组填充完毕后，如何对应得出题目最终解

## 打家劫舍系列

数组下标含义（有的题可能需要稍微变化）：先偷当前这家，当前这家之后的家随便偷，所获的最大收益。

力扣题目链接：[https://leetcode.cn/problems/house-robber/](https://leetcode.cn/problems/house-robber/)

力扣题目链接：[https://leetcode.cn/problems/house-robber-ii/](https://leetcode.cn/problems/house-robber-ii/)

力扣题目链接：[https://leetcode.cn/problems/house-robber-iii/](https://leetcode.cn/problems/house-robber-iii/)

## 股票系列

数组下标含义（有的题可能需要稍微变化）：

- 第一种：当天闭市时，持有股票所获最大收益。当天闭市时，未持有股票所获最大收益。
- 第二种：当天闭市时，持有股票且当天买入所获最大收益。当天闭市时，持有股票且当天未买入所获最大收益。当天闭市时，未持有股票且当天卖出所获最大收益。当天闭市时，未持有股票且当天未卖出所获最大收益。

力扣题目链接：[https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/)

力扣题目链接：[https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/)

力扣题目链接：[https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iii/](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iii/)

力扣题目链接：[https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iv/](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iv/)

力扣题目链接：[https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-cooldown/](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-cooldown/)

力扣题目链接：[https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/)

## 子序列系列

- 最长严格递增子序列

  数组下标含义：dp[i] = 最后一个数为 nums[i]的所有子序列中最长严格递增子序列的长度

  力扣题目链接：[https://leetcode.cn/problems/longest-increasing-subsequence/](https://leetcode.cn/problems/longest-increasing-subsequence/)

- 最长严格递增连续子序列：

  数组下标含义：dp[i] = 最后一个数为 nums[i]的所有连续子序列中最长严格递增连续子序列的长度

  力扣题目链接：[https://leetcode.cn/problems/longest-continuous-increasing-subsequence/](https://leetcode.cn/problems/longest-continuous-increasing-subsequence/)

- 最长公共连续子序列：

  数组下标含义：dp[i][j] = 把 nums1[i]和 nums2[j]对齐，一起向前走，最长能走多远。或者说以 nums1[i]结尾的所有连续子序列和以 nums2[j]结尾的所有连续子序列，相等的最大长度。

  力扣题目链接：[https://leetcode.cn/problems/maximum-length-of-repeated-subarray/](https://leetcode.cn/problems/maximum-length-of-repeated-subarray/)

- 最长公共子序列：

  数组下标含义：dp[i][j] = nums1[0]到 nums1[i]，nums2[0]到 nums2[j]，它们的最长公共子序列长度。

  力扣题目链接：[https://leetcode.cn/problems/longest-common-subsequence/](https://leetcode.cn/problems/longest-common-subsequence/)
