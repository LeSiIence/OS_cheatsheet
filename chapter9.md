按第一页高亮，这次重点复习 **9.1 Types of Processor Scheduling（处理器调度类型）** 和 **9.2 Scheduling Algorithms（调度算法）**。本章是 **Uniprocessor Scheduling（单处理器调度）**，核心就是：**CPU 只有一个，多个进程怎么排队、怎么轮流用 CPU。** 

---

## 1. Processor Scheduling 是什么？

**Processor scheduling（处理器调度）**：决定在某个时刻，让哪个进程使用 CPU。

英文原意是：

> assign processes to be executed by the processor over time
> 随着时间推移，把进程分配给处理器执行。

多道程序系统中，多个进程同时在内存里，但 CPU 一次只能真正执行一个进程。所以调度的目标是：

| 英文                                 | 中文      | 含义               |
| ---------------------------------- | ------- | ---------------- |
| response time                      | 响应时间    | 用户发出请求到开始看到响应的时间 |
| throughput                         | 吞吐量     | 单位时间完成多少进程       |
| processor efficiency / utilization | CPU 利用率 | CPU 忙碌时间占总时间比例   |
| fairness                           | 公平性     | 不让某些进程一直等        |

---

## 2. 三种 Processor Scheduling

第一页高亮的 9.1 重点就是这三个：

| 类型                     | 英文   | 中文作用                      | 发生频率 | 关键词                                |
| ---------------------- | ---- | ------------------------- | ---- | ---------------------------------- |
| Long-term scheduling   | 长程调度 | 决定哪些作业进入系统，变成进程           | 最少   | admit / degree of multiprogramming |
| Medium-term scheduling | 中程调度 | 决定哪些进程换入/换出内存             | 中等   | swapping                           |
| Short-term scheduling  | 短程调度 | 决定 ready queue 中哪个进程下一个运行 | 最频繁  | dispatcher                         |

---

### 2.1 Long-Term Scheduling 长程调度

**Long-term scheduler** 决定：

> which programs are admitted to the system
> 哪些程序被允许进入系统执行。

也就是：**新作业能不能进来。**

它控制的是：

> degree of multiprogramming
> 多道程序程度

意思是系统中同时存在多少个进程。

如果进程太少，CPU 可能空闲；如果进程太多，每个进程分到的 CPU 时间变少，内存压力也变大。

考试常问：

**Q: Long-term scheduling controls what?**
A: It controls the **degree of multiprogramming**.
中文：长程调度控制系统中同时存在的进程数量。

---

### 2.2 Medium-Term Scheduling 中程调度

**Medium-term scheduler** 和 **swapping（交换）** 相关。

它决定：

> whether to add a process to those that are at least partially in main memory
> 是否把某个进程放入主存，使其有机会执行。

简单说就是：
**内存不够时，先把某些进程换出；之后再决定把哪些进程换回来。**

相关状态：

| 状态              | 中文   |
| --------------- | ---- |
| Ready           | 就绪   |
| Blocked         | 阻塞   |
| Ready/Suspend   | 就绪挂起 |
| Blocked/Suspend | 阻塞挂起 |

中程调度负责在 **suspend（挂起）** 和内存中的状态之间移动进程。

---

### 2.3 Short-Term Scheduling 短程调度

**Short-term scheduler** 又叫：

> dispatcher
> 分派器

它决定：

> which ready process to execute next
> 下一个执行哪个就绪进程。

这是第 9 章后面算法的核心。

它执行最频繁，因为很多事件都会触发它：

| 触发事件                   | 英文     | 中文              |
| ---------------------- | ------ | --------------- |
| Clock interrupts       | 时钟中断   | 时间片到了           |
| I/O interrupts         | I/O 中断 | I/O 完成了         |
| Operating system calls | 系统调用   | 进程请求 OS 服务      |
| Signals                | 信号     | 如 semaphore 信号量 |

记忆：

**长程调度管“进不进系统”，中程调度管“进不进内存”，短程调度管“谁用 CPU”。**

---

## 3. 9.2 Scheduling Algorithms 调度算法

这一节重点看短程调度，因为短程调度才真正决定 CPU 分给谁。

### 3.1 调度评价标准 Scheduling Criteria

常见指标：

