---
title: 为什么 jdk 要按照操作系统和系统架构分类
date: 2023-10-29 13:26:53
tags: [计算机组成原理, 八股文]
---

# 为什么 jdk 要按照操作系统和系统架构分类

JDK 包括 JRE（Java 运行环境）和 Java 工具（编译器（javac.exe）、反编译工具（javap.exe）等），其中 JRE 又包含 Java 虚拟机 JVM（java.exe）和 java 基础类库。

上面提到的 jvm、编译器等等都是由 C/C++实现的可执行文件。因此不同的操作系统其系统调用接口不同、基础库不同，这就决定了为不同操作系统编写的相关 java 工具的源码肯定有所不同，其次不同的 CPU 架构其指令集不同，这意味着即使源码相同的情况下编译出来的可执行文件（机器指令序列）也一定是不同的。
