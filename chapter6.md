这里的“区别”主要是 **Deadlock Avoidance 死锁避免 / Banker’s Algorithm 银行家算法** 和 **Deadlock Detection 死锁检测** 的区别。它们看起来都在做“找一个进程能不能完成”，但本质完全不一样。

# 一句话区别

**Deadlock avoidance / 银行家算法**：
在资源真正分配之前，先问：

> 如果我把这次资源给它，系统以后还安全吗？

**Deadlock detection / 死锁检测**：
资源已经尽量分配了，现在问：

> 系统现在是不是已经死锁了？

书里说 avoidance 是动态判断当前请求如果被批准会不会可能导致死锁，需要知道未来资源需求；而 detection 则是不限制资源访问，资源能给就给，之后周期性检查是否出现 circular wait 循环等待。 

---

# 1. 使用的数据不同

## Avoidance 用 Claim / Max

银行家算法需要知道每个进程的最大需求：

```text
Claim matrix C / Max matrix
```

然后算：

```text
Need = Claim - Allocation
```

也就是：

```text
Need[i] = C[i] - A[i]
```

它判断的是：

```text
Need[i] <= Available
```

中文意思是：
“如果以后这个进程最多还需要这么多资源，当前资源能不能让它完成？”

书里明确说，deadlock avoidance 要求每个进程提前声明 maximum resource requirement 最大资源需求。

---

## Detection 用 Request Q

死锁检测不关心最大需求，它只关心当前已经在等什么资源。

它用的是：

```text
Request matrix Q
```

其中：

```text
Q[i]
```

表示进程 Pi **当前正在请求、正在等待** 的资源。

检测算法判断的是：

```text
Q[i] <= W
```

中文意思是：
“Pi 现在正在等的资源，当前能不能满足？”

书里说 detection 使用 Allocation matrix、Available vector，另外定义 Request matrix Q，其中 Qij 表示进程 i 当前请求资源 j 的数量。

---

# 2. 判断公式不同

## 银行家算法 / Avoidance

公式是：

```text
Claim - Allocation <= Available
```

也就是：

```text
Need <= Available
```

重点在 **最大剩余需求 Need**。

例如：

```text
P1 Max = (3, 2, 2)
P1 Allocation = (1, 0, 0)
P1 Need = (2, 2, 2)
```

如果当前：

```text
Available = (0, 1, 1)
```

那么：

```text
(2, 2, 2) <= (0, 1, 1)
```

不成立，所以 P1 暂时不能保证完成。

---

## 死锁检测 / Detection

公式是：

```text
Request Q <= Work W
```

重点在 **当前请求 Request**。

例如 Figure 6.10 里：

```text
P3 Request Q = (0, 0, 0, 0, 1)
W = (0, 0, 0, 0, 1)
```

所以：

```text
(0, 0, 0, 0, 1) <= (0, 0, 0, 0, 1)
```

成立，于是认为 P3 可以完成，并释放它已经占有的资源。

---

# 3. “找不到进程”时含义不同

这是最容易混的点。

## 银行家算法找不到进程

如果银行家算法找不到一个满足：

```text
Need <= Available
```

的进程，说明：

```text
unsafe state 不安全状态
```

但是：

```text
unsafe state ≠ deadlock
```

不安全状态只是说明“未来可能死锁”，不代表现在已经死锁。

书里 Figure 6.8(b) 就强调：P1 的请求如果批准，会进入 unsafe state，但这 **不是 deadlocked state**，只是有 deadlock potential 死锁风险，所以 avoidance 会提前拒绝。

---

## 死锁检测找不到进程

如果死锁检测算法找不到一个满足：

```text
Q <= W
```

的未标记进程，并且还有进程没被标记，那么：

```text
unmarked processes = deadlocked processes
```

中文：
剩下没标记的进程就是死锁进程。

书里说得很直接：算法结束时，如果还有 unmarked processes，则存在 deadlock；这些 unmarked rows 正好对应 deadlocked processes。

---

# 4. 对 Figure 6.8 和 Figure 6.10 的区别理解

## Figure 6.8：Avoidance 是“提前拦截”

Figure 6.8 中，P1 请求：

```text
Request(P1) = (1, 0, 1)
```