| 英文                    | 中文      | 解释                |
| --------------------- | ------- | ----------------- |
| Turnaround time       | 周转时间    | 从提交到完成的总时间        |
| Response time         | 响应时间    | 从请求提交到第一次响应出现     |
| Throughput            | 吞吐量     | 单位时间完成的进程数        |
| Processor utilization | CPU 利用率 | CPU 忙碌时间比例        |
| Fairness              | 公平性     | 不能让进程饿死           |
| Predictability        | 可预测性    | 同样任务在不同负载下表现不要差太多 |
| Deadlines             | 截止时间    | 实时系统中很重要          |

最常考的是：

### Turnaround Time 周转时间

公式：

[
Turnaround\ Time = Finish\ Time - Arrival\ Time
]

中文：
**周转时间 = 完成时间 - 到达时间**

---

### Normalized Turnaround Time 归一化周转时间

书里很重视这个指标：

[
\frac{T_r}{T_s} = \frac{Turnaround\ Time}{Service\ Time}
]

其中：

| 符号    | 英文                               | 中文                 |
| ----- | -------------------------------- | ------------------ |
| (T_r) | turnaround time / residence time | 周转时间 / 驻留时间        |
| (T_s) | service time                     | 服务时间，即真正需要 CPU 的时间 |

它表示：
**一个进程实际待在系统里的时间，是它真正运行时间的多少倍。**

最小值是 1。

如果 (T_r/T_s = 1)，说明进程一到就运行，没有等待。
如果 (T_r/T_s = 5)，说明它在系统中待的时间是运行时间的 5 倍，等待很多。

---

## 4. Priority Scheduling 优先级调度

很多系统会给每个进程分配 **priority（优先级）**。

调度器总是选择更高优先级的进程。

但是注意书里的习惯：

> larger priority values represent lower priority processes
> 数值越大，优先级越低。

也就是说在 UNIX 这类系统里：

| priority 数值 | 实际优先级 |
| ----------- | ----- |
| 小           | 高     |
| 大           | 低     |

优先级调度的问题：

> starvation
> 饥饿

如果高优先级进程一直来，低优先级进程可能永远得不到 CPU。

解决办法：

> aging
> 老化

等待时间越长，逐渐提高优先级。

---

## 5. 两种 Decision Mode：是否可抢占

调度算法分两大类：

| 英文            | 中文   | 含义                    |
| ------------- | ---- | --------------------- |
| Nonpreemptive | 非抢占式 | 一个进程一旦运行，就一直运行到完成或阻塞  |
| Preemptive    | 抢占式  | OS 可以强行暂停当前进程，让别的进程运行 |

例子：

| 算法          | 是否抢占 |
| ----------- | ---- |
| FCFS        | 非抢占  |
| SPN         | 非抢占  |
| HRRN        | 非抢占  |
| Round Robin | 抢占   |
| SRT         | 抢占   |
| Feedback    | 抢占   |

抢占式优点：响应时间好，避免一个进程长期霸占 CPU。
抢占式缺点：上下文切换更多，overhead 更高。

---

## 6. 重点调度算法

### 6.1 FCFS / FIFO

**First-Come-First-Served**
先来先服务。

规则：

> The process that has been in the ready queue the longest is selected.
> 在就绪队列里等得最久的进程先运行。

优点：

简单，overhead 最低。

缺点：

对短进程不友好。
如果短进程刚好排在长进程后面，会等很久。

这叫：

> convoy effect
> 护航效应 / 车队效应

尤其会惩罚：

| 进程类型              | 影响                 |
| ----------------- | ------------------ |
| short process     | 短进程等待长进程           |
| I/O-bound process | I/O 型进程被 CPU 型进程拖慢 |

---

### 6.2 Round Robin 轮转调度

**Round Robin, RR** 是抢占式调度。

核心概念：

> time quantum / time slice
> 时间片

规则：

每个进程最多运行一个时间片。时间片用完后，如果还没完成，就回到 ready queue 队尾。

适合：

> time-sharing system
> 分时系统

优点：

响应时间好，比较公平。

缺点：

时间片大小很关键。

| 时间片大小 | 结果                 |
| ----- | ------------------ |
| 太小    | 上下文切换频繁，overhead 大 |
| 太大    | 退化成 FCFS           |

书里强调：

> If quantum is longer than the longest-running process, RR degenerates to FCFS.
> 如果时间片比所有进程都长，RR 就退化成 FCFS。

