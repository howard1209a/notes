---
title: 0151. 反转字符串中的单词
date: 2023-10-31 14:07:26
tags: [Leetcode, 字符串, 部分翻转]
---

# 151.反转字符串中的单词

力扣题目链接：[https://leetcode.cn/problems/reverse-words-in-a-string/](https://leetcode.cn/problems/reverse-words-in-a-string/)

<p>给你一个字符串 <code>s</code> ，请你反转字符串中 <strong>单词</strong> 的顺序。</p>

<p><strong>单词</strong> 是由非空格字符组成的字符串。<code>s</code> 中使用至少一个空格将字符串中的 <strong>单词</strong> 分隔开。</p>

<p>返回 <strong>单词</strong> 顺序颠倒且 <strong>单词</strong> 之间用单个空格连接的结果字符串。</p>

<p><strong>注意：</strong>输入字符串 <code>s</code>中可能会存在前导空格、尾随空格或者单词间的多个空格。返回的结果字符串中，单词间应当仅用单个空格分隔，且不包含任何额外的空格。</p>

<p>&nbsp;</p>

<p><strong>示例 1：</strong></p>

<pre>
<strong>输入：</strong>s = "<code>the sky is blue</code>"
<strong>输出：</strong>"<code>blue is sky the</code>"
</pre>

<p><strong>示例 2：</strong></p>

<pre>
<strong>输入：</strong>s = " &nbsp;hello world &nbsp;"
<strong>输出：</strong>"world hello"
<strong>解释：</strong>反转后的字符串中不能存在前导空格和尾随空格。
</pre>

<p><strong>示例 3：</strong></p>

<pre>
<strong>输入：</strong>s = "a good &nbsp; example"
<strong>输出：</strong>"example good a"
<strong>解释：</strong>如果两个单词间有多余的空格，反转后的字符串需要将单词间的空格减少到仅有一个。
</pre>

<p>&nbsp;</p>

<p><strong>提示：</strong></p>

<ul>
	<li><code>1 &lt;= s.length &lt;= 10<sup>4</sup></code></li>
	<li><code>s</code> 包含英文大小写字母、数字和空格 <code>' '</code></li>
	<li><code>s</code> 中 <strong>至少存在一个</strong> 单词</li>
</ul>

<ul>
</ul>

<p>&nbsp;</p>

<p><strong>进阶：</strong>如果字符串在你使用的编程语言中是一种可变数据类型，请尝试使用&nbsp;<code>O(1)</code> 额外空间复杂度的 <strong>原地</strong> 解法。</p>

## 方法一：单独处理

切片，把每个单词单独拿出来，然后反序拼接。这种方式不满足空间 O(1)的要求，

- 时间复杂度$O(n)$
- 空间复杂度$O(n)$

## 方法二：整体翻转再局部翻转

一个数组或字符串被分为多个块，如果整体翻转就直接翻转，如果块内翻转就分别翻转，如果块间翻转就先整体翻转再分别块内翻转。

- 时间复杂度$O(n)$
- 空间复杂度$O(1)$

### AC 代码

```java
class Solution {
    public String reverseWords(String s) {
        int start = 0, end = s.length() - 1;
        StringBuilder sb = new StringBuilder();
        while (s.charAt(end) == ' ') {
            end--;
        }
        while (s.charAt(start) == ' ') {
            start++;
        }
        s = s.substring(start, end + 1);
        start = s.length() - 1;
        end = s.length() - 1;
        while (start >= 0) {
            while (start >= 0 && s.charAt(start) != ' ') {
                start--;
            }
            sb.append(s.substring(start + 1, end + 1));
            sb.append(" ");
            while (start > 0 && s.charAt(start - 1) == ' ') {
                start--;
            }
            start--;
            end = start;
        }
        return sb.substring(0, sb.charAt(sb.length() - 1) == ' ' ? sb.length() - 1 : sb.length());
    }
}
```
