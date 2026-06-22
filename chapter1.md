好的，后续我都按这种格式讲：

> **英文原文 / English**
> **中文解释 / Chinese explanation**
> **考点总结 / Exam points**

下面先把“中断 Interrupts”核心考点改成**中英对照版**。材料来自你上传的 *1.4 Interrupts*。

---

## 1. What is an interrupt? / 什么是中断？

**English original idea:**

> Virtually all computers provide a mechanism by which other modules, such as I/O and memory, may interrupt the normal sequencing of the processor.

**中文解释：**

几乎所有计算机都有一种机制，允许 I/O 设备、内存模块或其他硬件打断 CPU 正常执行程序的顺序。这种机制就叫做 **interrupt，中断**。

**考点总结：**

中断就是：

> CPU 正在执行程序时，被某个事件打断，转去执行中断处理程序，处理完后再回来继续原程序。

---

## 2. Why interrupts are needed? / 为什么需要中断？

**English original idea:**

> Interrupts are provided primarily as a way to improve processor utilization.

**中文解释：**

中断主要是为了提高 **processor utilization，处理器利用率**。

没有中断时，CPU 发出 I/O 命令后可能必须等待设备完成。因为 I/O 设备通常远慢于 CPU，所以 CPU 会浪费大量时间。

有中断时，CPU 发出 I/O 命令后可以继续执行别的程序。等 I/O 设备完成后，再通过中断通知 CPU。

**考点总结：**

> 中断的主要目的：提高 CPU 利用率。
> The main purpose of interrupts is to improve processor utilization.

---

## 3. Classes of interrupts / 中断的分类

**English terms:**

| English                    | 中文     | 例子               |
| -------------------------- | ------ | ---------------- |
| Program interrupt          | 程序中断   | 除零、溢出、非法指令、越界访问  |
| Timer interrupt            | 时钟中断   | 操作系统定期获得 CPU 控制权 |
| I/O interrupt              | 输入输出中断 | 磁盘完成读写、打印机完成输出   |
| Hardware failure interrupt | 硬件故障中断 | 掉电、内存奇偶校验错误      |

**中文解释：**

教材把中断分成四类：程序中断、时钟中断、I/O 中断、硬件故障中断。

其中最常考的是：

* **I/O interrupt**：设备完成操作后通知 CPU；
* **Timer interrupt**：用于操作系统定期调度进程；
* **Program interrupt**：程序执行出错，例如除零、越界；
* **Hardware failure**：硬件故障，例如掉电。

**考点总结：**

记忆口诀：

> **程序、时钟、I/O、硬件故障。**

---

## 4. Interrupts and the instruction cycle / 中断与指令周期

**English original idea:**

> To accommodate interrupts, an interrupt stage is added to the instruction cycle.

**中文解释：**

普通指令周期是：

> Fetch stage → Execute stage
> 取指阶段 → 执行阶段

加入中断后，变成：

> Fetch stage → Execute stage → Interrupt stage
> 取指阶段 → 执行阶段 → 中断检查阶段

CPU 通常是在**执行完当前指令之后**，才检查是否有中断请求。

**考点总结：**

考试常问：

> CPU does not usually interrupt an instruction in the middle.
> CPU 通常不会在一条指令执行到一半时响应中断，而是在当前指令完成后响应。

---

## 5. Interrupt handler / 中断处理程序

**English term:**

> Interrupt handler
> Interrupt service routine, ISR

**中文解释：**

**interrupt handler** 或 **ISR** 指的是中断处理程序。它通常是操作系统的一部分。

当中断发生时，CPU 会暂停当前程序，跳转到对应的 ISR。ISR 负责判断中断来源并执行相应处理。

例如打印机发出中断，打印机 ISR 可能会继续发送下一批打印数据。

**考点总结：**

> ISR = Interrupt Service Routine
> 中断服务程序 / 中断处理程序

---

## 6. Simple interrupt processing / 简单中断处理流程

**English sequence idea:**

1. Device issues an interrupt signal.
2. Processor finishes execution of current instruction.
3. Processor acknowledges the interrupt.
4. Processor saves PSW and PC.
5. Processor loads new PC value based on the interrupt.
6. Interrupt handler saves remaining process state.
7. Interrupt is processed.
8. Process state is restored.
9. Old PSW and PC are restored.

**中文解释：**

中断处理流程可以分为硬件部分和软件部分。

硬件部分：

1. 设备向 CPU 发出中断信号。
2. CPU 执行完当前指令。
3. CPU 检查到中断请求，并向设备发送确认信号。
4. CPU 保存当前程序的 **PSW** 和 **PC**。
5. CPU 将 **PC** 改成中断处理程序的入口地址。

软件部分：

6. 中断处理程序保存其他寄存器等现场信息。
7. 执行具体的中断处理。
8. 恢复寄存器等现场信息。
9. 恢复原来的 PSW 和 PC，返回原程序。

**考点总结：**

可以背成：

> Interrupt request → finish current instruction → save state → jump to ISR → process interrupt → restore state → return
> 中断请求 → 执行完当前指令 → 保存现场 → 跳转 ISR → 处理中断 → 恢复现场 → 返回原程序

---

## 7. PC and PSW / 程序计数器和程序状态字

**English terms:**

