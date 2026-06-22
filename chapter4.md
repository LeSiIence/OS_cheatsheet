第一页高亮区域是 **Learning Objectives（学习目标）**。结合后面 4.1–4.2，本次重点先复习前三个目标：**process vs thread（进程与线程区别）**、**thread design issues（线程设计问题）**、**ULT vs KLT（用户级线程与内核级线程区别）**。Windows / Solaris / Linux 的具体线程管理在后续章节会展开，本文件这里只先出现概念铺垫。

## 1. Process 和 Thread 的核心区别

英文原文核心意思：

**A process embodies two separate concepts: resource ownership and execution.**
进程其实包含两个可以分开的概念：**资源拥有** 和 **执行路径**。

### Process = resource ownership / 资源拥有单位

**Resource ownership（资源拥有）** 指进程拥有或管理一组资源，例如：

* **virtual address space**：虚拟地址空间
* **process image**：进程映像，包括 program、data、stack、PCB attributes
* **files / I/O devices / I/O channels**：文件、I/O 设备、I/O 通道
* **protection**：操作系统提供保护，防止进程之间互相干扰资源

中文理解：
**进程更像是“资源容器”。** 它负责装程序、数据、打开的文件、地址空间等资源。

### Thread = scheduling/execution / 调度与执行单位

**Scheduling/execution（调度/执行）** 指一条执行路径：

* **execution path / trace**：执行轨迹
* **execution state**：执行状态，如 Running、Ready、Blocked
* **dispatching priority**：调度优先级
* **scheduled and dispatched by the OS**：被操作系统调度和分派

中文理解：
**线程更像是“真正跑起来的执行流”。** CPU 调度时，调度的主要对象是线程，而不是整个资源容器。

一句话记忆：

> **Process owns resources; thread executes code.**
> **进程拥有资源，线程负责执行。**

---

## 2. Multithreading / 多线程

英文定义：

**Multithreading refers to the ability of an OS to support multiple, concurrent paths of execution within a single process.**
多线程是指操作系统支持在同一个进程内部存在多个并发执行路径。

### Single-threaded process / 单线程进程

传统模型中：

> **One process, one thread.**
> 一个进程只有一个线程。

这种情况下，进程既是资源单位，也是执行单位。

### Multithreaded process / 多线程进程

现代模型中：

> **One process, multiple threads.**
> 一个进程可以有多个线程。

同一个进程中的多个线程共享进程资源，但每个线程有自己的执行状态。

---

## 3. 进程中有什么？线程中有什么？

### Process has / 进程拥有

英文：

**A process is defined as the unit of resource allocation and a unit of protection.**
进程是资源分配单位和保护单位。

进程包含：

| English                                     | 中文                |
| ------------------------------------------- | ----------------- |
| virtual address space                       | 虚拟地址空间            |
| process image                               | 进程映像              |
| protected access to files and I/O resources | 对文件和 I/O 资源的受保护访问 |
| access to other processes via IPC           | 通过进程间通信访问其他进程     |

### Thread has / 线程拥有

线程包含：

| English                            | 中文           |
| ---------------------------------- | ------------ |
| thread execution state             | 线程执行状态       |
| saved thread context               | 保存的线程上下文     |
| program counter                    | 程序计数器        |
| execution stack                    | 执行栈          |
| per-thread static storage          | 每线程私有静态存储    |
| access to process memory/resources | 访问所属进程的内存和资源 |

重点区别：

> Threads in the same process share memory and resources.
> 同一进程中的线程共享内存和资源。

所以线程之间通信很快，但也容易互相影响。

---

## 4. PCB 和 TCB

图 4.2 的重点是：

### PCB = Process Control Block / 进程控制块

保存进程级信息，例如：

* 地址空间
* 资源
* 文件
* 进程级属性

### TCB = Thread Control Block / 线程控制块

保存线程级信息，例如：

* 寄存器值
* 程序计数器
* 栈指针
* 优先级
* 线程状态

中文理解：

单线程模型：

> 一个 PCB + 一个执行流。

多线程模型：

> 一个 PCB + 多个 TCB。

也就是：

> **Process has one address space; each thread has its own stack and context.**
> **进程共享一个地址空间；每个线程有自己的栈和上下文。**

---

## 5. Threads 的优点

教材列出线程的性能优势：

### 1. Faster to create / 创建更快

