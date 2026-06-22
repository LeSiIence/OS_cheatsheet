我看到这份 PDF 的高亮主要落在 **12.1 Overview / Files and File Systems / File Structure / File Management Systems、12.3 B-Trees、12.4 File Directories、12.6 Record Blocking、12.7 File Allocation 和 First/Best/Nearest fit**。下面按考试重点复习。

## 1. 先抓主线：File Management 在干什么

**File system 文件系统**的核心作用：把长期数据组织成 **files 文件**，让用户和程序可以通过名字访问，而不是直接操作磁盘块。

文件系统通常要支持：

| English | 中文   |
| ------- | ---- |
| Create  | 创建文件 |
| Delete  | 删除文件 |
| Open    | 打开文件 |
| Close   | 关闭文件 |
| Read    | 读文件  |
| Write   | 写文件  |

文件的基本层次是：

**Field 字段 → Record 记录 → File 文件 → Database 数据库**

例如学生信息：

* `Name`、`ID`、`Age` 是 **fields 字段**
* 一个学生的完整信息是一条 **record 记录**
* 全班学生记录组成一个 **file 文件**
* 多个相关文件组成 **database 数据库**

注意：UNIX / Linux 里很多文件被看成 **stream of bytes 字节流**，不一定显式区分 record 和 field。

---

## 2. File System Architecture 文件系统层次结构

从上到下：

| 层次  | English                          | 作用                                    |
| --- | -------------------------------- | ------------------------------------- |
| 最高层 | Access method                    | 提供不同访问方式，例如 sequential、indexed、hashed |
| 中间层 | Logical I/O                      | 面向 record 记录操作                        |
| 中间层 | Basic I/O supervisor             | 管理 I/O 请求、缓冲区、磁盘调度                    |
| 底层  | Basic file system / Physical I/O | 面向 block 块操作                          |
| 最底层 | Device drivers                   | 直接控制磁盘、磁带等设备                          |

考试常问一句话：

> 用户看到的是 **records / files**，磁盘真正传输的是 **blocks**。
> Users access records/files, but secondary storage performs I/O in blocks.

这句话直接引出 12.6 的 **Record Blocking**。

---

## 3. B-Trees：文件索引常用结构

**B-tree B 树**是用于文件系统和数据库索引的平衡多路搜索树。

它的核心特点：

1. **Balanced 平衡**：所有叶子在同一层，所以查找路径长度差不多。
2. **Broad and shallow 宽而浅**：一个节点可以有很多 key 和 pointer，所以高度低。
3. 支持高效的 **search 查找、insert 插入、delete 删除**。

如果 B-tree 的最小度数是 **d**：

| 规则                                   | 含义               |
| ------------------------------------ | ---------------- |
| 每个节点最多有 `2d - 1` 个 keys              | Maximum keys     |
| 每个节点最多有 `2d` 个 children / pointers   | Maximum children |
| 除 root 外，每个节点至少有 `d - 1` 个 keys      | 至少半满             |
| 有 k 个 pointers 的非叶节点有 `k - 1` 个 keys | 指针比 key 多 1      |

插入时最重要：

> 如果节点满了，就 **split 分裂**，把 **median key 中位 key** 提升到父节点。

例如 PDF 图 12.5 里，`d = 3`，所以一个节点最多 `2d - 1 = 5` 个 key。插入新 key 时，如果节点超过容量，就围绕中位数分裂，并把中位 key 提上去。

---

## 4. File Directories 文件目录

**Directory 目录**本质上也是一种文件，用来保存其他文件的信息。

目录项中常见信息：

| 类型                         | 内容                       |
| -------------------------- | ------------------------ |
| Basic information          | 文件名、文件类型、文件组织方式          |
| Address information        | 所在 volume、起始地址、已用大小、分配大小 |
| Access control information | owner、权限、允许的操作           |
| Usage information          | 创建时间、最后读取、最后修改、是否打开、是否锁定 |

目录要支持的操作：

| English          | 中文      |
| ---------------- | ------- |
| Search           | 查找文件    |
| Create file      | 创建文件目录项 |
| Delete file      | 删除文件目录项 |
| List directory   | 列出目录    |
| Update directory | 更新文件属性  |

