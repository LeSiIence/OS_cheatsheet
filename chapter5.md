下面按 **英文原文术语 + 中文解释** 复习 **5.1 Principles of Concurrency** 和 **5.3 Semaphores**，对应你上传的 5.1–5.3 PDF 内容。

## 5.1 Principles of Concurrency 并发原理

### 1. Concurrency 并发是什么

**Concurrency** 指多个进程或线程在同一段时间内推进执行。它不一定是真正同时运行。

在 **single-processor multiprogramming system 单处理器多道程序系统** 中，多个进程是 **interleaved in time 时间交错执行** 的，看起来像同时运行。

在 **multiple-processor system 多处理器系统** 中，多个进程可以真正 **overlap 重叠执行 / parallel 并行执行**。

考试要点是：
**并发问题在单处理器和多处理器都会出现。**
单处理器中，因为中断和调度导致执行顺序不可预测；多处理器中，因为进程可能真正同时访问共享资源。

---

### 2. 并发带来的三个主要困难

教材强调，进程执行的相对速度不可预测，所以会产生问题：

**Shared global resources are dangerous.**
共享全局资源很危险。比如两个进程同时读写同一个全局变量，最终结果取决于执行顺序。

**Resource allocation is difficult.**
资源分配更复杂。比如一个进程拿到了 I/O 设备但还没用就被挂起，可能导致其他进程等待，甚至形成死锁。

**Programming errors are hard to reproduce.**
并发 bug 很难定位，因为结果通常是 **nondeterministic 非确定性** 的，同一程序多次运行结果可能不同。

---

### 3. A Simple Example：echo 例子

代码大概是：

```c
void echo()
{
    chin = getchar();
    chout = chin;
    putchar(chout);
}
```

这里 `chin` 和 `chout` 是共享变量。假设：

1. P1 执行 `chin = getchar()`，输入了字符 `x`，然后被中断。
2. P2 执行完整个 `echo()`，输入并输出字符 `y`。
3. P1 恢复执行，但 `chin` 已经被 P2 改成了 `y`。

结果就是：

`x` 丢失，`y` 被输出两次。

这个例子说明：

**The only way to protect shared global variables is to control the code that accesses them.**
保护共享变量的关键不是只盯着变量本身，而是控制访问变量的那段代码。

这段需要保护的代码就是：

**Critical Section 临界区**。

---

### 4. Race Condition 竞态条件

**Race condition**：多个进程或线程读写共享数据，最终结果取决于执行顺序。

中文可以理解为：
几个进程在“抢着”修改共享变量，谁先谁后会影响最终结果。

例子 1：

```c
P1: a = 1;
P2: a = 2;
```

最终 `a` 是 1 还是 2，取决于谁最后写。

例子 2：

初始：

```c
b = 1;
c = 2;
```

两个进程：

```c
P3: b = b + c;
P4: c = b + c;
```

如果 P3 先执行，最后可能是：

```c
b = 3, c = 5
```

如果 P4 先执行，最后可能是：

```c
b = 4, c = 3
```

所以竞态条件的核心是：

**The final result depends on timing.**
最终结果依赖时序。

---

### 5. OS Concerns 操作系统要处理什么

并发存在时，操作系统必须处理四类问题：

**Keep track of processes**
跟踪进程状态，依靠 PCB / Process Control Block。

**Allocate and deallocate resources**
分配和回收资源，包括 CPU 时间、内存、文件、I/O 设备。

**Protect data and resources**
保护每个进程的数据和物理资源，防止被其他进程意外干扰。

**Ensure speed independence**
保证一个进程的结果不依赖于它和其他进程的相对执行速度。

最后这一点就是本章重点：
如何让并发进程正确同步。

---

### 6. Process Interaction 进程交互类型

教材把进程之间的关系分为三类。

#### 1. Processes unaware of each other 彼此不知道对方存在

关系是：

**Competition 竞争**

例如两个独立程序都想用打印机、磁盘、文件。

可能问题：

**Mutual exclusion 互斥**
**Deadlock 死锁**
**Starvation 饥饿**

---