**It takes far less time to create a thread than a process.**
创建线程比创建进程快得多。

原因：线程共享已有进程的地址空间和资源，不需要重新创建完整进程环境。

### 2. Faster to terminate / 终止更快

销毁线程通常比销毁进程更轻量。

### 3. Faster to switch / 切换更快

同一进程内线程切换，比进程切换开销小。

因为不需要切换整个地址空间。

### 4. Faster communication / 通信更快

同一进程内线程共享内存和文件，所以线程通信通常不需要内核介入。

对比：

| 通信对象                    | 通信方式                  | 开销 |
| ----------------------- | --------------------- | -- |
| Processes               | IPC / message passing | 较大 |
| Threads in same process | shared memory         | 较小 |

考试常考一句话：

> Threads are cheaper than processes because they share the same process resources.
> 线程比进程轻量，因为线程共享所属进程的资源。

---

## 6. Threads 的典型应用

教材给了几个例子。

### File server / 文件服务器

每来一个文件请求，就创建一个线程处理。

好处：

* 多个请求可以并发处理
* 创建线程比创建进程便宜
* 共享文件数据更方便

### Foreground and background work / 前台与后台工作

例如电子表格程序：

* 一个线程负责显示菜单和读取用户输入
* 另一个线程负责执行命令和更新表格

用户会感觉程序响应更快。

### Asynchronous processing / 异步处理

例如文字处理器每分钟自动保存：

* 主线程负责用户编辑
* 后台线程定期保存 RAM buffer 到磁盘

### Speed of execution / 提高执行速度

一个线程读下一批数据，另一个线程处理当前数据。

如果在多处理器系统上，多个线程还可以真正并行执行。

### Modular program structure / 模块化程序结构

程序中有多种活动时，用线程可以让结构更清晰。

---

## 7. Thread states / 线程状态

线程也有类似进程的状态：

| English | 中文  |
| ------- | --- |
| Running | 运行态 |
| Ready   | 就绪态 |
| Blocked | 阻塞态 |

注意：

> Suspend states are usually process-level concepts.
> 挂起状态通常是进程级概念。

因为线程共享进程地址空间，如果进程被换出内存，所有线程都会一起被换出。

---

## 8. Thread operations / 线程基本操作

教材列出四种基本线程操作：

### Spawn / 创建线程

一个线程可以创建另一个线程。

新线程会获得：

* instruction pointer / 指令指针
* arguments / 参数
* register context / 寄存器上下文
* stack space / 栈空间

然后进入 Ready queue / 就绪队列。

### Block / 阻塞线程

当线程等待某个事件，例如 I/O 完成时，会进入 Blocked 状态。

### Unblock / 解除阻塞

等待事件发生后，线程从 Blocked 进入 Ready。

### Finish / 结束线程

线程完成后，释放寄存器上下文和栈。

---

## 9. 线程阻塞是否导致整个进程阻塞？

这是本章非常重要的问题。

英文问题：

**If one thread in a process is blocked, does this prevent the running of any other thread in the same process?**
如果进程中的一个线程阻塞了，会不会导致同一进程中的其他线程也不能运行？

答案取决于线程实现方式：

| 线程类型                         | 一个线程阻塞时    |
| ---------------------------- | ---------- |
| User-Level Threads / 用户级线程   | 可能导致整个进程阻塞 |
| Kernel-Level Threads / 内核级线程 | 通常不会阻塞整个进程 |

这就是 4.2 要讲 ULT 和 KLT 的原因。

---

## 10. Thread synchronization / 线程同步

因为同一进程内线程共享地址空间和资源，所以必须同步。

英文核心：

**It is necessary to synchronize the activities of the various threads so that they do not interfere with each other or corrupt data structures.**
必须同步多个线程的活动，防止它们互相干扰或破坏数据结构。

例子：

两个线程同时向 doubly linked list / 双向链表插入元素，可能导致：

* 一个元素丢失
* 链表结构损坏
* 数据不一致

这就是后面 Chapter 5 / Chapter 6 会讲的同步问题，比如：

* mutual exclusion / 互斥
* semaphore / 信号量
* mutex / 互斥锁
* deadlock / 死锁

---

# 4.2 Types of Threads / 线程类型

## 11. User-Level Threads, ULT / 用户级线程

英文：