目录结构有三种常见形式：

1. **Single-level directory 单级目录**
   所有文件放一个表里，简单但命名冲突严重。

2. **Two-level directory 两级目录**
   每个用户一个目录，上面有 master directory。文件名只需在用户目录内唯一。

3. **Tree-structured directory 树形目录**
   现代系统常用。可以有子目录、路径名。

重要概念：

| English                               | 中文                               |
| ------------------------------------- | -------------------------------- |
| Pathname                              | 路径名，例如 `/User_B/Word/Unit_A/ABC` |
| Working directory / Current directory | 当前工作目录                           |
| Home directory                        | 用户主目录                            |

同名文件可以存在于不同目录，只要 **pathname 路径名唯一**。

---

# 5. 12.6 Record Blocking 记录成块

这是重点。

**Record 记录**是逻辑单位，用户或程序按 record 访问。
**Block 块**是物理 I/O 单位，磁盘按 block 读写。

所以需要 **record blocking 记录成块**：

> 把一个或多个 logical records 组织进 physical blocks。

---

## 5.1 Block size 的取舍

块越大：

优点：

* 一次 I/O 能读更多 records
* 顺序处理 sequential processing 更快
* I/O 次数减少

缺点：

* 如果随机访问 random access，可能读进很多用不到的 records
* 需要更大的 I/O buffer
* buffer management 更复杂

一句话：

> 大 block 适合顺序访问，小 block 更适合随机访问。

---

## 5.2 三种 Record Blocking 方法

### 第一种：Fixed blocking 固定成块

**Fixed-length records 固定长度记录**，一个 block 里放整数个 records。

例如：

* block size = 1000 bytes
* record size = 300 bytes
* 一个 block 能放 `⌊1000 / 300⌋ = 3` 条记录
* 剩下 100 bytes 浪费

这种浪费叫：

> **Internal fragmentation 内部碎片**

适合：固定长度记录的 sequential file。

---

### 第二种：Variable-length spanned blocking 可变长度跨块成块

**Variable-length records 可变长度记录**，尽量把 block 填满。如果一个 record 放不下，可以跨越两个 block。

优点：

* 几乎没有浪费空间
* record 大小不受单个 block 限制

缺点：

* 一个 record 可能需要两次 I/O
* 实现复杂
* 更新困难

关键词：

> **spanned** = 允许一条记录跨 block。

---

### 第三种：Variable-length unspanned blocking 可变长度非跨块成块

也是可变长度记录，但不允许 record 跨 block。

如果当前 block 剩余空间不够放下一条 record，就浪费掉这部分空间，从下一个 block 开始放。

优点：

* 实现比 spanned 简单
* 一条 record 不会分散在多个 block

缺点：

* 有空间浪费
* record 最大不能超过 block size

关键词：

> **unspanned** = 不允许一条记录跨 block。

---

## 5.3 三种 blocking 对比

| 方法                 | 记录长度 | 是否可跨块 | 空间浪费                     | 实现难度 |
| ------------------ | ---- | ----- | ------------------------ | ---- |
| Fixed blocking     | 固定   | 不跨    | 有 internal fragmentation | 简单   |
| Variable spanned   | 可变   | 可以跨   | 最少                       | 复杂   |
| Variable unspanned | 可变   | 不跨    | 有浪费                      | 中等   |

考试答题模板：

> Fixed blocking stores an integral number of fixed-length records in each block and may cause internal fragmentation. Variable-length spanned blocking packs records without unused space but allows records to span blocks, increasing implementation complexity. Variable-length unspanned blocking does not allow spanning, so it wastes space and limits record size to block size.

---

# 6. 12.7 Secondary Storage Management 二级存储管理

这一节核心问题：

> 文件在磁盘上由 blocks 组成，操作系统要决定如何给文件分配 blocks，以及如何管理空闲 blocks。

两个大问题：

1. **File allocation 文件分配**
   文件的数据块放在哪里？

2. **Free space management 空闲空间管理**
   哪些磁盘块还没被使用？