#### 2. Processes indirectly aware of each other 间接知道对方

关系是：

**Cooperation by sharing 通过共享合作**

例如多个进程共享缓冲区、共享变量、共享数据库。

可能问题：

**Mutual exclusion 互斥**
**Deadlock 死锁**
**Starvation 饥饿**
**Data coherence 数据一致性**

这里比单纯竞争多了一个重点：
共享数据不仅要防止同时写，还要保证数据之间的逻辑关系不被破坏。

例如本来要求：

```c
a = b
```

P1：

```c
a = a + 1;
b = b + 1;
```

P2：

```c
b = 2 * b;
a = 2 * a;
```

如果两个进程交错执行，即使每次只改一个变量，也可能破坏 `a = b`。所以有时必须把一整组操作都放进临界区，而不是只保护单个变量。

---

#### 3. Processes directly aware of each other 直接知道对方

关系是：

**Cooperation by communication 通过通信合作**

例如进程之间用消息传递：

**message passing 消息传递**

因为没有共享变量，所以一般不需要 mutual exclusion。
但是仍然可能有：

**Deadlock 死锁**：两个进程互相等对方发消息。
**Starvation 饥饿**：某个进程一直等不到通信机会。

---

### 7. Mutual Exclusion 互斥

**Mutual exclusion** 的意思是：

当一个进程正在访问某个共享资源的临界区时，其他进程不能进入访问同一资源的临界区。

典型结构：

```c
while (true) {
    entercritical(Ra);
    /* critical section */
    exitcritical(Ra);
    /* remainder */
}
```

其中：

**critical section 临界区**：访问共享资源的代码。
**critical resource 临界资源**：一次只能被一个进程安全使用的资源。
**remainder section 剩余区**：不访问共享资源的普通代码。

---

### 8. Requirements for Mutual Exclusion 互斥必须满足的要求

这个是考试很容易考简答题的地方。

1. **Only one process at a time is allowed into its critical section.**
   同一共享资源对应的临界区，同一时刻最多只能有一个进程进入。

2. **A process halted in its noncritical section must not interfere with others.**
   一个进程停在非临界区，不能影响其他进程进入临界区。

3. **No deadlock or starvation.**
   想进入临界区的进程不能无限期等待，不能死锁，不能饥饿。

4. **If no process is in the critical section, entry should not be delayed.**
   如果临界区空着，申请进入的进程应该可以立即进入。

5. **No assumptions about process speeds or number of processors.**
   不能假设进程速度，也不能假设处理器数量。

6. **A process stays in its critical section for finite time only.**
   进程只能在临界区待有限时间，不能一直占着不走。

可以背成：

**互斥、非临界不干扰、无死锁饥饿、空闲立即进、不假设速度处理器、临界区有限时间。**

---

## 5.3 Semaphores 信号量

### 1. Semaphore 信号量是什么

**Semaphore** 是用于进程同步的一个整数变量，但它不能随便读写，只能通过原子操作访问。

教材里说它只有三种操作：

1. **initialize 初始化**
2. **semWait / P / down 减一并可能阻塞**
3. **semSignal / V / up 加一并可能唤醒**

Dijkstra 原始记号中：

**P operation = semWait**
**V operation = semSignal**

中文常说：

P 操作：申请资源 / 等待 / down
V 操作：释放资源 / 发信号 / up

---

### 2. semWait 和 semSignal 的规则

对于一般信号量：

```c
semWait(s):
    s.count--;
    if (s.count < 0)
        block this process;
```

```c
semSignal(s):
    s.count++;
    if (s.count <= 0)
        unblock one blocked process;
```

核心解释：

如果 `s.count > 0`，表示还有资源可用，进程执行 `semWait` 后可以继续。

如果 `s.count == 0`，表示资源刚好没有了，下一个执行 `semWait` 的进程会被阻塞。

如果 `s.count < 0`，它的绝对值表示正在等待的进程数。

例如：

```c
s.count = -3
```

表示有 3 个进程阻塞在这个信号量队列上。

---

### 3. Semaphore 的三个重要理解

教材提到几个容易混淆的点：

