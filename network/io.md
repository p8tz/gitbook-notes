## 基本概念

### 文件描述符fd

文件描述符（`File descriptor`）用于表述指向文件的引用的抽象化概念。

文件描述符在形式上是一个非负整数。实际上，它是一个索引值，指向内核为每一个进程所维护的该进程打开文件的记录表。当程序打开一个现有文件或者创建一个新文件时，内核向进程返回一个文件描述符。在程序设计中，一些涉及底层的程序编写往往会围绕着文件描述符展开。但是文件描述符这一概念往往只适用于UNIX、Linux这样的操作系统。

### 缓存 I/O

缓存 I/O 又被称作标准 I/O，大多数文件系统的默认 I/O 操作都是缓存 I/O。在 Linux 的缓存 I/O 机制中，操作系统会将 I/O 的数据缓存在文件系统的页缓存（ page cache ）中，也就是说，**数据会先被拷贝到操作系统内核的缓冲区中，然后才会从操作系统内核的缓冲区拷贝到应用程序的地址空间。**

缺点：

数据在传输过程中需要在应用程序地址空间和内核进行多次数据拷贝操作，这些数据拷贝操作所带来的 CPU 以及内存开销是非常大的。

## I/O模型

上面说了，对于一次IO访问（以read举例），数据会先被拷贝到操作系统的**内核缓冲区**中，然后才会从操作系统的内核缓冲区拷贝到**应用程序的地址空间**。所以说，当一个read操作发生时，它会经历两个阶段：

1、等待数据准备 (Waiting for the data to be ready)

2、将数据从内核拷贝到进程中 (Copying the data from the kernel to the process)

对于一个套接字上的输入操作，第一步通常涉及等待数据从网络中到达。当所等待数据到达时，它被复制到**内核**中的某个**缓冲区**。第二步就是把数据从内核缓冲区复制到应用**进程缓冲区**。

正式因为这两个阶段，linux系统产生了下面五种网络模模型的方案。

- 阻塞 I/O（blocking IO）
- 非阻塞 I/O（nonblocking IO）
-  I/O 多路复用（ IO multiplexing）
- 信号驱动 I/O（ signal driven IO）
- 异步 I/O（asynchronous IO）

### 阻塞I/O

在linux中，默认情况下所有的socket都是blocking，一个典型的读操作流程大概是这样：

![20201106211200](https://gitee.com/p8t/picbed/raw/master/imgs/20201106211200.png)

当用户进程调用了recvfrom这个系统调用，kernel就开始了IO的第一个阶段：准备数据（对于网络IO来说，很多时候数据在一开始还没有到达。比如，还没有收到一个完整的UDP包。这个时候kernel就要等待足够的数据到来）。这个过程需要等待，也就是说数据被拷贝到操作系统内核的缓冲区中是需要一个过程的。而在用户进程这边，整个进程会被阻塞（当然，是进程自己选择的阻塞）。当kernel一直等到数据准备好了，它就会将数据从kernel中拷贝到用户内存，然后kernel返回结果，用户进程才解除block的状态，重新运行起来。

blocking IO的特点就是在IO执行的两个阶段都被block了。

### 非阻塞I/O

linux下，可以通过设置socket使其变为non-blocking。当对一个non-blocking socket执行读操作时，流程是这个样子：

![20201106211331](https://gitee.com/p8t/picbed/raw/master/imgs/20201106211331.png)

当用户进程发出read操作时，如果kernel中的数据还没有准备好，那么它并不会block用户进程，而是立刻返回一个error。从用户进程角度讲 ，它发起一个read操作后，并不需要等待，而是马上就得到了一个结果。用户进程判断结果是一个error时，它就知道数据还没有准备好，于是它可以再次发送read操作。一旦kernel中的数据准备好了，并且又再次收到了用户进程的system call，那么它马上就将数据拷贝到了用户内存，然后返回。不断询问的过程就是**轮询**。

nonblocking IO的特点是用户进程需要**不断的主动询问**kernel数据好了没有。

### 多路复用I/O

使用 select 或者 poll 等待数据，并且可以等待多个套接字中的任何一个变为可读。这一过程会被阻塞，当某一个套接字可读时返回，之后再使用 recvfrom 把数据从内核复制到进程中。

它可以让单个进程具有处理多个 I/O 事件的能力。又被称为 Event Driven I/O，即事件驱动 I/O。

如果一个 Web 服务器没有 I/O 复用，那么每一个 Socket 连接都需要创建一个线程去处理。如果同时有几万个连接，那么就需要创建相同数量的线程。相比于多进程和多线程技术，I/O 复用不需要进程线程创建和切换的开销，系统开销更小。

![20201106215008](https://gitee.com/p8t/picbed/raw/master/imgs/20201106215008.png)

### 信号驱动 I/O

应用进程使用 sigaction 系统调用，内核立即返回，应用进程可以继续执行，也就是说等待数据阶段应用进程是非阻塞的。内核在数据到达时向应用进程发送 SIGIO 信号，应用进程收到之后在信号处理程序中调用 recvfrom 将数据从内核复制到应用进程中。

相比于非阻塞式 I/O 的轮询方式，信号驱动 I/O 的 CPU 利用率更高。

![img](https://gitee.com/p8t/picbed/raw/master/imgs/20201106214636.png)

### 异步 I/O

应用进程执行 aio_read 系统调用会立即返回，应用进程可以继续执行，不会被阻塞，内核会在所有操作完成之后向应用进程发送信号。

异步 I/O 与信号驱动 I/O 的区别在于，异步 I/O 的信号是通知应用进程 I/O 完成，而信号驱动 I/O 的信号是通知应用进程可以开始 I/O。

![20201106214457](https://gitee.com/p8t/picbed/raw/master/imgs/20201106214457.png)

### 比较

- 同步 I/O：将数据从内核缓冲区复制到应用进程缓冲区的阶段（第二阶段），应用进程会阻塞。
- 异步 I/O：第二阶段应用进程不会阻塞。

同步 I/O 包括阻塞式 I/O、非阻塞式 I/O、多路复用I/O 和信号驱动 I/O ，它们的主要区别在第一个阶段。

非阻塞式 I/O 、信号驱动 I/O 和异步 I/O 在第一阶段不会阻塞。

![20201106215416](https://gitee.com/p8t/picbed/raw/master/imgs/20201106215416.png)

## 参考文章

[Linux IO模式及 select、poll、epoll详解](https://segmentfault.com/a/1190000003063859)

[CS-Notes](https://cyc2018.github.io/CS-Notes/#/notes/Socket)