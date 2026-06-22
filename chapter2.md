下面复习 **2.1 Operating System Objectives and Functions / 操作系统的目标与功能**。这一节核心就是回答：

> **操作系统是什么？它为什么存在？它主要做什么？**
> **What is an operating system? Why does it exist? What does it do?**

内容来自你上传的 2.1 PDF。

---

## 1. OS 的定义 / Definition of OS

英文原文核心意思：

> **An OS is a program that controls the execution of application programs and acts as an interface between applications and the computer hardware.**

中文解释：

**操作系统 OS / Operating System** 是一种程序，它有两个核心作用：

1. **控制应用程序的执行**
   Controls the execution of application programs.

2. **作为应用程序和硬件之间的接口**
   Acts as an interface between applications and computer hardware.

也就是说，应用程序一般不会直接操作硬件，而是通过操作系统来使用 CPU、内存、磁盘、I/O 设备等资源。

可以记成：

> **OS = 管程序 + 管硬件 + 做中介**
> **OS = Program controller + Resource manager + Interface**

---

## 2. OS 的三个目标 / Three Objectives of an OS

### 2.1 Convenience / 方便性

英文：

> **An OS makes a computer more convenient to use.**

中文：

操作系统让计算机更容易使用。

例如用户不需要知道磁盘如何寻道、键盘如何发中断、内存如何分配，只需要点击文件、运行程序、输入输出即可。

考点理解：

> 没有 OS，程序员要直接写机器指令控制硬件，非常复杂。
> With no OS, programmers would have to directly control hardware with machine instructions.

---

### 2.2 Efficiency / 效率性

英文：

> **An OS allows the computer system resources to be used in an efficient manner.**

中文：

操作系统让计算机资源被高效利用。

这里的资源包括：

* CPU / 处理器
* Main memory / 主存
* I/O devices / 输入输出设备
* Files / 文件
* Storage / 存储设备
* Network / 网络资源

例如 CPU 调度可以让多个程序轮流执行，内存管理可以让多个进程共存，I/O 管理可以减少设备空闲浪费。

---

### 2.3 Ability to evolve / 可演化性

英文：

> **An OS should be constructed in such a way as to permit the effective development, testing, and introduction of new system functions without interfering with service.**

中文：

操作系统应该设计得容易扩展、修改和维护。

原因是 OS 会随着时间不断变化，例如：

* 新硬件出现
  New hardware appears.

* 用户需要新服务
  Users need new services.

* 系统 bug 需要修复
  Bugs need to be fixed.

所以 OS 设计上要尽量 **modular / 模块化**，模块之间要有清晰接口。

---

## 3. OS as a User/Computer Interface / 操作系统作为用户与计算机的接口

这一部分讲的是：用户、应用程序、系统程序、操作系统、硬件之间的层次关系。

大致结构是：

```text
User / 用户
   ↓
Application Programs / 应用程序
   ↓
Libraries and Utilities / 库和工具程序
   ↓
Operating System / 操作系统
   ↓
Hardware / 硬件
```

重点：

> **The OS masks the details of the hardware from the programmer.**
> 操作系统向程序员隐藏硬件细节。

例如程序员写：

```c
read(fd, buffer, size);
```

他不需要知道磁盘控制器怎么工作、数据块怎么寻址、I/O 中断怎么处理，这些由 OS 完成。

---

## 4. OS 提供的主要服务 / Services Provided by OS

这一节列了很多 OS 的服务，是考试常考点。

### 4.1 Program development / 程序开发

英文：

> The OS provides facilities and services, such as editors and debuggers.

中文：

操作系统提供程序开发工具，比如：

* editors / 编辑器
* debuggers / 调试器
* compilers / 编译器
* libraries / 库

严格来说，有些工具不一定是 OS 内核的一部分，但通常随 OS 提供。

---

### 4.2 Program execution / 程序执行

程序执行前需要很多准备工作：