---

### 6.3 SPN

**Shortest Process Next**
最短进程优先。

规则：

选择 expected processing time 最短的进程。

它是：

> nonpreemptive
> 非抢占式

优点：

平均周转时间通常很好，对短进程友好。

缺点：

1. 需要估计服务时间
2. 长进程可能 starvation
3. 不适合交互式系统，因为它不抢占

---

### 6.4 SRT

**Shortest Remaining Time**
最短剩余时间优先。

它是 SPN 的抢占式版本。

规则：

总是运行剩余时间最短的进程。

如果新来的进程剩余时间比当前运行进程更短，就抢占当前进程。

优点：

短进程响应非常好，平均周转时间通常优秀。

缺点：

1. 仍然需要估计运行时间
2. 长进程可能 starvation
3. 要记录已经运行了多久，overhead 较高

---

### 6.5 HRRN

**Highest Response Ratio Next**
最高响应比优先。

公式：

[
R = \frac{w+s}{s}
]

其中：

| 符号  | 英文                    | 中文     |
| --- | --------------------- | ------ |
| (R) | response ratio        | 响应比    |
| (w) | waiting time          | 等待时间   |
| (s) | expected service time | 预计服务时间 |

展开看：

[
R = 1 + \frac{w}{s}
]

含义：

短作业因为 (s) 小，容易得到较高 (R)。
长作业等待久了，(w) 变大，也会逐渐得到较高 (R)。

所以 HRRN 是一个折中算法：

> favors shorter jobs, but also considers aging
> 偏向短作业，但也考虑等待时间，避免长作业饿死。

它是：

> nonpreemptive
> 非抢占式

---

### 6.6 Feedback Scheduling 反馈调度

**Feedback scheduling** 适用于不知道进程长度的情况。

核心思想：

> penalize jobs that have been running longer
> 惩罚运行时间更长的进程。

它使用多个 ready queues：

[
RQ_0, RQ_1, RQ_2, ...
]

规则：

1. 新进程先进入最高优先级队列 (RQ_0)。
2. 如果一个进程用完时间片还没完成，就降到下一级队列。
3. 短进程通常很快完成，不会降太多级。
4. 长进程会逐渐降到低优先级队列。

优点：

不用提前知道进程需要运行多久。
对短进程比较友好。

缺点：

长进程可能 starvation。

改进方法：

1. 低优先级队列给更长时间片，例如 (q=2^i)。
2. 等待太久的进程提升优先级，即 aging。

---

## 7. 几个算法对比记忆表

| 算法       | 英文全称                        | 抢占？ | 偏向谁                 | 是否可能饥饿 |
| -------- | --------------------------- | --- | ------------------- | ------ |
| FCFS     | First-Come-First-Served     | 否   | 长进程 / 先到进程          | 否      |
| RR       | Round Robin                 | 是   | 公平轮流                | 否      |
| SPN      | Shortest Process Next       | 否   | 短进程                 | 是      |
| SRT      | Shortest Remaining Time     | 是   | 剩余时间短的进程            | 是      |
| HRRN     | Highest Response Ratio Next | 否   | 短进程 + 等得久的进程        | 否      |
| Feedback | Feedback Scheduling         | 是   | 新进程 / 短进程 / I/O 型进程 | 可能     |

---

## 8. 这章最容易考的计算

给你一组进程：

| Process | Arrival Time | Service Time |
| ------- | -----------: | -----------: |
| A       |            0 |            3 |
| B       |            2 |            6 |
| C       |            4 |            4 |
| D       |            6 |            5 |
| E       |            8 |            2 |

然后让你用 FCFS / RR / SPN / SRT / HRRN 画甘特图，算：

[
Finish\ Time
]

[
Turnaround\ Time = Finish\ Time - Arrival\ Time
]

[
Normalized\ Turnaround\ Time = \frac{Turnaround\ Time}{Service\ Time}
]

特别注意：
**Arrival Time 不一定都是 0。**
算周转时间时一定要减 arrival time。

---

## 9. 本节考试重点总结

你这页高亮内容可以压缩成这几句话：

