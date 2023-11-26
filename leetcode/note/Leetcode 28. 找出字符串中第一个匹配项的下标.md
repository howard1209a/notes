---
title: 28. 找出字符串中第一个匹配项的下标
date: 2023-10-31 22:18:05
tags: [Leetcode题解]
---

# 28.找出字符串中第一个匹配项的下标

力扣题目链接：[https://leetcode.cn/problems/find-the-index-of-the-first-occurrence-in-a-string/](https://leetcode.cn/problems/find-the-index-of-the-first-occurrence-in-a-string/)

<p>给你两个字符串&nbsp;<code>haystack</code> 和 <code>needle</code> ，请你在 <code>haystack</code> 字符串中找出 <code>needle</code> 字符串的第一个匹配项的下标（下标从 0 开始）。如果&nbsp;<code>needle</code> 不是 <code>haystack</code> 的一部分，则返回&nbsp; <code>-1</code><strong> </strong>。</p>

<p>&nbsp;</p>

<p><strong class="example">示例 1：</strong></p>

<pre>
<strong>输入：</strong>haystack = "sadbutsad", needle = "sad"
<strong>输出：</strong>0
<strong>解释：</strong>"sad" 在下标 0 和 6 处匹配。
第一个匹配项的下标是 0 ，所以返回 0 。
</pre>

<p><strong class="example">示例 2：</strong></p>

<pre>
<strong>输入：</strong>haystack = "leetcode", needle = "leeto"
<strong>输出：</strong>-1
<strong>解释：</strong>"leeto" 没有在 "leetcode" 中出现，所以返回 -1 。
</pre>

<p>&nbsp;</p>

<p><strong>提示：</strong></p>

<ul>
	<li><code>1 &lt;= haystack.length, needle.length &lt;= 10<sup>4</sup></code></li>
	<li><code>haystack</code> 和 <code>needle</code> 仅由小写英文字符组成</li>
</ul>

## 方法一：暴力匹配

以匹配串的每个字符作为起点尝试匹配模式串。

- 时间复杂度$O(m*n)$ n 为匹配串长度，m 为模式串长度
- 空间复杂度$O(1)$

## 方法二：KMP

这里原理分为两部分，next 数组的生成和字符串匹配过程。

需要注意的是 KMP 算法只能找到第一个匹配的字符串或者判断是否存在匹配的字符串。

### next 数组生成

主要采用动态规划的方式，数组下标定义为当前字符一直到第一个字符的最长公共前后缀长度，而只要有前面所有字符的最长公共前后缀长度，我们就可以推出当前字符的最长公共前后缀长度，又因为第一个字符的最长公共前后缀长度为 0，因此我们从左向右遍历就可以生成 next 数组.

下面讲一下递推公式：

那么当我们知道前面所有字符的最长公共前后缀，如何求当前字符的最长公共前后缀呢？首先我们已知前一个字符的最长公共前后缀，当前字符的最长公共前后缀最大为前一个字符的最长公共前后缀+1（这个可以用反证法证明，如果存在更长的则前一个字符的最长公共前后缀也必然存在更长的，那前一个字符的最长公共前后缀就不是最长的了，矛盾），这时我们可以比较一下字符串当前字符和前一个字符的最长公共前后缀+1 这个位置的字符是否相等，如果相等的话当前字符最长公共前后缀显然为前一个字符的最长公共前后缀+1，如果不等的话，就要考虑前一个字符的次长公共前后缀，这里为什么可以直接考虑次长公共前后缀呢（仍然用反证法证明，如果存在更长的则前一个字符的次长公共前后缀也必然存在更长的，那它就不是次长了，矛盾）。那么前一个字符的次长公共前后缀是多少呢，其实是前一个字符的最长公共前后缀的位置的字符的最长公共前后缀（这里建议画图理解）。然后我们可以进一步比较字符串当前字符和前一个字符的次长公共前后缀+1 这个位置的字符是否相等，如果相等则当前字符的最长公共前后缀等于前一个字符的次长公共前后缀+1，如果不等的话我们再考虑前一个字符的次次长公共前后缀......

直到最后可能出现一种情况是，前一个字符的次......长公共前后缀等于 0 了，这时再不等的话，我们直接为当前字符的最长公共前后缀赋 0 即可。

### 字符串匹配过程

为模式串和匹配串分别设置一个指针，指针同时后移，从模式串和匹配串的第一个字符开始向后匹配，找到第一个不匹配的字符，将模式串指针调到这个字符的前一个字符（也就是最后一个匹配的字符）的最长公共前后缀长度的位置，继续匹配，直到找到匹配的第一个字符串，或者匹配到最后没找到。

next 数组生成的时间复杂度为$O(m)$，空间复杂度为$O(m)$。字符串匹配过程的时间复杂度为$O(n)$，空间复杂度为$O(1)$。至于为什么复杂度是这样我没太想明白，大概是一个平均的复杂度，类似于快排最坏$O(n^2)$，最好$O(n\log n)$，但我们一般认为快排是$O(n\log n)$一样。

- 时间复杂度$O(m+n)$
- 空间复杂度$O(m)$

### AC 代码

```java
class Solution {
  public int strStr(String haystack, String needle) {
      if (needle.length() > haystack.length()) {
          return -1;
      }
      int[] value = kmp(needle);
      int haystackIndex = 0, needleIndex = 0;
      int pos = 0;
      while (haystackIndex < haystack.length()) {
          if (haystack.charAt(haystackIndex) == needle.charAt(needleIndex)) {
              haystackIndex++;
              needleIndex++;
              if (needleIndex >= needle.length()) {
                  return pos;
              }
          } else if (needleIndex == 0) {
              pos = haystackIndex + 1;
              haystackIndex++;
          } else {
              needleIndex = value[needleIndex - 1];
              pos = haystackIndex - needleIndex;
          }
      }
      return -1;
  }

  private int[] kmp(String s) {
      int[] next = new int[s.length()];
      next[0] = 0;
      if (s.length() == 1) {
          return next;
      }
      int i = 1, j = 0;
      while (i < s.length()) {
          if (s.charAt(i) == s.charAt(j)) {
              next[i] = j + 1;
              i++;
              j++;
          } else if (j == 0) {
              next[i] = 0;
              i++;
          } else {
              j = next[j - 1];
          }
      }
      return next;
  }
}
```
