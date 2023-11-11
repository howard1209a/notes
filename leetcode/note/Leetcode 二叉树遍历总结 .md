---
title: 二叉树遍历总结
date: 2023-11-08 13:19:51
tags: [Leetcode, 二叉树]
---

# 二叉树遍历总结

## 二叉树的前序遍历

力扣题目链接：[https://leetcode.cn/problems/binary-tree-preorder-traversal/](https://leetcode.cn/problems/binary-tree-preorder-traversal/)

### 递归方式

```java
class Solution {
    private List<Integer> res = new ArrayList<>();

    public List<Integer> preorderTraversal(TreeNode root) {
        preorder(root);
        return res;
    }

    private void preorder(TreeNode root) {
        if (root == null) {
            return;
        }
        res.add(root.val);
        preorder(root.left);
        preorder(root.right);
    }
}
```

### 栈方式

栈方式实现前序遍历的关键在于，不断地重复：访问当前节点、右节点进栈、转移到左节点，如果当前节点为空则弹栈，直到当前节点为空且栈空。

```java
class Solution {
    public List<Integer> preorderTraversal(TreeNode root) {
        List<Integer> res = new ArrayList<>();
        if (root == null) {
            return res;
        }
        Stack<TreeNode> stack = new Stack<>();
        stack.push(root);
        TreeNode node = null;
        while (node != null || !stack.isEmpty()) {
            if (node == null) {
                node = stack.pop();
            }
            res.add(node.val);
            if (node.right != null) {
                stack.push(node.right);
            }
            node = node.left;
        }
        return res;
    }
}
```

## 二叉树的中序遍历

力扣题目链接：[https://leetcode.cn/problems/binary-tree-inorder-traversal/](https://leetcode.cn/problems/binary-tree-inorder-traversal/)

### 递归方式

```java
class Solution {
    private List<Integer> res = new ArrayList<>();

    public List<Integer> inorderTraversal(TreeNode root) {
        inorder(root);
        return res;
    }

    private void inorder(TreeNode root) {
        if (root == null) {
            return;
        }
        inorder(root.left);
        res.add(root.val);
        inorder(root.right);
    }
}
```

### 栈方式

栈方式实现中序遍历的关键在于，一直向左走并进栈，直到节点为空，然后弹栈，访问节点，取节点的右子节点重复这个过程，直到节点为空且栈空。

```java
class Solution {
    public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> res = new ArrayList<>();
        Stack<TreeNode> stack = new Stack<>();
        TreeNode node = root;
        while (node != null || !stack.isEmpty()) {
            if (node == null) {
                node = stack.pop();
                res.add(node.val);
                node = node.right;
                continue;
            }
            stack.push(node);
            node = node.left;
        }
        return res;
    }
}
```

## 二叉树的后序遍历

力扣题目链接：[https://leetcode.cn/problems/binary-tree-postorder-traversal/](https://leetcode.cn/problems/binary-tree-postorder-traversal/)

### 递归方式

```java
class Solution {
    private List<Integer> res = new ArrayList<>();

    public List<Integer> postorderTraversal(TreeNode root) {
        postorder(root);
        return res;
    }

    private void postorder(TreeNode root) {
        if (root == null) {
            return;
        }
        postorder(root.left);
        postorder(root.right);
        res.add(root.val);
    }
}
```

### 栈方式

栈方式实现后序遍历的关键在于，第一次访问栈顶节点时不要出栈，第二次访问栈顶节点才出栈，仍然是一直向左走并进栈直到节点为空，这时候查看栈顶节点，如果是第一次查看则取这个节点的右子节点再次一直向左走，如果是第二次查看则这个节点出栈并访问这个节点。

```java
class Solution {
    private class Entry {
        public TreeNode node;
        public boolean shouldOut;

        public Entry(TreeNode node) {
            this.node = node;
        }
    }

    public List<Integer> postorderTraversal(TreeNode root) {
        List<Integer> res = new ArrayList<>();
        Stack<Entry> stack = new Stack<>();
        TreeNode node = root;
        while (node != null || !stack.isEmpty()) {
            if (node == null) {
                if (!stack.peek().shouldOut) {
                    stack.peek().shouldOut = true;
                    node = stack.peek().node.right;
                } else {
                    res.add(stack.pop().node.val);
                }
                continue;
            }
            stack.push(new Entry(node));
            node = node.left;
        }
        return res;
    }
}
```

## 二叉树的广度优先遍历

### 队列方式

```java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> res = new ArrayList<>();
        if (root == null) {
            return res;
        }
        Queue<TreeNode> queue = new LinkedList<>();
        queue.add(root);
        while (!queue.isEmpty()) {
            int len = queue.size();
            List<Integer> list = new ArrayList<>();
            res.add(list);
            for (int i = 0; i < len; i++) {
                TreeNode node = queue.poll();
                list.add(node.val);
                if (node.left != null) {
                    queue.add(node.left);
                }
                if (node.right != null) {
                    queue.add(node.right);
                }
            }
        }
        return res;
    }
}
```

### 递归方式

递归方式访问节点的顺序并不是广度优先的，它本质上是先序遍历二叉树，把每一行的节点按从左到右的顺序存起来，最后拼接成一个序列，然后遍历这个序列才是广度优先遍历顺序。之所以能这么做的原因在于，先序遍历同一行内左边的节点一定比右边的节点先被遍历到。

```java
class Solution {
    private List<List<Integer>> res = new ArrayList<>();

    public List<List<Integer>> levelOrder(TreeNode root) {
        preorder(root, 0);
        return res;
    }

    private void preorder(TreeNode root, int depth) {
        if (root == null) {
            return;
        }
        if (res.size() <= depth) {
            res.add(new ArrayList<>());
        }
        List<Integer> list = res.get(depth);
        list.add(root.val);
        preorder(root.left, depth + 1);
        preorder(root.right, depth + 1);
    }
}
```
