---
title: git&github必备技能
date: 2023-10-29 11:52:59
tags: [日常笔记]
---

# git&github 必备技能

[猴子都能懂的 GIT 入门 | 贝格乐（Backlog）](https://backlog.com/git-tutorial/cn/)

[Learn Git Branching](https://learngitbranching.js.org/?locale=zh_CN)

官方 hello world 入门教程，实操一遍提交代码的流程

进阶：

github 漫游指南

开源指北

git 官方文档，跟着文档敲一遍命令，有个大致印象即可

帮助理解：

[(93 条消息) 解决 git pull 出现冲突 多重超详细分析\_叶孤崖的博客-CSDN 博客\_git pull 冲突](https://blog.csdn.net/qq_34246965/article/details/107553644)

注意命令行下的 git 默认是不使用代理的（即使开了 vpn），所以执行 git push 等命令时会因为网络问题报错，这时就需要我们手动为 git 配置代理，以下是开了 clash 后的配置命令（7890 是 clash 的默认端口号）。

git config --global http.proxy http://127.0.0.1:7890

git config --global https.proxy https://127.0.0.1:7890