> Program Counter, PC
> Program Status Word, PSW

**中文解释：**

**PC，Program Counter，程序计数器**
保存下一条要执行的指令地址。

**PSW，Program Status Word，程序状态字**
保存当前程序的状态信息，比如条件码、中断允许位、用户态/内核态等。

中断时至少要保存 PC 和 PSW。否则中断处理完后，CPU 不知道原程序从哪里继续，也不知道原来的运行状态。

**考点总结：**

> PC tells where to resume.
> PC 决定中断结束后从哪里继续执行。

> PSW records the processor state.
> PSW 保存处理器状态。

---

## 8. Save and restore state / 保存现场与恢复现场

**English original idea:**

> It is important to save all of the state information about the interrupted program for later resumption.

**中文解释：**

中断不是用户程序主动调用的函数，而是可能在任意时刻发生。因此，系统必须保存被中断程序的全部关键状态。

需要保存的内容包括：

* PC；
* PSW；
* 通用寄存器；
* 栈指针；
* 其他进程状态信息。

中断处理完成后，再恢复这些内容，使原程序能够继续运行。

**考点总结：**

> 保存现场是为了之后恢复执行。
> Saving the state allows the interrupted program to resume correctly.

---

## 9. Interrupts vs polling / 中断与轮询

**English contrast:**

> Without interrupts, the program may wait by repeatedly testing whether the I/O operation is done.

**中文解释：**

没有中断时，CPU 可能需要不断检查 I/O 设备是否完成，这叫 **polling，轮询**。

轮询类似于 CPU 一直问设备：

> “你好了吗？你好了吗？你好了吗？”

中断则是设备主动通知 CPU：

> “我好了，你来处理吧。”

**考点总结：**

| Method    | 中文 | 特点                  |
| --------- | -- | ------------------- |
| Polling   | 轮询 | CPU 主动反复检查设备，浪费 CPU |
| Interrupt | 中断 | 设备主动通知 CPU，提高效率     |

---

## 10. Multiple interrupts / 多重中断

**English term:**

> Multiple interrupts

**中文解释：**

多重中断指的是：

> 一个中断正在处理时，又来了另一个中断。

比如 CPU 正在处理打印机中断，这时通信设备又发来了中断。

---

## 11. Sequential interrupt processing / 顺序中断处理

**English original idea:**

> The first approach is to disable interrupts while an interrupt is being processed.

**中文解释：**

第一种方法是在处理中断时关闭其他中断。

也就是说，CPU 正在处理一个中断时，不响应新的中断。新的中断保持 pending 状态，等当前中断处理完后再处理。

**优点：**

简单，按顺序处理。

**缺点：**

不能处理紧急情况。高优先级中断可能被拖延。

**考点总结：**

> Disable interrupts during interrupt handling.
> 处理中断时关闭中断。

---

## 12. Nested interrupt processing / 嵌套中断处理

**English original idea:**

> A second approach is to define priorities for interrupts and to allow an interrupt of higher priority to cause a lower-priority interrupt handler to be interrupted.

**中文解释：**

第二种方法是设置中断优先级。

高优先级中断可以打断低优先级中断处理程序。

例如优先级：

> Communication line > Disk > Printer
> 通信设备 > 磁盘 > 打印机

如果 CPU 正在处理打印机中断，这时通信设备中断来了，因为通信设备优先级更高，所以可以打断打印机中断，先处理通信中断。

**考点总结：**

> Higher-priority interrupts can interrupt lower-priority interrupt handlers.
> 高优先级中断可以打断低优先级中断处理程序。

---

## 13. 最重要的英文术语表

| English                        | 中文         |
| ------------------------------ | ---------- |
| interrupt                      | 中断         |
| interrupt handler              | 中断处理程序     |
| interrupt service routine, ISR | 中断服务程序     |
| interrupt request              | 中断请求       |
| processor utilization          | 处理器利用率     |
| instruction cycle              | 指令周期       |
| fetch stage                    | 取指阶段       |
| execute stage                  | 执行阶段       |
| interrupt stage                | 中断检查阶段     |
| program counter, PC            | 程序计数器      |
| program status word, PSW       | 程序状态字      |
| control stack                  | 控制栈        |
| polling                        | 轮询         |
| pending interrupt              | 挂起/等待处理的中断 |
| multiple interrupts            | 多重中断       |
| nested interrupt               | 嵌套中断       |
| interrupt priority             | 中断优先级      |

---

## 14. 考试答题模板：中英对照版

**Question: What is an interrupt? Describe the interrupt processing sequence.**
**问题：什么是中断？简述中断处理过程。**

可以答：

> An interrupt is a mechanism by which other modules, such as I/O devices, timers, memory, or hardware, interrupt the normal execution sequence of the processor.
> 中断是一种机制，允许 I/O 设备、定时器、内存或硬件故障等事件打断 CPU 的正常执行顺序。
>
> When an interrupt occurs, the processor finishes the current instruction, saves the current state including PC and PSW, loads the address of the interrupt handler, and executes the interrupt service routine. After the interrupt is processed, the saved state is restored and the original program resumes execution.
> 当中断发生时，CPU 先执行完当前指令，然后保存当前程序状态，包括 PC 和 PSW，再装入中断处理程序的入口地址并执行中断服务程序。中断处理完成后，恢复原来的状态，继续执行被中断的程序。
>
> Interrupts improve processor utilization because the CPU does not need to wait idly for slow I/O devices.
> 中断可以提高处理器利用率，因为 CPU 不必在慢速 I/O 设备工作时一直空等。