---

## 6.1 Preallocation vs Dynamic allocation

### Preallocation 预分配

创建文件时就声明最大大小，一次性分配空间。

优点：

* 管理简单
* 可能提高连续性和性能

缺点：

* 用户很难准确估计文件最大大小
* 容易浪费空间

### Dynamic allocation 动态分配

文件需要增长时，再分配新的 blocks。

优点：

* 节省空间
* 适合大小不确定的文件

缺点：

* 管理复杂
* 可能导致文件块分散，影响性能

---

## 6.2 Portion size 分配单位大小

**Portion** 是一段连续分配的 blocks。

两个极端：

1. 整个文件分配为一个大的连续 portion
2. 每次只分配一个 block

取舍：

| 大 portion | 小 block |
| --------- | ------- |
| 顺序访问性能好   | 灵活      |
| 表项少       | 不容易浪费   |
| 可能产生外部碎片  | 表项可能很多  |
| 空间复用困难    | 局部性较差   |

---

# 7. File Allocation Methods 文件分配方法

这里是 12.7 最重要部分。

## 7.1 Contiguous allocation 连续分配

文件占用一段连续 blocks。

文件分配表只需要记录：

* start block 起始块
* length 长度

如果文件从 block `b` 开始，要访问第 `i` 个 block：

```text
physical block = b + i - 1
```

优点：

* 顺序访问非常快
* 直接访问也容易
* 表项很小

缺点：

* 需要预先知道文件大小
* 会产生 **external fragmentation 外部碎片**
* 文件增长困难
* 可能需要 compaction 紧凑整理

类比第 7 章内存管理：

> 连续文件分配的问题很像连续内存分配：都会有 external fragmentation。

---

## 7.2 Chained allocation 链式分配

文件的每个 block 里保存一个 pointer，指向下一个 block。

文件分配表只需要记录：

* start block
* length

优点：

* 不需要连续空间
* 没有 external fragmentation
* 文件容易动态增长

缺点：

* 随机访问很慢
  要找第 i 个 block，必须从头沿链走 i 次。
* 指针占用空间
* 文件块可能分散在磁盘各处，locality 局部性差
* 顺序读可能需要频繁磁盘寻道

适合：

> sequential access 顺序访问。

---

## 7.3 Indexed allocation 索引分配

每个文件有一个 index block，里面记录该文件所有数据块的位置。

优点：

* 支持 sequential access
* 支持 direct/random access
* 不需要连续空间
* 比 chained allocation 更适合随机访问

缺点：

* 需要额外 index block
* 小文件可能浪费索引空间
* 大文件需要多级索引或更复杂结构

这是最常见、最折中的方法。

---

## 7.4 三种 allocation 对比

| 方法                    | 优点           | 缺点          | 适合        |
| --------------------- | ------------ | ----------- | --------- |
| Contiguous allocation | 顺序和随机访问都快，表小 | 外部碎片，文件增长困难 | 大小已知、顺序读多 |
| Chained allocation    | 无外部碎片，易增长    | 随机访问慢，局部性差  | 顺序访问文件    |
| Indexed allocation    | 支持顺序和随机访问，灵活 | 有索引开销       | 通用文件系统    |

最容易考的英文问法：

> Compare contiguous, chained, and indexed allocation.

答题时按四点写：

1. 是否需要连续空间
2. 是否有 fragmentation
3. 顺序访问性能
4. 随机访问性能

---

# 8. First fit / Best fit / Nearest fit

这是你高亮的部分。

它们用于 **variable-size portions 可变大小连续块** 的分配。

### First fit 首次适应

选择空闲列表中第一个足够大的连续块。

优点：

* 快

缺点：

* 可能在前面留下很多小碎片

---

### Best fit 最佳适应

选择所有可用块中最小但足够大的那一块。

优点：

* 单次看起来最省空间

缺点：

* 容易产生很多很小的碎片
* 搜索成本较高

---

### Nearest fit 最近适应

选择离该文件上一次分配位置最近的足够大空闲块。

优点：

* 增强 locality 局部性
* 对顺序访问更友好

缺点：