* 把 instructions and data / 指令和数据 加载到主存
* 初始化 I/O devices / 输入输出设备
* 初始化 files / 文件
* 准备其他 resources / 资源

这些工作由 OS 帮用户完成。

中文记忆：

> 程序不是一点就“凭空运行”，OS 要先帮它准备运行环境。

---

### 4.3 Access to I/O devices / 访问 I/O 设备

不同 I/O 设备有不同控制方式。

例如：

* keyboard / 键盘
* printer / 打印机
* disk / 磁盘
* network card / 网卡

如果让应用程序直接控制这些设备，会非常复杂。

所以 OS 提供统一接口，例如：

```c
read()
write()
open()
close()
```

核心句：

> **The OS provides a uniform interface.**
> 操作系统提供统一接口。

---

### 4.4 Controlled access to files / 受控的文件访问

OS 管理文件时，不仅要知道设备本身，还要知道文件系统结构。

例如：

* 文件存在哪里
* 文件块如何组织
* 文件权限是什么
* 多用户是否可以访问

在多用户系统中，OS 还要提供 **protection mechanisms / 保护机制**，防止非法访问文件。

---

### 4.5 System access / 系统访问控制

对于共享系统或公共系统，OS 要控制：

* 谁能登录系统
* 谁能访问资源
* 哪些操作被允许
* 多个用户争用资源时如何处理

关键词：

> **protection of resources and data**
> 保护资源和数据

> **resource contention**
> 资源竞争

---

### 4.6 Error detection and response / 错误检测与响应

OS 要处理硬件和软件错误。

硬件错误例子：

* memory error / 内存错误
* device failure / 设备故障
* device malfunction / 设备异常

软件错误例子：

* division by zero / 除零错误
* forbidden memory access / 访问非法内存
* OS cannot grant request / 操作系统无法满足请求

OS 的响应方式可能包括：

* ending the program / 终止出错程序
* retrying the operation / 重试操作
* reporting the error / 向应用程序报告错误

---

### 4.7 Accounting / 计账与统计

Accounting 不是会计那么简单，在 OS 里主要指：

> 收集资源使用情况和性能数据。

例如：

* CPU 用了多久
* 内存用了多少
* I/O 频率是多少
* response time / 响应时间
* 系统性能是否下降

在多用户系统中，也可以用于 billing / 计费。

---

## 5. 三个重要接口 / Three Key Interfaces

这一部分容易混淆，考试可能会考 ISA、ABI、API 的区别。

---

### 5.1 ISA / Instruction Set Architecture / 指令集体系结构

英文：

> **The ISA defines the repertoire of machine language instructions that a computer can follow.**

中文：

ISA 定义 CPU 能执行哪些机器指令。

它是：

> **hardware and software boundary / 硬件和软件之间的边界**

可以分为：

| English    | 中文    | 含义          |
| ---------- | ----- | ----------- |
| user ISA   | 用户指令集 | 普通应用程序可用的指令 |
| system ISA | 系统指令集 | OS 可用的特权指令  |

例如普通程序不能随便修改页表、关闭中断，而 OS 可以通过特权指令管理资源。

---

### 5.2 ABI / Application Binary Interface / 应用二进制接口

英文：

> **The ABI defines a standard for binary portability across programs.**

中文：

ABI 关注的是 **二进制层面的兼容性**。

也就是说，已经编译好的程序能不能在某个系统上运行，和 ABI 有关。

ABI 包括：

* system call interface / 系统调用接口
* register usage / 寄存器使用约定
* calling convention / 函数调用约定
* binary format / 二进制格式

记忆：

> **ABI 面向编译后的二进制程序。**
> ABI is for compiled binary programs.

---

### 5.3 API / Application Programming Interface / 应用程序编程接口

英文：

> **The API gives a program access to hardware resources and services through high-level language library calls.**

中文：

API 是程序员写代码时使用的接口。

例如：

```c
printf()
malloc()
open()
read()
write()
```

API 更偏向源代码层面。

记忆：

> **API 面向程序员写代码。**
> API is for programmers writing source code.