**All of the work of thread management is done by the application and the kernel is not aware of the existence of threads.**
所有线程管理工作由应用程序完成，内核不知道线程的存在。

也就是说：

> Kernel sees only the process, not the threads.
> 内核只看到进程，看不到进程内部的线程。

线程管理由 **threads library / 线程库** 完成，包括：

* creating threads / 创建线程
* destroying threads / 销毁线程
* scheduling threads / 调度线程
* saving and restoring contexts / 保存和恢复上下文
* passing messages and data / 在线程间传递数据

### ULT 的优点

#### 1. No kernel mode switch / 不需要内核态切换

线程切换在用户态完成，不需要进入内核。

所以开销小。

#### 2. Application-specific scheduling / 应用可自定义调度

不同应用可以选择不同线程调度策略。

例如：

* round-robin / 轮转调度
* priority-based scheduling / 优先级调度

#### 3. Portable / 可移植

ULT 不需要修改内核，只要有线程库即可。

所以可以运行在不支持内核线程的操作系统上。

### ULT 的缺点

#### 1. Blocking system call blocks all threads / 阻塞系统调用会阻塞整个进程

因为内核不知道线程存在。

如果某个用户级线程调用阻塞 I/O，内核会认为整个进程阻塞。

结果：

> One ULT blocks, all ULTs in the process may block.
> 一个用户级线程阻塞，整个进程内的用户级线程都可能阻塞。

#### 2. Cannot exploit multiprocessing / 不能充分利用多处理器

纯 ULT 中，内核只把整个进程调度到一个处理器上。

所以同一进程内多个 ULT 不能真正同时在多个 CPU 上运行。

### Jacketing / 包装技术

教材提到一种补救方法：**jacketing**。

作用：

> Convert a blocking system call into a nonblocking system call.
> 把阻塞系统调用包装成非阻塞调用。

例如线程不直接调用系统 I/O，而是调用一个用户级 I/O jacket routine。这个函数先检查设备是否忙，如果忙，就让当前线程在用户级阻塞，并切换到其他线程。

---

## 12. Kernel-Level Threads, KLT / 内核级线程

英文：

**All of the work of thread management is done by the kernel.**
所有线程管理工作由内核完成。

也就是说：

> Kernel knows and schedules each thread.
> 内核知道每个线程，并且按线程调度。

Windows 是教材中给出的典型 KLT 例子。

### KLT 的优点

#### 1. Can use multiprocessing / 可以利用多处理器

内核可以把同一进程中的多个线程同时调度到多个处理器上执行。

#### 2. One blocked thread does not block the whole process / 一个线程阻塞不影响整个进程

如果某个线程阻塞，内核可以调度同一进程中的另一个线程运行。

#### 3. Kernel routines can be multithreaded / 内核例程也可以多线程化

内核自身的一些操作也可以用多线程方式实现。

### KLT 的缺点

#### Mode switch overhead / 模式切换开销

同一进程内两个线程切换，也需要进入内核态。

所以 KLT 比 ULT 更重。

一句话记忆：

> ULT is faster but less powerful; KLT is heavier but more flexible.
> 用户级线程更快但能力弱；内核级线程开销大但功能强。

---

## 13. ULT vs KLT 对比表

| 对比点      | ULT / User-Level Thread 用户级线程 | KLT / Kernel-Level Thread 内核级线程 |
| -------- | ----------------------------- | ------------------------------- |
| 管理者      | Application / threads library | Kernel                          |
| 内核是否知道线程 | No                            | Yes                             |
| 线程切换开销   | Low                           | Higher                          |
| 是否需要模式切换 | Usually no                    | Yes                             |
| 阻塞系统调用影响 | 可能阻塞整个进程                      | 只阻塞当前线程                         |
| 多处理器并行   | 纯 ULT 不行                      | 可以                              |
| 调度策略     | 应用可自定义                        | 内核统一调度                          |
| 可移植性     | 强                             | 依赖 OS 支持                        |

---

## 14. Combined ULT/KLT / 组合式线程

英文：

**The multiple ULTs from a single application are mapped onto some number of KLTs.**
一个应用中的多个用户级线程会映射到若干个内核级线程上。

这种方式想结合两者优点：

* 线程创建和大部分调度在用户态完成，速度快
* 内核仍然能看到部分线程，支持多处理器并行
* 一个阻塞系统调用不一定阻塞整个进程

