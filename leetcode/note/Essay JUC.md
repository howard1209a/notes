---
title: Java Util Concurrent
date: 2024-01-08 09:41:54
tags: [Java, 八股文]
---

# Java Util Concurrent

## Java 线程

### 多线程什么时候效率高

单核 cpu 下，多线程不能实际提高 cpu 密集型程序的运行效率（线程切换还会带来额外的资源损耗），但可以在不同的任务之间切换从而实现并发效果。多核 cpu 同时跑多个线程才能真正提高程序运行效率。

IO 操作不占用 cpu，只是我们一般拷贝文件使用的是【阻塞 IO】，这时相当于线程虽然不用 cpu，但需要一直等待 IO 结束，没能充分利用线程。所以才有后面的【非阻塞 IO】和【异步 IO】优化。

### FutureTask

FutureTask 能够接收 Callable 类型的参数，用来处理有返回结果的情况

```java
FutureTask<Integer> task3 = new FutureTask<>(() -> {
    log.debug("hello");
    return 100;
});
new Thread(task3, "t3").start();
Integer result = task3.get();
log.debug("结果是:{}", result);
```

### 查看进程线程

#### linux

`ps -ef` 查看当前时刻所有进程

`top` 动态监视所有进程状态，包括 cpu 利用率、内存占用率等等。

#### Java

`jps` 命令查看所有 Java 进程

`jstack PID` 查看某个 Java 进程(PID)的所有线程状态

### 线程运行原理

每个线程启动后，JVM 都会为其分配一块栈内存。一个线程栈由多个栈帧组成，对应着方法的递归调用，栈顶的栈帧是当前的活动栈帧，对应着正在执行的方法。每个栈帧中都要保存程序计数器（下一条字节码地址）、局部变量、返回地址（方法结束后返回的下一条字节码地址）。

### API 细节

| 方法名          | static | 功能说明                                                                                                                                                                                                                                                                                   |
| --------------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| start()         |        | native 方法，在 JVM 层面创建一个线程，初始化线程栈等资源，线程状态从 NEW 切换到 RUNNABLE，映射操作系统内核实际线程，切换到就绪态                                                                                                                                                           |
| join()          |        | 阻塞当前线程直到 thread 对象对应的线程结束，阻塞状态是 WAITING，底层依靠 synchronized 的 wait 实现                                                                                                                                                                                         |
| setPriority()   |        | 设置线程优先级，较大的优先级 能提高该线程被 CPU 调度的机率                                                                                                                                                                                                                                 |
| isInterrupted() |        | 判断是否被打断，不会清除打断标记                                                                                                                                                                                                                                                           |
| interrupted()   | static | 判断当前线程是否被打断，会清除打断标记                                                                                                                                                                                                                                                     |
| interrupt()     |        | 打断线程，如果被打断线程正在 sleep（TIMED_WAITING），wait（WAITING），join（本质是 wait，WAITING） 会导致被打断的线程抛出 InterruptedException，并清除打断标记。如果打断的正在运行的线程，则会设置打断标记。park 的线程被打断，也会设置打断标记，如果打断标记已经是 true, 则 park 会失效。 |
| sleep()         | static | native 方法，当前线程转到阻塞态（TIMED_WAITING），一定时间后恢复到 RANNABLE                                                                                                                                                                                                                |
| yield()         | static | native 方法，提示 JVM 层面的线程调度器让出当前线程对 CPU 的使用，循环调 yield()方法也相当于让当前线程几乎不运行                                                                                                                                                                            |

### 守护线程

默认情况下，Java 进程需要等待所有线程都运行结束，才会结束。有一种特殊的线程叫做守护线程，只要其它非守
护线程运行结束了，即使守护线程的代码没有执行完，也会强制结束。

垃圾回收器线程就是一种守护线程

### 线程状态

#### 操作系统层面

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240112171408.png)

- 初始状态：仅是在语言层面创建了线程对象，操作系统还未实际创建线程
- 就绪状态：指该线程已经被创建(分配了资源)，可以由 CPU 调度运行
- 运行状态：获取了 CPU 时间片正在运行，当 CPU 时间片用完，会从运行状态转换至就绪状态，会导致线程的上下文切换
- 阻塞状态：不再接受 CPU 的调度，设置了相应的唤醒条件，在一定条件下会被软中断触发唤醒，转换到就绪状态
- 终止状态：表示线程已经执行完毕，生命周期已经结束，不会再转换为其它状态

#### JVM 层面

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240112172052.png)

- NEW：只是 new 了 Thread 对象，还没有调用 start() 方法，线程没有注册到 JVM
- RUNNABLE：调用了 start() 方法之后，在 JVM 层面创建一个线程，初始化线程栈等资源，线程状态从 NEW 切换到 RUNNABLE，映射操作系统内核实际线程，切换到就绪态。注意 JVM 层面的 RUNNABLE 状态涵盖了操作系统层面的就绪状态和运行状态。
- BLOCKED：synchronized 没有抢到锁时线程会进入的阻塞态
- WAITING：wait()、await()、LockSupport.park()调用时会进入的阻塞态，其中 await()、LockSupport.park()底层走的都是 UNSAFE.park()
- TIMED_WAITING：Threap.sleep()调用时会进入的阻塞态

