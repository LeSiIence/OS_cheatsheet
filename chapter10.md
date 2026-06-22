下面继续复习 **10.1 Multiprocessor and Multicore Scheduling / 多处理器与多核调度**。这一节的重点不是“单个 CPU 怎么选下一个进程”，而是：**多个处理器/核心之间，任务怎么分配、线程怎么同时运行、缓存怎么利用**。原文主要从 granularity、design issues、process scheduling、thread scheduling、multicore thread scheduling 展开。

---

## 1. Multiprocessor systems / 多处理器系统分类

原文先把 multiprocessor system 分成三类：

### 1. Loosely coupled or distributed multiprocessor / 松耦合或分布式多处理器

每个 processor 有自己的 **main memory / 主存** 和 **I/O channels / I/O 通道**。

中文理解：
这更像多台独立机器组成的集群，比如 cluster，不是本节重点。

---

### 2. Functionally specialized processors / 功能专用处理器

例如 **I/O processor / I/O 处理器**。

中文理解：
有一个 master general-purpose processor / 主通用处理器，其他专用处理器帮它做特定工作，比如 I/O。

---

### 3. Tightly coupled multiprocessor / 紧耦合多处理器

多个 processors 共享同一个 **common main memory / 公共主存**，并由一个 OS 统一控制。

中文理解：
这是本节重点。因为多个 CPU/core 共享内存，所以 OS 要决定：
**哪个进程/线程放在哪个处理器上运行。**

---

## 2. Granularity / 同步粒度

**Granularity** 指并行任务之间同步的频率，也就是：

> 多个进程/线程之间多久需要互相沟通、同步一次？

同步越频繁，粒度越细；同步越少，粒度越粗。

| English term                    | 中文     | 含义           | 同步频率  |
| ------------------------------- | ------ | ------------ | ----- |
| Fine-grained parallelism        | 细粒度并行  | 单条指令流内部的并行   | 非常频繁  |
| Medium-grained parallelism      | 中粒度并行  | 一个应用内部多个线程协作 | 较频繁   |
| Coarse-grained parallelism      | 粗粒度并行  | 多个并发进程       | 不太频繁  |
| Very coarse-grained parallelism | 很粗粒度并行 | 网络节点间分布式处理   | 很少    |
| Independent parallelism         | 独立并行   | 多个无关任务       | 不需要同步 |

---

## 3. Independent parallelism / 独立并行

**Independent parallelism** 的特点是：

> There is no explicit synchronization among processes.
> 进程之间没有显式同步。

例如多个用户同时在系统上运行不同程序：一个人在写 Word，一个人在用表格，一个人在编译代码。

中文理解：
这些任务互不相关，所以多处理器系统只需要把它们分给不同 CPU/core 即可。
这种情况比较简单，调度策略不需要特别复杂。

---

## 4. Coarse and very coarse-grained parallelism / 粗粒度和很粗粒度并行

**Coarse-grained parallelism** 里面，进程之间有同步，但同步不频繁。

例如：
一个大型软件编译时，不同文件可以同时编译。每个编译任务之间关系不紧密。

中文理解：
这种情况用多处理器能加速，但仍然比较容易调度。
因为进程之间不需要经常等待彼此。

---

## 5. Medium-grained parallelism / 中粒度并行

这是考试重点。

**Medium-grained parallelism** 通常对应：

> A single application implemented as a collection of threads.
> 一个应用由多个线程组成。

例如：
一个程序有多个线程，一个线程负责计算，一个线程负责 I/O，一个线程负责图形界面。

中文理解：
线程之间共享同一个地址空间，协作频繁。
所以调度一个线程时，不能只看这个线程自己，还要考虑它和同一个进程中其他线程的关系。

关键句理解：

> Scheduling decisions concerning one thread may affect the performance of the entire application.
> 对一个线程的调度决策可能影响整个应用性能。

例如：
线程 A 正在运行，但它很快要等线程 B 的结果。
如果线程 B 没有被调度运行，线程 A 就会阻塞，整个程序变慢。

---

## 6. Design Issues / 多处理器调度的设计问题

原文说 multiprocessor scheduling 有三个相关问题：

### 1. Assignment of processes to processors

### 进程如何分配给处理器

问题是：

> 一个进程是固定在一个 CPU 上运行，还是可以在多个 CPU 之间移动？

有两种主要方式。

---

### Static assignment / 静态分配