中文理解：

> Combined approach = ULT speed + KLT parallelism.
> 组合式方法 = 用户级线程的速度 + 内核级线程的并行能力。

Solaris 是教材给出的例子。

---

## 15. Threads 和 Processes 的关系

教材后面还列出几种关系：

### 1:1

**Each thread of execution is a unique process.**
每个执行线程都是一个独立进程。

传统 UNIX 接近这种模型。

### M:1

**Multiple threads within one process.**
一个进程中有多个线程。

这是我们最常见的多线程模型。

### 1:M

**A thread may migrate from one process environment to another.**
一个线程可以从一个进程环境迁移到另一个进程环境。

这种更偏分布式系统或实验系统。

### M:N

多个线程与多个进程之间多对多映射。

这种模型结合 M:1 和 1:M 的特点。

---

# 考试速记版

最重要的三句话：

> **A process is a unit of resource ownership.**
> **进程是资源拥有单位。**

> **A thread is a unit of scheduling/execution.**
> **线程是调度和执行单位。**

> **Threads in the same process share the same address space and resources.**
> **同一进程中的线程共享地址空间和资源。**

最容易考的简答题：

### Q1: What is the difference between a process and a thread?

答：

A process is the unit of resource allocation and protection, while a thread is the unit of execution and scheduling. Threads in the same process share the process’s address space and resources, but each thread has its own execution state, context, and stack.

中文：

进程是资源分配和保护单位，线程是执行和调度单位。同一进程中的线程共享进程地址空间和资源，但每个线程有自己的执行状态、上下文和栈。

### Q2: Why are threads cheaper than processes?

答：

Because threads share the same address space and resources of a process. Creating, terminating, and switching threads usually requires less overhead than doing the same operations for processes.

中文：

因为线程共享同一进程的地址空间和资源，所以线程创建、终止和切换通常比进程开销更小。

### Q3: Compare ULT and KLT.

答：

ULTs are managed by a user-level thread library, and the kernel is unaware of them. They are fast and portable, but a blocking system call may block the whole process, and pure ULTs cannot exploit multiprocessing. KLTs are managed by the kernel, so they can run in parallel on multiple processors and one blocked thread does not block the whole process, but thread switching has higher overhead.

中文：

用户级线程由用户态线程库管理，内核不知道它们的存在。它们速度快、可移植，但阻塞系统调用可能阻塞整个进程，而且纯 ULT 不能利用多处理器。内核级线程由内核管理，可以在多个处理器上并行执行，一个线程阻塞不会阻塞整个进程，但线程切换开销更大。



已找到与本次 **4.1 Processes and Threads**、**4.2 Types of Threads** 直接相关的题目：**Review Questions 4.1–4.8**，以及 **Problems 4.1–4.9**。
**Problems 4.10–4.12** 主要考 Solaris / Linux 的具体线程实现，属于后面内容，暂时不作为本次 4.1–4.2 的重点。题目来自 Chapter 4 question sheet，概念依据 4.1–4.2 课文。 

# Review Questions

## RQ 4.1

**Question:** In a multithreaded system, which PCB elements should move to TCB?
**问题：** 多线程系统中，原来 PCB 里的哪些内容应该放到 TCB，哪些仍留在 PCB？

**Answer / 答案：**

| Belong to TCB / 属于线程控制块                     | Belong to PCB / 属于进程控制块         |
| ------------------------------------------- | ------------------------------- |
| thread state / 线程状态，如 Running、Ready、Blocked | process ID / 进程标识符              |
| program counter / 程序计数器                     | virtual address space / 虚拟地址空间  |
| registers / 寄存器                             | memory management info / 内存管理信息 |
| stack pointer / 栈指针                         | open files / 打开的文件              |
| thread priority / 线程优先级                     | I/O resources / I/O 资源          |
| scheduling information for thread / 线程调度信息  | process privileges / 进程权限       |
| thread-specific data / 线程私有数据               | IPC information / 进程间通信信息       |

一句话：

> **TCB stores execution information; PCB stores resource ownership information.**
> **TCB 保存执行相关信息；PCB 保存资源拥有相关信息。**

---

## RQ 4.2

**Question:** Why is a thread switch cheaper than a process switch?
**问题：** 为什么线程切换比进程切换便宜？

**Answer / 答案：**

