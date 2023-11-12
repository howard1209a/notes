---
title: 42. 接雨水
date: 2023-11-12 14:09:35
tags: [Leetcode, 单调栈]
---

# 42.接雨水

力扣题目链接：[https://leetcode.cn/problems/trapping-rain-water/](https://leetcode.cn/problems/trapping-rain-water/)

<p>给定&nbsp;<code>n</code> 个非负整数表示每个宽度为 <code>1</code> 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。</p>

<p>&nbsp;</p>

<p><strong>示例 1：</strong></p>

<p><img src="https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/22/rainwatertrap.png" style="height: 161px; width: 412px;" /></p>

<pre>
<strong>输入：</strong>height = [0,1,0,2,1,0,1,3,2,1,2,1]
<strong>输出：</strong>6
<strong>解释：</strong>上面是由数组 [0,1,0,2,1,0,1,3,2,1,2,1] 表示的高度图，在这种情况下，可以接 6 个单位的雨水（蓝色部分表示雨水）。 
</pre>

<p><strong>示例 2：</strong></p>

<pre>
<strong>输入：</strong>height = [4,2,0,3,2,5]
<strong>输出：</strong>9
</pre>

<p>&nbsp;</p>

<p><strong>提示：</strong></p>

<ul>
	<li><code>n == height.length</code></li>
	<li><code>1 &lt;= n &lt;= 2 * 10<sup>4</sup></code></li>
	<li><code>0 &lt;= height[i] &lt;= 10<sup>5</sup></code></li>
</ul>

## 方法一：单调栈

可以看到所有接到的雨水都是由一块块的矩形组成。而每一块矩形的下边沿都至少和一个柱子的上边沿重合，反过来，每个柱子的上边沿作为矩形下边沿所能构成的最大矩形（如果可以构成矩形的话），也必然是一个雨水矩形块。因此我们发现它们有一一对应的关系。也就是说接下来，我们只需要计算每个柱子的上边沿作为矩形下边沿所能构成的最大矩形的面积之和即可（这里要去掉重复矩形，防止一个矩形被计算多次）。我们只需要得到每个柱子的左边和右边第一个更高柱子的位置，就可以算出来最大矩形面积。

- 时间复杂度$O(n)$
- 空间复杂度$O(n*m)$ n 为柱子个数，m 为最高柱子的高度

### AC 代码

```java
class Solution {
    private static class Entry {
        public int start;
        public int end;
        public int height;

        public Entry(int start, int end, int height) {
            this.start = start;
            this.end = end;
            this.height = height;
        }

        @Override
        public boolean equals(Object obj) {
            Entry entry = (Entry) obj;
            return (this.start == entry.start) && (this.end == entry.end) && (this.height == entry.height);
        }

        @Override
        public int hashCode() {
            return start + end + height;
        }
    }

    public int trap(int[] height) { // 左边第一个大的->严格单调递减 右边第一个大的->非严格单调递减
        if (height.length <= 2) {
            return 0;
        }
        Deque<Integer> stack = new LinkedList<>();
        int[] leftFirstBigger = new int[height.length];
        Arrays.fill(leftFirstBigger, -1);
        for (int i = 0; i < height.length; i++) {
            while (!stack.isEmpty() && height[stack.peekLast()] <= height[i]) {
                stack.pollLast();
            }
            if (!stack.isEmpty()) {
                leftFirstBigger[i] = stack.peekLast();
            }
            stack.addLast(i);
        }
        int[] rightFirstBigger = new int[height.length];
        Arrays.fill(rightFirstBigger, -1);
        stack.clear();
        for (int i = 0; i < height.length; i++) {
            while (!stack.isEmpty() && height[stack.peekLast()] < height[i]) {
                Integer num = stack.pollLast();
                rightFirstBigger[num] = i;
            }
            stack.addLast(i);
        }
        Set<Entry> set = new HashSet<>();
        int sum = 0;
        for (int i = 0; i < height.length; i++) {
            if (leftFirstBigger[i] != -1 && rightFirstBigger[i] != -1) {
                Entry entry = new Entry(leftFirstBigger[i], rightFirstBigger[i], height[i]);
                if (!set.contains(entry)) {
                    set.add(entry);
                    sum += (rightFirstBigger[i] - leftFirstBigger[i] - 1) * (Math.min(height[rightFirstBigger[i]], height[leftFirstBigger[i]]) - height[i]);
                }
            }
        }
        return sum;
    }
}
```