---

### ISA、ABI、API 对比总结

| 接口  | English                           | 中文理解         | 面向对象    |
| --- | --------------------------------- | ------------ | ------- |
| ISA | Instruction Set Architecture      | CPU 能执行什么指令  | 硬件/机器指令 |
| ABI | Application Binary Interface      | 编译好的程序如何兼容运行 | 二进制程序   |
| API | Application Programming Interface | 程序员如何调用功能    | 源代码/程序员 |

一句话记忆：

> **API 是写代码用的，ABI 是编译后运行用的，ISA 是 CPU 真正执行的。**
> API is for coding, ABI is for binary execution, ISA is for CPU instructions.

---

## 6. OS as Resource Manager / 操作系统作为资源管理器

这一部分是 2.1 的第二条主线。

英文核心：

> **The OS is responsible for managing computer resources.**

中文：

计算机是资源集合，OS 负责管理这些资源。

主要资源包括：

* Processor / 处理器
* Main memory / 主存
* I/O devices / 输入输出设备
* Files / 文件
* Storage / 存储
* Network / 网络

---

## 7. OS 控制计算机的特殊方式

这一段比较抽象，但很重要。

OS 自己也是程序：

> **The OS functions in the same way as ordinary computer software.**
> 操作系统和普通软件一样，也是由处理器执行的程序。

但 OS 又要管理其他程序。

矛盾点在于：

> OS 要控制 CPU，但 OS 自己也要靠 CPU 执行。

所以 OS 的控制方式很特殊：

1. OS 获得 CPU 控制权
   OS gains control.

2. OS 做调度、资源分配等工作
   OS schedules and allocates resources.

3. OS 放弃 CPU，让用户程序运行
   OS relinquishes control.

4. 通过中断、系统调用、异常等机制重新获得控制权
   OS regains control through interrupts, system calls, or exceptions.

可以记成：

> **OS 不是一直运行，而是不断“让出控制权”和“重新获得控制权”。**
> The OS does not run all the time; it repeatedly gives up and regains control.

---

## 8. Kernel / Nucleus / 内核

英文提到：

> **The kernel, or nucleus, contains the most frequently used functions in the OS.**

中文：

**Kernel / 内核** 是 OS 中最核心、最常用的部分，通常驻留在主存中。

它负责：

* process management / 进程管理
* memory management / 内存管理
* I/O management / 输入输出管理
* file management / 文件管理
* interrupt handling / 中断处理
* system call handling / 系统调用处理

注意：

> 不是整个 OS 都一直在主存里，但 kernel 通常会常驻主存。

---

## 9. Ease of Evolution / 操作系统的可演化性

OS 必须能不断发展，原因有三个。

### 9.1 Hardware upgrades plus new types of hardware / 硬件升级与新硬件

例如早期 UNIX 和 Macintosh OS 没有使用 paging / 分页机制，因为当时处理器不支持分页硬件。

后来硬件支持分页后，OS 就可以改进来利用 paging。

中文理解：

> 硬件变了，OS 也要跟着变。

---

### 9.2 New services / 新服务

用户和系统管理员会提出新需求。

例如发现系统性能不好，就可能增加新的性能测量和控制工具。

中文理解：

> 用户需求变了，OS 要增加新功能。

---

### 9.3 Fixes / 修复

任何 OS 都可能有 bug。

修复 bug 是必须的，但修复也可能引入新 bug。

所以 OS 设计要便于维护。

---

## 10. 本节最重要的考试总结

### 必背 1：OS 的三个目标

| English           | 中文   |
| ----------------- | ---- |
| Convenience       | 方便性  |
| Efficiency        | 效率性  |
| Ability to evolve | 可演化性 |

---

### 必背 2：OS 的两个核心角色

| English                 | 中文          |
| ----------------------- | ----------- |
| User/computer interface | 用户与计算机之间的接口 |
| Resource manager        | 资源管理器       |

---

### 必背 3：OS 提供的服务

