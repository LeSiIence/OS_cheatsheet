下面按**第一页高亮区域**来复习，重点是：**Process / PCB、Process States、Five-State Model、Suspended Processes、Process Control、Process Switching**。内容来自你上传的 3.1–3.4 课件/教材页。

## 1. Process 与 PCB

## 1. 进程与进程控制块

### Process / 进程

英文原文常见定义：

> **A process is a program in execution.**
> 进程是正在执行的程序。

也可以理解为：

| English                                                                                   | 中文解释                   |
| ----------------------------------------------------------------------------------------- | ---------------------- |
| **An instance of a program running on a computer**                                        | 程序在计算机上运行的一次实例         |
| **The entity that can be assigned to and executed on a processor**                        | 可以被分配给处理器并执行的实体        |
| **A unit of activity characterized by instructions, current state, and system resources** | 由指令序列、当前状态和系统资源组成的活动单位 |

重点理解：

**Program / 程序**是静态的代码文件。
**Process / 进程**是程序运行起来之后的动态实体。

例如：你打开两次同一个浏览器程序，程序代码可能一样，但会形成两个不同的进程，因为它们有不同的状态、内存、寄存器信息和资源。

---

### Process Control Block, PCB / 进程控制块

**PCB = Process Control Block / 进程控制块**

这是操作系统管理进程最核心的数据结构。

可以这样记：

> **Process = Program code + Data + PCB**
> 进程 = 程序代码 + 数据 + 进程控制块

PCB 里通常保存：

| English term               | 中文                            |
| -------------------------- | ----------------------------- |
| **Identifier**             | 进程标识符，例如 PID                  |
| **State**                  | 进程状态，例如 Running、Ready、Blocked |
| **Priority**               | 优先级                           |
| **Program Counter, PC**    | 程序计数器，保存下一条要执行指令的地址           |
| **Memory Pointers**        | 内存指针，指向代码、数据、共享内存等            |
| **Context Data**           | 上下文数据，主要是寄存器内容                |
| **I/O Status Information** | I/O 状态信息，例如正在等待的 I/O、打开的文件    |
| **Accounting Information** | 统计信息，例如 CPU 时间、运行时间、账号信息      |

最重要的一句话：

> **The PCB contains sufficient information so that it is possible to interrupt a running process and later resume execution as if the interruption had not occurred.**
> PCB 保存了足够的信息，使得一个正在运行的进程被中断后，将来还能像没有被中断过一样继续执行。

考试常考点：

**为什么 PCB 重要？**
因为操作系统要实现多道程序设计 / multiprogramming，就必须能暂停一个进程、保存现场、切换到另一个进程、再恢复原来的进程。这个“保存现场”的核心就是 PCB。

---

## 2. Process States

## 2. 进程状态

操作系统不是只知道“进程存在”，还要知道它现在处于什么状态。

最开始可以用二状态模型：

| English         | 中文   |
| --------------- | ---- |
| **Running**     | 正在运行 |
| **Not Running** | 没有运行 |

但二状态模型太粗糙。因为 **Not Running** 里面有两种完全不同的情况：

一种是：进程已经准备好，只是等 CPU。
另一种是：进程在等 I/O，给它 CPU 也运行不了。

所以后面引出更重要的模型：**Five-State Process Model / 五状态进程模型**。

---

## 3. Five-State Process Model

## 3. 五状态进程模型

五个状态是本章核心。

| English state         | 中文状态      | 含义                       |
| --------------------- | --------- | ------------------------ |
| **New**               | 新建态       | 进程刚被创建，但还没有被操作系统接纳为可执行进程 |
| **Ready**             | 就绪态       | 进程已经准备好，只要获得 CPU 就可以运行   |
| **Running**           | 运行态       | 进程正在 CPU 上执行             |
| **Blocked / Waiting** | 阻塞态 / 等待态 | 进程正在等待某个事件，例如 I/O 完成     |
| **Exit**              | 退出态       | 进程已经结束或被终止               |

重点区分：

**Ready / 就绪**：缺 CPU。
**Blocked / 阻塞**：缺事件，例如 I/O 还没完成。
这两个状态都不是 Running，但含义完全不同。

---

### 五状态之间的转换

| Transition            | 中文解释      | 触发原因                           |
| --------------------- | --------- | ------------------------------ |
| **Null → New**        | 无 → 新建态   | 创建新进程                          |
| **New → Ready**       | 新建态 → 就绪态 | OS 接纳该进程，称为 **Admit**          |
| **Ready → Running**   | 就绪态 → 运行态 | 调度器选择它运行，称为 **Dispatch**       |
| **Running → Ready**   | 运行态 → 就绪态 | 时间片用完，或被更高优先级进程抢占              |
| **Running → Blocked** | 运行态 → 阻塞态 | 请求 I/O 或等待事件，称为 **Event wait** |
| **Blocked → Ready**   | 阻塞态 → 就绪态 | 等待的事件发生，称为 **Event occurs**    |
| **Running → Exit**    | 运行态 → 退出态 | 正常结束或异常终止，称为 **Release**       |