进程从开始到结束都固定分配给某个 processor。

优点：

* scheduling overhead / 调度开销较小；
* 处理器分配只做一次；
* 有利于 group scheduling / gang scheduling。

缺点：

* 可能负载不均衡；
* 一个 CPU 队列空了，另一个 CPU 队列很长。

中文例子：
CPU 1 没任务，CPU 2 排了一堆任务，但任务不能随便迁移，就会浪费 CPU 1。

---

### Dynamic assignment / 动态分配

所有进程进入一个 common queue / 公共队列，哪个 processor 空闲，就从队列里取任务。

优点：

* 负载更均衡；
* 不容易出现某个 CPU 空闲而另一个 CPU 很忙。

缺点：

* 进程可能在不同 CPU 间迁移；
* 可能影响 cache locality / 缓存局部性。

---

## 7. Master/slave architecture vs peer architecture

多处理器系统还要决定 OS 内核由谁运行。

---

### Master/slave architecture / 主从结构

一个 master processor 负责关键 OS kernel functions / 内核功能，比如调度、I/O 管理。
其他 slave processors 主要运行用户程序。

优点：

* 简单；
* 比较容易从单处理器 OS 改造过来；
* 资源冲突容易处理。

缺点：

* master 失败，整个系统可能崩溃；
* master 可能成为 performance bottleneck / 性能瓶颈。

---

### Peer architecture / 对等结构

每个 processor 都可以运行 OS kernel，并且自己调度任务。

优点：

* 没有单一 master 瓶颈；
* 更适合真正并行的系统。

缺点：

* OS 设计更复杂；
* 必须避免两个 processor 选中同一个进程；
* 需要同步机制保护共享队列和共享资源。

考试可以这样答：

> Master/slave is simple but has a single point of failure and bottleneck. Peer scheduling is more scalable but requires more complex synchronization.

中文：

> 主从结构简单，但 master 可能成为单点故障和性能瓶颈；对等结构扩展性更好，但需要复杂同步。

---

## 8. Multiprogramming on individual processors / 单个处理器上是否多道程序设计

问题：

> 如果一个进程已经被分配给某个 processor，这个 processor 是否还应该同时维护多个进程？

对于传统 coarse-grained 或 independent parallelism，答案通常是：

> Yes.
> 是的。

因为如果一个进程等待 I/O，CPU 可以切换去运行另一个进程，提高利用率。

但是对于 medium-grained multithreaded applications，情况不同。

如果一个应用由多个线程组成，而且这些线程需要同时运行，那么目标不一定是“让每个 CPU 永远忙”，而是：

> 让应用整体性能最好。

例如一个程序有 4 个线程，必须一起推进。
如果只运行其中 1 个线程，其他线程没运行，整体可能反而慢。

---

## 9. Process Dispatching / 进程派遣

在单处理器调度中，我们很重视：

* priorities / 优先级；
* shortest job first；
* round robin；
* execution history / 执行历史。

但在多处理器系统中，原文指出：
具体调度算法的重要性会降低。

原因是：
如果有多个 CPU，一个长任务不会像单 CPU 那样严重阻塞其他任务。其他任务仍然可以在另一个 CPU 上运行。

所以考试可以记：

> In multiprocessor systems, simple scheduling policies such as FCFS may be sufficient, because multiple processors reduce the negative effect of a long-running process.

中文：

> 在多处理器系统中，简单的 FCFS 可能已经足够，因为多个处理器会削弱长进程对其他进程的阻塞影响。

---

# 10. Thread Scheduling / 线程调度

这是 10.1 的核心考点。

原文给出四种 multiprocessor thread scheduling approaches：

1. **Load sharing / 负载共享**
2. **Gang scheduling / 组调度**
3. **Dedicated processor assignment / 专用处理器分配**
4. **Dynamic scheduling / 动态调度**

---

## 11. Load sharing / 负载共享

### 定义

所有 ready threads / 就绪线程 放入一个 global queue / 全局队列。
哪个 processor 空闲，就从队列中取一个线程运行。

中文理解：
大家都从一个公共任务池里拿任务。

---

### 优点

1. **Load is distributed evenly**
   负载比较均衡。

2. **No centralized scheduler is required**
   不需要一个单独的中央调度器。

3. **Can use priority or history-based scheduling**
   全局队列可以结合优先级、执行历史等策略。

---

### 缺点

1. **Global queue bottleneck / 全局队列瓶颈**

