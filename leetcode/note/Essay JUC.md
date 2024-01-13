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
- WAITING：wait()、join()、await()、LockSupport.park()调用时会进入的阻塞态，其中 await()、LockSupport.park()底层走的都是 UNSAFE.park()，join()底层是 wait()
- TIMED_WAITING：Threap.sleep()和 wait(n)调用时会进入的阻塞态

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

#### 批量重偏向

当前有一些偏向锁偏向 A 线程，B 线程尝试获取这些锁导致多线程非争用地共享锁，偏向锁升级成轻量级锁，当升级次数超过一定阈值的时候，JVM 会将剩余的偏向锁批量地重偏向给 B 线程。

#### 批量撤销偏向

当升级次数超过更高阈值的时候，JVM 会觉得根本不该偏向，因此会让该类的所有对象都不可偏向，即最低就是轻量级锁。

#### 重量级锁的六个过程

##### 上锁

将 Monitor 的 Owner 置为当前线程，继续执行

##### 上锁但锁已被占用

发现 Owner 非空，将当前线程加入 EntryList，状态从 RUNNABLE 变为 BLOCKED，阻塞

##### 释放锁

将当前线程从 Owner 删掉，按照某种算法从 EntryList 中选出一个线程放入 Owner，状态从 BLOCKED 置为 RUNNABLE，继续执行

##### wait

将当前线程放入 WaitSet，状态从 RUNNABLE 变成 WAITING，然后执行释放锁流程，都执行完后当前线程阻塞

##### notify/notifyAll

从 WaitSet 拿出一个线程（notify）或拿出全部线程（notifyAll）放入 EntryList，状态从 WAITING 转为 BLOCKED，继续执行

##### interrupt

如果一个线程正处于 WAITING 状态，打断它等同于从 WaitSet 拿出它并放入 EntryList，状态从 WAITING 转为 BLOCKED，抢到锁后直接抛出 InterruptedException 异常

#### join 原理

判断目标线程是否 alive，不断地 wait 当前线程

## park 和 unpark 原理

`LockSupport.park()`和`LockSupport.unpark()`底层实现依赖 native 的`UNSAFE.park`和`UNSAFE.unpark`。

每个线程都有自己的一个 Parker 对象，由三部分组成 `_counter`，`_cond`和`_mutex`，其中`_counter`为 0 代表耗尽，1 代表充足。

### park

如果当前`_counter`为 0，阻塞当前线程，状态从 RANNBLE 变为 WAITING

如果当前`_counter`为 1，将其置为 0，继续运行不会阻塞

### unpark

如果此时线程被 park 阻塞了，则唤醒线程，状态从 WAITING 变为 RUNNABLE

如果此时线程正在运行，则将线程的`_counter`置为 1

### 对比 wait & notify

wait，notify 和 notifyAll 必须配合 Object Monitor 一起使用，而 park，unpark 不必

park & unpark 是以线程为单位来【阻塞】和【唤醒】线程，而 notify 只能随机唤醒一个等待线程，notifyAll 是唤醒所有等待线程，就不那么【精确】

park & unpark 可以先 unpark，而 wait & notify 不能先 notify

## 死锁、活锁和饥饿

在设计锁的粒度时，要尽可能地细粒度以提高程序的并发度，但是需要注意死锁等问题。

### 死锁

多个线程因互相等待资源而共同陷入阻塞的情况

检测死锁可以使用 jconsole 工具，或者使用 jps 定位进程 id，再用 jstack 定位死锁

### 活锁

多个线程互相改变对方的结束条件，导致虽然大家都在运行，没有阻塞，但是都不能结束的情况

### 饥饿

线程因为某种原因一直得不到运行的情况

## ReentrantLock

相对于 synchronized ，ReentrantLock 阻塞式获取锁的过程具备如下特点：

- 可打断
- 可以设置超时时间
- 可以设置为公平锁
- 支持多个条件变量

synchronized 阻塞式获取锁的过程状态是 BLOCKED，不可打断，不可超时退出，非公平锁，只有一个 WaitSet 因此只支持一个条件变量。

与 synchronized 一样，都支持可重入

### 基本语法

#### 最常见使用

```java
ReentrantLock lock = new ReentrantLock();
lock.lock();
try {

} finally {
    lock.unlock();
}
```

#### 可打断

```java
ReentrantLock lock = new ReentrantLock();
try {
    lock.lockInterruptibly();
} catch (InterruptedException e) {
    throw new RuntimeException(e);
}
```

#### 非阻塞式获取锁

```java
ReentrantLock lock = new ReentrantLock();
lock.tryLock();
```

#### 可打断可超时退出

```java
ReentrantLock lock = new ReentrantLock();
try {
    if (!lock.tryLock(1, TimeUnit.SECONDS)) {
        return;
    }
} catch (InterruptedException e) {
    return;
}
try {

} finally {
    lock.unlock();
}
```

#### 公平锁

ReentrantLock 默认是非公平锁，公平锁需要在构造时指定

```java
ReentrantLock lock = new ReentrantLock(true);
```

#### 条件变量

synchronized 只支持一个条件变量 WaitSet，ReentrantLock 可以创建多个条件变量，除此之外两者几乎相同。