### 重点例子

如果一个进程正在运行，突然发起磁盘读操作：

**Running → Blocked**
因为磁盘 I/O 很慢，进程必须等待。

当磁盘读完：

**Blocked → Ready**
注意不是直接变成 Running。它只是重新具备运行条件，还要等调度器分配 CPU。

---

## 4. Queueing Model

## 4. 队列模型

操作系统通常用队列管理不同状态的进程。

| Queue                       | 中文              |
| --------------------------- | --------------- |
| **Ready Queue**             | 就绪队列            |
| **Blocked Queue**           | 阻塞队列            |
| **Multiple Blocked Queues** | 多个阻塞队列，每个事件一个队列 |

为什么要多个阻塞队列？

如果所有阻塞进程都放在一个队列里，当某个 I/O 完成时，操作系统需要扫描整个阻塞队列，找出等这个事件的进程。这样效率低。

更好的做法：

> **One queue for each event.**
> 每个事件对应一个队列。

例如：

磁盘 I/O 队列、键盘输入队列、网络接收队列。

当某个事件发生时，直接把对应队列里的进程移到 Ready Queue。

---

## 5. Suspended Processes

## 5. 挂起进程

### 为什么需要 Suspended / 挂起？

核心原因是：

> **Main memory is limited.**
> 主存有限。

如果内存中的进程都在等待 I/O，那么 CPU 会空闲。为了提高效率，操作系统可以把某些进程从主存移到磁盘，腾出内存给其他可运行进程。

这个过程叫：

| English      | 中文        |
| ------------ | --------- |
| **Swapping** | 换入换出 / 交换 |
| **Swap out** | 从主存换出到磁盘  |
| **Swap in**  | 从磁盘换入主存   |
| **Suspend**  | 挂起        |

---

### Blocked 和 Suspended 的区别

这是易错点。

| 状态                  | 是否在主存？ | 是否能立即运行？ | 卡在哪里？         |
| ------------------- | ------ | -------- | ------------- |
| **Blocked**         | 在主存    | 不能       | 等事件           |
| **Ready**           | 在主存    | 能，只缺 CPU | 等 CPU         |
| **Blocked/Suspend** | 在磁盘    | 不能       | 既不在主存，也在等事件   |
| **Ready/Suspend**   | 在磁盘    | 暂时不能     | 事件已满足，但还没换回主存 |

最关键的一句话：

**Blocked / 阻塞**表示进程在等事件。
**Suspended / 挂起**表示进程不在主存，不能立即执行。

所以它们是两个不同维度：

| Dimension                      | 中文      |
| ------------------------------ | ------- |
| **Blocked or not blocked**     | 是否在等待事件 |
| **Suspended or not suspended** | 是否被换出主存 |

因此会出现 2 × 2 状态：

| English             | 中文               |
| ------------------- | ---------------- |
| **Ready**           | 在主存，且可运行         |
| **Blocked**         | 在主存，但等事件         |
| **Ready/Suspend**   | 在磁盘，事件已满足，换入后可运行 |
| **Blocked/Suspend** | 在磁盘，且还在等事件       |

---

### Suspended 相关转换

| Transition                          | 中文解释             |
| ----------------------------------- | ---------------- |
| **Blocked → Blocked/Suspend**       | 阻塞进程被换出主存        |
| **Blocked/Suspend → Ready/Suspend** | 等待的事件发生，但进程仍在磁盘  |
| **Ready/Suspend → Ready**           | 进程被换入主存，可以等待 CPU |
| **Ready → Ready/Suspend**           | 就绪进程被换出主存        |
| **Blocked/Suspend → Blocked**       | 虽然还没等到事件，但被换回主存  |

---

### 挂起的原因 Reasons for Suspension

| English                      | 中文解释               |
| ---------------------------- | ------------------ |
| **Swapping**                 | 为了释放主存空间           |
| **Other OS reason**          | 操作系统因为监控、故障等原因挂起进程 |
| **Interactive user request** | 用户为了调试程序手动挂起       |
| **Timing**                   | 周期性进程暂时不用，先挂起      |
| **Parent process request**   | 父进程要求挂起子进程         |

记忆方法：

**挂起 = 暂时不让它执行。**
不一定是因为它在等 I/O，也可能是因为调试、内存不足、父进程控制等原因。

---

## 6. Process Control

## 6. 进程控制

进程控制主要讲操作系统如何创建、管理、切换进程。

第一页高亮了 **Process Control / 进程控制** 和 **Process Switching / 进程切换**，但理解进程切换之前，要先知道执行模式。

---

## 7. Modes of Execution

## 7. 执行模式

大多数处理器至少支持两种执行模式：