| English                      | 中文        |
| ---------------------------- | --------- |
| Program development          | 程序开发      |
| Program execution            | 程序执行      |
| Access to I/O devices        | 访问 I/O 设备 |
| Controlled access to files   | 受控文件访问    |
| System access                | 系统访问控制    |
| Error detection and response | 错误检测与响应   |
| Accounting                   | 计账与统计     |

---

### 必背 4：ISA、ABI、API 区别

| Term | 中文       | 关键词      |
| ---- | -------- | -------- |
| ISA  | 指令集体系结构  | CPU 指令   |
| ABI  | 应用二进制接口  | 编译后程序兼容  |
| API  | 应用程序编程接口 | 程序员写代码调用 |

---

## 11. 可能考题与答案

### Question 1

**What are the three objectives of an operating system?**
操作系统的三个目标是什么？

答案：

An operating system has three main objectives: convenience, efficiency, and ability to evolve.

操作系统有三个主要目标：方便性、效率性和可演化性。

---

### Question 2

**Why is the OS considered a user/computer interface?**
为什么说操作系统是用户和计算机之间的接口？

答案：

Because the OS hides the details of hardware from users and programmers, and provides convenient services and interfaces for applications to use hardware resources.

因为操作系统隐藏了硬件细节，并为应用程序提供方便的接口，使应用程序能够使用硬件资源。

---

### Question 3

**Why is the OS considered a resource manager?**
为什么说操作系统是资源管理器？

答案：

Because the OS manages computer resources such as the processor, main memory, I/O devices, files, and storage, and decides how these resources are allocated to programs.

因为操作系统管理 CPU、主存、I/O 设备、文件和存储等资源，并决定这些资源如何分配给程序。

---

### Question 4

**What is the difference between API, ABI, and ISA?**
API、ABI、ISA 有什么区别？

答案：

API is a source-level programming interface used by programmers. ABI is a binary-level interface that allows compiled programs to run on compatible systems. ISA defines the machine instructions that the CPU can execute.

API 是程序员写源代码时使用的接口；ABI 是二进制层面的接口，用于保证编译后的程序能够在兼容系统上运行；ISA 定义 CPU 能执行的机器指令。

---

### Question 5

**Why should an OS be designed to evolve?**
为什么操作系统要具有可演化性？

答案：

Because hardware changes, new services are needed, and bugs must be fixed. Therefore, an OS should be modular, with clear interfaces and good documentation.

因为硬件会变化，用户会需要新服务，系统 bug 也需要修复。因此操作系统应当模块化设计，具有清晰接口和良好文档。

---

## 12. 一句话总复习

**English:**
An operating system controls application execution, hides hardware details, provides services to programs, manages system resources, and must be designed to evolve.

**中文：**
操作系统控制应用程序执行，隐藏硬件细节，为程序提供服务，管理系统资源，并且必须设计成便于扩展和维护的形式。


和 **2.1 Operating System Objectives and Functions / 操作系统目标与功能** 最相关的题目是：

| 类型              |  题号 | 相关程度 | 原因                                                |
| --------------- | --: | ---- | ------------------------------------------------- |
| Review Question | 2.1 | 直接相关 | OS 的三个目标                                          |
| Review Question | 2.2 | 直接相关 | kernel / nucleus 在 2.1 中出现                        |
| Problem         | 2.4 | 较相关  | system calls 和 OS interface/API/ABI 有关            |
| Problem         | 2.5 | 较相关  | System Resource Manager 对应 OS as resource manager |
| Review Question | 2.6 | 间接相关 | 属于 OS 资源管理，但更偏后面内存/存储管理                           |

题目文件里 Review Questions 包含 2.1 到 2.11，Problems 包含 2.1 到 2.6，其中 2.1、2.2、2.4、2.5 与本节联系最强。 2.1 原文主要讲 OS 的三个目标、作为用户/计算机接口、作为资源管理器，以及 kernel、API/ABI/ISA 等内容。

---

