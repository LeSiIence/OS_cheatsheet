首页标注的重点是：**8.1 Hardware and Control Structures**，其中重点看 **Locality and Virtual Memory** 与 **Paging**；以及 **8.2 Operating System Software**，其中重点看 **Replacement Policy**。后面的 Segmentation、Combined Paging and Segmentation、UNIX/Solaris 可以先作为了解，不是首页重点。

## 1. Virtual Memory / 虚拟内存的核心思想

**Virtual memory（虚拟内存）**：让程序感觉自己拥有一个很大的连续内存空间，但实际上只有一部分内容在主存 **main memory / real memory（真实内存）** 中，其余内容放在磁盘等 **secondary memory（二级存储）** 中。

关键区别：

| English term                      | 中文          | 含义                    |
| --------------------------------- | ----------- | --------------------- |
| Virtual address / Logical address | 虚拟地址 / 逻辑地址 | 程序看到和使用的地址            |
| Real address / Physical address   | 实地址 / 物理地址  | 主存中真正的位置              |
| Address translation               | 地址转换        | CPU/MMU 把虚拟地址转换成物理地址  |
| Resident set                      | 驻留集         | 一个进程当前真正放在主存中的页集合     |
| Page fault                        | 缺页中断        | 访问的页不在主存中，需要 OS 从磁盘调入 |

虚拟内存能成立，靠两个基础：

1. **Dynamic run-time address translation / 运行时动态地址转换**
   程序访问的是逻辑地址，执行时才翻译成物理地址。

2. **Process can be divided into pieces / 进程可以被分成若干块**
   例如 pages（页）或 segments（段），而且这些块不必连续放在主存中。

所以，一个进程运行时不需要全部进入主存，只要当前要执行的代码页和要访问的数据页在主存中，就可以继续运行。

---

## 2. Locality and Virtual Memory / 局部性与虚拟内存

虚拟内存之所以有效，是因为有 **principle of locality（局部性原理）**：

> Program and data references tend to cluster.
> 程序和数据访问往往集中在一小部分区域。

分两类理解：

| 类型                | English | 中文解释                |
| ----------------- | ------- | ------------------- |
| Temporal locality | 时间局部性   | 最近用过的内容，很可能马上又会用    |
| Spatial locality  | 空间局部性   | 访问某个地址后，很可能访问它附近的地址 |

例如程序正在执行一个循环，那么短时间内反复访问的通常就是循环代码、循环变量、数组的一小段区域。

因此 OS 不需要把整个程序全部装入主存，而是只装入最近可能用到的页。

但是如果 OS 换页策略很差，就会出现：

**Thrashing / 抖动**：系统大部分时间都在换入换出页面，而不是执行真正的指令。

常见考法：

> 为什么虚拟内存不会因为频繁缺页而无法使用？
> 答：因为程序具有 locality，短时间内只集中访问少量页，所以只要把这些页放入 resident set，缺页率就可以保持较低。

---

## 3. Paging / 分页机制

分页是本章重点。

### 3.1 地址结构

虚拟地址通常分成：

[
\text{Virtual Address} = \text{Page Number} + \text{Offset}
]

中文：

[
\text{虚拟地址} = \text{页号} + \text{页内偏移}
]

物理地址分成：

[
\text{Physical Address} = \text{Frame Number} + \text{Offset}
]

中文：

[
\text{物理地址} = \text{页框号} + \text{页内偏移}
]

注意：**offset 不变**。分页地址转换时，只是把 page number 翻译成 frame number。

---

### 3.2 Page Table / 页表

每个进程一般有自己的 **page table（页表）**。

页表项叫：

**Page Table Entry, PTE / 页表项**

典型字段：

| 字段                          | English  | 中文解释        |
| --------------------------- | -------- | ----------- |
| Present bit, P              | 存在位      | 该页是否在主存     |
| Modified bit, M / Dirty bit | 修改位 / 脏位 | 该页是否被修改过    |
| Frame number                | 页框号      | 该页在主存中的页框位置 |
| Control bits                | 控制位      | 保护、共享、访问权限等 |