| English                                      | 中文              |
| -------------------------------------------- | --------------- |
| **User Mode**                                | 用户态             |
| **Kernel Mode / System Mode / Control Mode** | 内核态 / 系统态 / 控制态 |

### User Mode / 用户态

普通用户程序运行在用户态。

用户态权限低，不能直接执行危险指令，例如：

* 直接访问 I/O 设备
* 修改控制寄存器
* 修改内存管理结构
* 直接访问操作系统内核区域

### Kernel Mode / 内核态

操作系统核心代码运行在内核态。

内核态权限高，可以执行特权指令。

| English                      | 中文                  |
| ---------------------------- | ------------------- |
| **Privileged instructions**  | 特权指令                |
| **Program Status Word, PSW** | 程序状态字               |
| **Mode bit**                 | 模式位，用来表示当前是用户态还是内核态 |

为什么需要用户态和内核态？

> **To protect the OS and key system tables such as PCBs from user programs.**
> 为了保护操作系统和关键系统表，例如 PCB，不被用户程序破坏。

---

## 8. Process Creation

## 8. 进程创建

操作系统创建进程时，一般做这些步骤：

| Step | English                                    | 中文                                      |
| ---- | ------------------------------------------ | --------------------------------------- |
| 1    | **Assign a unique process identifier**     | 分配唯一进程标识符 PID                           |
| 2    | **Allocate space for the process**         | 分配进程空间，包括程序、数据、栈、PCB                    |
| 3    | **Initialize the PCB**                     | 初始化 PCB                                 |
| 4    | **Set linkages**                           | 把进程加入 Ready Queue 或 Ready/Suspend Queue |
| 5    | **Create or expand other data structures** | 创建或扩展其他数据结构，例如 accounting 信息            |

PCB 初始化时常见内容：

* 进程 ID
* 父进程 ID
* 程序计数器 PC，通常设为程序入口地址
* 栈指针
* 初始状态，例如 **Ready** 或 **Ready/Suspend**
* 优先级
* 初始资源信息

---

## 9. Process Switching

## 9. 进程切换

### Process Switching 是什么？

> **Process switching means the OS stops the currently running process and starts another process.**
> 进程切换是指操作系统停止当前运行进程，转而运行另一个进程。

这个过程表面看很简单，但实际涉及：

1. 什么事件会触发切换？
2. 模式切换和进程切换有什么区别？
3. 操作系统要保存和恢复哪些数据结构？

---

### 触发进程切换的机制

教材中列了三个机制：

| Mechanism           | 中文            | Cause        | Use      |
| ------------------- | ------------- | ------------ | -------- |
| **Interrupt**       | 中断            | 外部事件，与当前指令无关 | 响应异步外部事件 |
| **Trap**            | 陷阱 / 异常       | 当前指令执行导致     | 处理错误或异常  |
| **Supervisor Call** | 管理程序调用 / 系统调用 | 程序显式请求       | 调用操作系统功能 |

---

### Interrupt / 中断

中断来自当前进程外部。

例子：

| English             | 中文解释                 |
| ------------------- | -------------------- |
| **Clock interrupt** | 时钟中断，判断时间片是否用完       |
| **I/O interrupt**   | I/O 中断，表示某个 I/O 操作完成 |
| **Memory fault**    | 内存故障，例如访问的虚拟页不在主存中   |

#### Clock interrupt / 时钟中断

如果当前进程时间片用完：

**Running → Ready**

然后调度器选择另一个 Ready 进程运行。

#### I/O interrupt / I/O 中断

如果某个 I/O 完成，操作系统会把等待这个 I/O 的进程从：

**Blocked → Ready**

如果这个新 Ready 的进程优先级更高，可能会抢占当前进程。

---

### Trap / 陷阱

Trap 是当前进程自己执行指令时引发的异常。

例如：

* 非法访问文件
* 除零错误
* 非法指令
* 越界访问内存

Trap 可能导致进程终止：

**Running → Exit**

也可能由操作系统处理后继续执行。

---

### Supervisor Call / 系统调用

用户程序不能直接执行特权操作，所以要通过系统调用请求操作系统帮忙。

例如：

* 读文件
* 写文件
* 创建进程
* 申请内存
* 发起 I/O

系统调用会让 CPU 从：

**User Mode → Kernel Mode**

但注意：

**Mode switching / 模式切换** 不一定等于 **Process switching / 进程切换**。

---

## 10. Mode Switching vs Process Switching

## 10. 模式切换 vs 进程切换

这是考试容易混的点。

| English               | 中文                | 是否换进程？ |
| --------------------- | ----------------- | ------ |
| **Mode switching**    | 模式切换，用户态 ↔ 内核态    | 不一定    |
| **Process switching** | 进程切换，从一个进程换到另一个进程 | 一定     |

例子：

用户进程调用 `read()` 读文件：

1. 进入内核态：**User Mode → Kernel Mode**
2. 如果数据已经准备好，内核处理完返回原进程
   这只是 **mode switch**，不一定发生 process switch。