* 不一定最省空间
* 实现需要记录文件前一次分配位置

一句话区分：

| 策略          | 关键词          |
| ----------- | ------------ |
| First fit   | 找到第一个够大的     |
| Best fit    | 找最小但够大的      |
| Nearest fit | 找离上次分配最近且够大的 |

---

# 9. Free Space Management 空闲空间管理

文件系统必须知道哪些 blocks 是空闲的。

## 9.1 Bit table / Bitmap 位图

一个 block 对应一个 bit。

例如：

* `0` 表示 free block
* `1` 表示 used block

优点：

* 空间很小
* 容易找连续空闲块

公式：

```text
bitmap bytes = disk size in bytes / (8 × file system block size)
```

PDF 例子：

```text
16GB disk, 512-byte blocks
bitmap ≈ 4MB
```

缺点：

* 磁盘很大时 bitmap 仍然可能很大
* 如果放磁盘上，搜索慢
* 如果放内存里，占用内存

---

## 9.2 Chained free portions 空闲块链表

把所有空闲 portion 链起来，每个空闲 portion 里记录：

* pointer 指向下一个空闲 portion
* length 长度

优点：

* 几乎不需要额外表空间

缺点：

* 磁盘碎片化后，链会很长
* 分配和删除大量小块时慢

---

## 9.3 Indexing 索引法

把空闲空间也看成一个“文件”，用 index table 管理空闲 portions。

优点：

* 支持多种 allocation 方法
* 对 variable-size portions 比较高效

---

## 9.4 Free block list 空闲块列表

把所有空闲 block 的编号存在磁盘保留区域。

可以用两种方式优化：

1. **Stack 栈**
   分配时 pop，释放时 push。

2. **FIFO queue 队列**
   从队头分配，从队尾释放。

优点：

* 大部分时候只需要访问内存中的一小段列表
* 性能较好

缺点：

* 整个列表通常太大，不能全放内存

---

# 10. Volumes 卷

**Volume** 可以理解成 logical disk 逻辑磁盘。

可能情况：

1. 一个物理磁盘 = 一个 volume
2. 一个磁盘分成多个 partitions，每个 partition 是一个 volume
3. 多个磁盘组合成一个 volume

重点：

> Volume 是操作系统看到的逻辑存储单位，不一定等于一个物理磁盘。

---

# 11. Reliability 可靠性

这一部分讲的是文件系统崩溃一致性问题。

典型错误场景：

1. 用户 A 扩展文件，系统在内存中更新了 allocation table
2. 但还没写回磁盘
3. 系统崩溃
4. 重启后，系统以为那块空间还是空闲
5. 用户 B 又被分配到同一块
6. A 和 B 文件发生重叠，数据损坏

解决思路：

* lock disk allocation table
* update disk allocation table
* update file allocation table
* write changes back to disk
* unlock table

但如果每次小分配都这么做，性能很差。

所以可以用：

> **batch storage allocation 批量存储分配**

先批量拿一批空闲块，在内存中分配。崩溃后再清理未正确使用的块。

---

# 12. 最容易混淆的点

### 1. File organization vs File allocation

| 概念                | 中文   | 问题          |
| ----------------- | ---- | ----------- |
| File organization | 文件组织 | 记录逻辑上怎么访问   |
| File allocation   | 文件分配 | 文件块物理上怎么放磁盘 |

例如：

* sequential file、indexed file、hashed file 是 **file organization**
* contiguous、chained、indexed allocation 是 **file allocation**

不要把 **indexed file** 和 **indexed allocation** 混在一起。

---

### 2. Internal fragmentation vs External fragmentation

| 类型                     | 出现场景                  | 含义               |
| ---------------------- | --------------------- | ---------------- |
| Internal fragmentation | fixed blocking        | block 内部剩余空间浪费   |
| External fragmentation | contiguous allocation | 磁盘空闲空间分散，找不到大连续区 |

---

### 3. Spanned vs Unspanned

| 类型        | 是否允许一条 record 跨 block |
| --------- | --------------------- |
| Spanned   | 允许                    |
| Unspanned | 不允许                   |

---