访问过程：

1. CPU 生成虚拟地址：页号 + 偏移。
2. 用页号查 page table。
3. 如果 present bit = 1，说明页在主存。
4. 取出 frame number。
5. frame number + offset 得到物理地址。
6. 如果 present bit = 0，产生 page fault。

---

## 4. TLB / 转换检测缓冲区

如果每次访存都要先查页表，再访问数据，就可能变成两次内存访问，速度很慢。

所以使用：

**Translation Lookaside Buffer, TLB / 转换检测缓冲区**

它本质上是一个很小但很快的页表缓存。

### TLB 访问流程

| 情况         | English | 中文                           |
| ---------- | ------- | ---------------------------- |
| TLB hit    | TLB 命中  | 页表项在 TLB 中，直接得到 frame number |
| TLB miss   | TLB 未命中 | 去主存中的 page table 查页表项        |
| Page fault | 缺页      | 页表项显示该页不在主存，需要 OS 从磁盘调入      |

所以完整逻辑是：

```text
CPU 产生虚拟地址
        ↓
先查 TLB
   ↓          ↓
TLB hit     TLB miss
   ↓          ↓
得到页框号   查 page table
              ↓
        present bit = 1 → 更新 TLB，访问主存
        present bit = 0 → page fault
```

容易混淆点：

**TLB miss 不等于 page fault。**

TLB miss 只是“TLB 里没有这个页表项”；
page fault 是“这个页根本不在主存”。

---

## 5. Page Fault / 缺页中断过程

当访问的页不在主存时：

1. CPU 产生 **page fault interrupt（缺页中断）**。
2. OS 接管。
3. OS 把当前进程阻塞。
4. OS 从磁盘读取需要的页。
5. 如果主存满了，需要执行 **page replacement（页面置换）**。
6. 更新 page table 和 TLB。
7. 把进程重新放回 Ready state。
8. 回到导致缺页的那条指令重新执行。

考试常问：

> 缺页中断后，是不是程序直接崩溃？
> 不是。正常缺页是虚拟内存机制的一部分，OS 会把页调入主存，然后继续执行。

---

## 6. Page Size / 页面大小的影响

页面大小是硬件和 OS 都会受影响的设计问题。

### 页面太小

优点：

* **Internal fragmentation（内部碎片）**小。

缺点：

* 页数量变多。
* 页表变大。
* TLB 覆盖范围变小。
* 可能增加页表管理开销。

### 页面太大

优点：

* 页表较小。
* 一次 I/O 可以读入更多连续内容。
* TLB 一个表项能覆盖更大内存范围。

缺点：

* 内部碎片变大。
* 可能把暂时不用的数据也调入主存。
* 局部性效果可能变差。

所以页面大小不是越大越好，也不是越小越好。

---

# 7. Replacement Policy / 页面置换策略

这是首页标注的另一个重点。

当发生 page fault，且没有空闲 frame 时，OS 必须选择一个已经在主存中的页换出去。

目标：

> Replace the page least likely to be referenced in the near future.
> 换出未来最不可能被访问的页。

但 OS 不知道未来，所以只能根据过去的访问历史猜测。

---

## 8. 常见页面置换算法

### 8.1 Optimal, OPT / 最优置换算法

规则：

> Replace the page whose next reference is farthest in the future.
> 换出下一次使用时间最晚的页。

优点：

* 缺页次数最少。
* 理论最优。

缺点：

* 需要知道未来访问序列，实际无法实现。

作用：

* 作为 benchmark / 基准，用来比较其他算法。

---

### 8.2 LRU / Least Recently Used / 最近最少使用

规则：

> Replace the page that has not been referenced for the longest time.
> 换出最长时间没有被访问的页。

思想：

* 最近很久没用，将来也可能暂时不用。
* 利用 locality。