多个处理器同时访问一个 ready queue，需要互斥保护。
处理器很多时，这个队列会成为瓶颈。

2. **Poor cache locality / 缓存局部性差**

线程被抢占后，下一次不一定回到同一个 CPU。
原 CPU cache 里的数据可能浪费。

3. **Related threads may not run together / 相关线程不能同时运行**

同一个应用的多个线程可能被分散排队，无法同时获得 CPU。
对于需要频繁同步的线程，性能会下降。

---

## 12. Gang scheduling / 组调度

### 定义

> A set of related threads is scheduled to run on a set of processors at the same time.
> 一组相关线程被同时调度到一组处理器上运行。

中文理解：
一个进程的多个线程一起上 CPU，一起跑。

---

### 为什么有用？

假设一个程序有线程 A、B、C、D。
A 运行到一半需要 B 的结果。
如果 B 还没被调度，A 就只能等待。

Gang scheduling 会尽量让 A、B、C、D 同时运行，减少等待。

---

### 优点

1. **Reduce synchronization blocking / 减少同步阻塞**

相关线程一起运行，不容易出现一个线程等另一个线程的情况。

2. **Reduce process/thread switching / 减少切换开销**

一次调度决策可以影响一组 processor 和一组 threads。

3. **Good for medium-grained or fine-grained parallel applications**

适合线程协作频繁的应用。

---

### 考试表达

> Gang scheduling is useful when threads of the same application need frequent coordination. It schedules related threads simultaneously, reducing synchronization blocking and context switching overhead.

中文：

> 组调度适合同一应用内线程需要频繁协作的情况。它让相关线程同时运行，从而减少同步阻塞和上下文切换开销。

---

## 13. Dedicated processor assignment / 专用处理器分配

### 定义

一个应用运行期间，它的每个线程都固定分配一个 processor。
这些 processor 在应用结束前不会分配给其他应用。

中文理解：
比 gang scheduling 更极端。
Gang scheduling 是“一组线程一起被调度”；
Dedicated processor assignment 是“一组处理器直接包给这个应用”。

---

### 优点

* 几乎没有 thread/process switching；
* cache locality 较好；
* 对高度并行程序可能有明显加速。

---

### 缺点

* 如果线程阻塞，对应 processor 会空闲；
* processor utilization / 处理器利用率可能很低。

但原文也强调：
在拥有几十甚至几百个 processors 的高度并行系统中，**processor utilization 不一定是最重要指标**，应用整体性能可能更重要。

---

## 14. Dynamic scheduling / 动态调度

### 定义

进程中的线程数量可以在运行过程中动态改变。
OS 和 application 一起参与调度。

中文理解：
应用可以根据当前可用 processor 数量调整自己的线程数量。

例如：
系统空闲时，应用开 8 个线程；
系统忙时，应用减少到 2 个线程。

---

### 原文中的策略大意

当 job 请求 processors 时：

1. 如果有 idle processors，就分配给它。
2. 如果没有空闲 processor，且这是新来的 job，就从拥有多个 processor 的 job 那里拿走一个。
3. 如果请求无法满足，就先挂起等待。
4. 当 processor 被释放时，优先分给还没有 processor 的 job，然后按 FCFS 分配剩余 processor。

---

### 考试理解

Dynamic scheduling 的核心是：

> The number of threads in a process can change during execution.

中文：

> 一个进程中的线程数可以在执行期间动态变化。

优点是灵活，缺点是系统和程序都更复杂，调度开销可能较大。

---

# 15. Multicore Thread Scheduling / 多核线程调度

多核调度和传统多处理器调度的区别在于：

> Cache matters more.
> 缓存更重要。

传统多处理器调度主要想让所有 CPU 都忙。
但是多核芯片中，核心之间可能共享某些 cache，比如 L2 或 L3 cache。

所以调度时不仅要考虑 load balance / 负载均衡，还要考虑：

* cache locality / 缓存局部性；
* shared cache / 共享缓存；
* memory contention / 内存竞争；
* off-chip memory access / 片外内存访问。

---

## 16. Adjacent cores / 相邻核心

原文用 AMD Bulldozer 举例：
每个 core 有自己的 L1 cache；每两个 cores 共享 L2 cache；所有 cores 共享 L3 cache。

如果两个 cores 共享同一个 L2 cache，它们可以叫：

> adjacent cores / 相邻核心。

