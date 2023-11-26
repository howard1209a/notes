---
title: 15.三数之和
date: 2023-10-30 22:21:46
tags: [Leetcode题解]
---

# 15.三数之和

力扣题目链接：[https://leetcode.cn/problems/3sum/](https://leetcode.cn/problems/3sum/)

<p>给你一个整数数组 <code>nums</code> ，判断是否存在三元组 <code>[nums[i], nums[j], nums[k]]</code> 满足 <code>i != j</code>、<code>i != k</code> 且 <code>j != k</code> ，同时还满足 <code>nums[i] + nums[j] + nums[k] == 0</code> 。请</p>

<p>你返回所有和为 <code>0</code> 且不重复的三元组。</p>

<p><strong>注意：</strong>答案中不可以包含重复的三元组。</p>

<p>&nbsp;</p>

<p>&nbsp;</p>

<p><strong>示例 1：</strong></p>

<pre>
<strong>输入：</strong>nums = [-1,0,1,2,-1,-4]
<strong>输出：</strong>[[-1,-1,2],[-1,0,1]]
<strong>解释：</strong>
nums[0] + nums[1] + nums[2] = (-1) + 0 + 1 = 0 。
nums[1] + nums[2] + nums[4] = 0 + 1 + (-1) = 0 。
nums[0] + nums[3] + nums[4] = (-1) + 2 + (-1) = 0 。
不同的三元组是 [-1,0,1] 和 [-1,-1,2] 。
注意，输出的顺序和三元组的顺序并不重要。
</pre>

<p><strong>示例 2：</strong></p>

<pre>
<strong>输入：</strong>nums = [0,1,1]
<strong>输出：</strong>[]
<strong>解释：</strong>唯一可能的三元组和不为 0 。
</pre>

<p><strong>示例 3：</strong></p>

<pre>
<strong>输入：</strong>nums = [0,0,0]
<strong>输出：</strong>[[0,0,0]]
<strong>解释：</strong>唯一可能的三元组和为 0 。
</pre>

<p>&nbsp;</p>

<p><strong>提示：</strong></p>

<ul>
	<li><code>3 &lt;= nums.length &lt;= 3000</code></li>
	<li><code>-10<sup>5</sup> &lt;= nums[i] &lt;= 10<sup>5</sup></code></li>
</ul>

## 方法一：哈希表

本题可以先暴力搜索$O(n^3)$，然后再去重（每个三元组排个序，再利用 hashmap 去重），但是这种做法非常慢。我们可以采用排序去重的方式，即先排序数组，在找的过程中就不找重复的，这样最后就不用再去重了。具体来说，我们先锚定第一个数和第二个数，利用 map 在剩余的数里面$O(1)$找解，然后右移第二个数，找到下一个不同的数，重复过程直到超出数组长度。然后右移第一个数，找到下一个不同的数，重复过程。

- 时间复杂度$O(n^2)$
- 空间复杂度$O(n)$

### AC 代码

```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        List<List<Integer>> res = new ArrayList<>();
        Arrays.sort(nums);
        int last = Integer.MIN_VALUE;
        for (int i = 0; i < nums.length - 2; i++) {
            if (nums[i] == last) {
                continue;
            }
            last = nums[i];
            Map<Integer, Integer> map = new HashMap<>();
            for (int j = i + 1; j < nums.length; j++) {
                map.put(nums[j], map.getOrDefault(nums[j], 0) + 1);
            }
            int newLast = Integer.MIN_VALUE;
            for (int j = i + 1; j < nums.length - 1; j++) {
                Integer num = map.get(nums[j]);
                if (num == 1) {
                    map.remove(nums[j]);
                } else {
                    map.put(nums[j], num - 1);
                }
                if (nums[j] == newLast) {
                    continue;
                }
                newLast = nums[j];
                if (map.containsKey(-1 * nums[i] - nums[j])) {
                    List<Integer> list = new ArrayList<>();
                    list.add(nums[i]);
                    list.add(nums[j]);
                    list.add(-nums[i] - nums[j]);
                    res.add(list);
                }
            }
        }
        return res;
    }
}
```

## 方法二：双指针

本题能够用双指针的关键在于一个结论，即对于无序数组的两数之和问题可以用双$O(n)$的哈希表求解，而对于有序数组的两数之和问题既可以用双$O(n)$的哈希表求解，也可以用时间$O(n)$空间 O(1)的双指针求解。对于这个双指针法，其实是设置 left 在最左边的数，right 在最右边的数，然后 right 收缩直到$left+right=target$或$left+right<target$，这时将 left 左移一位，然后继续重复过程。

注意其实四数之和、五数之和都可以用同样的双指针方法，对应时间复杂度$O(n^3)$、$O(n^4)$...

四数之和：https://leetcode.cn/problems/4sum/description/

- 时间复杂度$O(n^2)$
- 空间复杂度$O(1)$

### AC 代码

```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        Arrays.sort(nums);
        List<List<Integer>> res = new ArrayList<>();
        for (int i = 0; i < nums.length - 2; i++) {
            if(i!=0) {
                if (nums[i] == nums[i - 1]) {
                    continue;
                }
            }
            int j = i + 1;
            int k = nums.length - 1;
            while (true) {
                int sum = nums[i]+nums[j]+nums[k];
                if (sum == 0) {
                    res.add(Arrays.asList(nums[i],nums[j],nums[k]));
                    j++;
                    while (j<k&&nums[j]==nums[j-1]) {
                        j++;
                    }
                    if (j == k) {
                        break;
                    }
                    k--;
                    while (j < k && nums[k] == nums[k + 1]) {
                        k--;
                    }
                    if (j == k) {
                        break;
                    }
                } else if (sum < 0) {
                    j++;
                    while (j < k && nums[j] == nums[j - 1]) {
                        j++;
                    }
                    if (j == k) {
                        break;
                    }
                } else {
                    k--;
                    while (j < k && nums[k] == nums[k + 1]) {
                        k--;
                    }
                    if (j == k) {
                        break;
                    }
                }
            }
        }
        return res;
    }
}
```