优点：

* 效果接近 OPT。

缺点：

* 实现开销大。
* 需要记录每个页的最近访问时间，或者维护访问栈。

---

### 8.3 FIFO / First-In-First-Out / 先进先出

规则：

> Replace the page that has been in memory the longest.
> 换出最早进入主存的页。

优点：

* 非常简单。
* 用队列即可实现。

缺点：

* 效果可能较差。
* 可能把经常使用的老页面换出去。

注意：FIFO 只看“进入内存的时间”，不看“最近有没有被访问”。

---

### 8.4 Clock / 时钟算法，也叫 Second Chance / 二次机会算法

Clock 是对 FIFO 的改进，用一个 **use bit / reference bit（使用位 / 引用位）** 判断页面最近是否被访问过。

规则：

1. 页刚进入内存时，use bit = 1。
2. 页被访问时，use bit = 1。
3. 置换时指针按圆形队列扫描：

   * 如果 use bit = 0，换出该页。
   * 如果 use bit = 1，把它改成 0，并跳过，给它“第二次机会”。

所以 Clock 比 FIFO 聪明：

* FIFO：到你了就换。
* Clock：到你了但最近用过，就先放过你一次。

---

## 9. Enhanced Clock / 改进时钟算法

改进 Clock 同时考虑：

| bit | English    | 中文     |
| --- | ---------- | ------ |
| u   | use bit    | 最近是否访问 |
| m   | modify bit | 是否被修改  |

四类页面：

| 类型       | 含义         | 替换优先级     |
| -------- | ---------- | --------- |
| u=0, m=0 | 最近没访问，没修改  | 最适合换出     |
| u=0, m=1 | 最近没访问，但修改过 | 次优，需要写回磁盘 |
| u=1, m=0 | 最近访问过，没修改  | 尽量保留      |
| u=1, m=1 | 最近访问过，也修改过 | 最不想换      |

为什么优先换出 m=0 的页？

因为没有修改过的页不需要写回磁盘，直接丢掉即可，I/O 成本低。

---

# 10. 用课本例子练页面置换

课本图 8.14 的访问串是：

[
2,\ 3,\ 2,\ 1,\ 5,\ 2,\ 4,\ 5,\ 3,\ 2,\ 5,\ 2
]

假设有 3 个 page frames。

注意：课本图中的 F 标注的是 **frame allocation is initially filled 后的缺页**，也就是不把最开始装入空闲帧的缺页算在图中 F 里。做考试题时，如果题目问“总缺页次数”，通常要把一开始空帧装入也算作缺页。

| 算法    | 初始装入空帧缺页 | 后续置换缺页 | 总缺页次数 |
| ----- | -------: | -----: | ----: |
| OPT   |        3 |      3 |     6 |
| LRU   |        3 |      4 |     7 |
| FIFO  |        3 |      6 |     9 |
| Clock |        3 |      5 |     8 |

从结果看：

[
OPT < LRU < Clock < FIFO
]

一般规律也是：

* OPT 最优，但不能实际实现。
* LRU 效果好，但开销大。
* FIFO 简单，但可能差。
* Clock 是折中方案，实际系统中常用其变体。

---

# 11. 本部分最容易考的点

## 概念题

### Q1: What is virtual memory? / 什么是虚拟内存？

虚拟内存是一种内存管理技术，使程序使用的地址空间可以大于实际主存。程序使用 virtual address，由硬件和 OS 自动转换成 physical address。进程的部分页面可以放在主存，其余页面放在磁盘中，需要时再调入。

---

### Q2: Why does virtual memory work? / 为什么虚拟内存可行？

因为程序具有 principle of locality。短时间内程序通常只访问少量代码和数据页，所以不必把整个进程装入主存，只要把当前 locality 对应的页放入 resident set 即可。

---

### Q3: What is a page fault? / 什么是缺页中断？