# Review Question 2.1

## 原题 / Original Question

**What are three objectives of an OS design?**
操作系统设计的三个目标是什么？

## 答案 / Answer

The three objectives of an OS design are:

1. **Convenience / 方便性**
   The OS makes the computer easier and more convenient to use.
   操作系统让计算机更容易使用，用户和程序员不需要直接面对复杂硬件细节。

2. **Efficiency / 效率性**
   The OS allows computer system resources to be used efficiently.
   操作系统负责高效利用 CPU、内存、I/O 设备、文件和存储等系统资源。

3. **Ability to evolve / 可演化性**
   The OS should be designed so that new functions can be developed, tested, and introduced without disrupting existing services.
   操作系统应该便于扩展、维护和修改，例如支持新硬件、新服务和 bug 修复。

## 考试版答案

**English:**
The three objectives of an operating system are convenience, efficiency, and ability to evolve.

**中文：**
操作系统的三个目标是：方便性、效率性和可演化性。

---

# Review Question 2.2

## 原题 / Original Question

**What is the kernel of an OS?**
什么是操作系统内核？

## 答案 / Answer

The **kernel**, also called the **nucleus**, is the core part of the operating system that contains the most frequently used OS functions.

**Kernel / 内核**，也叫 **nucleus / 核心**，是操作系统中最核心、最常用的部分。

它通常负责：

| English              | 中文     |
| -------------------- | ------ |
| process management   | 进程管理   |
| memory management    | 内存管理   |
| I/O management       | 输入输出管理 |
| file management      | 文件管理   |
| interrupt handling   | 中断处理   |
| system call handling | 系统调用处理 |

2.1 原文说，一部分 OS 位于主存中，其中包括 **kernel, or nucleus**，它包含 OS 中最常使用的功能。

## 考试版答案

**English:**
The kernel is the central part of the OS that remains in main memory and contains the most frequently used operating system functions.

**中文：**
内核是操作系统的核心部分，通常驻留在主存中，包含操作系统最常用、最核心的功能。

---

# Problem 2.4

## 原题 / Original Question

**What is the purpose of system calls, and how do system calls relate to the OS and to the concept of dual-mode operation?**
系统调用的目的是什么？系统调用与操作系统以及双模式操作有什么关系？

## 答案 / Answer

### 1. Purpose of system calls / 系统调用的目的

**System calls / 系统调用** 的作用是让用户程序请求操作系统服务。

应用程序不能随便直接操作硬件或执行特权指令，所以它必须通过 system call 请求 OS 帮它完成一些操作。

常见系统调用包括：

| System call service | 中文     |
| ------------------- | ------ |
| file operations     | 文件操作   |
| process creation    | 创建进程   |
| memory allocation   | 分配内存   |
| I/O operations      | 输入输出操作 |
| communication       | 进程间通信  |
| protection/security | 权限和保护  |

例如程序调用：

```c
read(fd, buffer, size);
```

看起来是普通函数，但底层会通过系统调用请求 OS 从文件或设备读取数据。

---

### 2. Relationship with OS / 和操作系统的关系

System calls are the interface between user programs and the operating system.

系统调用是用户程序和操作系统之间的接口。

2.1 中提到 ABI 定义了 **system call interface to the operating system**，API 通常通过库函数来执行 system calls。

也就是说：

```text
User Program / 用户程序
        ↓ API / library calls
System Call / 系统调用
        ↓
Operating System Kernel / 操作系统内核
        ↓
Hardware / 硬件
```

---

### 3. Relationship with dual-mode operation / 和双模式操作的关系

**Dual-mode operation / 双模式操作** 指 CPU 至少有两种运行模式：

| Mode        | 中文  | 权限           |
| ----------- | --- | ------------ |
| User mode   | 用户态 | 权限低，普通应用程序运行 |
| Kernel mode | 内核态 | 权限高，OS 内核运行  |

普通应用程序运行在 **user mode / 用户态**，不能执行危险操作，例如直接访问硬件、修改页表、关闭中断等。

