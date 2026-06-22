下面继续按这份 **7.1–7.3 Memory Management** 复习，默认中英对照，重点看首页高亮的 **Memory Partitioning / Paging**。

## 1. 本章核心：Memory Management / 内存管理

在 **multiprogramming system / 多道程序系统** 中，主存不再只放一个程序，而要同时容纳多个进程。操作系统要决定：

谁进内存、放在哪里、能不能访问、要不要换出、地址怎么转换。

教材把内存管理的基本要求列为五个：

| English               | 中文   | 考点理解                         |
| --------------------- | ---- | ---------------------------- |
| Relocation            | 重定位  | 程序可能被换入不同内存位置，所以逻辑地址要转换成物理地址 |
| Protection            | 保护   | 一个进程不能随便访问另一个进程或操作系统的内存      |
| Sharing               | 共享   | 多个进程可以受控地共享代码或数据             |
| Logical Organization  | 逻辑组织 | 程序通常按模块组织，不是简单一维连续空间         |
| Physical Organization | 物理组织 | 主存和辅存之间的数据调入调出应由系统负责         |

重点记：**Relocation / 重定位** 是后面分页、分段、虚拟内存的基础。因为进程不一定每次都放在同一个物理地址，所以程序中看到的地址不能直接等于真实内存地址。

---

## 2. Relocation / 重定位：逻辑地址到物理地址

教材区分几个地址：

| English                             | 中文          | 含义                |
| ----------------------------------- | ----------- | ----------------- |
| Logical Address                     | 逻辑地址        | 程序看到的地址，和实际内存位置无关 |
| Relative Address                    | 相对地址        | 一种逻辑地址，通常相对程序起始位置 |
| Physical Address / Absolute Address | 物理地址 / 绝对地址 | 主存中的真实地址          |

最简单的动态重定位硬件是：

**Base Register + Bounds Register**

中文理解：

进程运行时，CPU 产生一个相对地址。硬件把它加上 **base register / 基址寄存器**，得到物理地址；再用 **bounds register / 界限寄存器** 检查是否越界。

公式：

```text
physical address = base register + relative address
```

如果超过 bounds，就触发 interrupt / 中断，交给操作系统处理。

这同时实现两个功能：

1. **Relocation / 重定位**：进程放到哪里都行，只要 base 改掉。
2. **Protection / 保护**：不能访问自己范围外的地址。

---

## 3. Memory Partitioning / 内存分区

这一节是首页重点高亮内容。分区的思想是：把内存划成若干块，然后把进程装进去。

### 3.1 Fixed Partitioning / 固定分区

**Fixed Partitioning** 指系统生成时就把内存分成若干个固定大小的分区。

可以有两种：

| 类型      | English                       | 特点            |
| ------- | ----------------------------- | ------------- |
| 等长固定分区  | Equal-size fixed partitions   | 每个分区一样大，简单但浪费 |
| 不等长固定分区 | Unequal-size fixed partitions | 分区大小不同，更灵活    |

主要问题是：

**Internal Fragmentation / 内部碎片**

意思是：进程比所在分区小，分区内部剩余空间浪费了。

例如：

一个进程只需要 2 MB，但被装进 8 MB 分区，那么剩下 6 MB 就是内部碎片。

固定分区的缺点：

1. 分区数量固定，所以最多活跃进程数固定。
2. 分区大小固定，小进程容易浪费空间。
3. 大进程可能放不进去，需要 overlay / 覆盖技术。

---

### 3.2 Dynamic Partitioning / 动态分区

**Dynamic Partitioning** 指分区不是预先划好的，而是进程来了以后，按它实际大小分配。

优点：

没有内部碎片，因为进程需要多少就分多少。

缺点：

会产生 **External Fragmentation / 外部碎片**。

外部碎片的意思是：

内存中有很多空闲小洞，合起来可能够用，但不连续，所以无法放入一个大进程。

例如：

```text
空闲 4MB + 空闲 6MB + 空闲 8MB = 总共 18MB
```

但如果它们分散在不同位置，而新进程需要连续 16MB，就可能放不下。

解决方法：

**Compaction / 紧凑**

操作系统把内存里的进程移动到一起，让空闲空间合并成一个大块。

缺点：很耗 CPU 时间，而且要求系统支持动态重定位。

---

## 4. Placement Algorithms / 放置算法

动态分区中，如果有多个空闲块都能放入进程，操作系统要选择用哪一个。

教材讲了三个：

