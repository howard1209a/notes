---
title: 239. 滑动窗口最大值
date: 2023-11-07 23:02:25
tags: [Leetcode, 单调队列]
---
# 239.滑动窗口最大值

力扣题目链接：[https://leetcode.cn/problems/sliding-window-maximum/](https://leetcode.cn/problems/sliding-window-maximum/)

<p>给你一个整数数组 <code>nums</code>，有一个大小为&nbsp;<code>k</code><em>&nbsp;</em>的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口内的 <code>k</code>&nbsp;个数字。滑动窗口每次只向右移动一位。</p>

<p>返回 <em>滑动窗口中的最大值 </em>。</p>

<p>&nbsp;</p>

<p><strong>示例 1：</strong></p>

<pre>
<b>输入：</b>nums = [1,3,-1,-3,5,3,6,7], k = 3
<b>输出：</b>[3,3,5,5,6,7]
<b>解释：</b>
滑动窗口的位置                最大值
---------------               -----
[1  3  -1] -3  5  3  6  7       <strong>3</strong>
 1 [3  -1  -3] 5  3  6  7       <strong>3</strong>
 1  3 [-1  -3  5] 3  6  7      <strong> 5</strong>
 1  3  -1 [-3  5  3] 6  7       <strong>5</strong>
 1  3  -1  -3 [5  3  6] 7       <strong>6</strong>
 1  3  -1  -3  5 [3  6  7]      <strong>7</strong>
</pre>

<p><strong>示例 2：</strong></p>

<pre>
<b>输入：</b>nums = [1], k = 1
<b>输出：</b>[1]
</pre>

<p>&nbsp;</p>

<p><b>提示：</b></p>

<ul>
	<li><code>1 &lt;= nums.length &lt;= 10<sup>5</sup></code></li>
	<li><code>-10<sup>4</sup>&nbsp;&lt;= nums[i] &lt;= 10<sup>4</sup></code></li>
	<li><code>1 &lt;= k &lt;= nums.length</code></li>
</ul>

## 单调队列
本题为单调队列的一种应用。单调队列和优先队列都关注队列中的最小数和最大数，那么它们有哪些不同呢？
- 单调队列：一种单调顺序的线性表，进队元素直接进队，出队元素是最早进队的元素，单调队列并不关注于出队元素和进队元素（也没有办法关注），它提供O(1)的时间复杂度可以随时获取队列中所有元素的最小数或最大数，并且出队进队操作加起来往往是O(n)的时间复杂度。
- 优先队列：大根堆/小根堆，进队元素直接进队，出队元素是当前队列最大/最小的元素，优先队列可以关注出队元素和进队元素分别是什么，它也提供O(1)的时间复杂度可以随时获取队列中所有元素的最小数或最大数，但出队进队操作加起来往往是O(nlogk)的时间复杂度。

下面介绍单调队列的原理（以单调递减队列为例），对于原队列，我们从最后一个数开始向前遍历，如果前面的数更大则保留，更小或平则丢弃，丢弃是因为，这些数在自己出队之前，队列中一直有着大于等于自己的数，所以自己永远不可能是队列的最大值。然后我们就得到了一个从前到后单调递减的队列，此时队列所有元素的最大值显然是第一个数。接下来考虑进队和出队，对于出队，我们只需要查看第一个数是否需要出队即可，对于进队，我们则从后向前遍历并删掉所有小于等于进队数的元素。综上所述，我们就可以一直维护一个单调队列，它只可以随时返回队列最大值/最小值，虽然在接口外看来是一个完整的队列。

至于单调队列应用于本题，则就是滑动窗口滑动对应一出队一进队，总体下来每个元素最多出队进队一次。

- 时间复杂度$O(n)$
- 空间复杂度$O(k)$

### AC代码
```java
class Solution {
    private Deque<int[]> monotonicQueue = new LinkedList<>();
    private int index = 0;

    public int[] maxSlidingWindow(int[] nums, int k) {
        for (int i = 0; i < k; i++) {
            add(nums[i], i);
        }
        int[] res = new int[nums.length - k + 1];
        res[0] = getMax();
        for (int i = k; i < nums.length; i++) {
            poll();
            add(nums[i], i);
            res[i - k + 1] = getMax();
        }
        return res;
    }

    private void add(int num, int pos) {
        while (!monotonicQueue.isEmpty()) {
            if (monotonicQueue.peekLast()[0] > num) {
                monotonicQueue.addLast(new int[]{num, pos});
                return;
            }
            monotonicQueue.pollLast();
        }
        monotonicQueue.addLast(new int[]{num, pos});
    }

    private void poll() {
        if (monotonicQueue.peekFirst()[1] == index) {
            monotonicQueue.pollFirst();
        }
        index++;
    }

    private int getMax() {
        return monotonicQueue.peekFirst()[0];
    }
}
```