后续你发任何英文课件/教材内容，我都按这个中英对照方式讲。




下面解答 **Review Questions 1.1–1.10** 和 **Problems 1.1–1.14**。题目来自你上传的 *chpater1 questions.pdf*，其中 Review Questions 列在第 1 页，Problems 1.1–1.14 分布在第 1–3 页。

---

# Review Questions

## 1.1

**English original:**
List and briefly define the four main elements of a computer.

**中文：**
列出并简要定义计算机的四个主要组成部分。

**Answer / 解答：**

1. **Processor / 处理器**
   Executes instructions and controls the operation of the computer.
   执行指令，并控制整个计算机系统的运行。

2. **Main memory / 主存储器**
   Stores programs and data that are currently being used.
   存放当前正在使用的程序和数据。

3. **I/O modules / 输入输出模块**
   Move data between the computer and external devices.
   负责计算机与外部设备之间的数据传输。

4. **System bus / 系统总线**
   Provides communication among processor, memory, and I/O modules.
   连接处理器、内存和 I/O 模块，用于传输数据、地址和控制信号。

---

## 1.2

**English original:**
Define the two main categories of processor registers.

**中文：**
定义处理器寄存器的两大类。

**Answer / 解答：**

1. **User-visible registers / 用户可见寄存器**
   These can be referenced by machine-language instructions. They reduce memory accesses.
   机器指令可以直接使用的寄存器，用于减少访问内存的次数。比如通用寄存器、数据寄存器、地址寄存器等。

2. **Control and status registers / 控制和状态寄存器**
   Used by the processor and operating system to control program execution.
   由 CPU 和操作系统使用，用来控制程序执行状态。比如：

   * **PC, Program Counter / 程序计数器**
   * **IR, Instruction Register / 指令寄存器**
   * **MAR, Memory Address Register / 存储器地址寄存器**
   * **MBR, Memory Buffer Register / 存储器缓冲寄存器**
   * **PSW, Program Status Word / 程序状态字**

---

## 1.3

**English original:**
In general terms, what are the four distinct actions that a machine instruction can specify?

**中文：**
机器指令通常可以指定哪四类操作？

**Answer / 解答：**

1. **Processor-memory / 处理器与内存之间的数据传送**
   Data may be transferred from processor to memory or from memory to processor.
   例如 load、store。

2. **Processor-I/O / 处理器与 I/O 之间的数据传送**
   Data may be transferred between processor and I/O modules.
   例如输入输出指令。

3. **Data processing / 数据处理**
   Arithmetic or logical operations on data.
   例如加法、减法、逻辑与、逻辑或。

4. **Control / 控制操作**
   Change the sequence of instruction execution.
   例如 jump、branch、call、return。

---

## 1.4

**English original:**
What is an interrupt?

**中文：**
什么是中断？

**Answer / 解答：**

**Interrupt / 中断** is a mechanism by which other modules, such as I/O devices, memory, timers, or hardware, can interrupt the normal sequence of processor execution.

中断就是：CPU 正在执行程序时，被 I/O 设备、定时器、程序异常或硬件故障等事件打断，转去执行 **interrupt handler / interrupt service routine，中断处理程序**，处理完后再回到原程序继续执行。

教材 1.4 中强调，中断可以让 CPU 在 I/O 设备工作时继续执行其他指令，从而提高 **processor utilization / 处理器利用率**。

---

## 1.5

**English original:**
How are multiple interrupts dealt with?

**中文：**
如何处理多个中断？

**Answer / 解答：**

There are two main approaches.

有两种主要方法：

### 1. Sequential interrupt processing / 顺序中断处理

Disable interrupts while an interrupt is being processed.

处理中断时关闭其他中断。新来的中断保持 **pending / 挂起等待**，等当前中断处理完后再处理。

优点：简单。
缺点：不能考虑优先级，紧急中断可能被延迟。

### 2. Nested interrupt processing / 嵌套中断处理

Assign priorities to interrupts. A higher-priority interrupt can interrupt a lower-priority interrupt handler.

给中断设置优先级。高优先级中断可以打断低优先级中断处理程序。

例如：

```text
Communication interrupt > Disk interrupt > Printer interrupt
通信中断 > 磁盘中断 > 打印机中断
```

教材 1.4 的 multiple interrupts 部分正是这样区分顺序处理和嵌套处理的。

---

## 1.6

**English original:**
What characteristics distinguish the various elements of a memory hierarchy?

**中文：**
存储器层次结构中，不同层次主要由什么特征区分？

**Answer / 解答：**

Memory hierarchy elements are distinguished mainly by:

1. **Capacity / 容量**
   Higher levels are smaller; lower levels are larger.
   越靠近 CPU，容量越小；越往下，容量越大。

2. **Access time / 访问时间**
   Higher levels are faster; lower levels are slower.
   越靠近 CPU，访问越快；越往下，访问越慢。

