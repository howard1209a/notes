---
title: Netty总结
date: 2023-12-05 10:22:05
tags: [Netty, 八股文]
---

# Netty 总结

## 多路复用 IO 底层实现

多路复用 IO 特指对于多个网络 IO 的单线程阻塞式事件获取，java 中可以通过`selector.select()`完成，但是 selector 只是一个多路复用 IO 接口，不同操作系统底层有着不同的底层实现，其中最主要的实现方式是 select、poll 和 epoll 这三种系统调用。内核只实现了网络 IO 的多路复用，普通文件 IO 没有多路复用机制。

大多数文件系统的默认 IO 操作都是缓存 IO。在 Linux 的缓存 IO 机制中，操作系统会将 IO 的数据缓存在文件系统的页缓存（page cache）。也就是说，数据会先被拷贝到操作系统内核的缓冲区中，然后才会从操作系统内核的缓存区拷贝到应用程序的地址空间中。这种做法的缺点就是，需要在应用程序地址空间和内核进行多次拷贝，这些拷贝动作所带来的 CPU 以及内存开销是非常大的。

至于为什么不能直接让磁盘控制器把数据送到应用程序的地址空间中呢？最简单的一个原因就是应用程序不能直接操作底层硬件。

### select

在 linux 中，任何 IO 都被抽象为一个文件，有对应的文件描述符 fd。

select 系统调用时会首先将需要监控的所有 fd 从用户内存拷贝到内核内存，然后内核遍历自己监控的所有 fd，通过 poll 逻辑检查 socket 是否有读写事件，遍历完后如果有事件则返回遍历到的事件集合，如果没有事件则睡眠当前内核进程并设置唤醒条件为 fd 产生事件或 timeout 超时，内核进程唤醒后会再次遍历所有 fd（如果是 timeout 唤醒则不知道有没有事件，如果是事件唤醒则不知道哪些 IO 产生了哪些事件，所以都要整体遍历一遍），重复过程...

上述过程中的内核进程唤醒是通过软中断完成的，即 timeout 超时或某个 fd 产生事件的时候会触发一个软中断，操作系统探查时检测到中断会执行操作系统中断处理程序，唤醒相应的内核进程。

select 存在三个问题：

- 每次调用 select，都需要把被监控的 fd 集合从用户态空间拷贝到内核态空间，高并发场景下这样的拷贝会使得消耗的资源是很大的。
- 能监听端口的数量有限，单个进程所能打开的最大连接数受限于 FD_SETSIZE 宏定义。
- 每次都要遍历所有 fd，调用 socket 的 poll 函数查看事件

### poll

poll 的实现和 select 非常相似，只是描述 fd 集合的方式不同。针对 select 遗留的三个问题中的 fd 个数限制问题，poll 只是使用 pollfd 结构而不是 select 的 fd_set 结构，这使得不再存在 fd 个数限制，但是另外两个问题仍然存在。

### epoll

相比于 select，epoll 最大的好处在于它不使用轮询机制，所以不会随着监听 fd 数目的增长而降低效率。

epoll 系统调用函数内部有三个数据结构：

- wq：等待队列链表。软中断数据就绪的时候会通过 wq 来找到阻塞在 epoll 对象上的用户进程。
- rbr：红黑树。为了支持对海量连接的高效查找、插入和删除，eventpoll 内部使用的就是红黑树。通过红黑树来管理用户主进程 accept 添加进来的所有 socket 连接。
- rdllist：就绪的描述符链表。当有连接就绪的时候，内核会把就绪的连接放到 rdllist 链表里。**这样应用进程只需要判断链表就能找出就绪进程，而不用去遍历红黑树的所有节点了。**

### 总结