当前资源够，所以不是“资源不够给”。

但如果先假设给它，系统会变成：

```text
Available = (0, 1, 1)
```

这时每个进程至少还需要 1 个 R1，但 R1 已经没有了。

所以银行家算法说：

```text
不能批准 P1 的请求
```

原因不是“现在已经死锁”，而是：

```text
批准后会进入 unsafe state
```

也就是未来可能死锁。

---

## Figure 6.10：Detection 是“已经检查当前状态”

Figure 6.10 中，系统已经处在某个资源分配状态。它不是在问“要不要批准某个请求”，而是在问：

```text
现在谁已经死锁了？
```

检测过程：

```text
P4 没有占有资源，先 mark
W = Available = (0, 0, 0, 0, 1)
P3 的请求 <= W，所以 mark P3
W = W + A(P3)
剩下 P1、P2 都无法满足
```

最后：

```text
P1, P2 unmarked
```

所以：

```text
P1 和 P2 是 deadlocked processes
```

书里 Figure 6.10 的结论就是 P1 和 P2 未标记，因此它们发生死锁。

---

# 5. 是否需要“未来信息”不同

## Avoidance 需要未来信息

银行家算法必须知道：

```text
每个进程最多会要多少资源
```

也就是：

```text
Claim / Max
```

否则无法计算：

```text
Need = Claim - Allocation
```

所以它要求进程提前声明最大需求。这个要求在真实系统里有时很难满足，因为进程不一定知道自己以后要多少资源。

---

## Detection 不需要未来信息

死锁检测只需要当前状态：

```text
Allocation 当前已经占有的资源
Request 当前正在等待的资源
Available 当前可用资源
```

它不需要知道进程未来最大还会请求多少资源。

因此 detection 更现实一些，但代价是：
它不能阻止死锁，只能发现死锁。

---

# 6. 系统策略不同

| 对比点      | Deadlock Avoidance 死锁避免            | Deadlock Detection 死锁检测          |
| -------- | ---------------------------------- | -------------------------------- |
| 发生时间     | 死锁发生前                              | 死锁可能已经发生后                        |
| 主要问题     | 这个请求能不能批准？                         | 现在有没有死锁？                         |
| 使用矩阵     | Claim / Max, Allocation, Available | Request Q, Allocation, Available |
| 核心公式     | `Need = Claim - Allocation`        | `Q = current request`            |
| 判断条件     | `Need <= Available`                | `Q <= W`                         |
| 找不到进程表示  | unsafe state，不一定死锁                 | unmarked processes 就是死锁进程        |
| 是否阻止死锁   | 是                                  | 否                                |
| 后续动作     | 拒绝或阻塞请求                            | recovery：终止、回滚、抢占资源              |
| 是否需要最大需求 | 需要                                 | 不需要                              |
| 资源分配风格   | 比较谨慎                               | 很宽松，能给就给                         |

---

# 7. 可以这样记

## 银行家算法

```text
看未来：如果我现在给你，以后大家还能不能都完成？
```

关键词：

```text
future
safe state
Claim / Max
Need = Claim - Allocation
unsafe ≠ deadlock
```

---

## 死锁检测

```text
看现在：现在有没有进程已经互相等死了？
```

关键词：

```text
current
deadlocked state
Request Q
unmarked = deadlocked
detect and recover
```

---

# 8. 最重要的考试陷阱

## 陷阱一：unsafe state 不等于 deadlock

这是银行家算法里的概念。

```text
Unsafe state means deadlock may occur.
Unsafe state does not necessarily mean deadlock has occurred.
```

中文：

```text
不安全状态表示可能死锁，不表示已经死锁。
```

---

## 陷阱二：detection 中未标记就是死锁

这是死锁检测里的结论。

```text
If processes remain unmarked, they are deadlocked.
```

中文：

```text
检测算法结束后，未标记进程就是死锁进程。
```

---

# 9. 最短总结

**Avoidance / Banker’s Algorithm** 用 `Claim - Allocation` 预测未来，目标是让系统始终处于 **safe state**。如果找不到安全序列，只能说明状态 **unsafe**，不一定已经死锁。