| English   | 中文   | 做法                |
| --------- | ---- | ----------------- |
| First-fit | 首次适配 | 从头开始找，第一个够大的空闲块就用 |
| Next-fit  | 下次适配 | 从上次分配的位置继续往后找     |
| Best-fit  | 最佳适配 | 找最接近请求大小的空闲块      |

常考结论：

**First-fit 通常最快，也往往效果较好。**

**Best-fit 名字叫“最佳”，但实际经常最差。**

原因是 best-fit 每次都留下最小剩余空间，容易产生大量特别小、以后很难利用的碎片。

教材例子中，如果请求 16 MB：

| 算法        | 选择的空闲块 |  剩余碎片 |
| --------- | -----: | ----: |
| Best-fit  |  18 MB |  2 MB |
| First-fit |  22 MB |  6 MB |
| Next-fit  |  36 MB | 20 MB |

看起来 best-fit 当次最省，但长期会制造很多小碎片。

---

## 5. Buddy System / 伙伴系统

**Buddy System / 伙伴系统** 是固定分区和动态分区之间的折中方案。

核心思想：

内存块大小只能是 2 的幂：

```text
2^L, 2^(L+1), ..., 2^U
```

例如：

```text
64KB, 128KB, 256KB, 512KB, 1MB
```

当一个进程请求大小为 s 的空间时，系统找一个最小的、能容纳 s 的 2 的幂大小块。

例如：

```text
请求 100KB
不能给 64KB，因为不够
所以给 128KB
```

如果当前没有 128KB 块，就把更大的块不断对半拆开：

```text
1MB → 512KB + 512KB
512KB → 256KB + 256KB
256KB → 128KB + 128KB
```

这两个拆出来的块互称为 **buddies / 伙伴**。

释放时，如果两个 buddy 都空闲，就可以合并：

```text
128KB + 128KB → 256KB
256KB + 256KB → 512KB
```

伙伴系统的特点：

| 优点                | 缺点                   |
| ----------------- | -------------------- |
| 分配和回收比较快          | 会有内部碎片，因为请求会向上取 2 的幂 |
| 可以通过 buddy 合并减少碎片 | 不如分页/分段虚拟内存灵活        |

一句话记：

**Buddy System = split when allocating, coalesce when freeing.**

中文：

**分配时拆分，释放时合并。**

---

## 6. Paging / 分页

这是 7.3 的重点。

固定分区的问题是内部碎片，动态分区的问题是外部碎片。分页的想法是：

把主存分成很多固定大小的小块，叫：

**Frame / 页框**

把进程也分成同样大小的小块，叫：

**Page / 页**

| English            | 中文 | 所在位置         |
| ------------------ | -- | ------------ |
| Page               | 页  | 进程逻辑空间 / 辅存中 |
| Frame / Page Frame | 页框 | 主存中          |

分页的关键优势：

**一个进程的各个 page 不需要连续放在主存中。**

例如进程 D 有 5 页，可以放在 frame 4, 5, 6, 11, 12，不需要连续。

所以分页：

1. 没有外部碎片。
2. 只有少量内部碎片，最多出现在最后一页。
3. 需要页表进行地址转换。

---

## 7. Page Table / 页表

每个进程都有自己的 **Page Table / 页表**。

页表记录：

```text
page number → frame number
```

也就是：

进程的第几页现在放在主存的第几个页框里。

逻辑地址格式：

```text
logical address = page number + offset
```

物理地址格式：

```text
physical address = frame number + offset
```

分页地址转换公式：

```text
page number = logical address // page size
offset = logical address % page size

frame number = page_table[page number]

physical address = frame number * page size + offset
```

---

## 8. 教材例题：分页地址转换

教材例子：

16-bit 地址，page size = 1K = 1024 bytes。

因为：

```text
1024 = 2^10
```

所以 offset 需要 10 bits。

总地址是 16 bits，所以 page number 占：

```text
16 - 10 = 6 bits
```

因此逻辑地址结构是：

```text
6-bit page number + 10-bit offset
```

教材给出 relative address = 1502。

计算：

```text
1502 // 1024 = 1
1502 % 1024 = 478
```

所以：

```text
page number = 1
offset = 478
```

如果 page 1 存在 frame 6 中，那么物理地址是：

```text
physical address = 6 * 1024 + 478 = 6622
```

二进制理解：

```text
logical address:
page # = 000001
offset = 0111011110

frame # = 000110
offset = 0111011110

physical address = 0001100111011110
```

重点记忆：

**分页地址转换时，offset 不变，只把 page number 换成 frame number。**

---

## 9. Simple Paging vs Virtual Memory Paging