3. 如果需要等待磁盘 I/O，当前进程阻塞
   **Running → Blocked**
4. 操作系统调度另一个进程
   这时才发生 **process switch**。

---

## 11. 本章高亮区域总复习框架

可以按下面这条线记：

> **Process → PCB → Process States → Five-State Model → Suspended States → Process Control → Process Switching**

中文理解：

> 操作系统为了管理程序运行，把运行中的程序抽象成进程；
> 每个进程用 PCB 保存状态和上下文；
> 进程会在 New、Ready、Running、Blocked、Exit 之间转换；
> 为了解决内存不足和 CPU 空闲问题，引入 Suspended 状态；
> 最后，操作系统通过中断、陷阱、系统调用获得控制权，并完成进程切换。

---

## 12. 考试速记版

### Process

**Process = program in execution**
进程 = 正在执行的程序。

### PCB

**PCB stores all information needed to manage and resume a process.**
PCB 保存操作系统管理和恢复进程所需的信息。

### Ready vs Blocked

**Ready**：只差 CPU。
**Blocked**：在等事件，给 CPU 也不能运行。

### Five states

**New, Ready, Running, Blocked, Exit**
新建、就绪、运行、阻塞、退出。

### Suspended

**Suspended means not immediately available for execution.**
挂起表示暂时不能立即执行，通常因为被换出主存。

### Ready/Suspend vs Blocked/Suspend

**Ready/Suspend**：事件已经满足，但还在磁盘。
**Blocked/Suspend**：还在磁盘，也还在等事件。

### Process switching triggers

**Interrupt, Trap, Supervisor Call**
中断、陷阱、系统调用。

### User mode vs Kernel mode

**User mode** 权限低。
**Kernel mode** 权限高，可以执行特权指令。

---

## 13. 典型问答

### Q1: Why is PCB important?

### 为什么 PCB 重要？

Because it stores the process state, program counter, registers, memory information, I/O status, and other information needed to stop and later resume a process.

因为 PCB 保存了进程状态、程序计数器、寄存器、内存信息、I/O 状态等内容，使操作系统能够暂停进程并在之后恢复它。

---

### Q2: What is the difference between Ready and Blocked?

### Ready 和 Blocked 有什么区别？

Ready means the process can run as soon as it gets the CPU.

Ready 表示进程已经准备好，只要获得 CPU 就能运行。

Blocked means the process is waiting for some event, such as I/O completion.

Blocked 表示进程正在等待某个事件，例如 I/O 完成，因此即使给它 CPU 也不能继续执行。

---

### Q3: Why introduce suspended states?

### 为什么引入挂起状态？

Because main memory is limited, and the OS may need to swap processes out to disk to free memory and improve CPU utilization.

因为主存有限，操作系统可能需要把进程换出到磁盘，释放主存空间，从而提高 CPU 利用率。

---

### Q4: Is a mode switch always a process switch?

### 模式切换一定是进程切换吗？

No.

不是。

A mode switch only changes between user mode and kernel mode. The same process may continue running afterward.

模式切换只是用户态和内核态之间的切换，之后可能还是同一个进程继续运行。

A process switch means the CPU changes from one process to another.

进程切换则表示 CPU 从一个进程切换到另一个进程。



我按你第一页高亮的范围筛题：**Process / PCB、Process States、Five-State Model、Suspended Processes、Process Control、Process Switching**。这些对应题目页里的 Review Questions 3.1–3.14 和 Problems 3.1–3.5、3.7–3.12；其中 3.6、3.7、3.8 是 VAX/VMS 扩展题，和进程状态、执行模式相关，但不是最核心考点。题目来源见 Chapter 3 questions。

---

# 一、Review Questions 相关题解答

## 3.1 What is an instruction trace?

## 什么是指令轨迹？

**Instruction trace / 指令轨迹**：一个进程在执行过程中实际执行的指令序列。

也就是：

> **A trace is a listing of the sequence of instructions that execute for a process.**
> trace 是某个进程执行过的指令顺序列表。

---

## 3.2 What common events lead to the creation of a process?

## 哪些事件会导致进程创建？

常见四种：

| English                                | 中文                                            |
| -------------------------------------- | --------------------------------------------- |
| **New batch job**                      | 新的批处理作业提交                                     |
| **Interactive log-on**                 | 用户交互式登录                                       |
| **Created by OS to provide a service** | 操作系统为了提供服务而创建进程，例如打印服务                        |
| **Spawned by existing process**        | 由已有进程派生出新进程，即 parent process 创建 child process |

---

## 3.3 For Figure 3.6, briefly define each state.

## 简述五状态模型中的每个状态。

