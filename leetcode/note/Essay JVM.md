---
title: Java Virtual Machine
date: 2024-02-05 17:34:24
tags: [Java, 八股文]
---

# Java Virtual Machine

## 字节码文件详解

### Java 虚拟机的组成

JVM 本质上是一个运行在计算机上的进程，它的主要职责是

- 解释运行：解释运行字节码中的指令
- 内存管理：为对象、方法等分配内存，自动进行垃圾回收
- 即时编译：对热点代码进行优化，提升执行效率

常见的 JVM 有很多实现，他们都遵循 Oracle 制定的 Java 虚拟机规范

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240205193708.png)

### 字节码文件的组成

#### 字节码文件包含哪些信息

字节码文件中保存了源代码编译之后的内容，以二进制文件的方式存储，可以用 jclasslib 工具解析字节码文件信息。

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240205191944.png)

##### 基本信息

magic 魔数用来标识二进制文件的类型。版本号用来标识编译此字节码文件的编译器版本，此版本必须小于等于 JVM 版本。

##### 常量池

存放字符串常量

常量池中的数据都有一个编号，编号从 1 开始。在字段或者字节码指令中通过编号可以快速的找到对应的数据。

##### 方法

存放每个方法的字节码指令序列，下面是一个示例，两个核心部分分别是操作数栈和局部变量表

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240205193307.png)

### 类的生命周期

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240205195048.png)

#### 加载阶段

类加载器根据类的全限定名通过读取磁盘上的字节码文件或动态代理生成的方式获取字节码。

加载完成后，JVM 会在方法区内存中生成一个 C++的 InstanceKlass 对象，在堆区生成一个 class 对象，class 对象中存储了该类的静态属性。

#### 连接阶段

##### 验证

检测 Java 字节码文件是否遵守 Java 虚拟机规范

##### 准备

为所有静态属性分配内存并设置初始值

##### 解析

将常量池中的符号引用替换为内存直接引用。在类加载之前只看字节码文件的话，比如字节码指令序列中调了`#6`方法，我们只能在常量池中找到这个方法的类名方法名，而在类加载中替换为内存直接引用后，我们就能在方法区运行时常量池中直接得到此方法的内存地址，因为此时此方法所在类一定已经加载到了方法区类信息中。

#### 初始化阶段

按顺序进行所有静态属性的初始化和静态代码块的运行。本质是执行字节码文件中 clinit 部分的字节码指令。

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240205201612.png)

### 类加载器

类加载器(ClassLoader)只负责类加载中的加载阶段。读入字节码数据放入内存转换成 byte[]，接下来调用虚拟机底层 native 方法根据 byte[]在方法区生成 InstanceKlass 对象，在堆区生成 class 对象。

#### 类加载器的分类

##### 启动类加载器

使用 C++实现的类加载器，加载 Java 中最核心的类，默认加载 Java 安装目录/jre/lib 下的类文件。

##### 扩展类加载器

使用 Java 实现的类加载器，加载 Java 中的扩展类，默认加载 Java 安装目录/jre/lib/ext 下的类文件。

##### 应用程序类加载器

使用 Java 实现的类加载器，加载源码类和 jar 包类，默认加载 classpath 下的类文件。

#### 双亲委派机制

当我们使用应用程序类加载器或扩展类加载器去加载类的时候（注意我们在 java 代码中拿不到启动类加载器），会从下向上的判断要加载的类是否已经被当前类加载器加载，如果已被加载则直接返回。如果一直到启动类加载器都没有加载过，则从上到下判断哪个加载器可以加载该类，如果找到就加载后返回，如果找不到最后抛出 ClassNotFoundException

两种方法的区别：

- loadClass 方法：只加载，不会连接和初始化
- forName 方法：加载连接初始化

双亲委派机制的作用：

- 保证类加载的安全性，避免恶意代码替换 JDK 中的核心类库
- 避免重复加载

#### 打破双亲委派机制

自定义类加载器，重写 loadClass 方法，即可打破双亲委派机制。

##### 为什么要打破双亲委派机制

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240206112214.png)

两个自定义类加载器加载相同限定名的类，不会冲突，因为在同一个 Java 虚拟机中，只有相同类加载器+相同的类限定名才会被认为是同一个类。