1. **Long-term scheduling** 决定哪些作业进入系统，控制 **degree of multiprogramming**。
2. **Medium-term scheduling** 与 **swapping** 有关，决定哪些进程换入主存。
3. **Short-term scheduling / dispatcher** 决定 ready queue 中哪个进程下一个运行。
4. 调度目标包括 **response time, throughput, processor utilization, fairness**。
5. 调度算法分为 **preemptive** 和 **nonpreemptive**。
6. **FCFS** 简单但惩罚短进程和 I/O-bound processes。
7. **RR** 使用 **time quantum**，时间片太大退化成 FCFS，太小 overhead 大。
8. **SPN/SRT** 偏向短进程，但可能让长进程 starvation。
9. **HRRN** 用 (R=(w+s)/s)，兼顾短进程和等待时间，能避免 starvation。
10. **Feedback** 用多级队列，不需要提前知道进程长度，但长进程可能被降级太多。







按第一页高亮范围，相关题主要是：**Review Questions 9.1–9.11**，以及 **Problems 9.1、9.2、9.3、9.4、9.5、9.6、9.11、9.14、9.15、9.16**。这些都对应 9.1 三类调度和 9.2 的 FCFS、RR、SPN、SRT、HRRN、Feedback 等调度算法。题目来自 Chapter 9 的 questions 文件；教材高亮页也明确本节重点是 processor scheduling types 和 scheduling algorithms。  

---

# 一、Review Questions 9.1–9.11

| 题号                                                                                 | 答案                                                                                                                                                                                                     |
| ---------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **9.1** Briefly describe the three types of processor scheduling.                  | **Long-term scheduling** 决定哪些作业进入系统，控制 **degree of multiprogramming**；**Medium-term scheduling** 决定哪些进程换入/换出主存，与 **swapping** 有关；**Short-term scheduling / dispatcher** 决定 ready queue 中哪个进程下一个使用 CPU。 |
| **9.2** What is usually the critical performance requirement in an interactive OS? | **Response time 响应时间**。交互式系统最重要的是用户操作后多久能看到响应。                                                                                                                                                         |
| **9.3** Turnaround time 和 response time 区别？                                        | **Turnaround time 周转时间**：从提交到完成的总时间。**Response time 响应时间**：从提交请求到第一次响应开始出现的时间。                                                                                                                         |
| **9.4** Low-priority value 是低优先级还是高优先级？                                            | 在本书和 UNIX 习惯里，**priority 数值越小，实际优先级越高**。但有些系统如 Windows 相反。                                                                                                                                             |
| **9.5** Preemptive 和 nonpreemptive 区别？                                             | **Nonpreemptive 非抢占式**：进程运行后，直到完成或阻塞才让出 CPU。**Preemptive 抢占式**：OS 可以中断正在运行的进程，把 CPU 分给别的进程。                                                                                                            |
| **9.6** FCFS 是什么？                                                                  | **First-Come-First-Served**，先来先服务。ready queue 中等待最久的进程先运行。                                                                                                                                             |
| **9.7** Round Robin 是什么？                                                           | **RR 轮转调度**，每个进程获得一个 **time quantum / time slice 时间片**，时间片用完还没结束就回到 ready queue 队尾。                                                                                                                    |
| **9.8** SPN 是什么？                                                                   | **Shortest Process Next**，非抢占式，选择预计服务时间最短的进程。                                                                                                                                                          |
| **9.9** SRT 是什么？                                                                   | **Shortest Remaining Time**，SPN 的抢占式版本，总是运行剩余时间最短的进程。                                                                                                                                                  |
| **9.10** HRRN 是什么？                                                                 | **Highest Response Ratio Next**，选择最大 (R=\frac{w+s}{s}) 的进程。它既偏向短作业，也会让等待很久的长作业优先级升高。                                                                                                                   |
| **9.11** Feedback scheduling 是什么？                                                  | **反馈调度 / multilevel feedback queue**：新进程进入高优先级队列，运行越久越可能被降到低优先级队列，从而偏向短进程和 I/O-bound process。                                                                                                          |

---

# 二、Problem 9.1

题目给出：

| Process | Burst Time | Priority | Arrival Time |
| ------- | ---------: | -------: | -----------: |
| P1      |      50 ms |        4 |         0 ms |
| P2      |      20 ms |        1 |        20 ms |
| P3      |     100 ms |        3 |        40 ms |
| P4      |      40 ms |        2 |        60 ms |

公式：

[
Waiting\ Time = Finish\ Time - Arrival\ Time - Burst\ Time
]

---