| State                 | 中文        | 含义                     |
| --------------------- | --------- | ---------------------- |
| **New**               | 新建态       | 进程刚创建，但还没被 OS 接纳为可执行进程 |
| **Ready**             | 就绪态       | 已准备好，只差 CPU            |
| **Running**           | 运行态       | 正在 CPU 上执行             |
| **Blocked / Waiting** | 阻塞态 / 等待态 | 正在等待某事件，例如 I/O 完成      |
| **Exit**              | 退出态       | 进程已经结束或被终止             |

重点区分：

**Ready = 缺 CPU**
**Blocked = 缺事件**

---

## 3.4 What does it mean to preempt a process?

## 抢占一个进程是什么意思？

**Preempt / 抢占**：操作系统在一个进程还没主动放弃 CPU 的时候，强行收回 CPU，让另一个进程运行。

英文表达：

> **Preemption is reclaiming the processor from a process before it has finished using it.**
> 抢占就是在进程还没用完处理器前，OS 把处理器收回来。

常见原因：

* 时间片用完，**time slice expires**
* 更高优先级进程变为 Ready
* 时钟中断触发调度

---

## 3.5 What is swapping and what is its purpose?

## 什么是交换？目的是什么？

**Swapping / 交换**：把一个进程的部分或全部从主存移到磁盘，或者从磁盘移回主存。

| English      | 中文        |
| ------------ | --------- |
| **Swap out** | 换出，从主存到磁盘 |
| **Swap in**  | 换入，从磁盘到主存 |

目的：

1. 释放主存空间；
2. 让更多进程参与多道程序设计；
3. 当内存中的进程都阻塞时，换入其他可运行进程，提高 CPU 利用率。

---

## 3.6 Why does Figure 3.9b have two blocked states?

## 为什么 Figure 3.9b 有两个阻塞状态？

因为这里有两个独立维度：

| Dimension                      | 中文       |
| ------------------------------ | -------- |
| **Blocked or not blocked**     | 是否正在等待事件 |
| **Suspended or not suspended** | 是否被换出主存  |

所以阻塞进程分成两类：

| State               | 中文    | 含义           |
| ------------------- | ----- | ------------ |
| **Blocked**         | 阻塞态   | 在主存中，但正在等事件  |
| **Blocked/Suspend** | 阻塞挂起态 | 在磁盘中，并且正在等事件 |

核心理解：

**Blocked** 是在等事件。
**Suspend** 是不在主存。
二者不是同一回事。

---

## 3.7 List four characteristics of a suspended process.

## 列出挂起进程的四个特征。

**Suspended process / 挂起进程** 的四个特征：

1. **Not immediately available for execution**
   不能立即执行。

2. **May or may not be waiting on an event**
   可能在等事件，也可能不等事件。

3. **Placed in suspended state by an agent**
   是被某个主体挂起的，例如 OS、父进程或它自己。

4. **Cannot leave suspension until explicitly activated**
   必须由对应主体显式恢复，才能离开挂起状态。

---

## 3.8 For what entities does OS maintain tables?

## OS 会为哪些实体维护管理表？

这一题偏 3.3，但和进程管理相关。

OS 通常维护四类表：

| Table              | 中文    |
| ------------------ | ----- |
| **Memory tables**  | 内存表   |
| **I/O tables**     | I/O 表 |
| **File tables**    | 文件表   |
| **Process tables** | 进程表   |

---

## 3.9 List three general categories of information in a PCB.

## PCB 中三大类信息是什么？

**PCB = Process Control Block / 进程控制块**

三大类信息：

| English                         | 中文                         |
| ------------------------------- | -------------------------- |
| **Process identification**      | 进程标识信息，例如 PID、父进程 ID、用户 ID |
| **Processor state information** | 处理器状态信息，例如寄存器、PC、PSW、栈指针   |
| **Process control information** | 进程控制信息，例如状态、优先级、调度信息、资源信息  |

---

## 3.10 Why are two modes needed?

## 为什么需要用户态和内核态？

两种模式：

| Mode                          | 中文        |
| ----------------------------- | --------- |
| **User mode**                 | 用户态       |
| **Kernel mode / System mode** | 内核态 / 系统态 |

原因：

为了保护操作系统和关键数据结构，例如 PCB、页表、I/O 控制表。

用户程序不能随意执行：

* 修改 PSW 的指令；
* 直接操作 I/O；
* 修改内存管理结构；
* 访问内核内存。

所以普通程序运行在 **user mode**，需要 OS 服务时通过 **system call / supervisor call** 进入 **kernel mode**。

---

## 3.11 What are the steps performed by OS to create a new process?

## OS 创建新进程的步骤是什么？

1. **Assign a unique process identifier**
   分配唯一 PID。

2. **Allocate space for the process**
   分配进程映像空间，包括程序、数据、栈、PCB。

3. **Initialize the process control block**
   初始化 PCB，例如状态、PC、寄存器、优先级等。

4. **Set the appropriate linkages**
   设置链表/队列链接，把进程加入 Ready queue 或 Ready/Suspend queue。

5. **Create or expand other data structures**
   创建或扩展其他结构，例如 accounting 信息。

---