##### SPI 机制

JDBC 中使用了 DriverManager 来管理项目中引入的不同数据库的驱动，比如 mysql 驱动、oracle 驱动。实际应用中，只需要引入相关数据库驱动的 jar 包即可自动进行类加载，这个过程依靠 SPI 机制实现。

SPI 是一个为某个接口自动装配实现实例的机制。有点类似 IOC 的思想，就是将装配的控制权移到程序之外。当服务的提供者，提供了服务接口的一种实现之后，在 jar 包的 META-INF/services/目录里同时创建一个以服务接口命名的文件。该文件里就是实现该服务接口的具体实现类的类名。而当外部程序装配这个模块的时候，就能通过该 jar 包 META-INF/services/里的配置文件找到具体的实现类名，并装载实例化，完成模块的注入。

## JVM 内存模型

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240206192912.png)

### 程序计数器

每个线程会通过程序计数器记录当前要执行的的字节码指令的地址，程序计数器可以控制程序指令的进行，实现分支、跳转、异常等逻辑。

程序计数器不存在内存溢出问题

### 方法栈

Java 虚拟机栈和本地方法栈本质是一个方法栈。方法栈中是一个个栈帧，每个栈帧都存了局部变量表、操作数栈和帧数据。

- 局部变量表：存放方法中的 this 对象地址、形参以及局部变量
- 操作数栈：虚拟机在执行指令过程中用来存放临时数据的一块区域
- 帧数据：当前栈帧结束后的返回地址、异常表等

栈帧过多（比如无限递归），超过栈内存最大大小就会出现内存溢出 StackOverflowError

### 堆区

存放所有对象。堆空间有三个需要关注的值，used total max。used 指的是当前已使用的堆内存，total 是 java 虚拟机已经分配的可用堆内存，max 是 java 虚拟机可以分配的最大堆内存。

对象过多或对象中数据过大会出现堆内存溢出`OutOfMemory`

### 方法区

又称元空间，使用的是直接内存

存放类信息、运行时常量池、字符串常量池

- 类信息：所有已经加载的类的 InstanceKlass 对象
- 运行时常量池：存放常量符号到内存地址的直接映射，包括类、接口、属性等等
- 字符串常量池：存储在代码中定义的常量字符串内容

当无限加载类的时候，方法区也是会内存溢出的，报`java.lang.OutOfMemoryError: Metaspace`

### 直接内存

直接内存不会被 GC，常应用于 NIO，目的是为了减少内存复制次数。

## 自动垃圾回收

Java 中为了简化对象内存的释放，引入了自动的垃圾回收(Garbage Collection 简称 GC)机制。

### 方法区回收

线程独享的程序计数器和方法栈是不需要垃圾回收的。

方法区的回收主要就是类的卸载，当满足如下三个条件时，方法区类信息中该类的 InstanceKlass 对象被清除，运行时常量池和字符串常量池中该类相关的数据被清除，堆区中该类的 class 对象被清除。

- 此类所有实例对象都已经被回收，在堆中不存在任何该类的实例对象以及子类对象。
- 加载该类的类加载器已经被回收。
- 该类对应的 java.lang.Class 对象没有在任何地方被引用。

如果我们不自定义类加载器的话，则所有类都是由启动类加载器或扩展类加载器或应用程序类加载器加载的，则不会出现类的卸载。

调用 System.gc()方法并不一定会立即回收垃圾，仅仅是向 Java 虚拟机发送一个垃圾回收的请求，具体是否需要执行垃圾回收 Java 虚拟机会自行判断。

### 堆区回收

#### 引用计数法和可达性分析法

一个 java 对象是否可以被回收取决于这个对象是否仍被引用

##### 引用计数法

为每个对象维护一个引用计数器，当对象被引用时加 1，取消引用时减 1。

##### 可达性分析法

JVM 使用的就是这种方法。可达性分析法将对象分为两类:垃圾回收的根对象(GC Root 对象)和普通对象，对象与对象之间存在引用关系。一个对象只要能追溯到某个 GC Root 对象，则它就不能被回收。

GC Root 对象有四类：