3. **Cost per bit / 每位成本**
   Higher levels are more expensive per bit.
   越快的存储器，每 bit 成本越高。

4. **Frequency of access / 访问频率**
   Higher levels are accessed more frequently.
   CPU 更频繁访问靠近自己的高速层次。

一句话：

> The higher the level, the faster, smaller, and more expensive it is.
> 越靠近 CPU，越快、越小、越贵。

---

## 1.7

**English original:**
What is cache memory?

**中文：**
什么是高速缓存？

**Answer / 解答：**

**Cache memory / 高速缓存** is a small, fast memory placed between the processor and main memory.

Cache stores copies of recently or frequently used memory blocks, so the CPU can access them faster.

中文解释：

Cache 是位于 CPU 和主存之间的小容量高速存储器，用来保存最近访问过或很可能马上访问的数据和指令。它利用了：

* **Temporal locality / 时间局部性**
* **Spatial locality / 空间局部性**

目的是降低平均访存时间。

---

## 1.8

**English original:**
What is the difference between a multiprocessor and a multicore system?

**中文：**
多处理器系统和多核系统有什么区别？

**Answer / 解答：**

**Multiprocessor system / 多处理器系统**
A system with two or more separate processors.
有多个独立处理器，通常可能是多个 CPU 芯片或多个处理器插槽。

**Multicore system / 多核系统**
A single processor chip contains multiple processing cores.
一个 CPU 芯片内部集成多个核心。

区别：

> Multiprocessor emphasizes multiple processors.
> Multicore emphasizes multiple cores on one chip.

---

## 1.9

**English original:**
What is the distinction between spatial locality and temporal locality?

**中文：**
空间局部性和时间局部性有什么区别？

**Answer / 解答：**

**Temporal locality / 时间局部性**
If a memory location is referenced, it is likely to be referenced again soon.
一个数据刚被访问过，短时间内很可能再次被访问。

例子：

```c
sum = sum + a[i];
```

变量 `sum` 被反复使用。

**Spatial locality / 空间局部性**
If a memory location is referenced, nearby locations are likely to be referenced soon.
一个地址被访问后，它附近的地址也很可能很快被访问。

例子：

```c
a[0], a[1], a[2], a[3]
```

数组连续访问。

---

## 1.10

**English original:**
In general, what are the strategies for exploiting spatial locality and temporal locality?

**中文：**
通常如何利用空间局部性和时间局部性？

**Answer / 解答：**

To exploit **temporal locality / 时间局部性**:

> Keep recently used data and instructions in faster memory.
> 把最近访问过的数据和指令保存在高速存储器中。

比如 cache 保存最近使用的块。

To exploit **spatial locality / 空间局部性**:

> Fetch blocks of adjacent memory locations, not just one word.
> 一次取入一整块连续地址，而不是只取一个字。

比如 cache line 一次加载一段连续内存。

---

# Problems

## Problem 1.1

**English original summary:**
The processor has two I/O instructions:

```text
0011 = Load AC from I/O
0111 = Store AC to I/O
```

Program:

```text
1. Load AC from device 5.
2. Add contents of memory location 940.
3. Store AC to device 6.
```

Assume device 5 gives value 3, and memory location 940 contains 2.

**中文：**
增加两个 I/O 指令：从设备读入 AC，把 AC 写到设备。程序要从设备 5 读入 3，再加上内存 940 中的 2，最后把结果写到设备 6。

教材 Figure 1.4 的指令格式是 4 位十六进制：最高位是 opcode，后 3 位是地址或设备号。Figure 1.4 中类似 `1940`、`5941`、`2941` 的执行流程就是取指、执行、更新 PC 和 AC。

### Program memory / 程序内存

```text
300: 3005    Load AC from I/O device 5
301: 5940    Add M[940] to AC
302: 7006    Store AC to I/O device 6

940: 0002
Device 5 input: 0003
```

### Execution / 执行过程

| Step | English                        | 中文           | Result                  |
| ---- | ------------------------------ | ------------ | ----------------------- |
| 1    | Fetch instruction at 300       | 从 300 取指令    | IR = 3005, PC = 301     |
| 2    | Execute load from I/O device 5 | 从设备 5 读入 AC  | AC = 0003               |
| 3    | Fetch instruction at 301       | 从 301 取指令    | IR = 5940, PC = 302     |
| 4    | Execute add M[940]             | 加上内存 940 的内容 | AC = 0003 + 0002 = 0005 |
| 5    | Fetch instruction at 302       | 从 302 取指令    | IR = 7006, PC = 303     |
| 6    | Execute store to I/O device 6  | 把 AC 写到设备 6  | Device 6 receives 0005  |

**Final answer / 最终结果：**

```text
AC = 0005
Device 6 = 0005
```

---

## Problem 1.2

**English original:**
The program execution of Figure 1.4 is described in six steps. Expand this description to show the use of MAR and MBR.

**中文：**
把 Figure 1.4 的六步执行过程扩展出来，说明 MAR 和 MBR 的使用。

### Terms / 术语

| Register | English                 | 中文       |
| -------- | ----------------------- | -------- |
| MAR      | Memory Address Register | 存储器地址寄存器 |
| MBR      | Memory Buffer Register  | 存储器缓冲寄存器 |
| PC       | Program Counter         | 程序计数器    |
| IR       | Instruction Register    | 指令寄存器    |
| AC       | Accumulator             | 累加器      |

