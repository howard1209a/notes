---
title: 98. 验证二叉搜索树
date: 2023-11-09 09:48:13
tags: [Leetcode, 二叉搜索树]
---

# 98.验证二叉搜索树

力扣题目链接：[https://leetcode.cn/problems/validate-binary-search-tree/](https://leetcode.cn/problems/validate-binary-search-tree/)

<p>给你一个二叉树的根节点 <code>root</code> ，判断其是否是一个有效的二叉搜索树。</p>

<p><strong>有效</strong> 二叉搜索树定义如下：</p>

<ul>
	<li>节点的左子树只包含<strong> 小于 </strong>当前节点的数。</li>
	<li>节点的右子树只包含 <strong>大于</strong> 当前节点的数。</li>
	<li>所有左子树和右子树自身必须也是二叉搜索树。</li>
</ul>

<p>&nbsp;</p>

<p><strong>示例 1：</strong></p>
<img alt="" src="https://assets.leetcode.com/uploads/2020/12/01/tree1.jpg" style="width: 302px; height: 182px;" />
<pre>
<strong>输入：</strong>root = [2,1,3]
<strong>输出：</strong>true
</pre>

<p><strong>示例 2：</strong></p>
<img alt="" src="https://assets.leetcode.com/uploads/2020/12/01/tree2.jpg" style="width: 422px; height: 292px;" />
<pre>
<strong>输入：</strong>root = [5,1,4,null,null,3,6]
<strong>输出：</strong>false
<strong>解释：</strong>根节点的值是 5 ，但是右子节点的值是 4 。
</pre>

<p>&nbsp;</p>

<p><strong>提示：</strong></p>

<ul>
	<li>树中节点数目范围在<code>[1, 10<sup>4</sup>]</code> 内</li>
	<li><code>-2<sup>31</sup> &lt;= Node.val &lt;= 2<sup>31</sup> - 1</code></li>
</ul>

## 方法一：利用二叉搜索树性质

二叉搜索树的定义是每个节点的左子树所有节点均小于当前节点，右子树所有节点均大于当前节点，另一种递归定义是，根节点的左子树所有节点均小于根节点，根节点的右子树所有节点均大于根节点，且根节点的左右子树均为二叉搜索树。

一个二叉树是二叉搜索树的充分必要条件是关于这个二叉树的中序遍历（左中右）是单调递增的，或者关于这个二叉树的中序遍历（右中左）是单调递减的。

所以本题做一个中序遍历判断是否单调递增即可。

- 时间复杂度$O(n)$
- 空间复杂度$O(n)$

### AC 代码

```java
class Solution {
    private int last;
    private boolean isStart = true;

    public boolean isValidBST(TreeNode root) {
        return !inorder(root);
    }

    private boolean inorder(TreeNode root) {
        if (root == null) {
            return false;
        }
        if (inorder(root.left)) {
            return true;
        }
        if (!isStart && root.val <= last) {
            return true;
        }
        isStart = false;
        last = root.val;
        if (inorder(root.right)) {
            return true;
        }
        return false;
    }
}
```

## 方法二：递归

从二叉搜索树的定义出发

- 时间复杂度$O(n)$
- 空间复杂度$O(n)$

### AC 代码

```java
class Solution {
    private class Result {
        public boolean isValid;
        public int max;
        public int min;

        public Result(boolean isValid, int max, int min) {
            this.isValid = isValid;
            this.max = max;
            this.min = min;
        }
    }

    public boolean isValidBST(TreeNode root) {
        return isBST(root).isValid;
    }

    private Result isBST(TreeNode root) {
        if (root.left == null && root.right == null) {
            return new Result(true, root.val, root.val);
        } else if (root.left != null && root.right == null) {
            Result left = isBST(root.left);
            if (!left.isValid) {
                return new Result(false, 0, 0);
            }
            if (left.max < root.val) {
                return new Result(true, root.val, left.min);
            } else {
                return new Result(false, 0, 0);
            }
        } else if (root.left == null && root.right != null) {
            Result right = isBST(root.right);
            if (!right.isValid) {
                return new Result(false, 0, 0);
            }
            if (root.val < right.min) {
                return new Result(true, right.max, root.val);
            } else {
                return new Result(false, 0, 0);
            }
        } else {
            Result left = isBST(root.left);
            if (!left.isValid) {
                return new Result(false, 0, 0);
            }
            Result right = isBST(root.right);
            if (!right.isValid) {
                return new Result(false, 0, 0);
            }
            if (left.max < root.val && root.val < right.min) {
                return new Result(true, right.max, left.min);
            } else {
                return new Result(false, 0, 0);
            }
        }
    }
}
```