# 13. 考试答题版总结

这一章可以压缩成一句主线：

> 文件系统把用户看到的 files / records 映射到底层 secondary storage blocks。为了提高访问效率，需要设计 file organization、directory、record blocking、file allocation 和 free space management。

重点背：

1. **Record Blocking 三类**
   fixed、variable spanned、variable unspanned。

2. **File Allocation 三类**
   contiguous、chained、indexed。

3. **Free Space Management 四类**
   bitmap、chained free portions、indexing、free block list。

4. **B-tree**
   平衡、多路、适合索引；插入满节点时 split，中位 key promote。

5. **Directory**
   保存文件名、位置、大小、权限、使用信息；现代系统多用 tree-structured directory。





相关题目主要是 **Review Questions 12.1–12.11**，以及 **Problems 12.1–12.11**。后面的 **12.12、12.13** 属于 UNIX inode，更偏 12.8，不是这次 12.6、12.7 的重点，所以先不做。题目来源见 Chapter 12 questions 的 Review Questions 和 Problems 部分。 

---

# Review Questions

## 12.1 Field 和 Record 有什么区别？

**Field 字段**是最小的数据单位，例如姓名、日期、学号。
**Record 记录**是若干相关字段组成的整体，例如一个学生的完整信息记录。

英文答法：

> A field is a single data item, while a record is a collection of related fields treated as a unit.

---

## 12.2 File 和 Database 有什么区别？

**File 文件**是相似 records 的集合。
**Database 数据库**是相关数据的集合，通常包含多个文件，并且数据之间的关系是明确的。

英文答法：

> A file is a collection of similar records. A database is a collection of related data, often consisting of multiple files, with explicit relationships among data elements.

---

## 12.3 What is a file management system?

**File management system 文件管理系统**是操作系统中负责管理文件的软件部分。它提供创建、删除、读写、修改、共享、保护、备份和恢复文件等功能。

英文答法：

> A file management system is system software that provides services for users and applications to create, delete, access, modify, protect, and manage files.

---

## 12.4 选择文件组织方式时看哪些标准？

选择 **file organization 文件组织方式**时主要看：

1. **Short access time**：访问时间短
2. **Ease of update**：容易更新
3. **Economy of storage**：节省存储空间
4. **Simple maintenance**：维护简单
5. **Reliability**：可靠性高

教材明确把 file organization 定义为记录的逻辑组织方式，并列出这些选择标准。

---

## 12.5 五种 file organization

| English                 | 中文          | 特点                                  |
| ----------------------- | ----------- | ----------------------------------- |
| Pile                    | 堆文件         | 数据按到达顺序堆放，无结构，查找靠全表扫描               |
| Sequential file         | 顺序文件        | 按 key field 排序，适合批处理                |
| Indexed sequential file | 索引顺序文件      | 顺序文件 + index + overflow file，支持较快查找 |
| Indexed file            | 索引文件        | 多个索引，可按多个字段查找                       |
| Direct / hashed file    | 直接文件 / 哈希文件 | 用 hash function 根据 key 直接定位         |

---

## 12.6 为什么 indexed sequential file 平均查找时间少于 sequential file？

因为 **sequential file 顺序文件**要从头开始线性扫描，平均要查一半文件。

而 **indexed sequential file 索引顺序文件**先查 index，快速定位到目标记录附近，然后只在局部范围内顺序查找。

例如教材中说：100 万条记录顺序查找平均约 50 万次访问；加 1000 项索引后，平均搜索长度可降到约 1000 次。

英文答法：

> Because the index provides an entry point close to the desired record, so the system does not need to scan the whole file sequentially.

---

## 12.7 Directory 目录上有哪些典型操作？

典型目录操作：

| English          | 中文     |
| ---------------- | ------ |
| Search           | 查找目录项  |
| Create file      | 创建文件   |
| Delete file      | 删除文件   |
| List directory   | 列出目录内容 |
| Update directory | 更新目录项  |

---

## 12.8 Pathname 和 Working directory 的关系是什么？

**Pathname 路径名**是从某个目录到文件的路径。
**Working directory / Current directory 当前工作目录**是用户或进程当前所在目录。