**Detection** 用当前 `Request Q` 检查现在，目标是找出已经发生的 **deadlock**。如果算法结束后还有未标记进程，这些进程就是死锁进程。

我按首页勾画的 **6.1–6.4** 来筛，最相关的是：

**Review Questions：6.1–6.7**
**Problems：6.1–6.6，6.11，6.12，6.15，6.16**

其中 6.7 以后很多题开始涉及 spooling、THE 系统、管道、哲学家、Linux 同步机制，不是首页重点，先不作为核心。题目列表在 *chapter6 questions.pdf* 里，6.1–6.7 复习题和 6.1–6.6 习题都直接对应死锁原理、预防、避免、检测。

---

# Review Questions

## 6.1 Give examples of reusable and consumable resources.

**Reusable resources 可重用资源**：使用后不会消失，释放后还能再用。
例子：processor 处理器、memory 内存、I/O channels、devices 设备、files 文件、databases 数据库、semaphores 信号量。

**Consumable resources 可消耗资源**：被创建后会被消费，消费后就不存在。
例子：interrupts 中断、signals 信号、messages 消息、I/O buffer information 缓冲区信息。

---

## 6.2 What are the three conditions that must be present for deadlock to be possible?

死锁“可能发生”需要前三个条件：

1. **Mutual exclusion 互斥**：一个资源一次只能被一个进程使用。
2. **Hold and wait 占有并等待**：进程已经占有资源，同时等待其他资源。
3. **No preemption 不可抢占**：资源不能被系统强行从进程手里拿走。

书里强调前三个是 deadlock possible 的必要条件，但还不足以说明已经死锁；真正发生死锁还需要 circular wait。

---

## 6.3 What are the four conditions that create deadlock?

四个死锁条件：

1. **Mutual exclusion 互斥**
2. **Hold and wait 占有并等待**
3. **No preemption 不可抢占**
4. **Circular wait 循环等待**

前三个说明“有死锁可能”，加上第四个 **circular wait** 后，才构成实际死锁。

---

## 6.4 How can the hold-and-wait condition be prevented?

破坏 **hold and wait 占有并等待** 的方法：

要求进程在开始运行前，一次性申请所有需要的资源。

英文表达：

> A process must request all of its required resources at one time.

缺点是资源利用率低，因为进程可能很早就占有一些暂时不用的资源，导致其他进程无法使用。

---

## 6.5 List two ways in which the no-preemption condition can be prevented.

破坏 **no preemption 不可抢占** 有两种方法：

第一，如果进程申请新资源失败，就释放它已经持有的资源，然后以后重新申请。

第二，如果一个进程请求的资源正被另一个进程持有，系统可以强制让持有者释放资源。

但这种方法只适合状态容易保存和恢复的资源，比如 CPU，不太适合文件写入、打印机、数据库事务等。

---

## 6.6 How can the circular-wait condition be prevented?

破坏 **circular wait 循环等待** 的方法：

给所有资源类型规定一个线性顺序，比如：

```text
R1 < R2 < R3 < R4
```

进程必须按照资源编号递增顺序申请资源。
如果进程已经持有 R2，就只能继续申请 R3、R4，不能回头申请 R1。

这样不会形成等待环，所以不会 circular wait。

---

## 6.7 What is the difference among deadlock avoidance, detection, and prevention?

| English             | 中文   | 核心思想                         |
| ------------------- | ---- | ---------------------------- |
| Deadlock prevention | 死锁预防 | 破坏死锁四条件之一，让死锁不可能发生           |
| Deadlock avoidance  | 死锁避免 | 每次分配前判断是否仍处于 safe state 安全状态 |
| Deadlock detection  | 死锁检测 | 平时允许分配，之后定期检测是否已经死锁          |

一句话：

> Prevention breaks conditions; avoidance avoids unsafe states; detection finds existing deadlocks.

中文：

> 预防是破坏条件，避免是避免进入不安全状态，检测是发现已经发生的死锁。

---

# Problems

## 6.1 Show that the four conditions of deadlock apply to Figure 6.1a.

Figure 6.1 是十字路口汽车死锁例子。严格说，书里 Figure 6.1a 是 **deadlock possible 潜在死锁**，Figure 6.1b 才是实际 deadlock；但四个条件可以这样对应：