A switch between threads in the same process is cheaper because the threads share the same address space and resources. The OS usually only needs to save and restore the thread context, such as registers, program counter, and stack pointer.

同一进程内线程切换更便宜，因为它们共享同一个地址空间和资源。操作系统主要只需要保存和恢复线程上下文，例如寄存器、程序计数器和栈指针。

进程切换通常还要切换：

* address space / 地址空间
* page table / 页表
* memory mapping / 内存映射
* resource ownership information / 资源拥有信息

---

## RQ 4.3

**Question:** What are the two independent characteristics embodied in process?
**问题：** 进程概念中包含哪两个可以分离的特征？

**Answer / 答案：**

1. **Resource ownership / 资源拥有**
   A process owns an address space and resources.

2. **Scheduling and execution / 调度与执行**
   A process follows an execution path and has execution states.

现代操作系统把这两个概念分开：

> **Process = resource ownership.**
> **Thread = scheduling/execution.**
> **进程 = 资源拥有单位；线程 = 调度执行单位。**

---

## RQ 4.4

**Question:** Give four examples of thread use in a single-user multiprocessing system.
**问题：** 给出单用户多处理系统中线程使用的四个例子。

**Answer / 答案：**

1. **Foreground and background work / 前台与后台工作**
   例如一个线程处理用户输入，另一个线程后台计算。

2. **Asynchronous processing / 异步处理**
   例如文字处理器后台自动保存文件。

3. **Speed of execution / 提高执行速度**
   一个线程读数据，另一个线程处理数据。

4. **Modular program structure / 模块化程序结构**
   不同线程负责不同功能，使程序结构更清晰。

---

## RQ 4.5

**Question:** What resources are shared by all threads of a process?
**问题：** 同一进程中的所有线程共享哪些资源？

**Answer / 答案：**

Threads in the same process usually share:

* **address space / 地址空间**
* **program code / 程序代码**
* **global data / 全局数据**
* **heap / 堆**
* **open files / 打开的文件**
* **I/O resources / I/O 资源**
* **process privileges / 进程权限**

但是每个线程有自己的：

* **thread state / 线程状态**
* **register context / 寄存器上下文**
* **program counter / 程序计数器**
* **stack / 栈**

---

## RQ 4.6

**Question:** List three advantages of ULTs over KLTs.
**问题：** 用户级线程 ULT 相比内核级线程 KLT 有哪三个优点？

**Answer / 答案：**

1. **No kernel mode switch / 不需要内核态切换**
   线程切换在用户态完成，速度快。

2. **Application-specific scheduling / 应用可自定义调度**
   应用程序可以自己决定线程调度策略。

3. **Portability / 可移植性好**
   不需要操作系统内核支持，只要有线程库即可。

---

## RQ 4.7

**Question:** List two disadvantages of ULTs compared to KLTs.
**问题：** ULT 相比 KLT 有哪两个缺点？

**Answer / 答案：**

1. **Blocking system call problem / 阻塞系统调用问题**
   一个 ULT 调用阻塞系统调用时，可能导致整个进程阻塞。

2. **Cannot exploit multiprocessing / 不能充分利用多处理器**
   纯 ULT 中，内核只看到一个进程，看不到其中多个线程，所以不能把同一进程内多个线程分配到多个 CPU 上真正并行运行。

---

## RQ 4.8

**Question:** Define jacketing.
**问题：** 什么是 jacketing？

**Answer / 答案：**

**Jacketing** means wrapping a blocking system call with a user-level routine to make it behave like a nonblocking call.

**Jacketing / 包装技术** 是指用用户级包装函数把阻塞系统调用包装成非阻塞形式。

例如线程不直接调用阻塞 I/O，而是先调用一个 **I/O jacket routine**。如果设备忙，就让当前用户级线程阻塞，并切换到同一进程中的其他线程。

---

# Problems

## Problem 4.1

**Question:** Is switching between two threads in the same process cheaper than switching between two threads in different processes?
**问题：** 同一进程内两个线程之间切换，是否比不同进程中的两个线程之间切换更省开销？

**Answer / 答案：**

Yes.

是的。

Switching between two threads in the same process usually requires only a thread context switch. The address space and resources remain the same.

同一进程内线程切换通常只需要切换线程上下文，地址空间和资源不用换。