如果使用 **absolute pathname 绝对路径**，从根目录开始写。
如果使用 **relative pathname 相对路径**，就是相对于当前工作目录写。

例如当前目录是：

```text
/User_B/Word
```

那么文件：

```text
/User_B/Word/Unit_A/ABC
```

可以简写为：

```text
Unit_A/ABC
```

树形目录可以让同名文件存在于不同目录，只要完整 pathname 不同即可。

---

## 12.9 文件访问权限有哪些？

典型 **access rights 访问权限**：

| English             | 中文         |
| ------------------- | ---------- |
| None                | 无权限        |
| Knowledge           | 知道文件存在     |
| Execution           | 执行         |
| Reading             | 读取         |
| Appending           | 追加         |
| Updating            | 更新、修改、删除内容 |
| Changing protection | 修改保护权限     |
| Deletion            | 删除文件       |

这些权限通常是层级式的，例如拥有 updating 权限通常也意味着可以 read 和 append。

---

## 12.10 三种 blocking methods

| Blocking method                    | 中文      | 特点                                                      |
| ---------------------------------- | ------- | ------------------------------------------------------- |
| Fixed blocking                     | 固定成块    | 固定长度记录，一个 block 放整数个 records，可能有 internal fragmentation |
| Variable-length spanned blocking   | 可变长度跨块  | 记录可跨越两个 block，空间利用率高，但实现复杂                              |
| Variable-length unspanned blocking | 可变长度非跨块 | 记录不能跨 block，较简单，但会浪费空间，且记录不能大于 block                    |

教材强调：用户关心 records / fields，但 I/O 按 blocks 进行，所以记录必须组织成块。

---

## 12.11 三种 file allocation methods

| Method                | 中文   | 特点                                              |
| --------------------- | ---- | ----------------------------------------------- |
| Contiguous allocation | 连续分配 | 文件占连续 blocks，顺序和随机访问快，但有 external fragmentation |
| Chained allocation    | 链式分配 | 每个 block 指向下一个 block，无外部碎片，但随机访问慢               |
| Indexed allocation    | 索引分配 | 每个文件有 index block，支持顺序和随机访问，是常用折中方案             |

教材 12.7 明确把常见 file allocation methods 分为 contiguous、chained、indexed。

---

# Problems

## 12.1 Blocking factor 公式

题目定义：

```text
B = block size
R = record size
P = size of block pointer
F = blocking factor
```

### 1. Fixed blocking

固定长度记录，一个 block 放整数条记录：

```text
F = floor(B / R)
```

---

### 2. Variable-length spanned blocking

允许记录跨 block。若每个跨块 block 需要一个 pointer，近似可写：

```text
F ≈ (B - P) / R
```

如果题目忽略 pointer overhead，也常写成：

```text
F ≈ B / R
```

考试中看到 P 一般要写进公式。

---

### 3. Variable-length unspanned blocking

不允许跨块，因此一块中只能放得下的记录数量：

```text
F ≈ floor(B / R)
```

严格来说，如果 record length 真的是随机可变的，unspanned 的期望 F 需要知道记录长度分布；没有分布时，用平均 record size R 近似。

---

## 12.2 文件按 1、2、4、8…… blocks 增长

设：

```text
m = ceil(n / F)
```

其中 `m` 是文件实际需要的 block 数。

### a. File allocation table 最多多少项？

每次分配 portion 大小翻倍：

```text
1, 2, 4, ..., 2^(k-1)
```

总共分配：

```text
1 + 2 + 4 + ... + 2^(k-1) = 2^k - 1
```

要满足：

```text
2^k - 1 >= m
```

所以：

```text
k >= log2(m + 1)
```

答案：

```text
number of entries <= ceil(log2(ceil(n / F) + 1))
```

---

### b. 任意时刻最多浪费多少已分配空间？

最大浪费发生在刚刚分配一个新 portion 时。

如果新 portion 大小是：

```text
2^(k-1)
```

刚使用其中 1 个 block，则最多未使用：

```text
2^(k-1) - 1 blocks
```