### Expanded execution / 扩展执行过程

Initial state:

```text
PC = 300
M[300] = 1940
M[301] = 5941
M[302] = 2941
M[940] = 0003
M[941] = 0002
```

| Step | Operation                                                                       |
| ---- | ------------------------------------------------------------------------------- |
| 1    | `MAR ← PC = 300`; `MBR ← M[MAR] = 1940`; `IR ← MBR = 1940`; `PC ← PC + 1 = 301` |
| 2    | `MAR ← 940`; `MBR ← M[940] = 0003`; `AC ← MBR = 0003`                           |
| 3    | `MAR ← PC = 301`; `MBR ← M[301] = 5941`; `IR ← MBR = 5941`; `PC ← 302`          |
| 4    | `MAR ← 941`; `MBR ← M[941] = 0002`; `AC ← AC + MBR = 0003 + 0002 = 0005`        |
| 5    | `MAR ← PC = 302`; `MBR ← M[302] = 2941`; `IR ← MBR = 2941`; `PC ← 303`          |
| 6    | `MAR ← 941`; `MBR ← AC = 0005`; `M[941] ← MBR = 0005`                           |

**Final answer / 最终结果：**

```text
M[941] = 0005
AC = 0005
PC = 303
```

---

## Problem 1.3

**English original summary:**
A 32-bit microprocessor has 32-bit instructions. The first byte is opcode, and the remaining 24 bits are immediate operand or address.

**中文：**
一个 32 位微处理器，指令长度 32 位。第 1 个字节是操作码，剩下 24 位是立即数或地址。

---

### 1.3(a)

**Question:**
What is the maximum directly addressable memory capacity?

**解答：**

Address field = 24 bits.

```text
Maximum addressable locations = 2^24
```

If byte-addressable:

```text
2^24 bytes = 16 MB
```

**Answer:**

```text
16 MB
```

---

### 1.3(b)

**Question:**
Discuss the impact if the bus has:

1. 32-bit address bus and 16-bit data bus
2. 16-bit address bus and 16-bit data bus

**解答：**

#### Case 1: 32-bit address bus, 16-bit data bus

地址总线 32 位，足够表示 24 位地址，地址能力不是瓶颈。

但是数据总线只有 16 位，所以一次只能传 16 bit = 2 bytes。

A 32-bit instruction requires two data-bus transfers.

```text
32-bit instruction / 16-bit data bus = 2 bus transfers
```

所以速度会比 32-bit data bus 慢。

#### Case 2: 16-bit address bus, 16-bit data bus

地址总线只有 16 位，一次最多直接发出：

```text
2^16 addresses = 64 KB
```

这比指令中 24 位地址能表示的 16 MB 小得多。

所以若要访问更大内存，需要 bank switching、segmentation、paging 或额外地址寄存器等机制，会更慢、更复杂。

同时数据总线仍然只有 16 位，所以 32 位指令仍然要两次传输。

---

### 1.3(c)

**Question:**
How many bits are needed for the PC and IR?

**解答：**

* PC must hold an instruction address. Since directly addressable space is 24 bits:

```text
PC = 24 bits
```

* IR must hold the whole instruction:

```text
IR = 32 bits
```

**Answer:**

```text
PC = 24 bits
IR = 32 bits
```

---

## Problem 1.4

**English original summary:**
A microprocessor generates 16-bit addresses and has a 16-bit data bus.

**中文：**
一个微处理器产生 16 位地址，并且有 16 位数据总线。

---

### 1.4(a)

**Question:**
Maximum memory address space if connected to a 16-bit memory?

**解答：**

16-bit address means:

```text
2^16 = 65536 addresses
```

If each address refers to a 16-bit word:

```text
65536 words × 2 bytes/word = 131072 bytes = 128 KiB
```

**Answer:**

```text
65536 16-bit words = 128 KiB
```

---

### 1.4(b)

**Question:**
Maximum memory address space if connected to an 8-bit memory?

**解答：**

If each address refers to 1 byte:

```text
2^16 bytes = 65536 bytes = 64 KiB
```

**Answer:**

```text
64 KiB
```

---

### 1.4(c)

**Question:**
What architectural features allow access to a separate I/O space?

**解答：**

需要：

1. **Separate I/O instructions / 单独的 I/O 指令**
   例如 `IN`、`OUT`。

2. **Separate I/O address space / 独立 I/O 地址空间**
   I/O 地址和内存地址分开。

3. **Control signals to distinguish memory and I/O / 区分内存访问和 I/O 访问的控制信号**
   例如 memory read/write 和 I/O read/write 信号。

**Answer:**

The processor needs separate I/O instructions and control lines that distinguish memory references from I/O references.

---

### 1.4(d)

**Question:**
If an input/output instruction can specify an 8-bit I/O port number, how many 8-bit and 16-bit ports are supported?

**解答：**

8-bit port number gives:

```text
2^8 = 256 port numbers
```

If one port number names one port:

```text
256 8-bit I/O ports
256 16-bit I/O ports
```

因为端口号数量是 256，不是按内存字节地址计算。