1. **Mutual exclusion 互斥**：每个路口象限一次只能被一辆车占用。
2. **Hold and wait 占有并等待**：每辆车占着自己前方的一个象限，同时等待下一个象限。
3. **No preemption 不可抢占**：不能强行把一辆车从已占据的象限中移走。
4. **Circular wait 循环等待**：Car 1 等 Car 2，Car 2 等 Car 3，Car 3 等 Car 4，Car 4 又等 Car 1。

所以形成循环等待链。

---

## 6.2 Show how prevention, avoidance, and detection can be applied to Figure 6.1.

**Prevention 死锁预防**：
可以设置交通灯、交警指挥、规定优先级，或者规定车辆进入路口前必须确认能通过整个路口。这样破坏 circular wait 或 hold and wait。

**Avoidance 死锁避免**：
每次只允许不会导致危险状态的车辆进入路口。例如交警动态判断：如果同时放行四辆车会导致死锁，就只放行一辆或一组不冲突的车。

**Detection 死锁检测**：
先允许车辆进入，之后如果发现四辆车互相堵住，就检测到死锁。恢复方法可以是让某一辆车倒车、拖走、或由交警强制改变顺序。

---

## 6.3 For Figure 6.3, describe each of the six paths.

Figure 6.3 的程序逻辑是：

```text
Process P: Get A → Release A → Get B → Release B
Process Q: Get B → Get A → Release B → Release A
```

注意 P 在申请 B 之前已经释放 A，所以不会出现 “P 拿着 A 等 B，同时 Q 拿着 B 等 A” 的真正循环等待。

六条路径可以理解为：

1. **Path 1**：Q 基本先运行完，先拿 B，再拿 A，然后释放 B、A；之后 P 可以正常运行。
2. **Path 2**：P 先拿到并释放 A，之后 Q 获得 B 和 A 并完成；没有形成互等。
3. **Path 3**：Q 先拿 B，P 拿 A 后释放 A；即使 P 后面等 B，Q 也能拿到 A 并最终释放 B，所以 P 之后可继续。
4. **Path 4**：P 释放 A 后，Q 能进入使用 A 的阶段；之后 P 使用 B，不会互相持有对方需要的资源。
5. **Path 5**：P 使用 B 时，Q 可能等待 B；但 P 不再持有 A，释放 B 后 Q 可以继续。
6. **Path 6**：P 基本先完成，然后 Q 再完成。

共同点：每条路径都不会进入 fatal region 致命区域，所以没有死锁。

---

## 6.4 Why can deadlock not occur in Figure 6.3?

因为 P 的资源顺序是：

```text
Get A → Release A → Get B
```

P 不会在持有 A 的同时请求 B。

而 Q 是：

```text
Get B → Get A
```

即使 Q 持有 B 并等待 A，P 最终也会释放 A；P 之后如果等待 B，它已经不持有 A，所以 Q 可以继续执行并释放 B。

所以不存在：

```text
P holds A and waits for B
Q holds B and waits for A
```

这种循环等待，因此不会死锁。

---

# 6.5 Banker’s Algorithm 银行家算法题

题目给出 6 个进程 P0–P5，4 类资源 A、B、C、D，总量为：

```text
A = 15, B = 6, C = 9, D = 10
```

Available 给出为：

```text
Available = (6, 3, 5, 4)
```

题目中的 allocation 和 maximum demand 表格见 PDF 第 303 页。

---

## 6.5(a) Verify Available

当前分配 Allocation：

| Process |  A |  B |  C |  D |
| ------- | -: | -: | -: | -: |
| P0      |  2 |  0 |  2 |  1 |
| P1      |  0 |  1 |  1 |  1 |
| P2      |  4 |  1 |  0 |  2 |
| P3      |  1 |  0 |  0 |  1 |
| P4      |  1 |  1 |  0 |  0 |
| P5      |  1 |  0 |  1 |  1 |

各类资源已分配总量：

```text
A: 2+0+4+1+1+1 = 9
B: 0+1+1+0+1+0 = 3
C: 2+1+0+0+0+1 = 4
D: 1+1+2+1+0+1 = 6
```

所以 Available：