select，poll 实现需要自己不断轮询所有 fd 集合，直到产生 IO 事件，期间可能要睡眠和唤醒多次交替。而 epoll 其实也需要调用 epoll_wait 不断轮询就绪链表，期间也可能多次睡眠和唤醒交替，但是它是 IO 产生 IO 事件时，主动调用回调函数，把就绪 fd 放入就绪链表中，并唤醒在 epoll_wait 中睡眠的进程。虽然都要睡眠和交替，但是 select 和 poll 在“醒着”的时候要遍历整个 fd 集合，而 epoll 在“醒着”的时候只要判断一下就绪链表是否为空就行了，这节省了大量的 CPU 时间。这就是回调机制带来的性能提升。

### 参考资料

https://zhuanlan.zhihu.com/p/367591714

## Selector

Selector 是一个 java 中的多路复用 IO 接口，我们可以为 Selector 注册 channel，并绑定这个 channel 上的一些事件，绑定的事件类型可以有：

- connect：客户端连接成功时触发
- accept：服务器端内核 tcp 就绪队列中有数据时触发
- read：内核 tcp 接收缓冲区中有数据时触发
- write：内核 tcp 发送缓冲区中有空闲位置时触发

### 三种监听事件方式

- 阻塞直到绑定事件发生
  ```java
  int count = selector.select();
  ```
- 阻塞直到绑定事件发生，或是超时
  ```java
  int count = selector.select(long timeout);
  ```
- 不会阻塞，也就是不管有没有事件，立刻返回，自己根据返回值检查是否有事件
  ```java
  int count = selector.selectNow();
  ```

### 示例代码

#### 基础 CS 架构

注意示例代码中`iter.remove()`是必要的，因为 select 在事件发生后，就会将相关的 key 放入 selectedKeys 集合，但不会在处理完后从 selectedKeys 集合中移除，需要我们自己删除，如果不删除之后的每次`selector.selectedKeys()`调用都会携带这个 key。

另一个点就是如果`sc.read(buffer)`读取过后，tcp 接收缓冲区中还有没被读出来的数据（buffer 较小），那么读事件不会消失，下次的`selector.select()`会直接触发。

##### 服务端

