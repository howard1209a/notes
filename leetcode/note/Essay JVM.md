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

将常量池中的符号引用替换为内存直接引用。符号引用就是在字节码文件中使用编号来访问常量池中的内容。

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

SPI 是一个为某个接口寻找服务实现的机制。有点类似 IOC 的思想，就是将装配的控制权移到程序之外。当服务的提供者，提供了服务接口的一种实现之后，在 jar 包的 META-INF/services/目录里同时创建一个以服务接口命名的文件。该文件里就是实现该服务接口的具体实现类的类名。而当外部程序装配这个模块的时候，就能通过该 jar 包 META-INF/services/里的配置文件找到具体的实现类名，并装载实例化，完成模块的注入。

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

存放类信息、运行时常量池、字符串常量池

- 类信息：所有已经加载的类的 InstanceKlass 对象
- 运行时常量池：存放常量符号到内存地址的直接映射，包括类名接口名属性名等等
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

分代算法将堆内存划分为年轻代和老年代，其中年轻代又分为伊甸园区和幸存者区，幸存者区分为 from 空间和 to 空间。新创建的对象会被放入伊甸园区，伊甸园区满会触发 Minor GC，将伊甸园区和 From 空间中需要回收的对象回收，把没有回收的对象放入 To 区，然后交换 from 和 to 名字。每次 Minor GC 都会让对象年龄加一。如果 Minor GC 后对象的年龄达到阈值，该对象就会被转移到老年代。当老年代空间不足时会触发 Full GC，Full GC 会对整个堆进行垃圾回收。如果 Full GC 后老年代空间仍然不足，则会抛出 Out Of Memory 异常。

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
