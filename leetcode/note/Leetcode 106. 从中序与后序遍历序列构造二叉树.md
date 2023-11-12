---
title: 106. 从中序与后序遍历序列构造二叉树
date: 2023-11-08 23:08:32
tags: [Leetcode, 二叉树]
---

# 106.从中序与后序遍历序列构造二叉树

力扣题目链接：[https://leetcode.cn/problems/construct-binary-tree-from-inorder-and-postorder-traversal/](https://leetcode.cn/problems/construct-binary-tree-from-inorder-and-postorder-traversal/)

<p>给定两个整数数组 <code>inorder</code> 和 <code>postorder</code> ，其中 <code>inorder</code> 是二叉树的中序遍历， <code>postorder</code> 是同一棵树的后序遍历，请你构造并返回这颗&nbsp;<em>二叉树</em>&nbsp;。</p>

<p>&nbsp;</p>

<p><strong>示例 1:</strong></p>
<img alt="" src="https://assets.leetcode.com/uploads/2021/02/19/tree.jpg" />
<pre>
<b>输入：</b>inorder = [9,3,15,20,7], postorder = [9,15,7,20,3]
<b>输出：</b>[3,9,20,null,null,15,7]
</pre>

<p><strong>示例 2:</strong></p>

<pre>
<b>输入：</b>inorder = [-1], postorder = [-1]
<b>输出：</b>[-1]
</pre>

<p>&nbsp;</p>

<p><strong>提示:</strong></p>

<ul>
	<li><code>1 &lt;= inorder.length &lt;= 3000</code></li>
	<li><code>postorder.length == inorder.length</code></li>
	<li><code>-3000 &lt;= inorder[i], postorder[i] &lt;= 3000</code></li>
	<li><code>inorder</code>&nbsp;和&nbsp;<code>postorder</code>&nbsp;都由 <strong>不同</strong> 的值组成</li>
	<li><code>postorder</code>&nbsp;中每一个值都在&nbsp;<code>inorder</code>&nbsp;中</li>
	<li><code>inorder</code>&nbsp;<strong>保证</strong>是树的中序遍历</li>
	<li><code>postorder</code>&nbsp;<strong>保证</strong>是树的后序遍历</li>
</ul>

## 方法一：典型建树

这道题是一道典型题，给出一个节点数值不重复的二叉树的中序序列和后序序列，反向构建二叉树。后序遍历序列的最后一个数是根节点，这个数在中序遍历序列的某个中间位置，此位置之前是左子树的中序遍历序列，之后是右子树的中序遍历序列，而后序遍历序列则等同于左子树后序遍历序列+右子树后序遍历序列+根节点。因此问题就可以化成两个子问题从而用递归解决，为了在中序遍历序列中方便的查找数的位置，我们可以使用 map。

题目 105.从前序与中序遍历序列构造二叉树其实是一样的道理

力扣题目链接：[https://leetcode.cn/problems/construct-binary-tree-from-preorder-and-inorder-traversal/](https://leetcode.cn/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)

需要注意的是前序和中序可以唯一确定一棵二叉树，后序和中序可以唯一确定一棵二叉树，但前序和后序不可以唯一确定一棵二叉树！

- 时间复杂度$O(n)$
- 空间复杂度$O(n)$

### AC 代码

```java
class Solution {
    private Map<Integer, Integer> map = new HashMap<>();

    public TreeNode buildTree(int[] inorder, int[] postorder) {
        for (int i = 0; i < inorder.length; i++) {
            map.put(inorder[i], i);
        }
        return build(inorder, postorder, 0, inorder.length - 1, 0, postorder.length - 1);
    }

    private TreeNode build(int[] inorder, int[] postorder, int inStart, int inEnd, int postStart, int postEnd) {
        if (inEnd < inStart) {
            return null;
        }
        TreeNode root = new TreeNode(postorder[postEnd]);
        int pos = map.get(postorder[postEnd]);
        root.left = build(inorder, postorder, inStart, pos - 1, postStart, postStart + pos - 1 - inStart);
        root.right = build(inorder, postorder, pos + 1, inEnd, postStart + pos - inStart, postEnd - 1);
        return root;
    }
}
```