## Monitor 管程

### Java 中变量的自增和自减

Java 中变量的自增和自减并不是原子操作，因为它们对应多条字节码（JVM 只保证单条字节码的执行是原子的）。

对于 i++而言，实际会产生如下的 JVM 字节码指令：

```java
getstatic i // 获取静态变量i的值
iconst_1 // 准备常量1
iadd // 自增
putstatic i // 将修改后的值存入静态变量i
```

### synchronized 关键字

synchronized 方法将锁加在了 this 对象上，synchronized 静态方法将锁加在了类的 class 对象上

### 一个变量什么时候线程安全

#### 属性和静态属性

- 没有多线程共享：线程安全
- 被多线程共享：
  - 本身是不可变类型：线程安全
  - 可变类型：
    - 只存在并发读：线程安全
    - 存在并发读写：线程不安全

#### 局部变量

- 数值类型：线程安全
- 引用类型：
  - 对象没有逃离方法的作用访问：线程安全
  - 对象逃离方法的作用范围：线程不安全

#### String 引用辨析

```java
// 创建并指向一个堆中String对象"abc"，这个对象指向常量池中"abc"
String s1 = new String("abc");

// 直接指向常量池中"abc"
String s2 = "abc";

// 直接指向常量池中"abc"
String s3 = "ab" + "c";

// 直接指向常量池中"c"
String c = "c";

// 创建并指向一个堆中String对象"abc"，这个对象指向常量池中"abc"
String s4 = "ab" + c;
```

### Monitor 管程概念

#### Java 对象头

以 32 位虚拟机（JVM）为例：
![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240112213325.png)

#### 重量级锁（fat lock）

每个 Java 对象都可以关联一个 Monitor 对象，如果使用 synchronized 给对象上锁(重量级)之后，该对象头的
Mark Word 中就被设置指向 Monitor 对象的指针

Monitor 结构如下
![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240112214733.png)

- 刚开始 Monitor 中 Owner 为 null
- 当 Thread-2 执行 synchronized(obj) 就会将 Monitor 的所有者 Owner 置为 Thread-2
- 之后如果 Thread-3，Thread-4，Thread-5 也来执行 synchronized(obj)，就会进入 EntryList，线程状态变为 BLOCKED
- Thread-2 释放锁时，会按照某种算法（非公平）选择 EntryList 中某个线程置为 Owner，然后将这个线程的状态转为 Runnable

#### 重量级锁的自旋优化

当多个线程尝试获取同一个重量级锁的时候，没有获取到锁的线程可能不会立刻进入 EntryList 变成 BLOCKED 状态，而是会进行 CPU 自旋空转，多尝试几次获取重量级锁，自旋优化某些时候性能高于直接阻塞，因为防止了线程状态的切换。自旋优化在 Java 中是自适应的。

下图为自旋重试成功的情况
![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240112223816.png)

#### 轻量级锁

如果多个线程共享锁，但不存在锁争用，这时采用重量级锁维护 Monitor 是比较浪费的。轻量级锁可以保证多个线程不存在锁争用时的共享锁，并且能够在出现锁争用时升级重量级锁。

Lock Record 对象，每个线程的每个栈帧都会包含一个锁记录的结构，如下图所示
![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240112221512.png)

当某个线程取得轻量级锁时，会交换 Lock Record 中的`lock record 地址 00`部分和对象头中的`Mark Word`，然后`Object reference`部分指向对象头。结果如下图
![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240112221933.png)

当某个线程尝试取得轻量级锁时，有可能发现对象头正处于轻量级锁状态并且`lock record`地址非空，此时如果是其他线程占用，则说明出现了锁争用，锁膨胀成重量级锁，如果是本线程占用，则进行锁重入，再新增一个`lock record`，结果如下图
![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240112222536.png)

下图是 Thread-0 已经加了轻量级锁，之后 Thread-1 加轻量级锁失败，进入锁膨胀流程，创建 Monitor，Owner 置为 Thread-0，EntryList 放入 Thread-1
![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240112223117.png)

#### 偏向锁

如果只有单线程使用锁，那么不需要在栈帧中添加`lock record`，只需要在对象头的`Mark Word`中记录线程 ID 即可，这种方式相比于轻量级锁减少了很多 CAS 操作。偏向锁保证单个线程占用锁，并且能够在多线程共享锁时升级轻量级锁。

只有第一次使用 CAS 将线程 ID 设置到对象的 Mark Word 头，之后发现这个线程 ID 是自己的就表示没有线程共享锁，不用重新 CAS。以后只要不发生竞争，这个对象就归该线程所有。

#### 偏向锁升级轻量级锁示例

如图两个代码块是严格顺序执行的，多线程锁共享但不存在锁争用，所以第一次和第二次打印都是偏向锁偏向 t1 线程，第三次打印是锁升级的轻量级锁指向 t2 的`lock record`，第四次打印是轻量级锁释放后的 Normal 状态（之后不会再使用偏向锁了）。
![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240112234054.png)

下图是执行结果
![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240112234538.png)