**Before semWait, you cannot know whether it will block.**
执行 semWait 之前，进程一般不知道自己会不会被阻塞。

**After semSignal, both processes may continue concurrently.**
一个进程 signal 之后，被唤醒的进程和原进程都可能继续运行，谁先运行由调度决定。

**semSignal may wake zero or one process.**
signal 不一定真的唤醒进程。如果没有人在等，它只是把信号量加一。

---

### 4. Binary Semaphore 二元信号量

**Binary semaphore** 只能取 0 或 1。

规则是：

```c
semWaitB(s):
    if s == 1:
        s = 0
        continue
    else:
        block
```

```c
semSignalB(s):
    if queue is empty:
        s = 1
    else:
        unblock one process
```

二元信号量很适合做互斥锁，因为它表示：

```text
1 = unlocked / available
0 = locked / unavailable
```

---

### 5. Mutex 和 Binary Semaphore 的区别

**Mutex 互斥锁** 和 **binary semaphore 二元信号量** 很像，但有一个关键区别：

**Mutex must be unlocked by the same process that locked it.**
谁加锁，必须谁解锁。

**Binary semaphore can be waited by one process and signaled by another.**
二元信号量可以由一个进程 wait，另一个进程 signal。

考试中如果问区别，答这个最关键。

---

### 6. Strong Semaphore 和 Weak Semaphore

**Strong semaphore 强信号量**：等待队列按 FIFO 释放，先等的先被唤醒。
**Weak semaphore 弱信号量**：不规定等待进程的释放顺序。

区别：

强信号量更公平，可以避免某些饥饿问题。
弱信号量可能导致某个进程一直等不到。

---

## 5.3.1 Mutual Exclusion Using Semaphores 用信号量实现互斥

最标准模板：

```c
semaphore s = 1;

void P(int i)
{
    while (true) {
        semWait(s);
        /* critical section */
        semSignal(s);
        /* remainder */
    }
}
```

解释：

初始 `s = 1`，表示临界区可进入。

第一个进程执行：

```c
semWait(s)
```

后，`s` 从 1 变成 0，进入临界区。

其他进程再执行 `semWait(s)`，会让 `s` 变成负数并阻塞。

离开临界区时执行：

```c
semSignal(s)
```

如果有阻塞进程，就唤醒一个。

---

### 信号量初值的含义

如果：

```c
semaphore s = 1;
```

表示最多 1 个进程进入临界区，即互斥。

如果：

```c
semaphore s = k;
```

表示最多 k 个进程同时进入对应区域，相当于有 k 个同类资源。

所以：

**counting semaphore 计数信号量** 可以表示资源数量。

---

## 5.3.2 Producer / Consumer Problem 生产者消费者问题

### 1. 问题描述

**Producer 生产者**：生成数据，放入 buffer。
**Consumer 消费者**：从 buffer 中取数据并消费。

必须满足：

1. 生产者不能往满缓冲区里放数据。
   **Producer must not insert into a full buffer.**

2. 消费者不能从空缓冲区里取数据。
   **Consumer must not remove from an empty buffer.**

3. 同一时刻只能有一个进程访问 buffer。
   **Buffer operations must be mutually exclusive.**

---

### 2. Infinite Buffer 无限缓冲区

无限缓冲区不用担心满，只担心空。

常用两个信号量：

```c
semaphore n = 0;  // number of items
semaphore s = 1;  // mutual exclusion for buffer
```

生产者：

```c
while (true) {
    produce();
    semWait(s);
    append();
    semSignal(s);
    semSignal(n);
}
```

消费者：

```c
while (true) {
    semWait(n);
    semWait(s);
    take();
    semSignal(s);
    consume();
}
```

解释：

`n` 表示 buffer 中已有 item 数量。
消费者先 `semWait(n)`，如果没有 item，就阻塞。
`s` 是互斥锁，保护 `append()` 和 `take()`。

---

### 3. 为什么消费者必须先 wait(n)，再 wait(s)

正确：

```c
semWait(n);
semWait(s);
take();
semSignal(s);
```

如果写反：