```text
A = 15 - 9 = 6
B = 6 - 3 = 3
C = 9 - 4 = 5
D = 10 - 6 = 4
```

因此：

```text
Available = (6, 3, 5, 4)
```

题目给的是正确的。

---

## 6.5(b) Calculate Need matrix

公式：

```text
Need = Maximum demand - Allocation
```

| Process | Need A | Need B | Need C | Need D |
| ------- | -----: | -----: | -----: | -----: |
| P0      |      7 |      5 |      3 |      4 |
| P1      |      2 |      1 |      2 |      2 |
| P2      |      3 |      4 |      4 |      2 |
| P3      |      2 |      3 |      3 |      1 |
| P4      |      4 |      1 |      2 |      1 |
| P5      |      3 |      4 |      3 |      3 |

---

## 6.5(c) Show safe sequence

初始：

```text
Available = (6, 3, 5, 4)
```

找满足：

```text
Need <= Available
```

的进程。

可以选：

```text
P1
```

因为：

```text
Need(P1) = (2, 1, 2, 2) <= (6, 3, 5, 4)
```

P1 完成后释放 Allocation(P1)：

```text
Available = (6,3,5,4) + (0,1,1,1)
          = (6,4,6,5)
```

接着选 P2：

```text
Need(P2) = (3,4,4,2) <= (6,4,6,5)
```

释放后：

```text
Available = (6,4,6,5) + (4,1,0,2)
          = (10,5,6,7)
```

接着选 P0：

```text
Need(P0) = (7,5,3,4) <= (10,5,6,7)
```

释放后：

```text
Available = (10,5,6,7) + (2,0,2,1)
          = (12,5,8,8)
```

接着选 P3：

```text
Available = (12,5,8,8) + (1,0,0,1)
          = (13,5,8,9)
```

接着选 P4：

```text
Available = (13,5,8,9) + (1,1,0,0)
          = (14,6,8,9)
```

最后选 P5：

```text
Available = (14,6,8,9) + (1,0,1,1)
          = (15,6,9,10)
```

所以一个 safe sequence 是：

```text
P1 → P2 → P0 → P3 → P4 → P5
```

当前状态是 **safe state 安全状态**。

---

## 6.5(d) P5 requests (3,2,3,3). Should it be granted?

P5 的 Need 是：

```text
Need(P5) = (3,4,3,3)
```

请求：

```text
Request(P5) = (3,2,3,3)
```

先检查：

```text
Request <= Need
(3,2,3,3) <= (3,4,3,3)
```

成立。

再检查：

```text
Request <= Available
(3,2,3,3) <= (6,3,5,4)
```

也成立。

所以资源“现在够给”。但是银行家算法还要假设分配后检查 safe state。

假设分配给 P5：

```text
Available = (6,3,5,4) - (3,2,3,3)
          = (3,1,2,1)
```

P5 新的 Need：

```text
Need(P5) = (3,4,3,3) - (3,2,3,3)
         = (0,2,0,0)
```

现在检查所有进程：

```text
P0 Need = (7,5,3,4)  不 <= (3,1,2,1)
P1 Need = (2,1,2,2)  不 <= (3,1,2,1)，D 不够
P2 Need = (3,4,4,2)  不 <= (3,1,2,1)
P3 Need = (2,3,3,1)  不 <= (3,1,2,1)
P4 Need = (4,1,2,1)  不 <= (3,1,2,1)
P5 Need = (0,2,0,0)  不 <= (3,1,2,1)，B 不够
```

没有任何进程能完成。

所以分配后会进入 **unsafe state 不安全状态**。

答案：

```text
不能批准 P5 的请求。
```

原因：虽然当前资源数量够，但批准后找不到 safe sequence。

---

# 6.6 Resource allocation graph 资源分配图题

题目代码中三个进程分别请求：

```text
P0: A → B → C
P1: D → E → B
P2: C → F → D
```

题目要求画资源分配图说明死锁可能，并修改请求顺序避免死锁。

---

## 6.6(a) Show possibility of deadlock

可以出现如下执行顺序：

```text
P0 gets A
P0 gets B

P1 gets D
P1 gets E

P2 gets C
P2 gets F
```