- 线程 Thread 对象，关联该线程方法栈中所有 this 对象、形参、局部变量
- class 对象，关联该类的所有静态属性
- synchronized 关键字持有的对象
- 本地方法调用时使用的全局对象（不需要关注）

下图是一个示例：
![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240207145226.png)

#### 五种对象引用

- 强引用：可达性算法描述的引用
- 软引用：如果一个对象只有软引用关联到它，当堆内存不足时，就会将该对象回收，JDK 提供 SoftReference 类来实现软引用。软引用适用于缓存场景。
- 弱引用：弱引用包含的对象在垃圾回收时，不管内存够不够都会直接被回收。
- 虚引用
- 终结器引用

#### 垃圾回收算法

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240207172743.png)

##### GC 算法的评价标准

- 吞吐量：执行用户代码时间 / (执行用户代码时间 + GC 时间)
- 最大暂停时间：所有垃圾回收过程中的 STW 时间最大值
- 堆使用效率：百分之多少的堆内存能够用于实际存放对象

##### 标记清除算法

利用可达性分析算法，从 GC Root 开始通过引用链遍历并标记所有存活对象，然后清除所有未标记对象

##### 复制算法

将堆内存分割成两块相等的 from 空间 to 空间，在 from 空间存放对象，GC 时将 From 中存活对象复制到 To 空间，将两块空间的 From 和 To 名字互换

##### 标记整理算法

利用可达性分析算法，从 GC Root 开始通过引用链遍历并标记所有存活对象，将所有存活对象移动到堆的一端

##### 分代算法

分代算法将堆内存划分为年轻代和老年代，其中年轻代又分为伊甸园区和幸存者区，幸存者区分为 from 空间和 to 空间。新创建的对象会被放入伊甸园区，伊甸园区满会触发 Minor GC，将伊甸园区和 From 空间中需要回收的对象回收，把没有回收的对象放入 To 区，然后交换 from 和 to 名字。每次 Minor GC 都会让对象年龄加一。如果 Minor GC 后对象的年龄达到阈值，该对象就会被转移到老年代。当老年代空间不足时会触发 Full GC，Full GC 会对年轻代、老年代和元空间（方法区）进行回收。如果 Full GC 后老年代空间仍然不足，则会抛出 Out Of Memory 异常。

为什么分代算法要分年轻代和老年代，原因是防止长期存活对象在 from 空间和 to 空间之间被不断复制。

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240207181056.png)

#### 几种垃圾回收器

需要掌握下图三条实线连接的垃圾回收器和 G1 垃圾回收器，实际具体选择哪种垃圾回收器，和业务类型相关。

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240207183700.png)

##### Serial-SerialOld

Serial 和 SerialOld 都是单线程串行垃圾回收器

Serial-SerialOld 适用于单核 cpu

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240207184741.png)

##### ParNew-CMS

ParNew 是多线程串行垃圾回收器
![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240207190638.png)

CMS 的过程比较复杂，涉及到串行/并行，单线程/多线程
![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240207190753.png)

ParNew-CMS 适用于大型的互联网系统中用户请求数据量大、频率高的场景。比如订单接口、商品接口等

##### ps-po

JDK8 默认的垃圾回收器，Parallel Scavenge 和 Parallel Old 都是多线程串行垃圾回收器，可以自动调整堆内存大小

ps-po 相比于 STW 更关注吞吐量大小，因此适用于不需要与用户交互的后台任务处理。比如:大数据的处理，大文件导出

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240207191944.png)

##### G1 垃圾回收器

JDK9 及之后的默认垃圾回收器。性能要高于其他垃圾回收器。

## 实际业务调优

### 内存调优

观察程序的堆内存曲线，如果正常就不需要进行内存调优，如果异常比如发现了存在大量不可回收对象或存在内存泄露或出现内存突然飙高乃至溢出的现象，这时就需要进行内存调优，找到大量不可回收对象是什么、哪里出现的内存泄露、内存溢出是什么原因（是持续性的内存泄露导致的，还是内存峰值过高导致的，如果是内存峰值过高导致的，要么分配更多的堆内存、要么修改代码降低内存峰值）

堆内存泄漏：某个对象不再使用却仍然引用它，该对象依然在 GC ROOT 的引用链上

#### 一些可能引起内存溢出的原因

