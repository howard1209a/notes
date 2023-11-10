---
title: 108. 将有序数组转换为二叉搜索树
date: 2023-11-09 11:29:51
tags: [Leetcode, 二叉搜索树, 平衡二叉树]
---

# 108.将有序数组转换为二叉搜索树

力扣题目链接：[https://leetcode.cn/problems/convert-sorted-array-to-binary-search-tree/](https://leetcode.cn/problems/convert-sorted-array-to-binary-search-tree/)

<p>给你一个整数数组 <code>nums</code> ，其中元素已经按 <strong>升序</strong> 排列，请你将其转换为一棵 <strong>高度平衡</strong> 二叉搜索树。</p>

<p><strong>高度平衡 </strong>二叉树是一棵满足「每个节点的左右两个子树的高度差的绝对值不超过 1 」的二叉树。</p>

<p>&nbsp;</p>

<p><strong>示例 1：</strong></p>
<img alt="" src="https://assets.leetcode.com/uploads/2021/02/18/btree1.jpg" style="width: 302px; height: 222px;" />
<pre>
<strong>输入：</strong>nums = [-10,-3,0,5,9]
<strong>输出：</strong>[0,-3,9,-10,null,5]
<strong>解释：</strong>[0,-10,5,null,-3,null,9] 也将被视为正确答案：
<img alt="" src="https://assets.leetcode.com/uploads/2021/02/18/btree2.jpg" style="width: 302px; height: 222px;" />
</pre>

<p><strong>示例 2：</strong></p>
<img alt="" src="https://assets.leetcode.com/uploads/2021/02/18/btree.jpg" style="width: 342px; height: 142px;" />
<pre>
<strong>输入：</strong>nums = [1,3]
<strong>输出：</strong>[3,1]
<strong>解释：</strong>[1,null,3] 和 [3,1] 都是高度平衡二叉搜索树。
</pre>

<p>&nbsp;</p>

<p><strong>提示：</strong></p>

<ul>
	<li><code>1 &lt;= nums.length &lt;= 10<sup>4</sup></code></li>
	<li><code>-10<sup>4</sup> &lt;= nums[i] &lt;= 10<sup>4</sup></code></li>
	<li><code>nums</code> 按 <strong>严格递增</strong> 顺序排列</li>
</ul>

## 方法一：递归建树

二叉搜索树的定义：

- 直接定义：每个节点的左子树所有节点均小于当前节点，右子树所有节点均大于当前节点。
- 递归定义：根节点的左子树所有节点均小于根节点，根节点的右子树所有节点均大于根节点，且根节点的左右子树均为二叉搜索树。

二叉平衡树的定义：

- 直接定义：每个节点的左子树和右子树的深度之差的绝对值不大于 1。
- 递归定义：根节点的左子树和右子树的深度之差的绝对值不大于 1，且左右子树均为二叉平衡树。

本题是将二叉搜索树和二叉平衡树组合成了平衡二叉搜索树。

- 时间复杂度$O(n)$
- 空间复杂度$O(logn)$

### AC 代码

```java
class Solution {
    public TreeNode sortedArrayToBST(int[] nums) {
        return build(nums, 0, nums.length - 1);
    }

    private TreeNode build(int[] nums, int i, int j) {
        if (i > j) {
            return null;
        } else if (i == j) {
            return new TreeNode(nums[i]);
        }
        int mid = (i + j) / 2;
        TreeNode root = new TreeNode(nums[mid]);
        root.left = build(nums, i, mid - 1);
        root.right = build(nums, mid + 1, j);
        return root;
    }
}
```