## 9.1(a) SRT：Shortest Remaining Time

调度过程：

[
0\text{-}20:P1,\quad 20\text{-}40:P2,\quad 40\text{-}70:P1,\quad 70\text{-}110:P4,\quad 110\text{-}210:P3
]

| Process | Finish |         Waiting |
| ------- | -----: | --------------: |
| P1      |     70 |    (70-0-50=20) |
| P2      |     40 |    (40-20-20=0) |
| P3      |    210 | (210-40-100=70) |
| P4      |    110 |  (110-60-40=10) |

[
Average\ Waiting\ Time=\frac{20+0+70+10}{4}=25ms
]

---

## 9.1(a) Nonpreemptive Priority

题目说明：**smaller priority number implies higher priority**，所以优先级顺序是 P2 > P4 > P3 > P1。
但是非抢占式，所以 P1 一旦在 0ms 开始运行，就先跑完。

调度过程：

[
0\text{-}50:P1,\quad 50\text{-}70:P2,\quad 70\text{-}110:P4,\quad 110\text{-}210:P3
]

| Process | Finish |         Waiting |
| ------- | -----: | --------------: |
| P1      |     50 |     (50-0-50=0) |
| P2      |     70 |   (70-20-20=30) |
| P3      |    210 | (210-40-100=70) |
| P4      |    110 |  (110-60-40=10) |

[
Average\ Waiting\ Time=\frac{0+30+70+10}{4}=27.5ms
]

---

## 9.1(a) Round Robin，q = 30ms

调度过程：

[
0\text{-}30:P1,\quad 30\text{-}50:P2,\quad 50\text{-}70:P1,\quad 70\text{-}100:P3,
]
[
100\text{-}130:P4,\quad 130\text{-}160:P3,\quad 160\text{-}170:P4,\quad 170\text{-}210:P3
]

| Process | Finish |         Waiting |
| ------- | -----: | --------------: |
| P1      |     70 |    (70-0-50=20) |
| P2      |     50 |   (50-20-20=10) |
| P3      |    210 | (210-40-100=70) |
| P4      |    170 |  (170-60-40=70) |

[
Average\ Waiting\ Time=\frac{20+10+70+70}{4}=42.5ms
]

---

# 三、Problem 9.2

题目给出：

| Process | Arrival | Processing Time |
| ------- | ------: | --------------: |
| A       |       0 |               3 |
| B       |       1 |               5 |
| C       |       3 |               2 |
| D       |       9 |               5 |
| E       |      12 |               5 |

公式：

[
Turnaround\ Time = Finish\ Time - Arrival\ Time
]

[
Normalized\ Turnaround\ Time = \frac{Turnaround\ Time}{Processing\ Time}
]

核心结果如下：

| Algorithm  | Gantt 简写                                                        | Finish Time A B C D E | Mean TAT | Mean (T_r/T_s) |
| ---------- | --------------------------------------------------------------- | --------------------- | -------: | -------------: |
| **FCFS**   | A 0-3, B 3-8, C 8-10, D 10-15, E 15-20                          | 3, 8, 10, 15, 20      |      6.2 |           1.74 |
| **RR q=1** | 轮流每次 1 单位                                                       | 6, 11, 8, 18, 20      |      7.6 |           1.98 |
| **RR q=4** | A 0-3, B 3-7, C 7-9, B 9-10, D 10-14, E 14-18, D 18-19, E 19-20 | 3, 10, 9, 19, 20      |      7.2 |           1.88 |
| **SPN**    | A 0-3, C 3-5, B 5-10, D 10-15, E 15-20                          | 3, 10, 5, 15, 20      |      5.6 |           1.32 |
| **SRT**    | 本题结果同 SPN                                                       | 3, 10, 5, 15, 20      |      5.6 |           1.32 |
| **HRRN**   | A 0-3, B 3-8, C 8-10, D 10-15, E 15-20                          | 3, 8, 10, 15, 20      |      6.2 |           1.74 |

注意：本题里 **SPN 和 SRT 结果相同**，因为没有出现“新来的短进程抢占当前长进程”的情况。
Feedback scheduling 有多个教材约定，若考试要算，一般会明确给出队列降级、抢占和时间片规则。

---

# 四、Problem 9.3

题目：证明在所有作业同时到达、非抢占式调度中，**SPN / SJF 能最小化平均等待时间**。