如果两个线程共享大量数据，最好把它们放到 adjacent cores 上。
这样一个线程加载进 cache 的数据，另一个线程也能更快访问。

---

## 17. Cooperative resource sharing / 协作式资源共享

### 定义

多个线程访问相同的 memory locations / 内存位置。

例如：

* multithreaded application / 多线程应用；
* producer-consumer interaction / 生产者—消费者线程交互。

这种情况下，把线程放在共享 cache 的相邻核心上有利于性能。

中文理解：
如果两个线程经常用同一批数据，那就让它们靠近一点，共享 cache。

---

## 18. Resource contention / 资源竞争

另一种情况是：
两个线程不共享数据，但它们被放在共享 cache 的 cores 上，会抢 cache 空间。

这会导致：

* 一个线程占用更多 cache；
* 另一个线程 cache 空间变少；
* cache miss 增加；
* 性能下降。

所以调度器需要考虑：

> Contention-aware scheduling / 竞争感知调度。

中文理解：
不是所有线程都适合放在共享 cache 的核心上。
共享数据的线程适合靠近；互相抢 cache 的线程最好分开。

---

# 19. 本节最重要对比总结

## Load sharing vs Gang scheduling

| 对比   | Load sharing                  | Gang scheduling    |
| ---- | ----------------------------- | ------------------ |
| 中文   | 负载共享                          | 组调度                |
| 核心思想 | 所有线程进入公共队列，谁空闲谁拿              | 相关线程一起运行           |
| 优点   | 简单、负载均衡                       | 减少同步等待             |
| 缺点   | 相关线程可能不能同时运行；cache locality 差 | 需要一次分配多个 processor |
| 适合   | 线程独立性较强                       | 线程协作频繁             |

---

## Gang scheduling vs Dedicated processor assignment

| 对比         | Gang scheduling | Dedicated processor assignment |
| ---------- | --------------- | ------------------------------ |
| 中文         | 组调度             | 专用处理器分配                        |
| 核心思想       | 一组线程同时被调度       | 一组处理器长期分配给一个应用                 |
| 是否长期占有 CPU | 不一定             | 是                              |
| 优点         | 减少同步阻塞          | 减少切换，cache locality 好          |
| 缺点         | 分配复杂            | CPU 可能空闲浪费                     |

---

## Multiprocessor vs Multicore scheduling

| 对比   | Multiprocessor scheduling    | Multicore scheduling            |
| ---- | ---------------------------- | ------------------------------- |
| 中文   | 多处理器调度                       | 多核调度                            |
| 主要关注 | processor allocation / 处理器分配 | cache-aware scheduling / 缓存感知调度 |
| 传统目标 | load balancing / 负载均衡        | 减少片外内存访问，提高 cache 利用            |
| 新问题  | 多 CPU 如何分配任务                 | 共享 cache 带来的协作与竞争               |

---

# 20. 高频考试题

## Q1. What are the three design issues in multiprocessor scheduling?

答：

The three design issues are:

1. assignment of processes to processors;
2. use of multiprogramming on individual processors;
3. actual dispatching of a process.

中文：

多处理器调度的三个设计问题是：

1. 进程如何分配给处理器；
2. 单个处理器上是否进行多道程序设计；
3. 如何实际选择并派遣进程运行。

---

## Q2. What is load sharing?

答：

Load sharing is a multiprocessor thread scheduling approach in which all ready threads are placed in a global queue. When a processor becomes idle, it selects a thread from the queue.

中文：

负载共享是一种多处理器线程调度方法。所有就绪线程放入全局队列，哪个处理器空闲，就从队列中选择一个线程运行。

---

## Q3. What are the disadvantages of load sharing?

答：

The disadvantages include:

1. the global queue may become a bottleneck;
2. preempted threads may not resume on the same processor, reducing cache efficiency;
3. related threads of the same application may not run simultaneously, hurting performance when synchronization is frequent.

中文：

负载共享的缺点包括：

1. 全局队列可能成为瓶颈；
2. 被抢占的线程不一定回到原处理器，缓存效率下降；
3. 同一应用的相关线程可能不能同时运行，频繁同步时性能下降。

---

## Q4. What is gang scheduling?

答：

Gang scheduling schedules a set of related threads to run simultaneously on a set of processors.

中文：

组调度是把一组相关线程同时调度到一组处理器上运行。

---

## Q5. Why is gang scheduling useful?

答：