## 3.12 Difference between interrupt and trap?

## interrupt 和 trap 的区别？

| Term          | 中文      | 来源        | 用途       |
| ------------- | ------- | --------- | -------- |
| **Interrupt** | 中断      | 当前指令外部的事件 | 响应异步外部事件 |
| **Trap**      | 陷阱 / 异常 | 当前指令执行导致  | 处理错误或异常  |

例子：

**Interrupt**：I/O 完成、时钟中断。
**Trap**：除零、非法指令、越界访问、非法文件访问。

---

## 3.13 Give three examples of an interrupt.

## 举三个中断例子。

教材中常见三个：

| English             | 中文            |
| ------------------- | ------------- |
| **Clock interrupt** | 时钟中断          |
| **I/O interrupt**   | I/O 中断        |
| **Memory fault**    | 内存故障 / 缺页相关事件 |

补充：很多现代 OS 会把 **page fault / 缺页异常** 归为 trap/exception，但本题按教材列举即可。

---

## 3.14 Difference between mode switch and process switch?

## 模式切换和进程切换有什么区别？

| Term               | 中文                           | 是否换进程？ |
| ------------------ | ---------------------------- | ------ |
| **Mode switch**    | 模式切换，User mode ↔ Kernel mode | 不一定    |
| **Process switch** | 进程切换，从一个进程换到另一个进程            | 一定     |

例子：

用户程序调用 `read()`：

* 如果数据已经准备好：User mode → Kernel mode → User mode，可能只是 **mode switch**。
* 如果需要等磁盘 I/O：当前进程 Running → Blocked，OS 调度另一个进程，这才是 **process switch**。

---

# 二、Problems 相关题解答

## Problem 3.1 状态转换表

题目给出 READY、RUN、BLOCKED、NONRESIDENT 之间的转换编号。解答如下：

| 编号 | Transition                | 中文       | 可能事件               |
| -- | ------------------------- | -------- | ------------------ |
| 1  | **READY → RUN**           | 就绪 → 运行  | Dispatcher 选择该进程运行 |
| 2  | **RUN → READY**           | 运行 → 就绪  | 时间片用完，或被更高优先级进程抢占  |
| 3  | **RUN → BLOCKED**         | 运行 → 阻塞  | 进程请求 I/O，等待事件      |
| 4  | **BLOCKED → READY**       | 阻塞 → 就绪  | 等待的事件发生，例如 I/O 完成  |
| 5  | **READY → NONRESIDENT**   | 就绪 → 非驻留 | OS 将就绪进程换出主存       |
| 6  | **BLOCKED → NONRESIDENT** | 阻塞 → 非驻留 | OS 将阻塞进程换出主存       |

这里的 **NONRESIDENT** 可以理解为进程不在主存中，接近 **suspended / swapped out**。

---

## Problem 3.2 判断 t=22、t=37、t=47 时进程状态

先整理事件：

* t=5：P1 读 disk 3，所以 P1 阻塞，等 disk 3 read。
* t=18：P7 写 disk 3，所以 P7 阻塞，等 disk 3 write。
* t=20：P3 读 disk 2，所以 P3 阻塞，等 disk 2 read。
* t=24：P5 写 disk 3，所以 P5 阻塞，等 disk 3 write。
* t=28：P5 被 swapped out，所以 P5 进入 Blocked/Suspend。
* t=33：P3 disk 2 read 完成，所以 P3 变 Ready。
* t=36：P1 disk 3 read 完成，所以 P1 变 Ready。
* t=40：P5 disk 3 write 完成，所以 P5 从 Blocked/Suspend 变 Ready/Suspend。
* t=44：P5 swapped back in，所以 P5 变 Ready。
* t=48：P7 disk 3 write 完成，所以 P7 变 Ready。

### t = 22

| Process | State                         | 中文说明                                                    |
| ------- | ----------------------------- | ------------------------------------------------------- |
| P1      | **Blocked**                   | 等 disk 3 read 完成                                        |
| P3      | **Blocked**                   | 等 disk 2 read 完成                                        |
| P5      | **Running** 或至少 Ready/Running | t=15 时间片到期后，t=24 会执行写 disk 3；若按事件序列推断，t=22 通常认为 P5 正在运行 |
| P7      | **Blocked**                   | 等 disk 3 write 完成                                       |
| P8      | **Not determined**            | 题目尚未给足信息                                                |

### t = 37

| Process | State               | 中文说明                        |
| ------- | ------------------- | --------------------------- |
| P1      | **Ready**           | t=36 disk 3 read 已完成        |
| P3      | **Ready**           | t=33 disk 2 read 已完成        |
| P5      | **Blocked/Suspend** | 已换出，并仍在等 disk 3 write       |
| P7      | **Blocked**         | 仍在等 disk 3 write，到 t=48 才完成 |
| P8      | **Running**         | 若 t=38 是正常终止，则 t=37 可认为正在运行 |