此时：

```text
P0 holds A, B; requests C
P1 holds D, E; requests B
P2 holds C, F; requests D
```

形成循环：

```text
P0 → C → P2 → D → P1 → B → P0
```

中文解释：

* P0 等 C，而 C 被 P2 占有。
* P2 等 D，而 D 被 P1 占有。
* P1 等 B，而 B 被 P0 占有。

所以存在 circular wait，可能死锁。

---

## 6.6(b) Modify order to prevent deadlock

用 **resource ordering 资源排序法**。

规定全局顺序：

```text
A < B < C < D < E < F
```

所有进程都必须按照这个顺序申请资源。

修改后：

```c
void P0()
{
 while (true) {
   get(A);
   get(B);
   get(C);
   // use A, B, C
   release(A);
   release(B);
   release(C);
 }
}
```

P0 原本已经符合顺序。

P1 使用 B、D、E，所以改为：

```c
void P1()
{
 while (true) {
   get(B);
   get(D);
   get(E);
   // use D, E, B
   release(D);
   release(E);
   release(B);
 }
}
```

P2 使用 C、D、F，所以改为：

```c
void P2()
{
 while (true) {
   get(C);
   get(D);
   get(F);
   // use C, F, D
   release(C);
   release(F);
   release(D);
 }
}
```

这样所有请求都是从小编号资源到大编号资源，不会出现回头申请较小资源，因此不会形成 circular wait。

---

# 6.11 Banker’s Algorithm memory question

题目给出总内存：

```text
Total memory = 150
```

当前三个进程：

| Process | Max | Hold | Need |
| ------- | --: | ---: | ---: |
| P1      |  70 |   45 |   25 |
| P2      |  60 |   40 |   20 |
| P3      |  60 |   15 |   45 |

当前可用内存：

```text
Available = 150 - (45 + 40 + 15)
          = 50
```

---

## 6.11(a) Fourth process max 60, initial need 25

如果给 P4 初始 25：

| Process | Max | Hold | Need |
| ------- | --: | ---: | ---: |
| P1      |  70 |   45 |   25 |
| P2      |  60 |   40 |   20 |
| P3      |  60 |   15 |   45 |
| P4      |  60 |   25 |   35 |

新的 Available：

```text
Available = 50 - 25 = 25
```

现在 P1 的 Need 是 25，可以完成。

安全序列可以是：

```text
P1 → P2 → P3 → P4
```

计算：

```text
Available = 25
P1 finishes: Available = 25 + 45 = 70
P2 finishes: Available = 70 + 40 = 110
P3 finishes: Available = 110 + 15 = 125
P4 finishes: Available = 125 + 25 = 150
```

所以 6.11(a) 可以批准。

---

## 6.11(b) Fourth process max 60, initial need 35

如果给 P4 初始 35：

| Process | Max | Hold | Need |
| ------- | --: | ---: | ---: |
| P1      |  70 |   45 |   25 |
| P2      |  60 |   40 |   20 |
| P3      |  60 |   15 |   45 |
| P4      |  60 |   35 |   25 |

新的 Available：

```text
Available = 50 - 35 = 15
```

检查 Need：

```text
P1 Need = 25 > 15
P2 Need = 20 > 15
P3 Need = 45 > 15
P4 Need = 25 > 15
```

没有任何进程能完成。

所以这是 unsafe state，不应批准。

答案：

```text
6.11(a) safe，可以批准。
6.11(b) unsafe，不能批准。
```

---

# 6.12 Evaluate the banker’s algorithm for its usefulness in an OS.

银行家算法理论上很清楚，但在真实通用 OS 中不太实用。

优点：

* 可以避免死锁。
* 不需要真的发生死锁后再恢复。
* 比 deadlock prevention 更灵活，因为不是直接禁止某个条件。

缺点：

* 每个进程必须提前声明最大资源需求。
* 资源数量最好固定。
* 进程之间要相对独立。
* 每次资源请求都要做安全性检查，有运行时开销。
* 可能拒绝一些实际上不会造成死锁的请求，降低并发度。

结论：

> Banker’s algorithm is useful as a theoretical deadlock avoidance model, but it is often impractical for general-purpose operating systems.

中文：