Gang scheduling is useful because related threads often need to synchronize with each other. Running them simultaneously reduces synchronization blocking and context-switching overhead.

中文：

组调度有用是因为相关线程经常需要相互同步。让它们同时运行可以减少同步阻塞和上下文切换开销。

---

## Q6. What is dedicated processor assignment?

答：

Dedicated processor assignment allocates processors to an application for the entire duration of its execution. Each thread is assigned a processor, and the processor remains dedicated to that thread until the application completes.

中文：

专用处理器分配是在应用运行期间，把一组处理器固定分配给该应用。每个线程对应一个处理器，直到应用结束。

---

## Q7. What is dynamic scheduling?

答：

Dynamic scheduling allows the number of threads in a process to change during execution. The operating system allocates processors among jobs, while applications decide how to map tasks to available threads.

中文：

动态调度允许进程中的线程数量在运行过程中变化。操作系统负责给作业分配处理器，应用程序负责把任务映射到可用线程上。

---

## Q8. Why is cache important in multicore scheduling?

答：

In multicore systems, some cores may share caches. Scheduling related threads on adjacent cores can improve locality, while scheduling competing threads on the same shared cache may cause resource contention.

中文：

在多核系统中，一些核心可能共享缓存。把共享数据的相关线程放在相邻核心上可以提高局部性；但把相互竞争缓存的线程放在共享缓存上可能导致资源竞争。

---

# 21. 一句话总复习

**10.1 的核心：**

> Multiprocessor scheduling focuses on assigning processes or threads to multiple processors, while multicore scheduling further considers cache sharing and memory locality.

中文：

> 多处理器调度关注如何把进程或线程分配给多个处理器；多核调度在此基础上还要考虑缓存共享和内存局部性。

最重要的四种线程调度方法记住：

> Load sharing, Gang scheduling, Dedicated processor assignment, Dynamic scheduling.
> 负载共享、组调度、专用处理器分配、动态调度。





根据你刚复习的 **10.1 Multiprocessor and Multicore Scheduling**，以及最后一页已经进入的 **10.2 Real-Time Scheduling Background**，相关题目主要是 **Review Questions 10.1–10.5**。后面的 10.6–10.8 和 Problems 10.1–10.8 主要属于后续 real-time scheduling 算法计算题，现在先不用做。题目列表在 chapter10 questions 的 Review Questions 部分。

---

# 相关题目 10.1

## 题目原文

**10.1. List and briefly define five different categories of synchronization granularity.**

列出并简要定义五种不同的同步粒度。

---

## 答案

五种 **synchronization granularity / 同步粒度** 是：

| English                             | 中文     | 含义                     |
| ----------------------------------- | ------ | ---------------------- |
| **Fine-grained parallelism**        | 细粒度并行  | 单条指令流内部的并行，同步非常频繁      |
| **Medium-grained parallelism**      | 中粒度并行  | 一个应用内部多个线程并行执行，需要较频繁协作 |
| **Coarse-grained parallelism**      | 粗粒度并行  | 多个并发进程在多道程序环境中运行，同步较少  |
| **Very coarse-grained parallelism** | 很粗粒度并行 | 网络节点之间的分布式处理，同步更少      |
| **Independent parallelism**         | 独立并行   | 多个无关进程，基本不需要显式同步       |

英文背诵版：

> The five categories are fine, medium, coarse, very coarse, and independent parallelism.

中文理解：

> 粒度越细，进程或线程之间越需要频繁同步；粒度越粗，它们之间越独立。

教材 Table 10.1 正是按这五类列出同步粒度，并给出 fine、medium、coarse、very coarse、independent 的同步频率差异。

---

# 相关题目 10.2

## 题目原文

**10.2. List and briefly define four techniques for thread scheduling.**

列出并简要定义四种线程调度技术。

---

## 答案

四种 **multiprocessor thread scheduling / 多处理器线程调度** 技术是：

### 1. Load sharing / 负载共享

所有 ready threads / 就绪线程放入一个 **global queue / 全局队列**。哪个 processor 空闲，就从队列中取一个线程运行。

中文理解：

> 大家共用一个任务池，谁空了谁拿任务。

---

### 2. Gang scheduling / 组调度

一组相关线程同时被调度到一组处理器上运行。

中文理解：

> 同一个程序里的多个线程一起上 CPU，避免一个线程运行、另一个线程却没运行导致等待。

---

### 3. Dedicated processor assignment / 专用处理器分配