### t = 47

| Process | State                | 中文说明                        |
| ------- | -------------------- | --------------------------- |
| P1      | **Ready or Running** | 已经可运行，但调度器未说明               |
| P3      | **Ready or Running** | 已经可运行，但调度器未说明               |
| P5      | **Ready**            | t=40 写完成，t=44 换回主存          |
| P7      | **Blocked**          | 仍在等 disk 3 write，到 t=48 才完成 |
| P8      | **Exit**             | t=38 已终止                    |

注意：t=47 时，题目没有给出调度策略，所以无法唯一确定 P1、P3、P5 中哪个正在 Running。严谨写法是：**one of the ready processes may be running; the others are ready.**

---

## Problem 3.3 七状态模型中的 possible / impossible transitions

七个状态：

| Abbrev. | State           |
| ------- | --------------- |
| N       | New             |
| R       | Ready           |
| Run     | Running         |
| B       | Blocked         |
| E       | Exit            |
| RS      | Ready/Suspend   |
| BS      | Blocked/Suspend |

### Possible transitions

| From | To  | 例子                            |
| ---- | --- | ----------------------------- |
| N    | R   | OS admit 新进程，并放入主存            |
| N    | RS  | 新进程创建，但内存不够，先进入 Ready/Suspend |
| N    | E   | 新进程还未接纳就被取消                   |
| R    | Run | Dispatcher 调度该进程              |
| R    | RS  | OS 为释放内存，把 ready 进程换出         |
| R    | E   | 父进程或 OS 终止该进程                 |
| Run  | R   | 时间片用完或被抢占                     |
| Run  | B   | 请求 I/O 或等待事件                  |
| Run  | RS  | 被更高优先级进程抢占，同时换出               |
| Run  | E   | 正常结束或异常终止                     |
| B    | R   | 等待事件发生                        |
| B    | BS  | 阻塞进程被换出主存                     |
| B    | E   | 父进程或 OS 终止它                   |
| RS   | R   | 被换入主存                         |
| RS   | E   | 被终止                           |
| BS   | B   | 被换入主存，但仍在等事件                  |
| BS   | RS  | 等待事件发生，但仍在磁盘                  |
| BS   | E   | 被终止                           |

### Impossible transitions

按标准状态图，其余都不作为直接转换：

| From | Impossible direct transitions | 原因                                |
| ---- | ----------------------------- | --------------------------------- |
| N    | Run, B, BS                    | 新进程不能直接运行或阻塞，必须先被 admit           |
| R    | N, B, BS                      | Ready 不能直接阻塞，必须 Running 后才能等待事件   |
| Run  | N, BS                         | 运行进程不能回到 New；Run→BS 通常拆成 Run→B→BS |
| B    | N, Run, RS                    | 阻塞进程事件未发生不能运行；也不能回 New            |
| RS   | N, Run, B, BS                 | 不在主存，不能直接运行或发起等待事件                |
| BS   | N, Run, R                     | 仍不在主存，不能直接运行；BS→R 通常要 BS→RS→R     |
| E    | N, R, Run, B, RS, BS          | Exit 是终止状态，进程不再回到其他状态             |

---

## Problem 3.4 七状态模型的 queueing diagram

可以这样画：

```text
                    +----------------+
                    |      New       |
                    +----------------+
                       | Admit
                       v
+----------------+   Dispatch   +----------------+
|  Ready Queue   | -----------> |    Processor   |
+----------------+              |    Running     |
     ^       |                  +----------------+
     |       | Suspend              |   |   |
     |       v                      |   |   |
     |  +----------------+          |   |   +--> Exit
     |  | Ready/Suspend  | <--------+   |
     |  |     Queue      |  Suspend     |
     |  +----------------+              |
     |       ^                          |
     |       | Event occurs              | Event wait
     |       |                          v
+----------------------+        +----------------------+
| Blocked/Suspend      | <---- | Blocked queues       |
| queues by event      |       | by event             |
+----------------------+        +----------------------+
        | Activate                       |
        +--------------------------------+
```

解释：

* **Ready Queue**：在主存中，等 CPU。
* **Blocked queues by event**：在主存中，等某个事件。
* **Ready/Suspend Queue**：不在主存，但只要换入就能运行。
* **Blocked/Suspend queues by event**：不在主存，而且还在等事件。
* 每个 event 最好单独一个 blocked queue，例如 disk 1 queue、disk 2 queue、network queue。

---

## Problem 3.5 Ready 与 Ready/Suspend 同时存在时如何调度？

题目问：如果 Ready/Suspend 中有一个进程优先级高于所有 Ready 进程，是否应该立刻换入它？

两个极端策略：

1. 永远选 Ready，避免 swapping。
2. 永远选最高优先级，即使要 swapping。

折中策略：

> **Only swap in a Ready/Suspend process if its priority advantage is large enough to justify the swapping cost.**
> 只有当 Ready/Suspend 进程的优先级优势足够大，值得付出换入换出的代价时，才换入它。

