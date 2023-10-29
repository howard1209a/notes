---
title: git中的merge操作
date: 2023-10-29 11:56:42
tags: [git]
---

# git 中的 merge 操作

branch 相关的命令

```shell
// 查看所有分支
git branch

// 创建一个name分支
git branch name

// 切换到name分支
git checkout name

// 合并name分支到当前分支
git merge name
```

`git merge name`命令用于将其他分支合并到当前所在工作分支上，当前工作分支的内容会由于 merge 操作产生更新，但是其他分支则完全不受影响，`git merge`通常与其他几个 git 命令一起使用，包括使用`git checkout`命令来选择当前工作分支，以及使用`git branch -d`命令来删除已经合并过的废弃分支。

merge 合并的情形主要有两种，一种是快进合并，一种是三路合并。

快进合并的情形：master 分支分出一个 b1 分支后，master 分支没有进行任何改变，b1 分支做了一些文件的**增删改**，这时在 master 分支下`git merge b1`，b1 分支会完全覆盖 master 分支，而不会产生任何的 conflict

三路合并的情形：master 分支分出一个 b1 分支后，master 分支做了一些文件的**增删改**，b1 分支也做了一些文件的**增删改**，这时在 master 分支下`git merge b1`，这时就不是 b1 分支简单地覆盖 master 分支，而是需要对 b1 分支中**新增文件、删除文件和修改文件**做出分类讨论。

- 对于 b1 分支中的新增文件：如果 master 分支中没有新增同名文件，则直接 merge，不会产生 conflict，如果 master 分支中也新增了同名文件，则会产生 conflict，需要手动决策。
- 对于 b1 分支中的删除文件：如果 master 分支中的对应文件没有变化或也被删除了，则直接 merge，不会产生 conflict，如果 master 分支中的对应文件被修改了，则会产生 conflict，需要手动决策。
- 对于 b1 分支中的修改文件：如果 master 分支中的对应文件没有变化，则直接 merge，不会产生 conflict，如果 master 分支中的对应文件被修改或删除了，则会产生 conflict，需要手动决策。