对于最终大小 `m = ceil(n/F)`，可以写成：

```text
maximum unused <= 2^(ceil(log2(m + 1)) - 1) - 1 blocks
```

直观理解：这种翻倍策略最多大约浪费不到当前已用空间的一半，但换来了 allocation table 项数很少。

---

## 12.3 不同场景选择哪种 file organization？

### a. Updated infrequently and accessed frequently in random order

**更新少，随机访问多。**

选择：

```text
Direct / hashed file
```

如果只按一个 key 查找，hashed file 最快。

也可以答：

```text
Indexed file
```

如果需要按多个字段查找。

---

### b. Updated frequently and accessed in its entirety relatively frequently

**经常更新，而且经常整文件处理。**

选择：

```text
Indexed sequential file
```

原因：它保留 sequential file 的顺序处理优势，同时通过 overflow file 支持插入和更新。

如果题目更强调“每次都完整顺序处理”，也可说 sequential file，但更新频繁时 sequential file 维护成本较高。

---

### c. Updated frequently and accessed frequently in random order

**更新频繁，随机访问也频繁。**

选择：

```text
Indexed file
```

原因：支持随机查找，也比纯 sequential file 更适合频繁检索。
如果只按唯一 key 查找，也可以选：

```text
Direct / hashed file
```

---

## 12.4 B-tree 插入 key = 97

题目写的是 Figure 12.4c，但你给的 12.1–12.7 PDF 中对应可见的是 **Figure 12.5(c)** 的 B-tree。我按这个图解答。图中 B-tree 的 minimum degree 是 `d = 3`，所以每个节点最多：

```text
2d - 1 = 5 keys
```

Figure 12.5(c) 的根节点是：

```text
[23, 39, 51, 61, 71]
```

最右叶节点是：

```text
[73, 85, 88, 90, 96]
```

插入 97：

1. 97 应进入最右叶节点。
2. 最右叶节点已经满了。
3. 围绕 median key = 88 分裂：

   ```text
   [73, 85]   88   [90, 96, 97]
   ```
4. 88 要提升到父节点。
5. 根节点 `[23, 39, 51, 61, 71]` 也已经满了。
6. 根节点围绕 median key = 51 分裂。
7. 51 成为新 root。

最终结果：

```text
                [51]
              /      \
        [23, 39]     [61, 71, 88]
       /   |   \      /    |     |      \
 [2,10] [30,32] [43,44,45] [52,59,60] [67,68] [73,85] [90,96,97]
```

B-tree 插入规则就是：节点满时 split，中位 key promote。教材也明确说明插入时若节点已有 `2d - 1` 个 key，就围绕 median key 分裂，并把 median key 提升到上一层。

---

## 12.5 B-tree 插入时提前 split 满节点

题目说：插入时向下走，遇到 full node 就立即 split，即使最后可能不需要 split。

### a. 优点

优点是：

```text
不需要回溯，插入可以 top-down 一趟完成。
```

也就是说，往下走的时候就保证即将进入的节点不是满的。这样实现更简单，磁盘 I/O 也更好控制，因为父节点还在手里时就能处理子节点 split。

---

### b. 缺点

缺点是：

1. 可能 split 不必要的节点。
2. 增加写磁盘次数。
3. 增加节点数量。
4. 降低空间利用率。
5. 可能让树更早长高。

一句话：

> Advantage: avoids backtracking.
> Disadvantage: may perform unnecessary splits and waste space.

---

## 12.6 B-tree 高度上界

B-tree 的查找和插入时间取决于树高 `h`。

对于 minimum degree 为 `d` 的 B-tree，最坏情况下每个节点尽量少放 key。

标准结论：

```text
n >= 2d^h - 1
```

所以：

```text
h <= log_d((n + 1) / 2)
```

英文答法：

> For a B-tree of minimum degree d with n keys, the worst-case height h satisfies
> h ≤ log_d((n + 1) / 2).

注意：这里的 `h` 按常见定义是 root 到叶子的边数。如果教材或老师把 height 定义为层数，答案会相差 1。

---

## 12.7 16K block 的最后一块浪费率

Block size：