当应用程序需要 OS 服务时，会执行 system call。系统调用会触发一次受控切换：

```text
User mode / 用户态
   ↓ system call / trap
Kernel mode / 内核态
   ↓ OS 执行服务
User mode / 返回用户态
```

所以 system call 是用户程序进入内核服务的合法入口。

## 考试版答案

**English:**
System calls provide a controlled interface through which user programs request services from the operating system. In dual-mode operation, user programs run in user mode and cannot execute privileged instructions directly. A system call transfers control to the OS kernel in kernel mode, allowing the OS to perform the requested privileged operation safely and then return control to the user program.

**中文：**
系统调用为用户程序提供请求操作系统服务的受控接口。在双模式操作中，用户程序运行在用户态，不能直接执行特权指令。系统调用会使控制权从用户态切换到内核态，由操作系统内核安全地完成请求，然后再返回用户程序。

---

# Problem 2.5

## 原题简述 / Original Question Summary

题目说 IBM OS/390 的内核中有一个 **System Resource Manager, SRM / 系统资源管理器**。SRM 负责在 address spaces / 地址空间，也就是 processes / 进程之间分配资源。资源包括 processor / 处理器、real memory / 实存、I/O channels / I/O 通道。

SRM 每秒大约 20 次检查所有 page frames / 页框。如果某个页没有被 referenced / 引用 或 changed / 修改，就把计数器加 1。长期统计后，SRM 可以得到一个 page frame 平均多久没有被访问。题目问：

**What might be the purpose of this and what action might SRM take?**
这样做的目的是什么？SRM 可能采取什么行动？

题目原文说明 SRM 会收集处理器、通道、关键数据结构等利用率统计信息，并根据性能目标动态修改系统和作业特征。

---

## 答案 / Answer

SRM 这样做的目的，是判断 **real memory / 实存** 的使用压力，找出哪些 page frames / 页框 长时间没有被使用。

这其实是在估计：

| English                    | 中文      |
| -------------------------- | ------- |
| page activity              | 页面活跃程度  |
| memory pressure            | 内存压力    |
| working set behavior       | 工作集行为   |
| candidates for replacement | 页面置换候选页 |
| thrashing risk             | 抖动风险    |

---

## 分情况理解

### 情况 1：很多 page frames 长时间 untouched / 很多页框长期没被访问

说明内存比较宽裕，或者有些进程占着内存但不怎么用。

SRM 可能采取的行动：

* 把这些页面作为 **page replacement / 页面置换** 的优先候选；
* 回收部分 page frames；
* 给更需要内存的进程分配更多内存；
* 允许更多作业进入系统，提高 multiprogramming level / 多道程序程度。

中文理解：

> 如果很多页很久没被碰过，说明它们不活跃，可以被换出或重新分配。

---

### 情况 2：page frames 很快就被访问，untouched time 很短

说明内存压力大，所有页都频繁被访问，可能出现 **thrashing / 抖动**。

SRM 可能采取的行动：

* 降低 multiprogramming level / 多道程序程度；
* 暂停或换出一些进程；
* 给活跃进程更多内存；
* 调整作业优先级；
* 重新配置系统参数。

中文理解：

> 如果几乎所有页都经常被访问，说明内存很紧张，系统可能来回换页，性能下降。

---

## 考试版答案

**English:**
The purpose is to measure how actively real memory page frames are being used. If many frames remain untouched for a long time, SRM can identify them as good candidates for page replacement or reallocation. If frames are touched very frequently, this indicates high memory pressure and possible thrashing; SRM may reduce the degree of multiprogramming, swap out some address spaces, or adjust memory allocation to improve performance.

**中文：**
这样做的目的是衡量实存页框的活跃程度。如果许多页框长时间未被访问，SRM 可以把它们作为页面置换或重新分配的候选对象；如果页框频繁被访问，说明内存压力较大，可能出现抖动，SRM 可以降低多道程序程度、换出部分进程或重新调整内存分配，以改善系统性能。