```c
semWait(s);
semWait(n);
take();
semSignal(s);
```

可能死锁。

原因：

消费者先拿到了互斥锁 `s`，然后发现 `n = 0`，于是等待产品。
但生产者想生产后放入 buffer，也需要先拿 `s`。
结果消费者拿着 `s` 等产品，生产者等 `s` 才能放产品，双方卡死。

这就是典型考试陷阱。

---

### 4. Bounded Buffer 有界缓冲区

有界缓冲区既要防止空，也要防止满。

需要三个信号量：

```c
semaphore s = 1;             // mutual exclusion
semaphore n = 0;             // number of filled slots
semaphore e = sizeofbuffer;  // number of empty slots
```

其中：

`s`：保护 buffer 的互斥访问。
`n`：已有产品数量，消费者等它。
`e`：空槽数量，生产者等它。

生产者：

```c
while (true) {
    produce();
    semWait(e);
    semWait(s);
    append();
    semSignal(s);
    semSignal(n);
}
```

消费者：

```c
while (true) {
    semWait(n);
    semWait(s);
    take();
    semSignal(s);
    semSignal(e);
    consume();
}
```

记忆方法：

生产者：

```text
先等空位 e，再锁 buffer s，放入，解锁 s，增加产品 n
```

消费者：

```text
先等产品 n，再锁 buffer s，取出，解锁 s，增加空位 e
```

---

## 5.3.3 Implementation of Semaphores 信号量实现

信号量最关键的要求是：

**semWait and semSignal must be atomic.**
semWait 和 semSignal 必须是原子操作。

也就是说，执行过程中不能被中断，不能有两个进程同时修改同一个信号量。

实现方式可以有：

**Hardware / firmware 硬件或固件实现**
直接保证原子性。

**Software algorithms 软件算法**
比如 Dekker’s algorithm、Peterson’s algorithm，但开销较大。

**Hardware-supported mutual exclusion 硬件支持互斥**
比如 compare&swap。虽然会有 busy waiting，但 semWait 和 semSignal 很短，所以开销相对可接受。

**Disable interrupts 禁用中断**
在单处理器系统中，可以在执行 semWait / semSignal 时短暂关中断，保证不被打断。

---

## 本节最容易考的题型

### 题型 1：解释 race condition

答题模板：

**Race condition** occurs when multiple processes or threads access and modify shared data, and the final result depends on the relative timing or order of execution.

中文：
竞态条件是指多个进程或线程并发访问并修改共享数据，最终结果依赖于它们执行的相对顺序。

---

### 题型 2：解释 critical section 和 mutual exclusion

**Critical section** is a section of code that accesses shared resources and must not be executed by more than one process at the same time.

**Mutual exclusion** means that when one process is in its critical section, no other process may enter a corresponding critical section for the same shared resource.

---

### 题型 3：列出 mutual exclusion 的要求

背六点：

```text
1. 同一时刻最多一个进程进入临界区
2. 非临界区中停止的进程不能影响别人
3. 不能死锁或饥饿
4. 临界区空闲时，请求进入不能被延迟
5. 不假设进程速度或处理器数量
6. 进程只能在临界区停留有限时间
```

---

### 题型 4：P/V 操作含义

```text
P = semWait = wait = down = 申请资源，可能阻塞
V = semSignal = signal = up = 释放资源，可能唤醒
```

---

### 题型 5：信号量 count 的含义

```text
s.count > 0：还有 s.count 个资源可用
s.count = 0：资源刚好用完
s.count < 0：有 |s.count| 个进程正在等待
```

---

### 题型 6：生产者消费者代码填空

有界缓冲区必背：

```c
semaphore s = 1;
semaphore n = 0;
semaphore e = sizeofbuffer;

producer:
    produce();
    semWait(e);
    semWait(s);
    append();
    semSignal(s);
    semSignal(n);

consumer:
    semWait(n);
    semWait(s);
    take();
    semSignal(s);
    semSignal(e);
    consume();
```

核心顺序：

生产者先等空位，再锁缓冲区。
消费者先等产品，再锁缓冲区。
不能先拿 mutex 再等资源，否则可能死锁。