await 的线程可以被唤醒（signal、signalAll）、被打断（interrupt）、超时（如果 await 设置了时间），重新竞争 lock 锁。

```java
ReentrantLock lock = new ReentrantLock();
Condition condition = lock.newCondition();
condition.await();
condition.signal();
condition.signalAll();
```

## 共享模型之内存

### JMM

JMM 即 Java Memory Model，它定义了主存、工作内存抽象概念，底层对应着 CPU 寄存器、Cache 缓存和硬件内存。

### 一个错误的例子

![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240113141002.png)

因为 t 线程要频繁从主内存中读取 run 的值，JIT 编译器会将 run 的值缓存至自己工作内存中的高速缓存中， 减少对主存中 run 的访问，提高效率。1 秒之后，main 线程修改了 run 的值，并同步至主存，而 t 是从自己工作内存中的高速缓存中读取这个变量 的值，结果永远是旧值。

### volatile

#### 可见性

volatile 可以用来修饰属性或静态属性，volatile 修饰的变量在任意线程的读写都将直接针对主存，可以说某个变量如果被多线程共享一般都会设置为 volatile 的，以保证可见性和有序性。

这里其实和 CPU 缓存比较像，CPU 的 L1、L2Cache 中会缓存内存中的内存块，如果一个变量只有单个 CPU 访问，那么写回、写直达等策略就可以保证缓存一致性，但是如果一个变量被多个 CPU 访问，会使用总线嗅探来解决多核间的缓存一致性。但是 Java 内存模型没有总线嗅探功能，只能强制所有线程直接读写主存来保证可见性。

volatile 本身不能保证原子性，但上述例子中对静态属性 run 的读写本身是原子性的，因为读写都是单条字节码，如下图所示。
![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240113143347.png)

#### 有序性

指令重排：JVM 会在不影响单线程执行正确性的前提下，调整语句的执行顺序，目的是提高 cpu 流水线技术的工作效率。但是多线程下『指令重排』会影响正确性。

#### happens-before 原则

happens-before 原则规定了对共享变量的写操作对其它线程的读操作可见，除了 happens-before 原则之外的其他情况，JMM 并不能保证一个线程对共享变量的写，对于其它线程对该共享变量的读可见

#### 读写屏障

volatile 是通过读写屏障机制保证的可见性和有序性，对 volatile 变量的写指令后会加入写屏障，对 volatile 变量的读指令前会加入读屏障。

- 写屏障：同步工作内存到主存，并保证不会将写屏障之前的代码排在写屏障之后
- 读屏障：清空工作内存，并保证不会将读屏障之后的代码排在读屏障之前

## 共享模型之无锁

### CAS

CAS 全称 compareAndSet，compareAndSet 操作的执行过程是先比较，如果相等就 set 返回成功，不相等就返回失败，compareAndSet 操作本身是原子的，并且它的原子性来自于 CPU 指令级别的保证，CAS 的底层是 `lock cmpxchg` 指令(X86 架构)，在单核时单条机器指令自然是原子的，在多核时，`lock cmpxchg` 指令会锁住总线保证只有自己在执行。

### CAS 和 synchronized 对比

- CAS：CAS 是乐观锁，体现的是无锁并发、无阻塞并发，线程不会陷入阻塞，这是效率提升的因素之一，但如果竞争激烈，可以想到重试必然频繁发生，反而效率会受影响
- synchronized：悲观锁，线程会阻塞，线程上下文切换会带来较大资源开销

### JUC 包提供的 Atomic 实现

- 原子数值：提供对数值的 CAS 操作
  - AtomicBoolean
  - AtomicInteger
  - AtomicLong
- 原子引用：提供对引用对象的 CAS 操作，这里的 compare 对比的是两个对象是否==
  - AtomicReference
- 原子数组：提供对数组中的数值或引用的 CAS 操作
  - AtomicIntegerArray
  - AtomicLongArray
  - AtomicReferenceArray
- 字段更新器：利用字段更新器，可以针对对象的某个域(Field)进行原子操作，只能配合 volatile 修饰的字段使用，否则会出现异常。适用场景：有一个类，类里面有一个属性，属性并不是 Atomic 的，现在有一个此类的对象，我希望 CAS 原子地操作这个对象的属性。
  - AtomicReferenceFieldUpdater
  - AtomicIntegerFieldUpdater
  - AtomicLongFieldUpdater

### Unsafe

JUC 包提供的 Atomic 实现全部都是依靠 Unsafe 的 native 方法实现的。Unsafe 类是单例的， 它提供了非常底层的，操作内存、线程的方法，Unsafe 对象不能直接调用，只能通过反射获得。

## 共享模型之不可变

不可变类就是不能够修改内部属性

- 将类和类中所有属性都设置为 final 的，那么这个类的对象一定是不可变的
- 没有属性的类一定是不可变的

## 共享模型之工具

### 线程池

Java 线程池采用了常见的一种接口设计方式
![](https://raw.githubusercontent.com/howard1209a/image-resource/main/note/20240113165911.png)

具体线程池知识详见并发编程 pdf 的 P146