- 持续的内存泄露
  - 往 ThreadLocal 里面放对象后不去清除，这时只要这个线程没有结束，ThreadLocal 里面的对象就会一直在。这种情况常发生于线程池。
  - 大量的数据在静态属性中被引用，但是不再使用，成为了内存泄漏。
- 并发请求问题：某些接口由于用户的并发请求量有可能很大，同时处理数据的时间很长，导致大量的数据存在于内存中，最终超过了内存的上限，导致内存溢出。

#### 发现内存问题

- top 命令：实时地查看系统的资源
- VisualVM：监控一个 Java 进程的堆内存曲线，微服务比较多的时候不太方便用
- Arthas：功能很强大的线上监控诊断产品
- Prometheus + Grafana：企业中运维常用的监控方案，其中 Prometheus 用来采集系统或者应用的相关数据，同时具备告警功能。Grafana 可以将 Prometheus 采集到的数据以可视化的方式进行展示。

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240208160227.png)

#### 定位问题代码

我们可以通过`-XX:+HeapDumpOnOutOfMemoryError`虚拟参数，设置堆内存溢出时自动生成内存快照（hprof 文件），将整个堆内存保存下来。

`-XX:+HeapDumpBeforeFullGC`虚拟机参数可以在 FullGC 之前生成内存快照。

如果要导出运行中系统的内存快照

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240208164715.png)

##### MAT 软件分析内存快照文件

MAT 会首先生成一个支配树，在对象引用
图中，所有指向对象 B 的路径都经过对象 A，则认为对象 A 支配对象 B。

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240208162855.png)

对于支配树中的某个节点对象，对象本身占用的空间称为浅堆，对象子树的所有空间称为深堆，深堆的大小表示该对象如果可以被回收，能释放多大的内存空间。

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240208163333.png)

MAT 就是遍历支配树，如果发现某节点深堆的大小超过整个堆内存的一定比例阈值，就会将其标记成内存泄漏的"嫌疑对象"

#### 排查内存问题的方式

1. 服务出现 OOM 内存溢出时，生成内存快照
2. 使用 MAT 分析内存快照，找到内存溢出的对象
3. 尝试在开发环境中重现问题，分析代码中问题产生的原因
4. 修改代码
5. 测试并验证结果

#### 一些设计上引起的内存溢出

- 某分页查询接口 OOM，问题根源是高并发、单次查询数据量大
  1. 限制单次查询数据量
  2. 高峰期直接限流保护
- 某并发处理大量数据的程序 OOM，问题根源是线程每次读入内存的数据量过大，导致内存峰值过大
  1. 减少单批读入内存的数据量大小

### GC 调优

观察程序的 GC 吞吐量和 GC STW 值，如果 GC 吞吐量较高、STW 很低，那么就不需要进行 GC 调优，如果出现了 GC 吞吐量低、STW 较高的情况，那么就需要进行 GC 调优，首先如果出现了频繁 FULLGC，那么是由于内存过高导致的，这时又回到了内存调优，如果不存在频繁 FULLGC 现象，我们可以选择合适的垃圾回收器、设置合适的 JVM 参数来提高 GC 效率。

#### 发现 GC 问题

通过 GC 日志，可以得到每次 GC 的详细信息，比如本次 GC 使用什么垃圾回收器、是 MinorGC 还是 FullGC、回收了多少内存、耗时多久等等。

- 使用方法(JDK 8 及以下):`-XX:+PrintGCDetails -Xloggc:文件名`
- 使用方法(JDK 9+):`-Xlog:gc\*:file=文件名`

我们可以用 GCeasy 来分析 GC 日志，网址是`https://gceasy.io/`

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240210154617.png)

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240210154734.png)

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240210154816.png)

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240210160447.png)

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240210160625.png)

#### 解决 GC 问题的手段

##### 优化基础 JVM 参数

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240210161146.png)

-Xmx 和 –Xms 一般设置成相等的值

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240210161529.png)

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240210161607.png)

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240210163225.png)

其余 JVM 参数如年轻代大小、伊甸园区和幸存者区内存比例等不建议自己设置

##### 更换垃圾回收器

根据业务类型和垃圾回收器特性设置即可

### 性能调优