这里教材讲的是 **Simple Paging / 简单分页**，还不是完整虚拟内存。

区别：

| 类型                    | 中文     | 是否必须一次性装入整个进程     |
| --------------------- | ------ | ----------------- |
| Simple Paging         | 简单分页   | 是，所有 pages 都要装入内存 |
| Virtual Memory Paging | 虚拟内存分页 | 否，需要哪页再调入哪页       |

所以这章 7.3 的分页主要是为后面虚拟内存做铺垫。

---

## 10. 本节最常考对比

| 技术                   | Internal Fragmentation 内部碎片 | External Fragmentation 外部碎片 | 是否连续          |
| -------------------- | --------------------------- | --------------------------- | ------------- |
| Fixed Partitioning   | 有                           | 无                           | 每个进程占一个固定连续分区 |
| Dynamic Partitioning | 无                           | 有                           | 每个进程占一个动态连续分区 |
| Buddy System         | 有，向上取 2 的幂                  | 较少，可合并 buddy                | 每个块连续         |
| Paging               | 有，通常只在最后一页                  | 无                           | 进程页不需要连续      |

最重要判断题：

**Paging has no external fragmentation.**

中文：

**分页没有外部碎片。**

但分页仍可能有：

**Internal fragmentation in the last page.**

中文：

**最后一页可能有内部碎片。**

---

## 11. 一句话总复习

**Fixed Partitioning**：分区固定，简单，但内部碎片严重。

**Dynamic Partitioning**：按需分配，没有内部碎片，但有外部碎片，需要 compaction。

**Buddy System**：按 2 的幂分配，分配时拆，释放时合并。

**Paging**：进程分成 pages，内存分成 frames，靠 page table 映射；页可以不连续，所以没有外部碎片。

**地址转换核心**：

```text
logical address = page number + offset
physical address = frame number + offset
```

分页题最常用公式：

```text
page number = address // page size
offset = address % page size
physical address = frame * page size + offset
```



7.8 这题问的是 **buddy address / 伙伴块地址** 怎么算。题目给的是一个当前块地址：

```text
x = 011011110000
```

然后问：如果块大小是 4 或 16，它的 buddy 地址是多少。

---

## 1. Buddy 是什么？

在 buddy system 里，两个同样大小的块来自同一个更大的父块。

例如块大小是 4：

```text
父块大小 8
= 左边 4 + 右边 4
```

如果当前块是左边那个，它的 buddy 就是右边那个；
如果当前块是右边那个，它的 buddy 就是左边那个。

所以 buddy 地址只差一个块大小：

```text
buddy = x + block_size   或   x - block_size
```

更统一的公式是：

```text
buddy = x XOR block_size
```

---

## 2. 为什么是 XOR？

因为块大小如果是：

```text
block size = 2^k
```

那么 buddy 只需要翻转地址中代表 `2^k` 的那一位。

低 k 位表示块内部偏移，不动。
更高位表示属于哪个大区域，也不动。
只有第 k 位决定它是父块中的左 buddy 还是右 buddy。

所以：

```text
buddy_k(x) = x XOR 2^k
```

---

## 3. 块大小是 4

```text
4 = 2^2
```

所以要翻转从右往左数第 3 位，也就是 `2^2` 那一位。

原地址：

```text
011011110000
```

把它按 4 位分组：

```text
0110 1111 0000
```

block size = 4 的二进制是：

```text
0000 0000 0100
```

做 XOR：

```text
  0110 1111 0000
^ 0000 0000 0100
= 0110 1111 0100
```

所以答案是：

```text
011011110100
```

用十进制看更直观：

```text
011011110000 = 1776
1776 + 4 = 1780
1780 = 011011110100
```

说明当前 4-byte 块是左 buddy，它的伙伴在右边。

---

## 4. 块大小是 16

```text
16 = 2^4
```

所以翻转 `2^4` 那一位。

block size = 16 的二进制是：

```text
0000 0001 0000
```

做 XOR：

```text
  0110 1111 0000
^ 0000 0001 0000
= 0110 1110 0000
```

所以答案是：

```text
011011100000
```

十进制看：

```text
011011110000 = 1776
1776 - 16 = 1760
1760 = 011011100000
```

说明当前 16-byte 块是右 buddy，它的伙伴在左边。

---

## 最后记法

只要看到 buddy 地址题，直接用：

```text
buddy = address XOR block_size
```

所以这题：

```text
size = 4:
011011110000 XOR 000000000100 = 011011110100

size = 16:
011011110000 XOR 000000010000 = 011011100000
```