```java
@Slf4j
public class ChannelDemo6 {
    public static void main(String[] args) {
        try (ServerSocketChannel channel = ServerSocketChannel.open()) {
            channel.bind(new InetSocketAddress(8080));
            System.out.println(channel);
            Selector selector = Selector.open();
            channel.configureBlocking(false);
            channel.register(selector, SelectionKey.OP_ACCEPT);

            while (true) {
                int count = selector.select();
//                int count = selector.selectNow();
                log.debug("select count: {}", count);
//                if(count <= 0) {
//                    continue;
//                }

                // 获取所有事件
                Set<SelectionKey> keys = selector.selectedKeys();

                // 遍历所有事件，逐一处理
                Iterator<SelectionKey> iter = keys.iterator();
                while (iter.hasNext()) {
                    SelectionKey key = iter.next();
                    // 判断事件类型
                    if (key.isAcceptable()) {
                        ServerSocketChannel c = (ServerSocketChannel) key.channel();
                        // 必须处理
                        SocketChannel sc = c.accept();
                        sc.configureBlocking(false);
                        sc.register(selector, SelectionKey.OP_READ);
                        log.debug("连接已建立: {}", sc);
                    } else if (key.isReadable()) {
                        SocketChannel sc = (SocketChannel) key.channel();
                        ByteBuffer buffer = ByteBuffer.allocate(128);
                        int read = sc.read(buffer);
                        if(read == -1) {
                            key.cancel();
                            sc.close();
                        } else {
                            buffer.flip();
                            debugAll(buffer);
                        }
                    }
                    // 处理完毕，必须将事件移除
                    iter.remove();
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

##### 客户端

```java
public class Client {
    public static void main(String[] args) {
        try (Socket socket = new Socket("localhost", 8080)) {
            System.out.println(socket);
            socket.getOutputStream().write("world".getBytes());
            System.in.read();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

#### 分隔符解决黏包半包

buffer 被 attach 到 key 事件中，读事件触发后，可以从读事件 key 中取出这个 socket 的 buffer，如果 buffer 不够长还会动态扩容。这种方式每次都要检测分隔符，因此效率比较低。

##### 服务端

```java
private static void split(ByteBuffer source) {
    source.flip();
    for (int i = 0; i < source.limit(); i++) {
        // 找到一条完整消息
        if (source.get(i) == '\n') {
            int length = i + 1 - source.position();
            // 把这条完整消息存入新的 ByteBuffer
            ByteBuffer target = ByteBuffer.allocate(length);
            // 从 source 读，向 target 写
            for (int j = 0; j < length; j++) {
                target.put(source.get());
            }
            debugAll(target);
        }
    }
    source.compact(); // 0123456789abcdef  position 16 limit 16
}

public static void main(String[] args) throws IOException {
    // 1. 创建 selector, 管理多个 channel
    Selector selector = Selector.open();
    ServerSocketChannel ssc = ServerSocketChannel.open();
    ssc.configureBlocking(false);
    // 2. 建立 selector 和 channel 的联系（注册）
    // SelectionKey 就是将来事件发生后，通过它可以知道事件和哪个channel的事件
    SelectionKey sscKey = ssc.register(selector, 0, null);
    // key 只关注 accept 事件
    sscKey.interestOps(SelectionKey.OP_ACCEPT);
    log.debug("sscKey:{}", sscKey);
    ssc.bind(new InetSocketAddress(8080));
    while (true) {
        // 3. select 方法, 没有事件发生，线程阻塞，有事件，线程才会恢复运行
        // select 在事件未处理时，它不会阻塞, 事件发生后要么处理，要么取消，不能置之不理
        selector.select();
        // 4. 处理事件, selectedKeys 内部包含了所有发生的事件
        Iterator<SelectionKey> iter = selector.selectedKeys().iterator(); // accept, read
        while (iter.hasNext()) {
            SelectionKey key = iter.next();
            // 处理key 时，要从 selectedKeys 集合中删除，否则下次处理就会有问题
            iter.remove();
            log.debug("key: {}", key);
            // 5. 区分事件类型
            if (key.isAcceptable()) { // 如果是 accept
                ServerSocketChannel channel = (ServerSocketChannel) key.channel();
                SocketChannel sc = channel.accept();
                sc.configureBlocking(false);
                ByteBuffer buffer = ByteBuffer.allocate(16); // attachment
                // 将一个 byteBuffer 作为附件关联到 selectionKey 上
                SelectionKey scKey = sc.register(selector, 0, buffer);
                scKey.interestOps(SelectionKey.OP_READ);
                log.debug("{}", sc);
                log.debug("scKey:{}", scKey);
            } else if (key.isReadable()) { // 如果是 read
                try {
                    SocketChannel channel = (SocketChannel) key.channel(); // 拿到触发事件的channel
                    // 获取 selectionKey 上关联的附件
                    ByteBuffer buffer = (ByteBuffer) key.attachment();
                    int read = channel.read(buffer); // 如果是正常断开，read 的方法的返回值是 -1
                    if(read == -1) {
                        key.cancel();
                    } else {
                        split(buffer);
                        // 需要扩容
                        if (buffer.position() == buffer.limit()) {
                            ByteBuffer newBuffer = ByteBuffer.allocate(buffer.capacity() * 2);
                            buffer.flip();
                            newBuffer.put(buffer); // 0123456789abcdef3333\n
                            key.attach(newBuffer);
                        }
                    }

                } catch (IOException e) {
                    e.printStackTrace();
                    key.cancel();  // 因为客户端断开了,因此需要将 key 取消（从 selector 的 keys 集合中真正删除 key）
                }
            }
        }
    }
}
```

##### 客户端

```java
public static void main(String[] args) {
        try (Socket socket = new Socket("localhost", 6984)) {
            System.out.println(socket);
            socket.getOutputStream().write("hello\n".getBytes());
            Thread.sleep(1500);
            socket.getOutputStream().write("hellogqesabqeadrbdbr\n***".getBytes());
            Thread.sleep(1500);
            socket.getOutputStream().write("helloedqabaaaaaaehberqedbhrhbarbrwbrdw\n".getBytes());
            System.in.read();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
```

#### 写事件解决数据太长一次无法写完

非阻塞模式下，无法保证把 buffer 中所有数据都写入 channel，因此需要追踪 write 方法的返回值（代表实际写入字节数）。

注意只有一次没写完才注册写事件，数据都写完之后要及时取消写事件注册，否则之后只要 tcp 发送缓冲区有空闲空间就还会触发。

##### 服务端

```java
public static void main(String[] args) throws IOException {
        ServerSocketChannel ssc = ServerSocketChannel.open();
        ssc.configureBlocking(false);
        ssc.bind(new InetSocketAddress(8080));

        Selector selector = Selector.open();
        ssc.register(selector, SelectionKey.OP_ACCEPT);

        while(true) {
            selector.select();

            Iterator<SelectionKey> iter = selector.selectedKeys().iterator();
            while (iter.hasNext()) {
                SelectionKey key = iter.next();
                iter.remove();
                if (key.isAcceptable()) {
                    SocketChannel sc = ssc.accept();
                    sc.configureBlocking(false);
                    SelectionKey sckey = sc.register(selector, SelectionKey.OP_READ);
                    // 1. 向客户端发送内容
                    StringBuilder sb = new StringBuilder();
                    for (int i = 0; i < 3000000; i++) {
                        sb.append("a");
                    }
                    ByteBuffer buffer = Charset.defaultCharset().encode(sb.toString());
                    int write = sc.write(buffer);
                    // 3. write 表示实际写了多少字节
                    System.out.println("实际写入字节:" + write);
                    // 4. 如果有剩余未读字节，才需要关注写事件
                    if (buffer.hasRemaining()) {
                        // read 1  write 4
                        // 在原有关注事件的基础上，多关注 写事件
                        sckey.interestOps(sckey.interestOps() + SelectionKey.OP_WRITE);
                        // 把 buffer 作为附件加入 sckey
                        sckey.attach(buffer);
                    }
                } else if (key.isWritable()) {
                    ByteBuffer buffer = (ByteBuffer) key.attachment();
                    SocketChannel sc = (SocketChannel) key.channel();
                    int write = sc.write(buffer);
                    System.out.println("实际写入字节:" + write);
                    if (!buffer.hasRemaining()) { // 写完了
                        key.interestOps(key.interestOps() - SelectionKey.OP_WRITE);
                        key.attach(null);
                    }
                }
            }
        }
    }
```

##### 客户端

```java
public static void main(String[] args) throws IOException {
        Selector selector = Selector.open();
        SocketChannel sc = SocketChannel.open();
        sc.configureBlocking(false);
        sc.register(selector, SelectionKey.OP_CONNECT | SelectionKey.OP_READ);
        sc.connect(new InetSocketAddress("localhost", 8080));
        int count = 0;
        while (true) {
            selector.select();
            Iterator<SelectionKey> iter = selector.selectedKeys().iterator();
            while (iter.hasNext()) {
                SelectionKey key = iter.next();
                iter.remove();
                if (key.isConnectable()) {
                    System.out.println(sc.finishConnect());
                } else if (key.isReadable()) {
                    ByteBuffer buffer = ByteBuffer.allocate(1024 * 1024);
                    count += sc.read(buffer);
                    buffer.clear();
                    System.out.println(count);
                }
            }
        }
    }
```

#### 多线程优化-每个线程可以处理事件和执行异步任务

- 主线程：做一些准备就结束
- accept 线程：针对每个新连接都轮询地提交异步任务
- 多个 eventloop 线程：处理事件和执行异步任务

##### 服务端

```java
public class ChannelDemo7 {
    public static void main(String[] args) throws IOException {
        new BossEventLoop().register();
    }


    @Slf4j
    static class BossEventLoop implements Runnable {
        private Selector boss;
        private WorkerEventLoop[] workers;
        private volatile boolean start = false;
        AtomicInteger index = new AtomicInteger();

        public void register() throws IOException {
            if (!start) {
                ServerSocketChannel ssc = ServerSocketChannel.open();
                ssc.bind(new InetSocketAddress(8080));
                ssc.configureBlocking(false);
                boss = Selector.open();
                SelectionKey ssckey = ssc.register(boss, 0, null);
                ssckey.interestOps(SelectionKey.OP_ACCEPT);
                workers = initEventLoops();
                new Thread(this, "boss").start();
                log.debug("boss start...");
                start = true;
            }
        }

        public WorkerEventLoop[] initEventLoops() {
//        EventLoop[] eventLoops = new EventLoop[Runtime.getRuntime().availableProcessors()];
            WorkerEventLoop[] workerEventLoops = new WorkerEventLoop[2];
            for (int i = 0; i < workerEventLoops.length; i++) {
                workerEventLoops[i] = new WorkerEventLoop(i);
            }
            return workerEventLoops;
        }

        @Override
        public void run() {
            while (true) {
                try {
                    boss.select();
                    Iterator<SelectionKey> iter = boss.selectedKeys().iterator();
                    while (iter.hasNext()) {
                        SelectionKey key = iter.next();
                        iter.remove();
                        if (key.isAcceptable()) {
                            ServerSocketChannel c = (ServerSocketChannel) key.channel();
                            SocketChannel sc = c.accept();
                            sc.configureBlocking(false);
                            log.debug("{} connected", sc.getRemoteAddress());
                            workers[index.getAndIncrement() % workers.length].register(sc);
                        }
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    @Slf4j
    static class WorkerEventLoop implements Runnable {
        private Selector worker;
        private volatile boolean start = false;
        private int index;

        private final ConcurrentLinkedQueue<Runnable> tasks = new ConcurrentLinkedQueue<>();

        public WorkerEventLoop(int index) {
            this.index = index;
        }

        public void register(SocketChannel sc) throws IOException {
            if (!start) {
                worker = Selector.open();
                new Thread(this, "worker-" + index).start();
                start = true;
            }
            tasks.add(() -> {
                try {
                    SelectionKey sckey = sc.register(worker, 0, null);
                    sckey.interestOps(SelectionKey.OP_READ);
                    worker.selectNow();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            });
            worker.wakeup();
        }

        @Override
        public void run() {
            while (true) {
                try {
                    worker.select();
                    Runnable task = tasks.poll();
                    if (task != null) {
                        task.run();
                    }
                    Set<SelectionKey> keys = worker.selectedKeys();
                    Iterator<SelectionKey> iter = keys.iterator();
                    while (iter.hasNext()) {
                        SelectionKey key = iter.next();
                        if (key.isReadable()) {
                            SocketChannel sc = (SocketChannel) key.channel();
                            ByteBuffer buffer = ByteBuffer.allocate(128);
                            try {
                                int read = sc.read(buffer);
                                if (read == -1) {
                                    key.cancel();
                                    sc.close();
                                } else {
                                    buffer.flip();
                                    log.debug("{} message:", sc.getRemoteAddress());
                                    debugAll(buffer);
                                }
                            } catch (IOException e) {
                                e.printStackTrace();
                                key.cancel();
                                sc.close();
                            }
                        }
                        iter.remove();
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

#### UDP

因为 UDP 是无连接的，不是面向字节流而是面向数据报的，UDP 客户端在传输层发送一个 UDP 数据报，会在网络层拆成多个 IP 数据报，在服务端的网络层中如果所有 IP 数据报都到齐了，则会组装成 UDP 数据报给到传输层。

这样带来的结果是，无论服务端是否绑定端口，客户端都可以直接发送 UDP 数据报，如果 UDP 数据报已经到达服务端的时候，服务端仍未绑定端口并等待接收，那么 UDP 数据报不会被缓存而是被直接丢弃。

##### 服务端

```java
public static void main(String[] args) {
        try (DatagramChannel channel = DatagramChannel.open()) {
            channel.socket().bind(new InetSocketAddress(9999));
            System.out.println("waiting...");
            ByteBuffer buffer = ByteBuffer.allocate(32);
            channel.receive(buffer);
            buffer.flip();
            debug(buffer);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

##### 客户端

```java
public static void main(String[] args) {
        try (DatagramChannel channel = DatagramChannel.open()) {
            ByteBuffer buffer = StandardCharsets.UTF_8.encode("hello");
            InetSocketAddress address = new InetSocketAddress("localhost", 9999);
            channel.send(buffer, address);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

## 概念解析

### java 中 stream api 和 channel api 的区别

- stream 不会自动缓冲数据，channel 会利用系统提供的发送缓冲区、接收缓冲区（更为底层）
- stream 仅支持阻塞 API，channel 同时支持阻塞、非阻塞 API，网络 channel 可配合 selector 实现多路复用

### 零拷贝

#### 传统 IO

传统 IO 将磁盘文件写入网卡的构成存在两次系统调用和四次数据拷贝，分别是磁盘数据通过 DMA 读到内核内存，内核内存复制数据到用户内存，用户内存复制数据到 socket 缓冲区，socket 缓冲区数据通过 DMA 发到网卡（注：DMA 用于解放 cpu 完成文件 IO）。

```java
File f = new File("helloword/data.txt");
RandomAccessFile file = new RandomAccessFile(file, "r");

byte[] buf = new byte[(int)f.length()];
file.read(buf);

Socket socket = ...;
socket.getOutputStream().write(buf);
```

#### 三种零拷贝优化

所谓零拷贝指的是尽可能减少数据复制的次数以及用户态内核态切换的次数。

- 第一种：内核内存不再复制数据到用户内存，我们可以使用 DirectByteBuf 将堆外内存（内核内存）映射到 jvm 内存中来直接访问使用，需要注意 DirectByteBuf 不会被 java gc，需要手动释放。这种情况涉及两次系统调用和三次数据拷贝。
- 第二种：依赖 linux 2.1 后提供的 sendFile 方法。java 调用 transferTo 方法后，在内核态依次执行磁盘数据读入内核缓冲区，内核缓冲区数据复制到 socket 缓冲区，socket 缓冲区数据写入网卡三个操作。这种情况涉及一次系统调用和三次数据拷贝。
- 第三种：linux 2.4 的进一步优化。在第二种的基础上将内核缓冲区的数据直接写入网卡。这种情况涉及一次系统调用和两次数据拷贝。

### AIO

AIO 指的是异步 IO，之前无论是阻塞 read 还是非阻塞 read 都是同步的，也就是说它们从网卡 IO 读数据到内核内存，再从内核内存复制数据到用户内存的过程都是当前线程来完成的。AIO 把这个过程交给了一个异步线程来完成，并通过注册回调的方式通知当前线程数据结果。异步 read 函数的执行是最快的，因为它甚至不需要陷入内核态。

而在异步线程中执行的 read 任务，其方式是阻塞/非阻塞/多路复用其实都是可以的。

#### 文件 AIO

```java
public static void main(String[] args) throws IOException {
        try{
            AsynchronousFileChannel s =
                AsynchronousFileChannel.open(
                	Paths.get("1.txt"), StandardOpenOption.READ);
            ByteBuffer buffer = ByteBuffer.allocate(2);
            log.debug("begin...");
            s.read(buffer, 0, null, new CompletionHandler<Integer, ByteBuffer>() {
                @Override
                public void completed(Integer result, ByteBuffer attachment) {
                    log.debug("read completed...{}", result);
                    buffer.flip();
                    debug(buffer);
                }

                @Override
                public void failed(Throwable exc, ByteBuffer attachment) {
                    log.debug("read failed...");
                }
            });

        } catch (IOException e) {
            e.printStackTrace();
        }
        log.debug("do other things...");
        System.in.read();
    }
```

#### 网络 AIO

下面的整个过程都是异步完成的，包括`AsynchronousServerSocketChannel`和`AsynchronousSocketChannel`，代码首先提交了 accept 任务并在 accept 任务的 Completion 处理中提交一个 read 任务、一个 write 任务和一个 accept 任务，在 read 任务的 Completion 处理中，会打印 buffer 数据并再次提交 read 任务，在 write 任务的 Completion 处理中会查看当前待发送 buffer 中是否还有剩余数据，如果还有剩余数据，则再次提交 write 任务。所有任务都是在一个异步线程池中执行的。

```java
public static void main(String[] args) throws IOException {
        AsynchronousServerSocketChannel ssc = AsynchronousServerSocketChannel.open();
        ssc.bind(new InetSocketAddress(8080));
        ssc.accept(null, new AcceptHandler(ssc));
        System.in.read();
    }

    private static void closeChannel(AsynchronousSocketChannel sc) {
        try {
            System.out.printf("[%s] %s close\n", Thread.currentThread().getName(), sc.getRemoteAddress());
            sc.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static class ReadHandler implements CompletionHandler<Integer, ByteBuffer> {
        private final AsynchronousSocketChannel sc;

        public ReadHandler(AsynchronousSocketChannel sc) {
            this.sc = sc;
        }

        @Override
        public void completed(Integer result, ByteBuffer attachment) {
            try {
                if (result == -1) {
                    closeChannel(sc);
                    return;
                }
                System.out.printf("[%s] %s read\n", Thread.currentThread().getName(), sc.getRemoteAddress());
                attachment.flip();
                System.out.println(Charset.defaultCharset().decode(attachment));
                attachment.clear();
                // 处理完第一个 read 时，需要再次调用 read 方法来处理下一个 read 事件
                sc.read(attachment, attachment, this);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void failed(Throwable exc, ByteBuffer attachment) {
            closeChannel(sc);
            exc.printStackTrace();
        }
    }

    private static class WriteHandler implements CompletionHandler<Integer, ByteBuffer> {
        private final AsynchronousSocketChannel sc;

        private WriteHandler(AsynchronousSocketChannel sc) {
            this.sc = sc;
        }

        @Override
        public void completed(Integer result, ByteBuffer attachment) {
            // 如果作为附件的 buffer 还有内容，需要再次 write 写出剩余内容
            if (attachment.hasRemaining()) {
                sc.write(attachment);
            }
        }

        @Override
        public void failed(Throwable exc, ByteBuffer attachment) {
            exc.printStackTrace();
            closeChannel(sc);
        }
    }

    private static class AcceptHandler implements CompletionHandler<AsynchronousSocketChannel, Object> {
        private final AsynchronousServerSocketChannel ssc;

        public AcceptHandler(AsynchronousServerSocketChannel ssc) {
            this.ssc = ssc;
        }

        @Override
        public void completed(AsynchronousSocketChannel sc, Object attachment) {
            try {
                System.out.printf("[%s] %s connected\n", Thread.currentThread().getName(), sc.getRemoteAddress());
            } catch (IOException e) {
                e.printStackTrace();
            }
            ByteBuffer buffer = ByteBuffer.allocate(16);
            // 读事件由 ReadHandler 处理
            sc.read(buffer, buffer, new ReadHandler(sc));
            // 写事件由 WriteHandler 处理
            sc.write(Charset.defaultCharset().encode("server hello!"), ByteBuffer.allocate(16), new WriteHandler(sc));
            // 处理完第一个 accpet 时，需要再次调用 accept 方法来处理下一个 accept 事件
            ssc.accept(null, this);
        }

        @Override
        public void failed(Throwable exc, Object attachment) {
            exc.printStackTrace();
        }
    }
```

## Netty