可以具体写成：

* 如果 Ready/Suspend 进程优先级只高一点：先运行主存中的 Ready 进程。
* 如果 Ready/Suspend 进程优先级高很多，或等待时间过长：换入它并运行。
* 可以设置一个阈值：
  **priority difference > threshold** 才触发 swap-in。
* 也可以结合 aging，防止高优先级进程一直在磁盘中饿死。

---

# 三、扩展相关 Problems 简答

## Problem 3.7 四种 access modes 的优缺点

VAX/VMS 有四个模式：

| Mode           | 中文   | 职责             |
| -------------- | ---- | -------------- |
| **Kernel**     | 内核模式 | 内存管理、中断处理、I/O  |
| **Executive**  | 执行模式 | 文件、记录管理等 OS 服务 |
| **Supervisor** | 监督模式 | 用户命令等 OS 服务    |
| **User**       | 用户模式 | 普通用户程序         |

### 优点

* 更细粒度保护，**finer-grained protection**；
* 减少所有 OS 服务都拥有最高权限的风险；
* 更符合最小权限原则，**least privilege**；
* 不同系统服务可以放在不同权限层。

### 缺点

* 硬件和 OS 设计更复杂；
* 模式切换成本更高；
* 调试和维护更困难；
* 权限层越多，越容易设计错误。

是否可以更多模式？

理论上可以，例如给驱动、文件系统、网络栈、安全监控各自不同 ring。但实际中模式太多会让系统复杂度和切换开销变大。

---

## Problem 3.8 ring protection 的 need-to-know 问题

**Ring protection / 环保护结构** 的问题是：权限是层级包含关系。

如果更内层 ring 权限更高，那么它不仅能访问自己的对象，也能访问外层能访问的对象。

问题在于：

> **Need-to-know requires selective access, but ring protection gives nested access.**
> need-to-know 要求“只访问自己需要知道的对象”，但 ring 结构天然给了“层级包含式权限”。

例子：

* A 进程需要访问文件 X；
* B 进程需要访问文件 Y；
* 但 B 不应该访问 X。

如果为了让 B 访问某个高权限对象，把 B 放进更高权限 ring，B 可能自动获得 A 所能访问的东西。这就违反了 need-to-know。

---

## Problem 3.9 一个进程能不能同时等待多个事件？

可以。

例子：

一个进程可能等待：

* 网络数据到达；
* 用户键盘输入；
* 超时 timeout。

只要任意一个事件发生，它就可以继续执行。

修改 queueing structure 的方法：

原来一个 PCB 只在一个 event queue 中。现在可以改成：

1. 允许一个进程的 PCB 出现在多个 event queue 中；
2. 或者使用 **wait descriptor / 等待描述符**，每个等待事件有一个描述符；
3. 当任意事件发生时，把进程从其他等待队列中删除，移动到 Ready Queue。

---

## Problem 3.10 早期机器把寄存器保存到固定位置是否可行？

可行条件：

* 单处理器；
* 不允许嵌套中断；
* 中断处理期间关闭其他中断；
* 系统简单，没有复杂多进程切换；
* 每种中断不会重入。

为什么一般不方便？

因为现代系统有：

* nested interrupts / 嵌套中断；
* multiprogramming / 多道程序；
* process switching / 进程切换；
* 多核 CPU；
* 每个进程都要保存自己的上下文。

如果所有寄存器都存在固定地址，后来的中断可能覆盖前一个中断保存的信息，所以现代 OS 更倾向于把上下文保存到 **kernel stack / PCB** 中。

---

## Problem 3.11 为什么不可抢占内核不适合实时系统？

题目说：UNIX 中进程在 kernel mode 执行时可能不能被 preempt。

实时系统需要：

> **bounded response time**
> 有界响应时间。

如果低优先级进程进入内核，正在执行一个较长系统调用，而内核不可抢占，那么高优先级实时任务即使变成 Ready，也必须等低优先级进程退出内核。

这会导致：

* 响应时间不可预测；
* 优先级反转；
* 高优先级任务错过 deadline；
* 不适合 hard real-time。

所以实时 OS 通常需要 **preemptible kernel / 可抢占内核**。

---

## Problem 3.12 fork 程序的可能输出

程序：

```c
main ()
{
    int pid;
    pid = fork();
    printf("%d \n", pid);
}
```

`fork()` 成功后会产生两个进程：

| Process        | fork 返回值                     |
| -------------- | ---------------------------- |
| Parent process | 返回 child process 的 PID，一个正整数 |
| Child process  | 返回 0                         |

所以会打印两行：

```text
0
child_pid
```

或者顺序反过来：

```text
child_pid
0
```

其中 `child_pid` 是子进程的 PID，例如：

```text
12345
0
```

或：

```text
0
12345
```

顺序不确定，因为 parent 和 child 谁先运行由调度器决定。