我按你刚复习的 **5.1 Principles of Concurrency** 和 **5.3 Semaphores** 来筛。直接相关的是：

**Review Questions：5.1–5.10**
**Problems：5.1、5.3、5.4、5.5、5.12、5.16、5.19、5.20、5.21、5.22**
其中 5.11、5.12 Review 后半、5.13 Review 属于 Monitor / Message Passing / Readers-Writers，不是本轮重点；Problems 5.24、5.25 也是后面小节内容。题目来自 Chapter 5 questions，对照 5.1–5.3 内容作答。 

# Review Questions

## 5.1 List four design issues for which concurrency is relevant.

并发相关的四个设计问题：

1. **Communication among processes**：进程之间通信。
2. **Sharing and competing for resources**：共享资源或竞争资源，例如 memory、files、I/O devices。
3. **Synchronization of process activities**：多个进程活动的同步。
4. **Allocation of processor time**：CPU 时间分配，也就是 scheduling。

---

## 5.2 What are three contexts in which concurrency arises?

并发出现的三个场景：

1. **Multiple applications**：多个应用程序同时运行。
2. **Structured applications**：一个应用被设计成多个并发模块或进程。
3. **Operating system structure**：操作系统本身由多个进程或线程组成。

---

## 5.3 What is the basic requirement for the execution of concurrent processes?

基本要求是：

**Mutual exclusion 互斥**。

也就是当一个进程进入访问共享资源的 **critical section 临界区** 时，其他进程不能同时进入访问同一共享资源的临界区。

---

## 5.4 List three degrees of awareness between processes.

三种进程感知程度：

1. **Processes unaware of each other**：彼此不知道对方存在。
   关系是 **competition 竞争**。

2. **Processes indirectly aware of each other**：间接知道对方。
   它们共享某些对象，例如 buffer、file、database。关系是 **cooperation by sharing 通过共享合作**。

3. **Processes directly aware of each other**：直接知道对方。
   它们可以通过 process ID 或通信原语交流。关系是 **cooperation by communication 通过通信合作**。

---

## 5.5 What is the distinction between competing processes and cooperating processes?

**Competing processes 竞争进程**：
进程之间没有信息交换，只是争抢同一个资源，例如 printer、disk、file。

**Cooperating processes 合作进程**：
进程之间需要共同完成任务，可能通过共享数据或消息通信协作，某个进程的结果可能依赖其他进程。

---

## 5.6 List the three control problems associated with competing processes.

竞争进程的三个控制问题：

1. **Mutual exclusion 互斥**：同一时刻只能一个进程使用临界资源。
2. **Deadlock 死锁**：多个进程互相等待对方释放资源，导致都不能继续。
3. **Starvation 饥饿**：某个进程长期得不到资源，虽然系统整体还在运行。

---

## 5.7 List the requirements for mutual exclusion.

互斥的六个要求：

1. 同一时刻最多一个进程进入对应临界区。
2. 停在非临界区的进程不能影响其他进程。
3. 不能发生 deadlock 或 starvation。
4. 如果临界区空闲，请求进入的进程不能被无故延迟。
5. 不能假设进程相对速度，也不能假设处理器数量。
6. 进程只能在临界区停留有限时间。

---

## 5.8 What operations can be performed on a semaphore?

Semaphore 只能进行三类操作：

1. **Initialize 初始化**：初始化为非负整数。
2. **semWait / P operation**：信号量减 1，若结果为负，则阻塞进程。
3. **semSignal / V operation**：信号量加 1，若有进程等待，则唤醒一个。

注意：不能直接随便读写 semaphore 的值。

---

## 5.9 What is the difference between binary and general semaphores?

**Binary semaphore 二元信号量**：
只能取 0 或 1，常用于互斥。

**General semaphore / Counting semaphore 一般信号量 / 计数信号量**：
可以取整数值，用来表示多个资源的数量，也可以用于同步。

---

## 5.10 What is the difference between strong and weak semaphores?

**Strong semaphore 强信号量**：
等待队列按 FIFO 释放，先等待的进程先被唤醒。