补充：如果某个体系结构规定 16-bit port 必须占用两个 8-bit port numbers，那么就是 128 个 16-bit ports。但按题目通常理解，port number 直接编号端口，所以答案是：

```text
256 8-bit ports
256 16-bit ports
```

---

## Problem 1.5

**English original summary:**
A 32-bit microprocessor has a 16-bit external data bus, 8-MHz input clock, and one bus cycle takes 4 clock cycles.

**中文：**
32 位微处理器，外部数据总线 16 位，输入时钟 8 MHz，一个总线周期需要 4 个时钟周期。

---

### Step 1: Bus cycles per second

```text
Clock = 8 MHz = 8,000,000 cycles/s
1 bus cycle = 4 clock cycles
Bus cycles/s = 8,000,000 / 4 = 2,000,000
```

### Step 2: Bytes per bus cycle

16-bit data bus:

```text
16 bits = 2 bytes
```

### Step 3: Maximum transfer rate

```text
2,000,000 bus cycles/s × 2 bytes/bus cycle
= 4,000,000 bytes/s
```

**Answer:**

```text
Maximum transfer rate = 4,000,000 bytes/s = 4 MB/s
```

### Which improvement is better?

#### Option 1: Make data bus 32 bits

```text
32 bits = 4 bytes
2,000,000 cycles/s × 4 bytes = 8,000,000 bytes/s
```

#### Option 2: Double clock to 16 MHz

```text
16,000,000 / 4 = 4,000,000 bus cycles/s
4,000,000 × 2 bytes = 8,000,000 bytes/s
```

**Conclusion / 结论：**

Under ideal assumptions, both improvements double the transfer rate:

```text
Both give 8 MB/s
```

前提是内存和外设都能跟上，没有额外 wait states。

---

## Problem 1.6

**English original summary:**
A keyboard/printer Teletype uses:

```text
INPR: Input Register
OUTR: Output Register
FGI: Input Flag
FGO: Output Flag
IEN: Interrupt Enable
```

题目要求先只用前四个寄存器实现 I/O，再说明加入 IEN 后如何更高效。

---

### 1.6(a)

**Question:**
Describe how CPU uses INPR, OUTR, FGI, FGO to achieve I/O.

**解答：**

This is **programmed I/O / 程序控制 I/O**, also called **polling / 轮询**.

#### Input / 输入

```text
while FGI == 0:
    keep checking

read INPR
clear FGI
```

中文解释：

CPU 不断检查 FGI。
如果 FGI = 0，说明键盘还没有输入字符。
如果 FGI = 1，说明 INPR 中已经有一个 8-bit 字符，CPU 读取 INPR，然后清除 FGI。

#### Output / 输出

```text
while FGO == 0:
    keep checking

write character to OUTR
clear FGO
```

中文解释：

CPU 不断检查 FGO。
如果 FGO = 0，说明打印机还没准备好。
如果 FGO = 1，说明打印机可以接收新字符，CPU 把字符写入 OUTR，然后清除 FGO。

**缺点：**

CPU wastes time repeatedly checking flags.
CPU 会浪费时间反复检查标志位。

---

### 1.6(b)

**Question:**
How can it be more efficient by employing IEN?

**解答：**

Using **IEN / Interrupt Enable / 中断允许位**, the system can use **interrupt-driven I/O / 中断驱动 I/O**.

Process:

1. CPU sets:

```text
IEN = 1
```

2. CPU continues executing other instructions.

3. When keyboard input arrives:

```text
FGI = 1
I/O module generates interrupt
```

4. When printer is ready:

```text
FGO = 1
I/O module generates interrupt
```

5. CPU enters interrupt handler, checks whether FGI or FGO caused the interrupt, performs input or output, clears the flag, and returns.

这对应教材 1.4 中的中断处理过程：设备发出中断信号，CPU 执行完当前指令后响应，保存 PC 和 PSW，进入中断处理程序，处理完后恢复现场。

**Conclusion / 结论：**

```text
Without IEN: polling
With IEN: interrupt-driven I/O
```

---

## Problem 1.7

**English original:**
In virtually all systems that include DMA modules, DMA access to main memory is given higher priority than processor access to main memory. Why?

**中文：**
为什么在包含 DMA 模块的系统中，DMA 访问主存通常比处理器访问主存优先级更高？

**Answer / 解答：**

DMA transfers data directly between I/O devices and main memory.

DMA 直接在 I/O 设备和主存之间传输数据。

DMA should have higher priority because I/O devices often have strict timing requirements and limited buffers. If DMA is delayed, the device buffer may overflow or underflow, causing data loss.

中文解释：

I/O 设备通常有固定速度，比如磁盘、网卡、串口。如果 DMA 不能及时把数据搬到内存，设备缓冲区可能溢出，导致数据丢失。

CPU 被延迟几个内存周期通常只是变慢，不会丢数据。

所以：

```text
DMA delay may cause data loss.
CPU delay usually only causes slower execution.
```

因此 DMA 访问主存优先级更高。

---

## Problem 1.8

**English original summary:**
A DMA module transfers characters from a device at 9600 bps. Processor fetches instructions at 1 million instructions/s. How much is the processor slowed?

**中文：**
DMA 设备速率 9600 bps，处理器每秒取 100 万条指令。DMA 会让处理器慢多少？