当进程访问的虚拟页不在主存中时，硬件产生 page fault interrupt，OS 负责从磁盘把该页调入主存。如果主存没有空闲页框，还需要执行 page replacement。

---

### Q4: TLB miss 和 page fault 有什么区别？

TLB miss：TLB 中没有对应页表项，但页可能在主存中。
Page fault：页表显示该页不在主存中，需要从磁盘调入。

所以：

[
\text{page fault 一定比 TLB miss 更严重}
]

---

### Q5: 为什么 Clock 算法比 FIFO 好？

FIFO 只看页面进入内存的先后顺序，可能换出经常使用的页面。Clock 给 use bit = 1 的页面一次机会，说明该页最近被访问过，不立即换出，因此更符合 locality。

---

## 12. 一句话总结

第 8 章首页标注的核心就是：

> **Virtual memory** 依靠 **locality**，通过 **paging + page table + TLB + page fault handling** 实现“进程不用全部在主存也能运行”；当内存满时，OS 用 **replacement policy**，如 **OPT / LRU / FIFO / Clock**，决定换出哪一页。





按首页标注，本次相关重点是 **8.1 Hardware and Control Structures** 里的 **Locality and Virtual Memory / Paging**，以及 **8.2 Operating System Software** 里的 **Replacement Policy**。所以我从题目文件里筛出相关题：**Review Questions 8.1–8.12**，以及 **Problems 8.1、8.2、8.3、8.4、8.5、8.6、8.7、8.10、8.11、8.12、8.13、8.14、8.18**。题目来源是 Chapter 8 的问题页；首页标注来自 8.1–8.3 讲义。 

---

# 一、Review Questions 相关题答案

## 8.1 Simple paging 和 virtual memory paging 的区别

**Simple paging / 简单分页**：进程运行前，所有 pages 必须都在 main memory 中。
**Virtual memory paging / 虚拟内存分页**：进程不需要全部在内存，只要当前需要的 pages 在内存即可；缺页时通过 **page fault** 从磁盘调入。

核心区别：

| Simple Paging      | Virtual Memory Paging          |
| ------------------ | ------------------------------ |
| 所有页必须在主存           | 部分页在主存即可                       |
| 不需要 page fault 机制  | 需要 page fault 机制               |
| 页表只记录 page → frame | 页表还要有 present bit、modify bit 等 |

---

## 8.2 Explain thrashing / 解释抖动

**Thrashing / 抖动**：系统频繁发生 page fault，大量时间花在页面换入换出，而不是执行程序。

中文理解：

> 内存给每个进程分得太少，进程刚换进来的页马上又被换出，导致 CPU 利用率急剧下降。

---

## 8.3 为什么 locality 对 virtual memory 很关键？

**Principle of locality / 局部性原理** 指程序短时间内通常集中访问少量代码和数据。

虚拟内存可行，是因为：

1. 程序短时间只需要少数 pages。
2. OS 可以只把这些 pages 放入 resident set。
3. Page fault rate 会保持较低。
4. 如果没有 locality，程序会不断缺页，虚拟内存就会退化成 thrashing。

---

## 8.4 Page table entry 通常有什么元素？

典型 **Page Table Entry, PTE / 页表项** 包括：

| English             | 中文        | 作用              |
| ------------------- | --------- | --------------- |
| Present / Valid bit | 存在位 / 有效位 | 表示该页是否在主存       |
| Modify / Dirty bit  | 修改位 / 脏位  | 表示该页是否被写过       |
| Reference / Use bit | 引用位 / 使用位 | 表示最近是否访问过       |
| Frame number        | 页框号       | 该页在物理内存中的 frame |
| Protection bits     | 保护位       | 控制读、写、执行权限      |

---

## 8.5 TLB 的作用是什么？

**Translation Lookaside Buffer, TLB / 转换检测缓冲区** 是一个高速页表缓存。

作用：

> 缓存最近用过的 page table entries，减少每次地址转换时访问主存页表的次数。

注意区分：

