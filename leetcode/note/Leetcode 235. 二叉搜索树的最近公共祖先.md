---
title: 235. 二叉搜索树的最近公共祖先
date: 2023-11-09 10:35:56
tags: [Leetcode题解]
---

# 235.二叉搜索树的最近公共祖先

力扣题目链接：[https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-search-tree/](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-search-tree/)

<p>给定一个二叉搜索树, 找到该树中两个指定节点的最近公共祖先。</p>

<p><a href="https://baike.baidu.com/item/%E6%9C%80%E8%BF%91%E5%85%AC%E5%85%B1%E7%A5%96%E5%85%88/8918834?fr=aladdin" target="_blank">百度百科</a>中最近公共祖先的定义为：&ldquo;对于有根树 T 的两个结点 p、q，最近公共祖先表示为一个结点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（<strong>一个节点也可以是它自己的祖先</strong>）。&rdquo;</p>

<p>例如，给定如下二叉搜索树:&nbsp; root =&nbsp;[6,2,8,0,4,7,9,null,null,3,5]</p>

<p><img alt="" src="https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/14/binarysearchtree_improved.png" style="height: 190px; width: 200px;"></p>

<p>&nbsp;</p>

<p><strong>示例 1:</strong></p>

<pre><strong>输入:</strong> root = [6,2,8,0,4,7,9,null,null,3,5], p = 2, q = 8
<strong>输出:</strong> 6 
<strong>解释: </strong>节点 <code>2 </code>和节点 <code>8 </code>的最近公共祖先是 <code>6。</code>
</pre>

<p><strong>示例 2:</strong></p>

<pre><strong>输入:</strong> root = [6,2,8,0,4,7,9,null,null,3,5], p = 2, q = 4
<strong>输出:</strong> 2
<strong>解释: </strong>节点 <code>2</code> 和节点 <code>4</code> 的最近公共祖先是 <code>2</code>, 因为根据定义最近公共祖先节点可以为节点本身。</pre>

<p>&nbsp;</p>

<p><strong>说明:</strong></p>

<ul>
	<li>所有节点的值都是唯一的。</li>
	<li>p、q 为不同节点且均存在于给定的二叉搜索树中。</li>
</ul>

## 方法一：走出一条路径

对于二叉搜索树中的一个点，我们将它和根节点相比较就可以得出它是根节点还是左子树中的节点还是右子树中的节点，而两个节点的最近公共祖先一定是唯一的一个节点，这个节点满足一个点在右子树，一个点在左子树。因此我们就可以从根节点开始向下搜出一条路径。
二叉搜索树经常不涉及遍历所有节点，而是根据其有序性走出一条路，这时就没必要递归了，迭代法一个循环即可解决。

- 时间复杂度$O(n)$
- 空间复杂度$O(n)$

### AC 代码

```java
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if(p.val<root.val&&q.val<root.val) {
            return lowestCommonAncestor(root.left,p,q);
        }
        if(p.val>root.val&&q.val>root.val) {
            return lowestCommonAncestor(root.right,p,q);
        }
        return root;
    }
}
```

## 方法二：通用方法

任何二叉树找最近公共祖先通用方法
任何节点到根节点存在唯一路径，这个路径上的点都是节点的祖先，我们只需要遍历二叉树保存两个节点对应的两条路径，再找到它们的交点即可。

- 时间复杂度$O(n)$
- 空间复杂度$O(n)$

### AC 代码

```java
class Solution {
    private List<TreeNode> pList = new ArrayList<>();
    private List<TreeNode> qList = new ArrayList<>();

    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        preorder(root, new ArrayList<>(), p, q);
        int length = Math.min(pList.size(), qList.size());
        TreeNode res = null;
        for (int i = 0; i < length; i++) {
            if (pList.get(i) != qList.get(i)) {
                return res;
            }
            res = pList.get(i);
        }
        return res;
    }

    private boolean preorder(TreeNode root, ArrayList<TreeNode> list, TreeNode p, TreeNode q) {
        if (root == null) {
            return false;
        }
        list.add(root);
        if (root == p) {
            pList = (List<TreeNode>) (list.clone());
        } else if (root == q) {
            qList = (List<TreeNode>) (list.clone());
        }
        if (pList.size() > 0 && qList.size() > 0) {
            return true;
        }
        if (preorder(root.left, list, p, q)) {
            return true;
        }
        boolean bool = preorder(root.right, list, p, q);
        list.remove(list.size() - 1);
        return bool;
    }
}
```
