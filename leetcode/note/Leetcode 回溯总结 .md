---
title: 回溯总结
date: 2023-11-09 13:10:57
tags: [Leetcode, 回溯]
---

# 回溯总结

## 组合

力扣题目链接：[https://leetcode.cn/problems/combinations/](https://leetcode.cn/problems/combinations/)

根节点是空节点，根节点分支考虑每一个节点，每一个节点的分支再考虑这个节点之后的每一个节点，递归中有两种情况会返回，一种是 path 中元素数目已够，另一种是 path 中元素数目虽然没够，但是当前节点已经是最后一个节点（之后没有节点了）。

思考：多重循环和回溯方法有什么区别呢？

多重循环每层循环次数可变，循环层数固定不可变。回溯方法每层循环次数可变，循环层数也可变（可受变量控制）。

### AC 代码

```java
class Solution {
    private LinkedList<Integer> path = new LinkedList<>();
    private List<List<Integer>> res = new ArrayList<>();

    public List<List<Integer>> combine(int n, int k) {
        backtacking(n, k, 0, 0);
        return res;
    }

    private void backtacking(int n, int k, int depth, int num) {
        if (depth == k) {
            res.add(new ArrayList<>(path));
            return;
        }
        for (int i = num + 1; i <= n; i++) {
            path.add(i);
            backtacking(n, k, depth + 1, i);
            path.removeLast();
        }
    }
}
```

## 情况相乘

力扣题目链接：[https://leetcode.cn/problems/letter-combinations-of-a-phone-number/](https://leetcode.cn/problems/letter-combinations-of-a-phone-number/)

第一个位置考虑所有的情况、第二个位置考虑所有的情况...... 相乘即可

### AC 代码

```java
class Solution {
    private List<String> res = new ArrayList<>();
    private StringBuilder sb = new StringBuilder();
    private Map<Character, List<Character>> map = new HashMap<>();

    public List<String> letterCombinations(String digits) {
        if (digits.length() == 0) {
            return res;
        }
        build('2', 'a', 'b', 'c');
        build('3', 'd', 'e', 'f');
        build('4', 'g', 'h', 'i');
        build('5', 'j', 'k', 'l');
        build('6', 'm', 'n', 'o');
        build('7', 'p', 'q', 'r', 's');
        build('8', 't', 'u', 'v');
        build('9', 'w', 'x', 'y', 'z');
        backtracking(0, digits);
        return res;
    }

    private void build(char c, Character... chars) {
        map.put(c, Arrays.asList(chars));
    }

    private void backtracking(int index, String digits) {
        if (index == digits.length()) {
            res.add(sb.toString());
            return;
        }
        List<Character> characters = map.get(digits.charAt(index));
        for (char c : characters) {
            sb.append(c);
            backtracking(index + 1, digits);
            sb.deleteCharAt(sb.length() - 1);
        }
    }
}
```

## 子集

力扣题目链接：[https://leetcode.cn/problems/subsets/](https://leetcode.cn/problems/subsets/)

根节点是空节点，根节点分支考虑每一个节点，每一个节点的分支再考虑这个节点之后的每一个节点，递归中只有一种情况会返回，即当前节点已经是最后一个节点（之后没有节点了）。子集的树和组合的树是同一棵树，只不过组合找的是叶子节点中长度等于组合元素个数的节点，而子集找的是所有节点。

### AC 代码

```java
class Solution {
    private List<List<Integer>> res = new ArrayList<>();
    private LinkedList<Integer> path = new LinkedList<>();

    public List<List<Integer>> subsets(int[] nums) {
        backtracking(nums, 0);
        return res;
    }

    private void backtracking(int[] nums, int index) {
        res.add(new ArrayList<>(path));
        for (int i = index; i < nums.length; i++) {
            path.add(nums[i]);
            backtracking(nums, i + 1);
            path.removeLast();
        }
    }
}
```

## 可多取的子集

力扣题目链接：[https://leetcode.cn/problems/combination-sum/](https://leetcode.cn/problems/combination-sum/)

子集是每个元素可以取或不取，可多取的子集是每个元素可以取 0 到多个。根节点是空节点，根节点分支考虑每一个节点，每一个节点的分支再考虑这个节点及这个节点之后的每一个节点，递归中有两种情况会返回，一种情况是因为某种原因而剪枝，另一种情况是当前分支下面的所有分支已经遍历完。

### AC 代码

```java
class Solution {
    private List<List<Integer>> res = new ArrayList<>();
    private LinkedList<Integer> path = new LinkedList<>();
    private int sum = 0;

    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        backtracking(candidates, target, 0);
        return res;
    }

    private void backtracking(int[] candidates, int target, int index) {
        if (sum == target) {
            res.add(new ArrayList<>(path));
            return;
        } else if (sum > target) {
            return;
        }
        for (int i = index; i < candidates.length; i++) {
            path.add(candidates[i]);
            sum += candidates[i];
            backtracking(candidates, target, i);
            path.removeLast();
            sum -= candidates[i];
        }
    }
}
```

## 去重子集

力扣题目链接：[https://leetcode.cn/problems/combination-sum-ii/](https://leetcode.cn/problems/combination-sum-ii/)

本题如果允许结果重复或者 candidates 数组中每个元素都不重复，那么就是一个简单的子集回溯，但是本题要求在集合存在重复元素的情况下找到所有不重复子集。如果采用对结果用 map 去重的方式就太低效了，我们可以考虑先排序再在回溯中去重。

回溯如下：根节点是空节点，根节点分支从左向右考虑集合中的元素构成分支，相同元素只考虑第一个。对于每一个分支，考虑下一个元素，并且仍然相同元素只考虑第一个。最终递归形成的树的所有节点都是去重的子集。递归中只有一种情况会返回，那就是已经没有节点可以走了。

上述去重过程为什么成立呢？因为其实树中任何一个节点为根的子树都对应着这个节点对应的子集+这个节点元素之后的所有元素构成的子集的所有情况，打个比方，假如排序后集合中连着两个 1，那么第一个 1 对应的所有情况已经把第二个 1 对应的所有情况覆盖了！

### AC 代码

```java
class Solution {
    private List<List<Integer>> res = new ArrayList<>();
    private LinkedList<Integer> path = new LinkedList<>();

    public List<List<Integer>> combinationSum2(int[] candidates, int target) {
        Arrays.sort(candidates);
        backtracking(candidates, target, 0, 0);
        return res;
    }

    private void backtracking(int[] candidates, int target, int index, int sum) {
        if (sum > target) {
            return;
        } else if (sum == target) {
            res.add(new ArrayList<>(path));
            return;
        }
        if (index == candidates.length) {
            return;
        }
        Set<Integer> set = new HashSet<>();
        for (int i = index; i < candidates.length; i++) {
            if (set.contains(candidates[i])) {
                continue;
            } else {
                set.add(candidates[i]);
            }
            path.add(candidates[i]);
            sum += candidates[i];
            backtracking(candidates, target, i + 1, sum);
            path.removeLast();
            sum -= candidates[i];
        }
    }
}
```

## 全排列

取排列树的所有叶节点即可。

### AC 代码

```java
class Solution {
    private List<List<Integer>> res = new ArrayList<>();
    private LinkedList<Integer> path = new LinkedList<>();

    public List<List<Integer>> permute(int[] nums) {
        backtracking(nums);
        return res;
    }

    private void backtracking(int[] nums) {
        if (path.size() == nums.length) {
            res.add(new ArrayList<>(path));
        }
        for (int i = 0; i < nums.length; i++) {
            if (nums[i] == 11) {
                continue;
            }
            path.add(nums[i]);
            nums[i] = 11;
            backtracking(nums);
            nums[i] = path.getLast();
            path.removeLast();
        }
    }
}
```

## 去重全排列

保证排列树每层相同元素只取一次即可。

```java
class Solution {
    private List<List<Integer>> res = new ArrayList<>();
    private LinkedList<Integer> path = new LinkedList<>();
    private boolean[] used;

    public List<List<Integer>> permuteUnique(int[] nums) {
        used = new boolean[nums.length];
        for (int i = 0; i < used.length; i++) {
            used[i] = false;
        }
        backtracking(nums);
        return res;
    }

    private void backtracking(int[] nums) {
        if (path.size() == nums.length) {
            res.add(new ArrayList<>(path));
            return;
        }
        boolean[] hash = new boolean[21];
        Arrays.fill(hash, false);
        for (int i = 0; i < nums.length; i++) {
            if (used[i]) {
                continue;
            }
            if (hash[nums[i] + 10]) {
                continue;
            } else {
                hash[nums[i] + 10] = true;
            }
            path.add(nums[i]);
            used[i] = true;
            backtracking(nums);
            used[i] = false;
            path.removeLast();
        }
    }
}
```