* **TLB miss**：TLB 里没有页表项，但页可能在主存。
* **Page fault**：页不在主存，需要从磁盘调入。

---

## 8.6 Alternative page fetch policies / 页面调入策略

两种常见策略：

1. **Demand paging / 请求分页**
   只有访问某页且该页不在主存时，才调入该页。

2. **Prepaging / 预调页**
   缺页时不只调入当前页，还提前调入相邻或预测会用到的页。

考试回答重点：

> Demand paging 保守，避免调入无用页；prepaging 利用磁盘连续读取优势，但如果预测错，会浪费内存和 I/O。

---

## 8.7 Resident set management 和 page replacement policy 的区别

**Resident set management / 驻留集管理**：决定一个进程在主存中保留多少页框，以及替换范围是 local 还是 global。

**Page replacement policy / 页面置换策略**：在必须换出页面时，决定具体换出哪一页。

简单说：

> Resident set management 管“给多少 frame”；replacement policy 管“换掉哪一页”。

---

## 8.8 FIFO 和 Clock 的关系

**FIFO / First-In-First-Out**：换出最早进入内存的页面。

**Clock / 时钟算法**：是 FIFO 的改进版。它也用循环队列，但给每个页加一个 **use bit / 使用位**：

* use bit = 1：最近访问过，给第二次机会。
* use bit = 0：可以换出。

所以 Clock 也叫 **Second Chance Algorithm / 二次机会算法**。

---

## 8.9 Page buffering 有什么作用？

**Page buffering / 页面缓冲**：被替换的页面不立刻丢弃，而是放入 free list 或 modified list。

作用：

1. 如果页面很快又被访问，可以低成本恢复。
2. 修改过的 dirty pages 可以批量写回磁盘。
3. 减少 I/O 次数。
4. 允许使用简单 FIFO，同时提升性能。

---

## 8.10 为什么不能 fixed allocation + global replacement？

**Fixed allocation / 固定分配**：每个进程的 frame 数量固定。
**Global replacement / 全局置换**：可以从所有进程的页中选择 victim page。

矛盾在于：

> 如果全局置换从别的进程拿走一个 frame，那么那个进程的 resident set size 就变了，这违反 fixed allocation。

所以：

[
\text{Fixed allocation} \Rightarrow \text{Local replacement}
]

---

## 8.11 Resident set 和 working set 的区别

**Resident set / 驻留集**：当前实际在主存中的该进程页面集合。

**Working set / 工作集**：最近一段时间窗口 (\Delta) 内被访问过的页面集合。

区别：

| Term         | 中文  | 含义        |
| ------------ | --- | --------- |
| Resident set | 驻留集 | 实际在内存中的页  |
| Working set  | 工作集 | 最近真正需要用的页 |

理想情况是：

[
\text{Resident set} \supseteq \text{Working set}
]

否则容易频繁 page fault。

---

## 8.12 Demand cleaning 和 precleaning 的区别

**Demand cleaning / 请求清理**：只有当 dirty page 被选中替换时，才写回磁盘。

**Precleaning / 预清理**：提前把 dirty pages 写回磁盘。

对比：

| Demand cleaning | Precleaning        |
| --------------- | ------------------ |
| 写回次数少           | 可批量写回              |
| 缺页时可能要等写回 + 读入  | 可能写回了后来又被修改，浪费 I/O |

---

# 二、Problems 相关题答案

## Problem 8.1 地址转换

页大小：

[
1024 \text{ bytes}
]

虚拟地址转换公式：

[
\text{virtual page number} = \left\lfloor \frac{\text{virtual address}}{1024} \right\rfloor
]

[
\text{offset} = \text{virtual address} \bmod 1024
]

[
\text{physical address} = \text{frame number} \times 1024 + \text{offset}
]

### (i) 1052

[
1052 = 1 \times 1024 + 28
]

Virtual page = 1，有效，frame = 7。

[
PA = 7 \times 1024 + 28 = 7196
]