性能调优适用于，在不需要进行内存调优和 GC 调优时，发现程序导致 cpu 飙高或某个接口响应时间过长或出现死锁，这时可以进行性能调优。

#### cpu 飙高问题

利用`top`命令定位哪个进程的哪个线程存在很高的 cpu 占用率，使用`jstack 进程ID > 文件名`命令进行 Thread Dump 线程转储（打一份线程当前状态的快照）。然后查看下 cpu 飙高线程当前的方法栈。

#### 死锁问题

通过`jstack -l 进程ID > 文件名`命令进行线程 dump，然后在文件中搜索 deadlock 即可找到死锁位置。

#### 接口响应时间很长问题

我们需要定位接口中的哪一部分代码导致的性能问题，可以借助 Arthas 的 trace 命令。

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240210184244.png)

#### 如何测试一个方法的耗时

由于存在懒加载对象，所以第一次方法调用的时间有可能很长，由于多次执行方法 JIT 会进行优化，因此经过充分预热后的测试才是准确的。因此我们一般使用 JMH 框架进行预热后的测试。

## JVM 扩展知识

### GraalVM

GraalVM 是 Oracle 官方推出的一款高性能 JDK，核心优势是可以编译成可执行文件（失去跨平台特性），具有更快的启动速度、更低的内存和 cpu 使用率。

### 新一代的 GC

#### Shenandoah

Shenandoah 是由 Red Hat 开发的一款低延迟的垃圾回收器，无论堆大小如何，Shenandoah 都能通过多次回收的方式将 STW 尽量控制在 10ms 以下。

Shenandoah 不是 oraclejdk 的可选垃圾回收器，它只存在于 openjdk 中。

#### ZGC

ZGC 是 oracle 开发的低延迟垃圾回收器，将 STW 尽量控制在了 1ms 以下，ZGC 吞吐量不佳，且垃圾回收会占用大量额外堆内存，OracleJDK 和 OpenJDK 中都支持 ZGC。

### Java Agent

Java Agent 技术是 JDK 提供的用来编写 Java 工具的技术，使用这种技术生成一种特殊的 jar 包，这种 jar 包可以让 Java 程序静态或动态地运行其中的代码。

#### 静态加载模式

`java`命令运行时就指定好 Java Agent 的 jar 包，主线程会在执行 main 方法前先执行 premain 方法

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240211181717.png)

#### 动态加载模式

首先开启被加载的 java 进程，然后打开另一个 java 进程，给这个 java 进程输入被加载的 java 进程的进程号和 Java Agent 的 jar 包路径，这样这两个 java 进程可以通信，被加载的 java 进程也就能拿到 jar 包中的字节码，它会开一个新的线程来运行这段字节码。

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240211182146.png)

#### 字节码增强技术

静态加载模式是在主线程 main 方法之前执行一段代码，动态加载模式是随时单开一个线程执行一段代码。

借助 ASM 或 Byte Buddy 这两个字节码增强技术框架，我们可以修改加载到 JVM 内存中的字节码，比如在字节码指令序列的开头统计当前时间，在字节码指令序列的结尾统计当前时间然后相减得到该方法的执行时间。ASM 框架比较难用，Byte Buddy 框架基于 ASM 框架开发，要好用一些。

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240211185740.png)

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240211185805.png)

## JVM 底层原理

### 栈上的数据存储

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240212104643.png)

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240212113147.png)

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240212113301.png)

无论是 32 位还是 64 位虚拟机，对于任意数值类型来说，栈内存字节数一定大于等于堆内存字节数。比如在 32 位虚拟机中 double 堆内存 8 字节、栈内存 8 字节，在 64 位虚拟机中 double 堆内存 8 字节、栈内存 16 字节。

栈内存比堆内存大是存在空间浪费的，这是一种空间换时间的方案，在栈中不区分数值类型直接按槽 slot 进行计算，省去了根据不同数值类型进行不同计算方式的过程。

堆中数据保存到栈上，无符号高位补 0 有符号高位补 0 或 1，栈中数据保存到堆上，超出部分舍弃即可。

### 对象在堆上是如何存储的

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240212122153.png)