**解答：**

Assume 1 character = 8 bits.

```text
9600 bits/s ÷ 8 bits/character = 1200 characters/s
```

Assume each character DMA transfer steals one memory cycle that could otherwise be used for instruction fetch.

Processor instruction fetch rate:

```text
1,000,000 instructions/s
```

Slowdown:

```text
1200 / 1,000,000 = 0.0012 = 0.12%
```

**Answer / 答案：**

```text
Processor is slowed by about 0.12%.
```

也可以说：

```text
Instruction fetch capacity decreases by about 1200 instructions/s.
```

---

## Problem 1.9

**English original summary:**
CPU maximum rate = 10^6 instructions/s.
Average instruction = 5 processor cycles.
3 of these cycles use the memory bus.
Background programs require 95% of instruction rate.
One processor cycle = one bus cycle.
Estimate programmed I/O and DMA transfer rates.

**中文：**
CPU 每秒最多执行 100 万条指令。平均每条指令 5 个处理器周期，其中 3 个周期使用内存总线。后台程序占用 95% 的指令执行能力。求 programmed I/O 和 DMA 的最大传输率。

---

### Basic data / 基本数据

CPU full instruction rate:

```text
1,000,000 instructions/s
```

Each instruction:

```text
5 cycles/instruction
```

So total processor cycles per second:

```text
1,000,000 × 5 = 5,000,000 cycles/s
```

Background uses 95% instruction rate:

```text
0.95 × 1,000,000 = 950,000 instructions/s
```

Background CPU cycles:

```text
950,000 × 5 = 4,750,000 cycles/s
```

Remaining CPU cycles:

```text
5,000,000 - 4,750,000 = 250,000 cycles/s
```

Background memory-bus cycles:

```text
950,000 × 3 = 2,850,000 bus cycles/s
```

Remaining bus cycles:

```text
5,000,000 - 2,850,000 = 2,150,000 bus cycles/s
```

---

### 1.9(a) Programmed I/O

Each one-word I/O transfer requires 2 CPU instructions.

剩余 CPU 指令能力：

```text
5% × 1,000,000 = 50,000 instructions/s
```

每传 1 个 word 需要 2 条指令：

```text
50,000 / 2 = 25,000 words/s
```

**Answer:**

```text
Programmed I/O rate = 25,000 words/s
```

---

### 1.9(b) DMA

DMA does not require CPU instructions for each word transfer. It mainly consumes bus cycles.

Available bus cycles:

```text
2,150,000 bus cycles/s
```

One word transfer uses one bus cycle:

```text
DMA rate = 2,150,000 words/s
```

**Answer:**

```text
DMA rate = 2.15 × 10^6 words/s
```

---

## Problem 1.10

**English original code:**

```c
for (i = 0; i < 20; i++)
    for (j = 0; j < 10; j++)
        a[i] = a[i] * j
```

**中文：**
指出这段代码中的空间局部性和时间局部性。

---

### 1.10(a) Spatial locality / 空间局部性

Array elements are accessed sequentially:

```text
a[0], a[1], a[2], ..., a[19]
```

数组元素在内存中通常连续存放，所以访问 `a[i]` 后，接下来很可能访问相邻的 `a[i+1]`。

**Answer:**

```text
Sequential access to a[0], a[1], ..., a[19] shows spatial locality.
```

---

### 1.10(b) Temporal locality / 时间局部性

For a fixed `i`, the inner loop repeatedly accesses the same `a[i]` ten times:

```text
a[i] = a[i] * 0
a[i] = a[i] * 1
...
a[i] = a[i] * 9
```

同一个 `a[i]` 在短时间内被反复访问。

**Answer:**

```text
Repeated access to the same a[i] in the inner loop shows temporal locality.
```

---

## Problem 1.11

**English original:**
Generalize Equations (1.1) and (1.2) in Appendix 1A to n-level memory hierarchies.

**中文：**
把两级存储层次的公式推广到 n 级存储层次。

---

Assume:

```text
Level 1, Level 2, ..., Level n
```

Let:

```text
T_i = access time of level i
C_i = cost per bit of level i
S_i = size of level i
H_i = conditional hit ratio at level i
```

Assume level n always contains the needed item:

```text
H_n = 1
```

---

### Average access time / 平均访问时间

For serial lookup:

```text
T_avg =
T_1
+ (1 - H_1)T_2
+ (1 - H_1)(1 - H_2)T_3
+ ...
+ [(1 - H_1)(1 - H_2)...(1 - H_{n-1})]T_n
```

更严格地写：

```text
T_avg =
Σ_{i=1}^{n}
[
    (Π_{j=1}^{i-1}(1 - H_j)) H_i
    (Σ_{k=1}^{i} T_k)
]
```

中文解释：

只有第 1 级 miss，才访问第 2 级；只有第 1、2 级都 miss，才访问第 3 级；以此类推。

---

### Average cost / 平均每 bit 成本

```text
C_avg = (C_1S_1 + C_2S_2 + ... + C_nS_n) / (S_1 + S_2 + ... + S_n)
```

也就是：

```text
C_avg = Σ(C_iS_i) / ΣS_i
```

---

## Problem 1.12

**English original summary:**