答案：

[
\boxed{7196}
]

### (ii) 2221

[
2221 = 2 \times 1024 + 173
]

Virtual page = 2，valid bit = 0。

答案：

[
\boxed{\text{page fault，无 physical address}}
]

### (iii) 5499

[
5499 = 5 \times 1024 + 379
]

Virtual page = 5，有效，frame = 0。

[
PA = 0 \times 1024 + 379 = 379
]

答案：

[
\boxed{379}
]

---

## Problem 8.2 二维数组访问与缺页频率

数组大小：

[
64 \times 64
]

每个 int 是 4 bytes，所以一个数组大小：

[
64 \times 64 \times 4 = 16384 \text{ bytes} = 16 \text{ KB}
]

页大小 1 KB，所以每个数组占：

[
16 \text{ pages}
]

题目给 4-page working set，其中：

* 1 page 给程序代码。
* 3 pages 给数据 A、B、C。

原程序：

```c
for (j = 0; j < Size; j++)
    for (i = 0; i < Size; i++)
        C[i][j] = A[i][j] + B[i][j];
```

这是按列访问，但数组按行连续存储。每 4 行占 1 页，所以访问同一列时，每 4 次执行换到下一组页面。

### (a) 原程序缺页频率

每 4 次执行语句，需要访问 A、B、C 的新页面，因此大约：

[
\boxed{\text{每 4 次赋值产生 3 次 data page faults}}
]

总执行次数：

[
64 \times 64 = 4096
]

每列有 16 组页面，每组 3 次缺页：

[
64 \times 16 \times 3 = 3072
]

答案：

[
\boxed{3072 \text{ data page faults}}
]

---

### (b) 修改程序

改成按行访问：

```c
for (i = 0; i < Size; i++)
    for (j = 0; j < Size; j++)
        C[i][j] = A[i][j] + B[i][j];
```

---

### (c) 修改后缺页频率

每 4 行共：

[
4 \times 64 = 256
]

次赋值才需要换一次 A、B、C 的页面。

所以：

[
\boxed{\text{每 256 次赋值产生 3 次 data page faults}}
]

总共有 16 组行页面：

[
16 \times 3 = 48
]

答案：

[
\boxed{48 \text{ data page faults}}
]

---

## Problem 8.3 Page table 空间

### (a)

Figure 8.3 是 32-bit address，page size = 4 KB。

Offset：

[
4KB = 2^{12}
]

所以 page number 位数：

[
32 - 12 = 20
]

页数：

[
2^{20}
]

每个 PTE = 4 bytes。

[
2^{20} \times 4 = 2^{22} \text{ bytes} = 4 \text{ MB}
]

答案：

[
\boxed{4 \text{ MB}}
]

---

### (b)

Hash value 是 6 bits，所以 hash entries：

[
2^6 = 64
]

每个 hash entry 最多 3 个 overflow entries，所以总 entry 数：

[
64 \times 4 = 256
]

每个 entry 包含：

* page number：20 bits
* frame number：20 bits
* chain pointer：通常按 6 bits 计算

所以每项：

[
20 + 20 + 6 = 46 \text{ bits}
]

总空间：

[
256 \times 46 = 11776 \text{ bits}
]

[
11776 / 8 = 1472 \text{ bytes}
]

答案：

[
\boxed{1472 \text{ bytes}}
]

如果按整字节对齐，约为：

[
\boxed{1.5 \text{ KB}}
]

---

## Problem 8.4 页面置换算法

Reference string：

[
7,0,1,2,0,3,0,4,2,3,0,3,2
]

假设 3 个 frames。题目要求 **只统计所有 frames 初始化完成之后的 page faults**。前三次 7、0、1 用于初始化，不计入后续 fault 统计。

记法：`701` 表示三个 frames 中分别是 7、0、1。