**Weak semaphore 弱信号量**：
不规定等待进程的释放顺序，所以可能导致 starvation。

# Problems

## Problem 5.1

**题意：** multiprogramming 和 multiprocessing 在并发问题上类似，但也有区别。说出两个区别。

答案：

1. **Multiprogramming 多道程序系统** 中，单处理器上进程只是 **interleaving 交错执行**；
   **Multiprocessing 多处理器系统** 中，多个进程可以真正 **overlap / parallel 并行执行**。

2. 单处理器中，关中断可以临时防止进程切换，从而保护临界区；
   多处理器中，关掉一个 CPU 的中断不能阻止其他 CPU 同时访问共享内存，所以需要硬件原子指令或更强同步机制。

---

## Problem 5.3

题目给两个并发进程，都反复执行：

```c
x = x - 1;
x = x + 1;
if (x != 10)
    printf("x is %d", x);
```

初始 `x = 10`。

### a. 构造输出 `x is 10`

一种 interleaving：

```text
初始 x = 10

P1: x = x - 1        // x = 9
P1: x = x + 1        // x = 10
P2: x = x - 1        // x = 9
P1: 判断 if (x != 10)，条件为 true
P2: x = x + 1        // x = 10
P1: printf 读取 x，输出 x is 10
```

关键点：`if` 判断和 `printf` 读取 `x` 不是一个不可分割的 atomic operation。

---

### b. 构造输出 `x is 8`

因为 `x = x - 1` 不是 atomic，实际可能是：

```asm
LD R0, x
DECR R0
STO R0, x
```

构造如下：

```text
初始 x = 10

P1: LD 10, DECR, STO 9          // P1 完成 x = x - 1，x = 9

P2: LD x = 9
P2: DECR 得到 8
P2: 暂停，还没 STO

P1: LD x = 9
P1: INCR 得到 10
P1: STO 10                     // P1 完成 x = x + 1，x = 10

P2: STO 8                      // P2 把旧计算结果写回，x = 8

P1: if (x != 10) 为 true
P1: printf 读取 x = 8
```

所以输出：

```text
x is 8
```

这就是典型 **race condition 竞态条件**。

---

## Problem 5.4

两个进程同时执行：

```c
for (count = 1; count <= 50; count++) {
    tally++;
}
```

初始：

```c
tally = 0;
```

### a. 两个进程时 tally 的上下界

每个进程执行 50 次 `tally++`，总共 100 次。

由于 `tally++` 不是 atomic，可能丢失更新。

**Upper bound 上界：**

```text
100
```

如果没有任何 lost update，最终就是 100。

**Lower bound 下界：**

```text
2
```

原因：可以让一个进程前 49 次更新完成，然后另一个进程用很早读到的旧值覆盖为 1，再让第一个进程最后一次基于 1 写回 2。最终可以低到 2。

所以：

```text
2 <= tally <= 100
```

---

### b. 如果允许任意多个进程并行

假设有 k 个进程，每个执行 50 次。

上界：

```text
50k
```

下界在 k ≥ 2 时仍然可以是：

```text
2
```

如果进程数不固定且可以任意大，则上界不再固定，会随进程数增长。

---

## Problem 5.5

**Is busy waiting always less efficient than a blocking wait?**

不是。

**Busy waiting 忙等待** 会一直占用 CPU 检查条件，长时间等待时效率低。
但如果等待时间非常短，busy waiting 可能比 blocking 更高效，因为 blocking 需要进程切换、进入内核、重新调度，开销更大。

所以：

```text
短等待：busy waiting 可能更好
长等待：blocking wait 通常更好
```

---

## Problem 5.12

题目给了两种 semaphore 定义。

教材定义中：

```c
semWait(s):
    s.count--;
    if (s.count < 0)
        block;
```

所以 `s.count` 可以是负数，负数绝对值表示等待进程数。

题目中的另一个定义中，`s.count` 永远不为负，等待进程只放在 queue 里。

结论：

**两种定义在程序效果上等价。**

区别只是表示方法不同：