一个程序执行期间，分配给它固定数量的处理器，通常等于它的线程数。程序结束后，这些处理器才回到系统池中。

中文理解：

> 把一组 CPU/core 包给一个程序用，直到程序结束。

---

### 4. Dynamic scheduling / 动态调度

进程中的线程数量可以在执行过程中动态改变，操作系统根据负载给程序分配处理器。

中文理解：

> 系统空闲时多给线程/处理器，系统忙时减少资源。

英文背诵版：

> The four techniques are load sharing, gang scheduling, dedicated processor assignment, and dynamic scheduling.

教材在 Thread Scheduling 部分直接列出了这四种 general approaches。

---

# 相关题目 10.3

## 题目原文

**10.3. List and briefly define three versions of load sharing.**

列出并简要定义三种负载共享版本。

---

## 答案

三种 **load sharing / 负载共享** 版本是：

### 1. First-come-first-served, FCFS / 先来先服务

当一个 job 到达时，它的所有线程依次放入 shared queue / 共享队列末尾。
处理器空闲时，从队列头部取下一个 ready thread 运行，直到它完成或阻塞。

中文理解：

> 谁先来，谁先排队；CPU 空了就拿队首线程。

---

### 2. Smallest number of threads first / 最少线程数优先

共享就绪队列按 priority queue / 优先队列组织。
优先调度那些 **unscheduled threads / 未调度线程数最少** 的 job。

中文理解：

> 哪个作业剩下没运行的线程少，就先让它跑，争取快点完成。

---

### 3. Preemptive smallest number of threads first / 抢占式最少线程数优先

仍然优先选择未调度线程数最少的 job。
如果一个新到达 job 的线程数比当前正在运行的 job 更少，它可以抢占当前线程。

中文理解：

> 新来的小作业可以抢占大作业。

英文背诵版：

> The three versions are FCFS, smallest number of threads first, and preemptive smallest number of threads first.

教材在 Load Sharing 部分明确分析了这三种版本。

---

# 相关题目 10.4

## 题目原文

**10.4. What is the difference between hard and soft real-time tasks?**

硬实时任务和软实时任务有什么区别？

---

## 答案

### Hard real-time task / 硬实时任务

必须在 deadline / 截止时间 前完成。
如果错过 deadline，会造成不可接受的损害，甚至系统致命错误。

例子：

> 飞机控制系统、汽车刹车控制、工业安全控制。

---

### Soft real-time task / 软实时任务

也有 deadline，但 deadline 不是强制性的。
即使错过 deadline，任务完成后仍然有意义，只是效果变差。

例子：

> 视频播放、在线会议、音频缓冲。

英文背诵版：

> A hard real-time task must meet its deadline, otherwise unacceptable damage or fatal error may occur. A soft real-time task has a desirable but not mandatory deadline; it is still useful even if completed late.

教材在 Real-Time Scheduling Background 部分说明，hard real-time task 必须满足 deadline，而 soft real-time task 的 deadline 是 desirable but not mandatory。

---

# 相关题目 10.5

## 题目原文

**10.5. What is the difference between periodic and aperiodic real-time tasks?**

周期性实时任务和非周期性实时任务有什么区别？

---

## 答案

### Periodic task / 周期性任务

任务按照固定周期反复出现。

可以描述为：

> once per period T
> 每 T 时间单位执行一次

或者：

> exactly T units apart
> 每次执行间隔正好是 T

例子：

> 每 10ms 读取一次传感器数据。

---

### Aperiodic task / 非周期性任务

任务不是固定周期出现，但它有 deadline。
它可能要求在某个时间前开始，或者在某个时间前完成。

例子：

> 用户点击按钮后，系统需要在 100ms 内响应。

英文背诵版：

> A periodic task occurs repeatedly at regular intervals. An aperiodic task does not occur at fixed intervals but has a deadline for starting or finishing.

教材指出，aperiodic task 有开始或完成时间约束，而 periodic task 的要求通常是 once per period T 或 exactly T units apart。

---

# 当前阶段要背的题目

现在最该背的是：

1. **10.1 五种 synchronization granularity**
2. **10.2 四种 thread scheduling techniques**
3. **10.3 三种 load sharing versions**
4. **10.4 hard vs soft real-time task**
5. **10.5 periodic vs aperiodic task**

其中最重要的是 **10.1–10.3**，因为它们和你刚复习的 **Multiprocessor and Multicore Scheduling** 最直接相关。