```text
Ref:   7   0   1   2   0   3   0   4   2   3   0   3   2

FIFO:  7-- 70- 701 201 201 231 230 430 420 423 023 023 023
LRU:   7-- 70- 701 201 201 203 203 403 402 432 032 032 032
Clock: 7-- 70- 701 201 201 203 203 403 423 423 420 320 320
OPT:   7-- 70- 701 201 201 203 203 243 243 243 203 203 203
```

统计结果：

| Algorithm | 初始化后 page faults | Miss rate，分母为后 10 次 |
| --------- | ---------------: | ------------------: |
| FIFO      |                7 |                 70% |
| LRU       |                6 |                 60% |
| Clock     |                6 |                 60% |
| Optimal   |                4 |                 40% |

答案重点：

[
\boxed{OPT < LRU \approx Clock < FIFO}
]

---

## Problem 8.5 FIFO 与 Belady anomaly

Reference string：

[
A,B,C,D,A,B,E,A,B,C,D,E
]

### 3 frames，FIFO

缺页次数：

[
\boxed{9}
]

### 4 frames，FIFO

缺页次数：

[
\boxed{10}
]

这里出现了 **Belady anomaly / Belady 异常**：

> 对 FIFO 来说，frame 数量增加，page fault 反而可能增加。

---

## Problem 8.6 LRU 和 FIFO hit ratio

Reference string：

[
1,0,2,2,1,7,6,7,0,1,2,0,3,0,4,5,1,5,2,4,5,6,7,6,7,2,4,2,7,3,3,2,3
]

共：

[
33
]

次访问，4 个 frames。

### LRU

缺页次数：

[
17
]

命中次数：

[
33 - 17 = 16
]

Hit ratio：

[
\frac{16}{33} \approx 48.5%
]

答案：

[
\boxed{LRU \text{ hit ratio} = 16/33 \approx 48.5%}
]

### FIFO

缺页次数：

[
17
]

命中次数：

[
16
]

Hit ratio：

[
\frac{16}{33} \approx 48.5%
]

答案：

[
\boxed{FIFO \text{ hit ratio} = 16/33 \approx 48.5%}
]

这条 trace 里 FIFO 和 LRU 结果相同，但这只是特例。一般情况下：

[
\boxed{LRU 通常比 FIFO 更接近 OPT}
]

---

## Problem 8.7 User page tables 放在 virtual memory 的优缺点

### Advantage / 优点

把 user page tables 放在 virtual memory 中，可以节省 real memory。因为每个进程的页表可能很大，不需要整张页表一直占用主存。

### Disadvantage / 缺点

访问页表本身也可能 page fault。也就是说：

> 为了访问某个页面，可能先因为页表页不在主存而缺页，再因为目标页面不在主存而缺页。

这会导致 **double page fault / 双重缺页**，增加开销。

---

## Problem 8.10 64-bit address space 需要几级页表？

Page size：

[
4KB = 2^{12}
]

Offset = 12 bits。

64-bit address 中 page number 位数：

[
64 - 12 = 52
]

每个 PTE = 4 bytes，每个 page table page 大小 = 4 KB，所以一个页表页能放：

[
\frac{4096}{4} = 1024 = 2^{10}
]

每级页表索引 10 bits。

需要页表级数：

[
\left\lceil \frac{52}{10} \right\rceil = 6
]

答案：

[
\boxed{6 \text{ levels}}
]

---

## Problem 8.11 TLB 和 EMAT

Memory reference time：

[
200ns
]

### (a) 没有 TLB

需要：

1. 访问 page table。
2. 访问 data。

[
200 + 200 = 400ns
]

答案：

[
\boxed{400ns}
]

---

### (b) TLB hit rate = 85%

MMU overhead = 20ns。

TLB hit：

[
20 + 200 = 220ns
]

TLB miss：

[
20 + 200 + 200 = 420ns
]

EMAT：

[
0.85 \times 220 + 0.15 \times 420
]

[
= 187 + 63 = 250ns
]

答案：

[
\boxed{250ns}
]