---

# Review Question 2.6

这题和 2.1 的 **OS as Resource Manager / 操作系统作为资源管理器** 有关系，但具体内容更偏后续内存管理章节。

## 原题 / Original Question

**List and briefly explain five storage management responsibilities of a typical OS.**
列出并简要解释典型操作系统的五个存储管理职责。

## 答案 / Answer

这里的 **storage management / 存储管理** 可以理解为 OS 对内存和外存资源的管理。

典型 OS 的五个职责可以写成：

---

### 1. Process isolation / 进程隔离

The OS must prevent one process from interfering with the memory space of another process.

操作系统要防止一个进程随意访问或破坏另一个进程的内存空间。

例如进程 A 不能随便修改进程 B 的数据。

---

### 2. Automatic allocation and management / 自动分配与管理

The OS automatically allocates and deallocates memory and storage resources.

操作系统负责自动分配和回收内存、磁盘空间等资源。

例如程序运行时需要内存，OS 分配；程序结束后，OS 回收。

---

### 3. Support for modular programming / 支持模块化程序设计

The OS should support programs divided into modules, such as code modules, data modules, shared libraries, and dynamically linked libraries.

操作系统要支持程序分模块组织，例如代码段、数据段、共享库、动态链接库等。

---

### 4. Protection and access control / 保护与访问控制

The OS must control who can access which memory areas or files.

操作系统要控制用户和程序对内存、文件、设备的访问权限。

例如只读文件不能被普通用户修改，用户进程不能访问内核内存。

---

### 5. Long-term storage / 长期存储

The OS must manage files and persistent storage on disks or other storage devices.

操作系统要管理长期保存的数据，例如磁盘文件、目录、文件系统结构等。

---

## 考试版答案

**English:**
Five storage management responsibilities of an OS are process isolation, automatic allocation and management, support for modular programming, protection and access control, and long-term storage management.

**中文：**
操作系统的五个存储管理职责包括：进程隔离、自动分配与管理、支持模块化程序设计、保护与访问控制、长期存储管理。

---

# 不建议现在重点解的题目

这些题也在 Chapter 2 Questions 里，但不是 2.1 的重点：

|                                          题号 | 原因                         |
| ------------------------------------------: | -------------------------- |
|                 Review 2.3 multiprogramming | 属于后面 OS evolution / 多道程序部分 |
|                          Review 2.4 process | 属于进程章节重点                   |
|                Review 2.5 execution context | 属于进程上下文切换                  |
|  Review 2.7 real address vs virtual address | 属于虚拟内存                     |
|                      Review 2.8 round-robin | 属于调度                       |
| Review 2.9 monolithic kernel vs microkernel | 属于 OS 结构                   |
|                  Review 2.10 multithreading | 属于线程                       |
|                   Review 2.11 SMP OS design | 属于多处理器                     |
|                             Problem 2.1–2.3 | 属于调度和多道程序                  |
|                                 Problem 2.6 | 属于资源分配/死锁/多处理器，不是 2.1 核心   |

---

# 最后背诵版

和 2.1 最相关的题目，优先背这几个：

1. **2.1 What are three objectives of an OS design?**
   Convenience, efficiency, ability to evolve.
   方便性、效率性、可演化性。

2. **2.2 What is the kernel of an OS?**
   The core part of the OS containing the most frequently used functions.
   内核是操作系统核心部分，包含最常用的 OS 功能。

3. **Problem 2.4 What is the purpose of system calls?**
   System calls provide a controlled interface for user programs to request OS services and switch from user mode to kernel mode.
   系统调用让用户程序受控地请求 OS 服务，并从用户态进入内核态。

4. **Problem 2.5 What is the purpose of SRM checking page frames?**
   To monitor memory usage, find inactive pages, guide page replacement, and avoid memory pressure or thrashing.
   目的是监控内存使用情况，发现不活跃页面，指导页面置换，并避免内存压力和抖动。