```text
Tc = 100 ns
Cc = 0.01 cents/bit
Tm = 1200 ns
Cm = 0.001 cents/bit
```

题目要求计算 1 MByte 主存成本、1 MByte 用 cache 技术的成本，以及有效访问时间比 cache 访问时间大 10% 时的 hit ratio。

这里按常见计算机教材习惯取：

```text
1 MByte = 2^20 bytes = 8,388,608 bits
```

---

### 1.12(a) Cost of 1 MByte main memory

```text
Cost = 8,388,608 bits × 0.001 cents/bit
     = 8,388.608 cents
     = $83.88608
```

**Answer:**

```text
About $83.89
```

---

### 1.12(b) Cost of 1 MByte using cache memory technology

```text
Cost = 8,388,608 bits × 0.01 cents/bit
     = 83,886.08 cents
     = $838.8608
```

**Answer:**

```text
About $838.86
```

---

### 1.12(c) Required hit ratio H

Effective access time is 10% greater than cache access time:

```text
T_eff = 1.1 × Tc = 1.1 × 100 = 110 ns
```

Using the standard two-level formula:

```text
T_eff = Tc + (1 - H)Tm
```

Substitute:

```text
110 = 100 + (1 - H) × 1200
```

```text
10 = (1 - H) × 1200
```

```text
1 - H = 10 / 1200 = 0.008333
```

```text
H = 0.991667
```

**Answer:**

```text
H ≈ 99.17%
```

---

## Problem 1.13

**English original summary:**

* Cache hit: 20 ns
* If in main memory but not cache: 60 ns to load into cache, then reference restarts
* If not in main memory: 12 ms from disk, then 60 ns to copy to cache, then reference restarts
* Cache hit ratio = 0.9
* Main-memory hit ratio = 0.6

**中文：**
求平均访问时间。

---

### Convert disk time / 转换磁盘时间

```text
12 ms = 12,000,000 ns
```

---

### Cases / 分情况

#### Case 1: Cache hit

Probability:

```text
0.9
```

Time:

```text
20 ns
```

Contribution:

```text
0.9 × 20 = 18 ns
```

---

#### Case 2: Cache miss, main memory hit

Probability:

```text
0.1 × 0.6 = 0.06
```

Time:

```text
60 ns to load into cache + 20 ns restarted cache access
= 80 ns
```

Contribution:

```text
0.06 × 80 = 4.8 ns
```

---

#### Case 3: Cache miss, main memory miss, disk access

Probability:

```text
0.1 × 0.4 = 0.04
```

Time:

```text
12,000,000 ns + 60 ns + 20 ns
= 12,000,080 ns
```

Contribution:

```text
0.04 × 12,000,080 = 480,003.2 ns
```

---

### Average access time

```text
T_avg = 18 + 4.8 + 480,003.2
      = 480,026 ns
```

**Answer:**

```text
Average access time ≈ 480,026 ns
```

也就是大约：

```text
0.480 ms
```

---

## Problem 1.14

**English original:**
Suppose a stack is to be used by the processor to manage procedure calls and returns. Can the program counter be eliminated by using the top of the stack as a program counter?

**中文：**
如果处理器用栈管理过程调用和返回，能不能取消程序计数器 PC，直接用栈顶作为 PC？

**Answer / 解答：**

No, the program counter cannot generally be eliminated.

不能。

原因：

1. **PC stores the address of the next instruction.**
   PC 保存下一条要执行的指令地址。

2. **The stack stores return addresses and procedure state.**
   栈主要保存返回地址、局部变量、参数、现场信息等。

3. **The top of stack changes frequently.**
   栈顶会随着 push、pop、函数调用、局部变量分配不断变化。它不稳定，不能直接当作当前指令地址。

4. **Sequential execution still needs a current instruction pointer.**
   顺序执行、跳转、中断返回都需要一个明确的当前执行位置。

所以：

```text
The top of stack may contain a return address, but it is not the current PC.
```

栈顶可能保存某个返回地址，但它不是当前正在执行的下一条指令地址。

如果某种特殊机器真的用“栈顶寄存器”来指示下一条指令，那它本质上仍然是在用一个 PC，只是换了名字。

**Conclusion / 结论：**

```text
No. The PC is still needed.
```

---

# 最重要的数值答案汇总

| Problem | Answer                                                  |
| ------- | ------------------------------------------------------- |
| 1.1     | Device 6 receives `0005`                                |
| 1.3(a)  | 16 MB                                                   |
| 1.3(c)  | PC = 24 bits, IR = 32 bits                              |
| 1.4(a)  | 128 KiB with 16-bit memory                              |
| 1.4(b)  | 64 KiB with 8-bit memory                                |
| 1.4(d)  | 256 8-bit ports; usually 256 16-bit ports               |
| 1.5     | 4 MB/s; widening bus or doubling clock both give 8 MB/s |
| 1.8     | about 0.12% slowdown                                    |
| 1.9(a)  | 25,000 words/s                                          |
| 1.9(b)  | 2.15 × 10^6 words/s                                     |
| 1.12(a) | about $83.89                                            |
| 1.12(b) | about $838.86                                           |
| 1.12(c) | H ≈ 99.17%                                              |
| 1.13    | about 480,026 ns                                        |
| 1.14    | No, PC cannot be eliminated                             |
