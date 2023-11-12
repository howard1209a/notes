---
title: 84.柱状图中最大的矩形
date: 2023-11-12 14:42:02
tags: [Leetcode, 单调栈]
---

# 【LetMeFly】84.柱状图中最大的矩形

力扣题目链接：[https://leetcode.cn/problems/largest-rectangle-in-histogram/](https://leetcode.cn/problems/largest-rectangle-in-histogram/)

<p>给定 <em>n</em> 个非负整数，用来表示柱状图中各个柱子的高度。每个柱子彼此相邻，且宽度为 1 。</p>

<p>求在该柱状图中，能够勾勒出来的矩形的最大面积。</p>

<p> </p>

<p><strong>示例 1:</strong></p>

<p><img src="https://assets.leetcode.com/uploads/2021/01/04/histogram.jpg" /></p>

<pre>
<strong>输入：</strong>heights = [2,1,5,6,2,3]
<strong>输出：</strong>10
<strong>解释：</strong>最大的矩形为图中红色区域，面积为 10
</pre>

<p><strong>示例 2：</strong></p>

<p><img src="https://assets.leetcode.com/uploads/2021/01/04/histogram-1.jpg" /></p>

<pre>
<strong>输入：</strong> heights = [2,4]
<b>输出：</b> 4</pre>

<p> </p>

<p><strong>提示：</strong></p>

<ul>
	<li><code>1 <= heights.length <=10<sup>5</sup></code></li>
	<li><code>0 <= heights[i] <= 10<sup>4</sup></code></li>
</ul>

## 方法一：单调栈

在柱状图中勾勒任何一个矩形，这个矩形的上边沿必定覆盖至少一个柱子的上边沿，沿着任何一个柱子的上边沿去勾勒矩形，勾勒出来的矩形一定是一个符合要求的矩形。因此问题转化为了沿着所有柱子的上边沿可以勾勒出来的所有矩形中，矩形面积最大是多少。根据贪心，我们沿着任何一个矩形的上边沿勾勒时，往两边走肯定越长越好，可以利用单调栈求出任何一个柱子左边和右边第一个更矮的柱子的位置，这样就可以求出所有矩形面积了。

- 时间复杂度$O(n)$
- 空间复杂度$O(n)$

### AC 代码

```java
class Solution {
    public int largestRectangleArea(int[] heights) { // 左边第一个小于的->严格递增，右边第一个小于的->非严格递增
        int[] leftFirstSmaller = new int[heights.length];
        Arrays.fill(leftFirstSmaller, -1);
        int[] rightFirstSmaller = new int[heights.length];
        Arrays.fill(rightFirstSmaller, -1);
        Deque<Integer> stack = new LinkedList<>();
        for (int i = 0; i < heights.length; i++) {
            while (!stack.isEmpty() && heights[stack.peekLast()] >= heights[i]) {
                stack.pollLast();
            }
            if (!stack.isEmpty()) {
                leftFirstSmaller[i] = stack.peekLast();
            }
            stack.add(i);
        }
        stack.clear();
        for (int i = 0; i < heights.length; i++) {
            while (!stack.isEmpty() && heights[stack.peekLast()] > heights[i]) {
                Integer num = stack.pollLast();
                rightFirstSmaller[num] = i;
            }
            stack.add(i);
        }
        int max = 0;
        for (int i = 0; i < heights.length; i++) {
            int left = (leftFirstSmaller[i] != -1 ? leftFirstSmaller[i] + 1 : 0);
            int right = (rightFirstSmaller[i] != -1 ? rightFirstSmaller[i] - 1 : heights.length - 1);
            max = Math.max(max, (right - left + 1) * heights[i]);
        }
        return max;
    }
}
```