其中 Klass Word(Klass pointer)指向方法区栈信息中 InstanceKlass 对象的地址，32 位虚拟机中 Mark Word 和 Klass Word 都占 4 字节，64 位虚拟机中 Mark Word 和 Klass Word 都占 8 字节

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240212115608.png)

#### 指针压缩

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240212115816.png)

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240212120119.png)

指针压缩虽然可以降低对象占用内存，但也有两个问题，第一是要求所有对象占用堆内存的大小是 8 字节的倍数（内存对齐），第二是寻址范围变小，不用压缩指针，应该是 2 的 64 次方 = 16EB，用了压缩指针就变成了 8(字节) = 2 的 3 次方 \* 2 的 32 次方 = 2 的 35 次方

#### 内存对齐

内存对齐主要目的是为了解决并发情况下 CPU 缓存失效的问题，一个 cpu 缓存内存块大小是 8 字节，要求所有对象占用堆内存的大小是 8 字节倍数的情况下，使得每个对象完整的占满整数个 cpu 缓存内存块，这样某个对象 cpu 缓存失效并不会导致其他对象 cpu 缓存失效。

##### 字段重排列

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240212121819.png)

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240212121843.png)

### 方法调用的原理

方法调用的本质是在栈上创建栈帧，然后程序计数器指向方法中的第一行字节码指令。以 invoke 开头的字节码指令的作用是执行方法的调用。

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240212123003.png)

#### 静态绑定

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240212122542.png)

#### 动态绑定

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240212122827.png)

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240212122845.png)

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240212122902.png)

### 异常捕获的原理

#### try-catch 实现

编译时字节码中生成的异常表在类加载后会存储到方法区的类信息中，在执行该方法时，异常表会被加载到栈帧的帧数据中，异常表中包含了该方法异常捕获的生效范围以及异常发生后跳转到的字节码指令位置。

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240212125203.png)

程序运行中触发异常时，Java 虚拟机会从上至下遍历异常表中的所有条目。当触发异常的字节码的索引值在某个异常表条目的监控范围内，Java 虚拟机会判断所抛出的异常和该条目想要捕获的异常是否匹配。

1. 如果匹配，跳转到“跳转 PC”对应的字节码位置。
2. 如果遍历完都不能匹配，说明异常无法在当前方法执行时被捕获，此方法栈帧直接弹出，在上一层的栈帧中进行异常捕获的查询。

#### finally 实现

finally 中的字节码指令会插入到 try 和 catch 代码块中，保证在 try 和 catch 执行之后一定会执行 finally 中的代码。如下图所示，其中`iconst_3 istore1`这两条指令在三个位置出现了，这样就保证了没有抛异常、抛异常但捕获、抛异常但未捕获三种情况都会执行 finally 中的代码。

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240212131857.png)

### JIT 即时编译器

执行一条字节码指令的时候，首先解释器会进行语法分析，然后根据结果进行对应的操作，这个逻辑是在 C++层面实现的，但是 JVM 本质是一个可执行文件，因此运行的还是机器码。JIT 会对一些热点方法的字节码进行优化，直接将其编译成机器码同时进行一些效率优化，之后执行这段字节码时就不再解释执行而是直接执行机器码，相比于解释执行更快的原因在于：

1. 这段机器码不是 C++逻辑编译生成的，而是字节码直接编译生成的
2. 进行了一些优化
3. 省去了语法解析的过程

#### JIT 即时编译器种类

在 HotSpot 中，有三款即时编译器，C1、C2 和 Graal，其中 Graal 在 GraalVM 章节中已经介绍过。 C1 编译效率比 C2 快，但是优化效果不如 C2。

C1 即时编译器和 C2 即时编译器都有独立的线程去进行处理，内部会保存一个队列，队列中存放需要编译的任务。一般即时编译器是针对方法级别来进行优化的，当然也有对循环进行优化的设计。

#### 常见的 JIT 即时编译器优化手段

##### 方法内联

方法体中的字节码指令直接复制到调用方的字节码指令中，节省了创建栈帧的开销。

##### 逃逸分析

如果方法中的一个对象不会逃逸，只会被当前局部变量引用（不会调别的方法把这个对象传进去，也不会将这个对象 return 出去），那么该对象可以直接在栈上分配内存，不用分配堆内存。下面是一个例子

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240212134513.png)