但是如果两个线程属于不同进程，就不只是线程切换，还涉及进程级上下文切换，例如页表、地址空间、内存映射和资源保护信息等。

所以：

> **Same-process thread switch < different-process thread switch < full process switch.**
> **同进程线程切换开销最小，不同进程线程切换开销更大。**

---

## Problem 4.2

**Question:** Why does a blocking system call by one ULT block all threads in the process?
**问题：** 为什么一个 ULT 执行阻塞系统调用，会导致整个进程内所有线程阻塞？

**Answer / 答案：**

Because the kernel is not aware of user-level threads.

因为内核不知道用户级线程的存在。

In a pure ULT system, the kernel only sees the process. When one ULT makes a blocking system call, the kernel blocks the entire process, not just that one thread.

在纯 ULT 系统中，内核只看到进程。某个用户级线程调用阻塞系统调用时，内核会把整个进程置为 Blocked，而不是只阻塞那个线程。

---

## Problem 4.3

**Question:** OS/2 has sessions, processes, and threads. If sessions are removed and UI is associated directly with processes:
a. What benefits are lost?
b. Where should resources be assigned?

**问题：** OS/2 中有 session、process、thread。如果去掉 session，把用户界面直接和进程绑定：
a. 会失去什么好处？
b. 资源应该分配给 process 还是 thread？

### a. Lost benefits / 失去的好处

You lose the ability to group multiple processes under one user interface session.

会失去把多个进程组织到同一个用户界面会话中的能力。

原来的 **session** 可以表示一个完整的交互式应用，例如一个文字处理器或电子表格。这个应用可能包含多个进程，但共享同一个键盘、鼠标、窗口和屏幕上下文。

如果去掉 session，则每个进程都要单独管理前台/后台状态，复杂应用的结构会变差。

### b. Resource assignment / 资源分配

Resources should still be assigned at the **process level**.

资源仍然应该分配给 **process / 进程**。

原因：

> **Process is the unit of resource ownership; thread is the unit of execution.**
> **进程是资源拥有单位，线程是执行单位。**

所以 memory、files、I/O resources 应该归进程管理；线程只保留自己的执行上下文和栈。

---

## Problem 4.4

**Question:** Why can one-to-one ULT/KLT mapping make multithreaded programs faster on a uniprocessor?
**问题：** 在单处理器上，为什么一对一 ULT/KLT 映射仍能让多线程程序比单线程程序快？

**Answer / 答案：**

Even on a uniprocessor, only one thread can run at a time, but multithreading can overlap waiting time with useful work.

即使在单处理器上，同一时刻也只能运行一个线程，但多线程可以把等待时间和有用工作重叠起来。

For example, if one thread is blocked on I/O, the kernel can run another ready thread in the same process.

例如一个线程等待 I/O 时，内核可以调度同一进程中的另一个就绪线程运行。

所以性能提升来自：

* better CPU utilization / 更高 CPU 利用率
* overlapping I/O and computation / I/O 与计算重叠
* better responsiveness / 响应性更好

不是因为真正并行，而是因为并发切换更有效。

---

## Problem 4.5

**Question:** If a process exits while its threads are still running, will they continue to run?
**问题：** 如果进程退出，但它的线程还在运行，这些线程会继续运行吗？

**Answer / 答案：**

No.

不会。

When a process terminates, all threads within that process are terminated.

进程终止时，该进程中的所有线程都会被终止。

原因是线程依赖进程的地址空间和资源。进程退出后，地址空间、文件、内存等资源被回收，线程没有独立存在的环境。

---

## Problem 4.6

**Question:** In OS/390, what is the advantage of splitting control information into global ASCB and local TCB?
**问题：** OS/390 中，把控制信息拆成全局 ASCB 和局部 TCB 有什么好处？

**Answer / 答案：**

The advantage is better separation between address-space management and task execution management.

好处是把地址空间管理和任务执行管理分开。

**ASCB / Address Space Control Block** 是全局结构，保存地址空间级别的信息，例如：

* memory allocation / 内存分配
* dispatching priority / 调度优先级
* whether swapped out / 是否被换出
* number of ready tasks / 就绪任务数量

**TCB / Task Control Block** 是局部结构，保存具体任务执行信息，例如：

* processor state / 处理器状态
* task state / 任务状态
* pointers to programs / 程序指针

这样设计的好处：