证明思路：用 **exchange argument 交换论证**。

假设有两个相邻作业 (i) 和 (j)，服务时间分别是：

[
s_i \le s_j
]

如果顺序是 (i \rightarrow j)，设前面已经用了时间 (T)，那么这两个作业贡献的等待时间是：

[
T + (T+s_i)=2T+s_i
]

如果反过来 (j \rightarrow i)，等待时间是：

[
T + (T+s_j)=2T+s_j
]

因为：

[
s_i \le s_j
]

所以：

[
2T+s_i \le 2T+s_j
]

也就是说，**短作业放在长作业前面不会增加总等待时间，只会减少或相等**。

不断把“长作业在短作业前面”的逆序交换掉，最后得到按服务时间从短到长排序，也就是 **SPN / SJF**。因此，SPN 最小化平均等待时间。

---

# 五、Problem 9.4

Burst-time pattern：

[
6,\ 4,\ 6,\ 4,\ 13,\ 13,\ 13
]

Initial guess：

[
S_1=10
]

指数平均公式：

[
S_{n+1}=\alpha T_n+(1-\alpha)S_n
]

| Step     | Observed (T_n) | Simple Average | (\alpha=0.5) | (\alpha=0.8) |
| -------- | -------------: | -------------: | -----------: | -----------: |
| (S_1)    |        initial |         10.000 |       10.000 |       10.000 |
| after 6  |              6 |          6.000 |        8.000 |        6.800 |
| after 4  |              4 |          5.000 |        6.000 |        4.560 |
| after 6  |              6 |          5.333 |        6.000 |        5.712 |
| after 4  |              4 |          5.000 |        5.000 |        4.342 |
| after 13 |             13 |          6.600 |        9.000 |       11.268 |
| after 13 |             13 |          7.667 |       11.000 |       12.654 |
| after 13 |             13 |          8.429 |       12.000 |       12.931 |

结论：
(\alpha) 越大，越相信最近一次 burst，所以变化更快；(\alpha) 越小，估计更平滑，但反应更慢。教材也强调较大的 (\alpha) 会更快反映近期变化，但也更容易产生剧烈波动。

---

# 六、Problem 9.5

题目问 (\alpha) 和 (\beta) 的作用。

公式核心是：

[
S_{n+1}=\alpha T_n+(1-\alpha)S_n
]

[
X_{n+1}=\min(Ubound,\max(Lbound,\beta S_{n+1}))
]

其中 (X_{n+1}) 才是真正给 SPN 使用的估计值。

答案：

| 参数       | 作用                   | 值较大时                               | 值较小时                               |
| -------- | -------------------- | ---------------------------------- | ---------------------------------- |
| (\alpha) | 控制指数平均对最近 burst 的敏感度 | 更重视最近一次，反应快，但波动大                   | 更重视历史平均，反应慢，但稳定                    |
| (\beta)  | 对预测值整体放大或缩小          | (\beta>1)：估计偏大，进程看起来更长，不容易被 SPN 选中 | (\beta<1)：估计偏小，进程看起来更短，更容易被 SPN 选中 |
| (Ubound) | 上限                   | 防止估计过大                             | —                                  |
| (Lbound) | 下限                   | 防止估计过小                             | —                                  |

---

# 七、Problem 9.6

题目问：Figure 9.5 底部 feedback (q=2^i) 中，为什么一种情况 A 运行 2 个时间单位后交给 B，另一种情况 A 可以运行 3 个时间单位？

关键差异是：

**新到达的高优先级进程是否能立即抢占当前低优先级队列中的进程。**

解释：

A 一开始运行后会被降到较低优先级队列。
B 到达时，如果系统规定：

1. **高优先级新进程可以立即抢占低优先级运行进程**，那么 A 运行 2 个时间单位后就让给 B。
2. **只有当前时间片用完才重新调度**，那么 A 可以继续跑完当前时间片，也就是可能运行到 3 个时间单位后才让给 B。

---

# 八、Problem 9.11

题目：RR ready queue 里放的是 PCB 指针，如果同一个进程放两个指针会怎样？

## (a)

同一个进程在 ready queue 中出现两次，相当于它在一轮 RR 中能获得两次时间片。

所以它获得的 CPU 份额变大。

## (b)

主要优点：

可以实现 **weighted round robin 加权轮转调度**。

比如：