---

### (c) TLB hit rate 对 EMAT 的影响

设 hit rate 为 (h)：

[
EMAT = h \times 220 + (1-h) \times 420
]

[
EMAT = 420 - 200h
]

所以 hit rate 越高，EMAT 越低。

---

## Problem 8.12 Page fault 数量上下界

Reference string 长度为 (P)，有 (N) 个 distinct pages。

### Lower bound / 下界

每个不同页面第一次访问时至少缺页一次：

[
\boxed{N}
]

### Upper bound / 上界

最坏情况下，每次访问都缺页：

[
\boxed{P}
]

更精确地说：

* 如果 (M \ge N)，frame 足够装下所有不同页面，则最多：

[
\boxed{N}
]

* 如果 (M < N)，最坏可达：

[
\boxed{P}
]

---

## Problem 8.13 Snowplow analogy

### (a)

这个类比适合：

[
\boxed{\text{Clock page replacement algorithm}}
]

也就是 **时钟页面置换算法**。

### (b)

Snowplow 在圆形轨道上扫雪，就像 Clock pointer 在 circular buffer 中扫描 pages。

含义：

* 被访问过的页类似“重新积雪”，说明最近活跃。
* 指针扫到 use bit = 1 的页，会清零并跳过。
* 指针扫到 use bit = 0 的页，说明最近没被访问，可以换出。

所以 Clock 是一种低成本近似 LRU 的算法。

---

## Problem 8.14 只用 reference bit 近似 LRU

不能精确实现 LRU，因为只有 1 个 reference bit 不足以记录完整访问顺序。

可以用 **aging / 老化算法** 近似：

1. 给每个 frame 一个 age counter。
2. 定期扫描 reference bit。
3. 如果 reference bit = 1，把 age counter 调大或在高位写 1。
4. 如果 reference bit = 0，让 age counter 右移或衰减。
5. 每次扫描后把 reference bit 清零。
6. 置换时选择 age counter 最小的页面。

结论：

[
\boxed{\text{用 reference bit + 周期性扫描，可以近似 LRU}}
]

---

## Problem 8.18 Logical address 和 page table

Logical address space：

[
32 \text{ pages}
]

Page size：

[
2KB = 2^{11}
]

Physical memory：

[
1MB = 2^{20}
]

---

### (a) Logical address format

Page number bits：

[
\log_2 32 = 5
]

Offset bits：

[
\log_2 2KB = 11
]

所以 logical address 格式：

[
\boxed{\text{Page number: 5 bits, Offset: 11 bits}}
]

总长度：

[
5 + 11 = 16 \text{ bits}
]

---

### (b) Page table length and width

Page table length = logical pages 数量：

[
\boxed{32 \text{ entries}}
]

Physical frames：

[
\frac{1MB}{2KB} = \frac{2^{20}}{2^{11}} = 2^9 = 512
]

Frame number width：

[
\boxed{9 \text{ bits}}
]

所以页表：

[
\boxed{32 \text{ entries} \times 9 \text{ bits}}
]

---

### (c) Physical memory 减半

Physical memory 变为：

[
512KB = 2^{19}
]

Frames：

[
\frac{2^{19}}{2^{11}} = 2^8 = 256
]

Frame number width 变为：

[
\boxed{8 \text{ bits}}
]

Page table length 不变，仍然是：

[
\boxed{32 \text{ entries}}
]

变化是：

[
\boxed{\text{页表每项宽度从 9 bits 变为 8 bits}}
]

---

# 最重点要会的题

如果时间不够，优先掌握：

[
\boxed{8.1,\ 8.2,\ 8.3,\ 8.4,\ 8.5,\ 8.11}
]

其中最可能考计算的是：

1. **8.1 地址转换**
2. **8.4 FIFO / LRU / Clock / OPT**
3. **8.5 Belady anomaly**
4. **8.11 TLB effective access time**
5. **8.2 locality 对缺页率的影响**