```text
16K = 16 × 1024 = 16384 bytes
```

公式：

```text
allocated = ceil(file_size / 16384) × 16384
waste = allocated - file_size
waste percentage = waste / allocated × 100%
```

### a. 41,600 bytes

```text
ceil(41600 / 16384) = 3 blocks
allocated = 3 × 16384 = 49152
waste = 49152 - 41600 = 7552
percentage = 7552 / 49152 ≈ 15.36%
```

答案：

```text
15.36%
```

---

### b. 640,000 bytes

```text
ceil(640000 / 16384) = 40 blocks
allocated = 40 × 16384 = 655360
waste = 655360 - 640000 = 15360
percentage = 15360 / 655360 ≈ 2.34%
```

答案：

```text
2.34%
```

---

### c. 4,064,000 bytes

```text
ceil(4064000 / 16384) = 249 blocks
allocated = 249 × 16384 = 4079616
waste = 4079616 - 4064000 = 15616
percentage = 15616 / 4079616 ≈ 0.383%
```

答案：

```text
0.383%
```

---

## 12.8 使用 directories 的优点

Directories 目录的优点：

1. 让用户用 symbolic name 文件名访问文件。
2. 把 file name 映射到文件位置和属性。
3. 支持 hierarchical organization 层次化组织。
4. 避免命名冲突，不同目录可有同名文件。
5. 支持 access control 权限控制。
6. 方便 list、search、delete、update 文件。

---

## 12.9 Directory 作为 special file 或 ordinary file 的优缺点

### Special file 特殊文件

只能通过受限系统调用访问。

优点：

* 更安全。
* 不容易被用户程序破坏。
* OS 可以强制维护目录格式。
* 方便保护文件系统一致性。

缺点：

* 灵活性较差。
* 用户不能用普通文件工具直接操作。
* 实现接口更特殊。

---

### Ordinary data file 普通文件

目录也当普通文件处理。

优点：

* 接口统一。
* 可以复用普通文件操作。
* 灵活性高。

缺点：

* 用户或程序可能错误修改目录内容。
* 文件系统更容易损坏。
* 需要额外保护机制防止目录格式被破坏。

---

## 12.10 限制 tree-structured file system 的深度有什么影响？

### 对用户的影响

限制目录深度会让用户不能建立很深的目录结构。例如不能：

```text
/course/os/chapter12/review/questions/final
```

这会降低文件组织灵活性。

### 对系统设计的简化

可能简化：

1. pathname 解析。
2. 递归目录遍历。
3. 内核缓冲区大小。
4. 最大路径长度控制。
5. 防止过深目录导致搜索成本过高。

但现代系统一般不太需要靠限制深度来简化设计。

---

## 12.11 Free space list 指针丢失

### a. 如果 free space pointer 丢了，能重建 free space list 吗？

可以，但代价很高。

方法：

1. 扫描所有目录和 file allocation table。
2. 标记所有已经被文件占用的 blocks。
3. 没有被标记的 blocks 就是 free blocks。
4. 用这些 free blocks 重建 free space list。

缺点是非常慢，而且如果文件分配表本身损坏，结果可能不可靠。

---

### b. 如何保证单次内存故障不会导致 pointer 丢失？

可以保存冗余副本：

```text
keep multiple copies of the free-space pointer
```

例如：

1. 在磁盘 superblock 中保存两份或多份 free-list pointer。
2. 每份带 checksum 或 version number。
3. 更新时用 journaling / write-ahead logging。
4. 重启时选择 checksum 正确且 version 最新的副本。

英文答法：

> Store redundant copies of the free-space pointer in stable storage, protected by checksums or version numbers, and update them atomically or through a log.

---

# 最重点要背的题

优先级最高：

```text
Review 12.10：三种 blocking methods
Review 12.11：三种 file allocation methods
Problem 12.1：blocking factor 公式
Problem 12.2：翻倍 allocation
Problem 12.7：block 浪费率计算
Problem 12.11：free space list 恢复
```

一句话总结：

> 12.6 考 record 如何装进 block；12.7 考 block 如何分配给 file，以及 free blocks 如何管理。