```text
教材定义：s.count < 0 表示有 |s.count| 个进程等待
题目定义：s.count 不为负，等待进程数由 queue 表示
```

因为程序不能直接检查或修改 semaphore 内部值，所以替换后语义不变。

---

## Problem 5.16

题目要求用 binary semaphore 实现 general semaphore，但给出的程序有 bug。

### bug 是什么？

假设初始：

```text
s = 0
delay = 0
mutex = 1
```

两个进程 P1、P2 都执行 `semWait(s)`。

可能出现：

```text
P1: s--，s = -1，释放 mutex，但还没真正 wait(delay)
P2: s--，s = -2，释放 mutex，但还没真正 wait(delay)
```

此时如果有两个 `semSignal(s)`，由于 `delay` 是 binary semaphore，它只能记住一个 signal，另一个 signal 可能丢失，导致一个进程永远阻塞。

这就是 **lost wakeup 丢失唤醒**。

### 修正方法

把 `semSignal` 末尾释放 `mutex` 的那一行移动到被唤醒进程返回之后。

修正思路：

```c
void semWait(semaphore s)
{
    semWaitB(mutex);
    s--;
    if (s < 0) {
        semSignalB(mutex);
        semWaitB(delay);
        semSignalB(mutex);   // 被唤醒后释放 mutex
    }
    else {
        semSignalB(mutex);
    }
}

void semSignal(semaphore s)
{
    semWaitB(mutex);
    s++;
    if (s <= 0) {
        semSignalB(delay);   // 唤醒一个等待者，但不立刻释放 mutex
    }
    else {
        semSignalB(mutex);
    }
}
```

本质是 **pass the baton 传递接力棒**：唤醒者把继续执行的权利交给被唤醒者。

---

## Problem 5.19

题目问：为什么不能把 Figure 5.9 中 consumer 的：

```c
if (n == 0) semWaitB(delay);
```

简单移动到由 `s` 控制的 critical section 里面？

如果移动后大概变成：

```c
semWaitB(s);
take();
n--;
if (n == 0) semWaitB(delay);
semSignalB(s);
```

会死锁。

构造：

```text
初始：n = 0, s = 1, delay = 0

Producer:
semWaitB(s)       // s = 0
append()
n++               // n = 1
semSignalB(delay) // delay = 1
semSignalB(s)     // s = 1

Consumer:
semWaitB(delay)   // delay = 0
semWaitB(s)       // s = 0
take()
n--               // n = 0
if (n == 0) semWaitB(delay)
```

此时 consumer 在持有 `s` 的情况下等待 `delay`。

接下来 Producer 想生产：

```text
Producer: semWaitB(s)
```

但 `s = 0`，producer 被阻塞。

于是：

```text
Consumer 等 Producer signal(delay)
Producer 等 Consumer 释放 s
```

双方互相等待，发生 **deadlock 死锁**。

---

## Problem 5.20

题目要求改进 infinite-buffer producer/consumer，减少常见情况下的 semaphore 开销。

可以让：

```text
n = -1
```

表示：

```text
buffer 是空的，而且 consumer 已经检测到空，准备阻塞
```

改进版：

```c
int n = 0;
binary_semaphore s = 1, delay = 0;

void producer()
{
    while (true) {
        produce();

        semWaitB(s);
        append();
        n++;

        if (n == 0) {
            semSignalB(delay);
        }

        semSignalB(s);
    }
}

void consumer()
{
    while (true) {
        semWaitB(s);

        if (n == 0) {
            n = -1;
            semSignalB(s);

            semWaitB(delay);

            semWaitB(s);
            take();
            semSignalB(s);
        }
        else {
            take();
            n--;
            semSignalB(s);
        }

        consume();
    }
}
```

核心思想：

* `n > 0`：buffer 中有 n 个 item。
* `n = 0`：buffer 空，但 consumer 还没阻塞。
* `n = -1`：buffer 空，consumer 已经准备等待 producer。

---

## Problem 5.21

Figure 5.13 的 bounded-buffer 代码：

Producer：

```c
semWait(e);
semWait(s);
append();
semSignal(s);
semSignal(n);
```