* 一个进程放 1 个指针：每轮 1 个时间片
* 一个进程放 2 个指针：每轮 2 个时间片
* 一个进程放 3 个指针：每轮 3 个时间片

这相当于用很简单的方式实现不同权重或不同优先级。

## (c)

不用重复指针，也可以给 PCB 加一个字段：

[
weight
]

调度时让进程每轮运行 (weight) 个时间片，或者直接给它：

[
effective\ quantum = weight \times base\ quantum
]

---

# 九、Problem 9.14

题目：Multilevel feedback queue 一般偏向 CPU-bound 还是 I/O-bound？

答案：一般偏向 **I/O-bound process**。

原因：

* **I/O-bound process** 通常 CPU burst 很短，还没用完整个时间片就去等待 I/O 了，所以不容易被降级。
* **Processor-bound / CPU-bound process** 经常用完整个时间片，所以会被逐渐降到低优先级队列。

所以 multilevel feedback queue 通常会让 I/O 型、交互型进程获得较好的响应时间。

---

# 十、Problem 9.15

题目：在静态优先级调度中，为什么 Dekker’s solution 可能危险？

核心问题：**priority inversion / starvation 导致忙等无法结束**。

假设：

1. 低优先级进程 (P_L) 进入临界区，或者设置了 Dekker 算法中的 flag。
2. 此时高优先级进程 (P_H) 想进入临界区。
3. (P_H) 发现 (P_L) 还没退出，于是开始 busy waiting。
4. 由于 (P_H) 优先级更高，调度器一直让 (P_H) 运行。
5. (P_L) 因为优先级低，得不到 CPU，无法退出临界区。
6. (P_H) 又一直等 (P_L) 退出。

结果就是：
**高优先级进程一直忙等，低优先级进程无法运行，系统卡住。**

这就是为什么 busy waiting 的互斥算法在优先级调度系统中可能很危险。

---

# 十一、Problem 9.16

五个 batch jobs 同时到达：

| Job | Running Time | Priority |
| --- | -----------: | -------: |
| A   |           15 |        6 |
| B   |            9 |        3 |
| C   |            3 |        7 |
| D   |            6 |        9 |
| E   |           12 |        4 |

所有作业同时到达，所以：

[
Turnaround\ Time = Finish\ Time
]

---

## (a) Round Robin，q = 1

初始顺序 A, B, C, D, E。

| Job | Turnaround Time |
| --- | --------------: |
| A   |              45 |
| B   |              35 |
| C   |              13 |
| D   |              26 |
| E   |              42 |

[
Average=\frac{45+35+13+26+42}{5}=32.2
]

---

## (b) Priority Scheduling

优先级数值越小，优先级越高。

顺序：

[
B \rightarrow E \rightarrow A \rightarrow C \rightarrow D
]

| Job | Turnaround Time |
| --- | --------------: |
| A   |              36 |
| B   |               9 |
| C   |              39 |
| D   |              45 |
| E   |              21 |

[
Average=\frac{36+9+39+45+21}{5}=30
]

---

## (c) FCFS

题目指定顺序：

[
A \rightarrow B \rightarrow C \rightarrow D \rightarrow E
]

| Job | Turnaround Time |
| --- | --------------: |
| A   |              15 |
| B   |              24 |
| C   |              27 |
| D   |              33 |
| E   |              45 |

[
Average=\frac{15+24+27+33+45}{5}=28.8
]

---

## (d) Shortest Job First

按照运行时间从短到长：

[
C \rightarrow D \rightarrow B \rightarrow E \rightarrow A
]

| Job | Turnaround Time |
| --- | --------------: |
| A   |              45 |
| B   |              18 |
| C   |               3 |
| D   |               9 |
| E   |              30 |

[
Average=\frac{45+18+3+9+30}{5}=21
]

---

# 本次最该背的题

最重要的是：

1. **Review 9.1–9.11**：概念题，高频。
2. **Problem 9.1**：SRT、priority、RR 算 waiting time。
3. **Problem 9.2**：照 Table 9.5 算 finish time、turnaround time、normalized turnaround time。
4. **Problem 9.3**：证明 SJF/SPN 最小平均等待时间。
5. **Problem 9.14**：Feedback 偏向 I/O-bound process。
6. **Problem 9.16**：RR、Priority、FCFS、SJF 算 turnaround time。
