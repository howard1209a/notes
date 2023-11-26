---
title: 2003. 每棵子树内缺失的最小基因值
date: 2023-10-31 13:23:41
tags: [Leetcode题解]
---

# 2003.每棵子树内缺失的最小基因值

力扣题目链接：[https://leetcode.cn/problems/smallest-missing-genetic-value-in-each-subtree/](https://leetcode.cn/problems/smallest-missing-genetic-value-in-each-subtree/)

<p>有一棵根节点为 <code>0</code>&nbsp;的 <strong>家族树</strong>&nbsp;，总共包含 <code>n</code>&nbsp;个节点，节点编号为 <code>0</code>&nbsp;到 <code>n - 1</code>&nbsp;。给你一个下标从 <strong>0</strong>&nbsp;开始的整数数组 <code>parents</code>&nbsp;，其中&nbsp;<code>parents[i]</code>&nbsp;是节点 <code>i</code>&nbsp;的父节点。由于节点 <code>0</code>&nbsp;是 <strong>根</strong>&nbsp;，所以&nbsp;<code>parents[0] == -1</code>&nbsp;。</p>

<p>总共有&nbsp;<code>10<sup>5</sup></code>&nbsp;个基因值，每个基因值都用 <strong>闭区间</strong>&nbsp;<code>[1, 10<sup>5</sup>]</code>&nbsp;中的一个整数表示。给你一个下标从&nbsp;<strong>0</strong>&nbsp;开始的整数数组&nbsp;<code>nums</code>&nbsp;，其中&nbsp;<code>nums[i]</code>&nbsp;是节点 <code>i</code>&nbsp;的基因值，且基因值 <strong>互不相同</strong>&nbsp;。</p>

<p>请你返回一个数组<em>&nbsp;</em><code>ans</code>&nbsp;，长度为&nbsp;<code>n</code>&nbsp;，其中&nbsp;<code>ans[i]</code>&nbsp;是以节点&nbsp;<code>i</code>&nbsp;为根的子树内 <b>缺失</b>&nbsp;的&nbsp;<strong>最小</strong>&nbsp;基因值。</p>

<p>节点 <code>x</code>&nbsp;为根的 <strong>子树&nbsp;</strong>包含节点 <code>x</code>&nbsp;和它所有的 <strong>后代</strong>&nbsp;节点。</p>

<p>&nbsp;</p>

<p><strong>示例 1：</strong></p>

<p><img alt="" src="https://assets.leetcode.com/uploads/2021/08/23/case-1.png" style="width: 204px; height: 167px;"></p>

<pre><b>输入：</b>parents = [-1,0,0,2], nums = [1,2,3,4]
<b>输出：</b>[5,1,1,1]
<b>解释：</b>每个子树答案计算结果如下：
- 0：子树包含节点 [0,1,2,3] ，基因值分别为 [1,2,3,4] 。5 是缺失的最小基因值。
- 1：子树只包含节点 1 ，基因值为 2 。1 是缺失的最小基因值。
- 2：子树包含节点 [2,3] ，基因值分别为 [3,4] 。1 是缺失的最小基因值。
- 3：子树只包含节点 3 ，基因值为 4 。1是缺失的最小基因值。
</pre>

<p><strong>示例 2：</strong></p>

<p><img alt="" src="https://assets.leetcode.com/uploads/2021/08/23/case-2.png" style="width: 247px; height: 168px;"></p>

<pre><b>输入：</b>parents = [-1,0,1,0,3,3], nums = [5,4,6,2,1,3]
<b>输出：</b>[7,1,1,4,2,1]
<b>解释：</b>每个子树答案计算结果如下：
- 0：子树内包含节点 [0,1,2,3,4,5] ，基因值分别为 [5,4,6,2,1,3] 。7 是缺失的最小基因值。
- 1：子树内包含节点 [1,2] ，基因值分别为 [4,6] 。 1 是缺失的最小基因值。
- 2：子树内只包含节点 2 ，基因值为 6 。1 是缺失的最小基因值。
- 3：子树内包含节点 [3,4,5] ，基因值分别为 [2,1,3] 。4 是缺失的最小基因值。
- 4：子树内只包含节点 4 ，基因值为 1 。2 是缺失的最小基因值。
- 5：子树内只包含节点 5 ，基因值为 3 。1 是缺失的最小基因值。
</pre>

<p><strong>示例 3：</strong></p>

<pre><b>输入：</b>parents = [-1,2,3,0,2,4,1], nums = [2,3,4,5,6,7,8]
<b>输出：</b>[1,1,1,1,1,1,1]
<b>解释：</b>所有子树都缺失基因值 1 。
</pre>

<p>&nbsp;</p>

<p><strong>提示：</strong></p>

<ul>
	<li><code>n == parents.length == nums.length</code></li>
	<li><code>2 &lt;= n &lt;= 10<sup>5</sup></code></li>
	<li>对于&nbsp;<code>i != 0</code>&nbsp;，满足&nbsp;<code>0 &lt;= parents[i] &lt;= n - 1</code></li>
	<li><code>parents[0] == -1</code></li>
	<li><code>parents</code>&nbsp;表示一棵合法的树。</li>
	<li><code>1 &lt;= nums[i] &lt;= 10<sup>5</sup></code></li>
	<li><code>nums[i]</code>&nbsp;互不相同。</li>
</ul>

## 方法一：分类讨论+深度优先搜索

无论是 2 叉树还是 K 叉树，常见的表示方式就是链表方式和数组方式，本题就是使用数组方式对非完全 K 叉树进行表示。关于数组表示 K 叉树，主要分为三种方法：

- 第一种直接表示法，只适用于二叉树，完全二叉树不用动，非完全二叉树需要补所有空节点，然后按照广度优先遍历的顺序存数组，这样任何节点数组下标/2 就是父节点，*2 就是左子节点，*2+1 就是右子节点。
- 第二种父节点表示法，适用于任何 K 叉树，任何一个节点的数组下标位置都存了此节点父节点的数组下标，根节点存-1，注意二叉树一定是 n 个节点，n-1 条边。
- 第三种子节点表示法，适用于任何 K 叉树，任何一个节点的数组下标位置都存了一个 list，这个 list 里面放了此节点的所有子节点的数组下标。

对于任意 K 叉树，都可以用第二种或第三种方法确定地唯一表示，并且第二种方法表示可以和第三种方法表示互相转换，就比如本题给了第二种表示，我们为了进行 dfs 自己转换成了第三种表示。

说回本题，本题的关键在于一个分类讨论：如果树中不存在 1 会怎样，存在 1 又会怎样。如果树中不存在 1，那么任意节点的最小缺失基因都是 1。如果树中存在 1，那么这个 1 节点和它所有的父节点（也就是从 1 节点到根节点的这条路径）的最小缺失基因都不是 1，其他所有节点的最小缺失基因都是 1，至于这条路径上所有节点的最小缺失基因，我们可以维护一个 set，结合不断地 dfs，自底向上地把所有的最小缺失基因都求出来。

关于时间复杂度和空间复杂度，我们发现虽然整个过程做了很多事情，但是它们的复杂度都没有超过 O(n)。

- 时间复杂度$O(n)$
- 空间复杂度$O(n)$

### AC 代码

```java
class Solution {
    public int[] smallestMissingValueSubtree(int[] parents, int[] nums) {
        int[] res = new int[parents.length];
        Arrays.fill(res, 1);
        int index = -1;
        for (int i = 0; i < nums.length; i++) {
            if (nums[i] == 1) {
                index = i;
                break;
            }
        }
        if (index < 0) {
            return res;
        }
        List<Integer>[] childArr = new ArrayList[parents.length];
        Arrays.setAll(childArr, new IntFunction<List<Integer>>() {
            @Override
            public List<Integer> apply(int value) {
                return new ArrayList<>();
            }
        });
        for (int i = 1; i < parents.length; i++) {
            childArr[parents[i]].add(i);
        }
        Set<Integer> set = new HashSet<>();
        int num = 1;
        int last = -1;
        while (index != -1) {
            set.add(nums[index]);
            for (int i = 0; i < childArr[index].size(); i++) {
                Integer child = childArr[index].get(i);
                if (child != last) {
                    preorder(set, childArr[index].get(i), childArr, nums);
                }
            }
            while (set.contains(num)) {
                num++;
            }
            res[index] = num;
            last = index;
            index = parents[index];
        }
        return res;
    }

    private void preorder(Set<Integer> set, int root, List<Integer>[] childArr, int[] nums) {
        if (root > childArr.length - 1) {
            return;
        }
        set.add(nums[root]);
        for (int i = 0; i < childArr[root].size(); i++) {
            preorder(set, childArr[root].get(i), childArr, nums);
        }
    }
}
```