Consumer：

```c
semWait(n);
semWait(s);
take();
semSignal(s);
semSignal(e);
```

### a. 交换 `semWait(e); semWait(s)`

如果变成：

```c
semWait(s);
semWait(e);
```

会改变程序含义，可能死锁。

原因：如果 buffer 满了，producer 先拿到 mutex `s`，然后等待空位 `e`。但 consumer 要取走 item 也需要 `s`，于是 consumer 进不来，producer 等不到空位，死锁。

---

### b. 交换 `semSignal(s); semSignal(n)`

如果变成：

```c
semSignal(n);
semSignal(s);
```

一般不改变正确性。

只是 consumer 可能被唤醒后发现 `s` 还没释放，于是再等一下。语义上仍然正确，最多影响调度效率。

---

### c. 交换 `semWait(n); semWait(s)`

如果变成：

```c
semWait(s);
semWait(n);
```

会改变程序含义，可能死锁。

原因：如果 buffer 空了，consumer 先拿到 `s`，再等待 item `n`。但 producer 要放入 item 也必须先拿 `s`，所以 producer 进不来，consumer 等不到 item，死锁。

---

### d. 交换 `semSignal(s); semSignal(e)`

如果变成：

```c
semSignal(e);
semSignal(s);
```

一般不改变正确性。

producer 可能被提前唤醒，但如果 `s` 还没释放，它会继续等待 `s`。语义仍然正确，只是可能多一点调度开销。

---

## Problem 5.22

这是 bounded-buffer 的 semaphore trace 表。下面给出从题目空白处开始的填表结果。记号：

```text
X = occupied slot 已占用缓冲区
O = empty slot 空缓冲区
队列用 [ ] 表示
```

| Scheduled Step    |        full 状态 | Buffer |            empty 状态 |
| ----------------- | -------------: | ------ | ------------------: |
| Pb executes p1    | full = -1 [Cb] | OOO    |          empty = +2 |
| Pa executes p1    | full = -1 [Cb] | OOO    |          empty = +1 |
| Pa executes p2    | full = -1 [Cb] | XOO    |          empty = +1 |
| Pb executes p2    | full = -1 [Cb] | XXO    |          empty = +1 |
| Pb executes p3    |       full = 0 | XXO    |          empty = +1 |
| Pc executes p1    |       full = 0 | XXO    |           empty = 0 |
| Cb executes c2    |       full = 0 | XOO    |           empty = 0 |
| Pc executes p2    |       full = 0 | XXO    |           empty = 0 |
| Cb executes c3    |       full = 0 | XXO    |          empty = +1 |
| Pa executes p3    |      full = +1 | XXO    |          empty = +1 |
| Pb executes p1-p3 |      full = +2 | XXX    |           empty = 0 |
| Pc executes p3    |      full = +3 | XXX    |           empty = 0 |
| Pa executes p1    |      full = +3 | XXX    |     empty = -1 [Pa] |
| Pd executes p1    |      full = +3 | XXX    | empty = -2 [Pa, Pd] |
| Ca executes c1-c3 |      full = +2 | XXO    |     empty = -1 [Pd] |
| Pa executes p2    |      full = +2 | XXX    |     empty = -1 [Pd] |
| Cc executes c1-c2 |      full = +1 | XXO    |     empty = -1 [Pd] |
| Pa executes p3    |      full = +2 | XXO    |     empty = -1 [Pd] |
| Cc executes c3    |      full = +2 | XXO    |           empty = 0 |
| Pd executes p2-p3 |      full = +3 | XXX    |           empty = 0 |

最重要的不是死记整张表，而是会按规则更新：

```text
wait(full): full--
wait(empty): empty--
signal(full): full++，若有人等 full，则唤醒 FIFO 队首
signal(empty): empty++，若有人等 empty，则唤醒 FIFO 队首
```

核心考点总结：

```text
5.1 重点：race condition、critical section、mutual exclusion、deadlock、starvation
5.3 重点：P/V 操作、binary/general semaphore、producer-consumer、wait 顺序不能乱
```