* OS 可以在全局层面管理地址空间
* 任务级信息可以留在对应地址空间内
* 减少系统全局结构的负担
* 有利于 swapping / 换入换出管理

---

## Problem 4.7

**Question:** What does `count_positives` do? Does two-thread execution cause problems?
**问题：** `count_positives` 做什么？两个线程并行执行会不会有问题？

代码核心：

```c
if (p -> val > 0.0)
    ++global_positives;
```

### a. What does the function do? / 函数作用

It counts the number of positive values in a linked list and adds that count to the global variable `global_positives`.

它遍历链表，把其中大于 0 的元素数量累加到全局变量 `global_positives` 中。

### b. Problem? / 是否有问题

In the specific case given, thread A scans a list containing only negative values, so intuitively it should not modify `global_positives`. Thread B increments `global_positives` once.

在题目给定的特殊情况下，A 线程遍历的链表全是负数，所以按直观执行，A 不会修改 `global_positives`；B 线程只加一次。

但是从多线程角度看，`global_positives` 是共享变量。如果多个线程同时读写它，而且没有 lock / mutex / atomic，就会有 **data race / 数据竞争** 风险。

---

## Problem 4.8

**Question:** What problem occurs with the optimized version?
**问题：** 编译器优化后的版本会出现什么问题？

优化后代码大意：

```c
r = global_positives;
...
global_positives = r;
```

**Answer / 答案：**

The optimized version may overwrite another thread’s update.

优化后的版本可能覆盖另一个线程的更新。

例如：

1. Thread A reads `global_positives = 0` into local register `r`.
2. Thread B executes `++global_positives`, so the value becomes 1.
3. Thread A finishes scanning the negative list and writes `r` back.
4. `global_positives` becomes 0 again.

所以 B 的更新被 A 覆盖了。

这叫：

> **lost update / 丢失更新**
> **data race / 数据竞争**

这说明普通 C/C++ 代码如果没有同步机制，编译器优化可能破坏多线程程序的正确性。

---

## Problem 4.9

**Question:** What does the Pthreads program do? Why is the output `myglobal equals 21`?
**问题：** 这个 Pthreads 程序做了什么？为什么输出是 `myglobal equals 21`？

### a. What does it accomplish? / 程序作用

The program creates two threads:

程序创建了两个线程：

1. **main thread / 主线程**
   每秒执行一次 `myglobal = myglobal + 1`，执行 20 次，并打印 `o`。

2. **new thread / 新线程**
   每秒读取 `myglobal`，加 1，再写回，也执行 20 次，并打印 `.`。

理想情况下，如果每次加法都正确同步，最终应该是：

```text
myglobal equals 40
```

因为两个线程各加 20 次。

### b. What went wrong? / 问题在哪里

The output `myglobal equals 21` is caused by a race condition.

输出 21 是因为发生了 **race condition / 竞态条件**。

关键原因是：

```c
myglobal = myglobal + 1;
```

不是原子操作。

它实际上包含三个步骤：

1. read / 读取 `myglobal`
2. add / 加 1
3. write / 写回 `myglobal`

两个线程可能交错执行，导致其中一个线程的写回覆盖另一个线程的写回。

例如：

```text
Thread A reads myglobal = 5
Thread B reads myglobal = 5
Thread A writes 6
Thread B writes 6
```

两个线程都加了一次，但结果只增加了 1。

所以最后不是 40，而可能是 21、22、30 等不确定值。

解决方法：

```c
pthread_mutex_lock(&lock);
myglobal++;
pthread_mutex_unlock(&lock);
```

或者使用 atomic variable / 原子变量。

---

# 本次最该背的题

优先背这些：

1. **RQ 4.3**：process 的两个特征
   **resource ownership** 和 **scheduling/execution**

2. **RQ 4.5**：线程共享哪些资源
   共享 address space、files、I/O resources；私有 stack、registers、PC。

3. **RQ 4.6 / 4.7**：ULT 优缺点
   ULT 快、可移植、可自定义调度；但阻塞系统调用会阻塞整个进程，不能利用多处理器。

4. **Problem 4.2**：为什么 ULT 阻塞会阻塞整个进程
   因为 kernel 不知道 ULT，只看到 process。

5. **Problem 4.9**：多线程共享变量为什么出错
   因为 `++` 不是 atomic，会发生 race condition / lost update。
