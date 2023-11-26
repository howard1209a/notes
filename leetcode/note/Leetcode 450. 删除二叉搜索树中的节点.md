---
title: 450. 删除二叉搜索树中的节点
date: 2023-11-09 11:01:30
tags: [Leetcode题解]
---

# 450.删除二叉搜索树中的节点

力扣题目链接：[https://leetcode.cn/problems/delete-node-in-a-bst/](https://leetcode.cn/problems/delete-node-in-a-bst/)

<p>给定一个二叉搜索树的根节点 <strong>root </strong>和一个值 <strong>key</strong>，删除二叉搜索树中的&nbsp;<strong>key&nbsp;</strong>对应的节点，并保证二叉搜索树的性质不变。返回二叉搜索树（有可能被更新）的根节点的引用。</p>

<p>一般来说，删除节点可分为两个步骤：</p>

<ol>
	<li>首先找到需要删除的节点；</li>
	<li>如果找到了，删除它。</li>
</ol>

<p>&nbsp;</p>

<p><strong>示例 1:</strong></p>

<p><img src="https://assets.leetcode.com/uploads/2020/09/04/del_node_1.jpg" style="width: 800px;" /></p>

<pre>
<strong>输入：</strong>root = [5,3,6,2,4,null,7], key = 3
<strong>输出：</strong>[5,4,6,2,null,null,7]
<strong>解释：</strong>给定需要删除的节点值是 3，所以我们首先找到 3 这个节点，然后删除它。
一个正确的答案是 [5,4,6,2,null,null,7], 如下图所示。
另一个正确答案是 [5,2,6,null,4,null,7]。

<img src="https://assets.leetcode.com/uploads/2020/09/04/del_node_supp.jpg" style="width: 350px;" />
</pre>

<p><strong>示例 2:</strong></p>

<pre>
<strong>输入:</strong> root = [5,3,6,2,4,null,7], key = 0
<strong>输出:</strong> [5,3,6,2,4,null,7]
<strong>解释:</strong> 二叉树不包含值为 0 的节点
</pre>

<p><strong>示例 3:</strong></p>

<pre>
<strong>输入:</strong> root = [], key = 0
<strong>输出:</strong> []</pre>

<p>&nbsp;</p>

<p><strong>提示:</strong></p>

<ul>
	<li>节点数的范围&nbsp;<code>[0, 10<sup>4</sup>]</code>.</li>
	<li><code>-10<sup>5</sup>&nbsp;&lt;= Node.val &lt;= 10<sup>5</sup></code></li>
	<li>节点值唯一</li>
	<li><code>root</code>&nbsp;是合法的二叉搜索树</li>
	<li><code>-10<sup>5</sup>&nbsp;&lt;= key &lt;= 10<sup>5</sup></code></li>
</ul>

<p>&nbsp;</p>

<p><strong>进阶：</strong> 要求算法时间复杂度为&nbsp;O(h)，h 为树的高度。</p>

## 方法一：理解切分

二叉搜索树的本质就是切分，根节点切一下将区间分为左边和右边，左子节点切一下将左区间分为左边和右边，右子节点切一下将右区间分为左边和右边，依次进行。

如果我们要删除一个点，完全可以找到这个切分右边的最近一个切分（即要删除点的右子节点一直向左走直到走不了的那个节点），把这个切分替换到删除点的位置，然后进行适当的调整即可。做这种题本质还是要理解二叉搜索树的每个节点都是一个切分。

- 时间复杂度$O(h)$，h 为树的高度
- 空间复杂度$O(1)$

### AC 代码

```java
class Solution {
    public TreeNode deleteNode(TreeNode root, int key) {
        if (root == null) {
            return null;
        }
        TreeNode node = root, parent = root;
        int flag = 0;
        while (node != null) {
            if (node.val == key) {
                break;
            } else if (node.val < key) {
                parent = node;
                flag = 1;
                node = node.right;
            } else {
                parent = node;
                flag = 0;
                node = node.left;
            }
            if (node == null) {
                return root;
            }
        }
        if (node.left == null && node.right == null) {
            if (node == root) {
                return null;
            } else {
                if (flag == 0) {
                    parent.left = null;
                } else {
                    parent.right = null;
                }
                return root;
            }
        } else if (node.left != null && node.right == null) {
            if (node == root) {
                return node.left;
            } else {
                if (flag == 0) {
                    parent.left = node.left;
                } else {
                    parent.right = node.left;
                }
                return root;
            }
        } else if (node.left == null && node.right != null) {
            if (node == root) {
                return node.right;
            } else {
                if (flag == 0) {
                    parent.left = node.right;
                } else {
                    parent.right = node.right;
                }
                return root;
            }
        } else {
            if (node == root) {
                root = node.right;
                TreeNode tmp = root;
                while (tmp.left != null) {
                    tmp = tmp.left;
                }
                tmp.left = node.left;
                return root;
            } else {
                if (flag == 0) {
                    parent.left = node.right;
                    TreeNode tmp = parent.left;
                    while (tmp.left != null) {
                        tmp = tmp.left;
                    }
                    tmp.left = node.left;
                    return root;
                } else {
                    parent.right = node.right;
                    TreeNode tmp = parent.right;
                    while (tmp.left != null) {
                        tmp = tmp.left;
                    }
                    tmp.left = node.left;
                    return root;
                }
            }
        }
    }
}
```