> 银行家算法适合作为理论模型或资源需求可预测的系统方法，但不太适合复杂通用操作系统。

---

# 6.15 Minimum available units for safe state

题目给：

```text
C = (3, 2, 9, 7)
A = (1, 1, 3, 2)
```

单一资源，所以 Need：

```text
Need = C - A
     = (2, 1, 6, 5)
```

问：最少 Available 是多少，才能 safe？

试 Available = 1：

```text
P2 Need = 1，可以完成
Available = 1 + 1 = 2
P1 Need = 2，可以完成
Available = 2 + 1 = 3
```

剩下：

```text
P3 Need = 6
P4 Need = 5
```

都不能完成，所以 Available = 1 不够。

试 Available = 2：

```text
P1/P2 可以完成
完成后 Available 最多到 4
```

仍然无法满足 P4 Need = 5，所以不够。

试 Available = 3：

安全序列：

```text
P2 → P1 → P4 → P3
```

计算：

```text
Available = 3
P2 finishes: Available = 3 + 1 = 4
P1 finishes: Available = 4 + 1 = 5
P4 finishes: Available = 5 + 2 = 7
P3 finishes: Available = 7 + 3 = 10
```

所以最小 Available 是：

```text
3
```

---

# 6.16 Compare deadlock handling methods

题目列了 6 种方法：

1. Banker’s algorithm 银行家算法
2. Detect deadlock and kill thread 检测死锁并杀死线程
3. Reserve all resources in advance 预先保留全部资源
4. Restart thread and release all resources if thread needs to wait 需要等待就重启并释放资源
5. Resource ordering 资源排序
6. Detect deadlock and roll back thread’s actions 检测死锁并回滚线程

题目 6.16 在 PDF 第 306 页。

---

## 6.16(a) Rank by greatest concurrency

并发度最高的是 detection，因为它平时不限制资源请求，只有死锁发生后才处理。

一个合理排序是：

| Rank | Method                              |
| ---: | ----------------------------------- |
|    1 | 2. Detect and kill thread           |
|    2 | 6. Detect and roll back             |
|    3 | 1. Banker’s algorithm               |
|    4 | 5. Resource ordering                |
|    5 | 4. Restart if need to wait          |
|    6 | 3. Reserve all resources in advance |

解释：

* **Detection** 最宽松，所以并发度最高。
* **Banker’s algorithm** 会拒绝 unsafe request，所以并发度低一些。
* **Resource ordering** 限制申请顺序。
* **Restart on wait** 会浪费已经做过的工作。
* **Reserve all resources in advance** 最保守，并发度最低。

---

## 6.16(b) Rank by efficiency when deadlock is rare

如果 deadlock 很少发生，最省开销的一般是 prevention 类方法，尤其是资源排序，因为不需要频繁检测。

一个合理排序：

| Rank | Method                              |
| ---: | ----------------------------------- |
|    1 | 5. Resource ordering                |
|    2 | 3. Reserve all resources in advance |
|    3 | 2. Detect and kill thread           |
|    4 | 4. Restart if need to wait          |
|    5 | 1. Banker’s algorithm               |
|    6 | 6. Detect and roll back             |

解释：

* **Resource ordering** 运行时开销很低。
* **Reserve all resources in advance** 算法简单，但资源利用率低。
* **Detect and kill** 如果死锁很少，恢复成本很少发生。
* **Banker’s algorithm** 每次请求都要安全性检查，所以开销较大。
* **Rollback** 需要保存检查点或日志，系统实现复杂，开销最大。

如果 deadlock 很频繁，排序会变化：detection + kill/rollback 的恢复成本会大幅增加，此时 banker’s algorithm 或 resource ordering 会相对更好。

---

# 考前重点背这几题

最核心是：

```text
Review 6.2：死锁可能条件
Review 6.3：死锁四条件
Review 6.7：prevention / avoidance / detection 区别
Problem 6.5：银行家算法
Problem 6.6：资源分配图 + 资源排序破坏 circular wait
Problem 6.15：单资源银行家算法
```

尤其是银行家算法，记住三步：

```text
Need = Max - Allocation
找 Need <= Available 的进程
进程完成后 Available += Allocation
```
