# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Congestion — Basic Ideas（拥塞——基本思想）

> <span style="color:#7f8c8d;">Source: 慕课/Unit4-Congestion-Control/congestion_-_basics.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1：封面</span>

CS144：计算机网络导论。**Congestion（拥塞）：Basic Ideas（基本思想）**。Nick McKeown，斯坦福大学电气工程与计算机科学教授。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 2：Outline of next few videos（接下来几讲的大纲）</span>

- 什么是拥塞控制（congestion control，拥塞控制）？
- 拥塞控制的基本途径：在网络中（in the network）—— 在终端主机（from the end host）
- TCP 拥塞控制：TCP Tahoe —— TCP Reno —— TCP 的 RTT 估计（RTT estimation）—— 实际中的性能

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 3：Time Scales of Congestion（拥塞的时间尺度）</span>

拥塞发生在三个跨度悬殊的时间尺度上（图示：输入曲线 In1(t)、In2(t) 与输出 Out 在三个时间尺度下的对照）：

- **微秒尺度（100 μs / 200 μs / 300 μs）**：两个分组（packet，分组）在路由器（router，路由器）处相撞；
- **毫秒尺度（10 ms / 20 ms / 30 ms）**：若干条流（flow，流）用尽了链路（link，链路）容量；
- **小时尺度（7:00 / 8:00 / 9:00）**：高峰小时内过多用户挤占同一条链路。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这页值得停下来想一想："拥塞"其实是一族现象而非一个现象。微秒尺度上，拥塞就是两台主机的分组在路由器里撞车；毫秒尺度上，是几条流的速率之和超过了链路容量；小时尺度上，是晚高峰时全城用户挤同一条链路。不同尺度的拥塞要用不同机制去对付：微秒尺度靠路由器的缓冲区（buffer，缓冲区）与调度（scheduling，调度），毫秒尺度靠端到端的拥塞控制——这正是本单元的主角，小时尺度则要靠容量规划与流量工程。从概率论的角度看，这与你学过的排队论一脉相承：到达过程的短时突发（burst，突发）与长时间平均速率是两个不同的统计量，前者决定队列的瞬时溢出概率，后者决定平均利用率。任何只盯着一个时间尺度的设计，都会在另一个尺度上翻车。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 4：What causes congestion?（什么导致拥塞？）</span>

图示：主机 A 与主机 B 各自经一条 12 Mb/s 接入链路向路由器发送分组，到达过程分别记为 $A_1(t)$、$A_2(t)$；路由器经另一条 12 Mb/s 链路 $X$ 向外发送，输出过程记为 $D(t)$。当 $A_1(t) + A_2(t)$ 的瞬时速率超过输出链路容量时，路由器内部开始积压。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 5：What causes congestion?（什么导致拥塞？——续）</span>

把 $A_1(t)$、$A_2(t)$ 各自的累积曲线、二者的和 $A_1(t)+A_2(t)$，与输出 $D(t)$ 的累积曲线画在同一张"比特—时间"图上：

- 曲线 $A_1(t)+A_2(t)$ 与 $D(t)$ 之间的垂直距离，就是路由器里排队（queuing，排队）的比特数 $Q(t)$；
- 只要到达速率持续高于链路速率，$Q(t)$ 就会增长。

问题：**如果缓冲区（buffer，缓冲区）是有限的呢？**

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 把 $A_1(t)+A_2(t)$ 与 $D(t)$ 画成累积曲线，两条曲线之间的垂直距离就是队列长度 $Q(t)$——这是排队论里最朴素也最有用的图像，与你学过的利特尔法则（Little's law，利特尔法则）一脉相承：平均队列长度等于平均到达率乘以平均时延。只要到达率超过服务率，$Q(t)$ 就会增长；缓冲区有限时，增长终会撞上容量上限，溢出的那部分就是丢包（packet loss，丢包）。注意这页的隐含对比：缓冲区无限时，拥塞只表现为排队时延（queuing delay，排队时延）变大；缓冲区有限时，拥塞才表现为丢包。TCP 恰恰把丢包当作拥塞信号，所以"缓冲区设多大"不是工程琐事，而是决定整个互联网行为方式的根本参数——后面 4-3 节会给出单流情形下缓冲区 $B = RTT \times C$ 的答案。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 6：Another example（另一个例子）</span>

图示：四台主机 A、B、C、D 由多条 12 Mb/s 链路相连，其中间一段 1 Mb/s 链路是瓶颈（bottleneck，瓶颈）：A 与 B 两条流同时穿过它，各分到不足一半的容量，其余链路皆有余。要点：拥塞的根源不在"哪条链路坏了"，而在**速率的需求之和与瓶颈容量不匹配**。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 7：Congestion is unavoidable（拥塞不可避免——甚至可以说是好事！）</span>

1. 我们使用分组交换（packet switching，分组交换）正是因为它高效利用链路。因此，路由器中的缓冲区经常被占用。
2. 如果缓冲区总是空的：时延（latency，时延）低，但我们对网络的使用率也低。
3. 如果缓冲区总是被占用：时延高，但我们更充分地利用了网络。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "拥塞不可避免、甚至是好事"这个论断值得用第一性原理拆开看。分组交换的全部好处来自统计复用（statistical multiplexing，统计复用）：多条流共享一条链路，各自的需求峰值在时间上错开，于是用一条容量 $C$ 的链路服务 $N$ 条均值远小于 $C$ 的流。但复用的收益恰好来自"排队"，排队的代价恰好是时延，而时延的爆炸点恰好出现在利用率趋近 1 的时候——你在概率论的排队模型里已经见过这个形状：时延随利用率 $\rho \to 1$ 急剧增长。所以"缓冲区总是空的"意味着网络花了钱却没干活的容量，"缓冲区总是满的"意味着时延爆炸，工程上永远活在这两者的折中里。承认拥塞的必然性，才不会去追求"消灭拥塞"这种不可能目标，而是追求"拥塞发生时仍然公平、高效地工作"——这正是下一页的五条观察和整个本单元的出发点。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 8：Observations（观察）</span>

1. 拥塞不可避免，甚至可以说是合乎需要的。
2. 拥塞发生在不同的时间尺度上——从两个分组的相撞，到一些流发送得太快，再到突然涌进网络的闪发人群（flash crowd，闪发人群）。
3. 如果分组被丢弃，重传（retransmission，重传）会让拥塞雪上加霜。
4. 被丢弃的分组在丢弃之前已经浪费了上游的资源。
5. 我们需要一个公平性（fairness，公平性）的定义，来决定希望各条流如何共享一条瓶颈链路。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 9：Fairness and throughput（公平性与吞吐量）</span>

纯图示 slide：三条流 A、B、C 汇入网络，其中链路 1 由 B 独占、链路 2 由 A 与 B 共享，C 经链路 1 与链路 2 之间的路径接入。本页提出问题：这些流各自的速率 $r_A, r_B, r_C$ 该怎么分配，才算既高吞吐又公平？下一页给出具体数字。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 10：Fairness and throughput（公平性与吞吐量：一个例子）</span>

沿用上页拓扑（链路 2 容量为 2），两种分配（Alloca/ons → Allocations，分配）方案：

1. $r_A = 0.25$，$r_B = 1.75$，$r_C = 0.75$，总吞吐量（throughput，吞吐量）$= 2.75$；
2. $r_A = 0.5$，$r_B = 1.5$，$r_C = 0.5$，总吞吐量 $= 2.5$。

分配 #2 更"公平"。**那么，公平到底该如何定义？**

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 11：Max-min Fairness（最大-最小公平）</span>

<span style="color:#00838f;">**定义**：一个分配是最大-最小公平（max-min fair，最大-最小公平）的，当且仅当你**无法在不减少另一个速率更低的流的速率的前提下，增加某条流的速率**。</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这个定义读起来绕，结构上其实是一个字典序最大化的贪心：先最大化境况最差者的分配额，再在保持最差者不变的前提下最大化次差者，依此类推——与你在 6.042J 里处理良序关系、在 CLRS 里见过的"先保底再贪心"思路同源。用上一页的例子验证：容量 2 的链路上，$(0.25,\ 1.75,\ 0.75)$ 总吞吐更大却更不公平——把 B 的 0.25 份额让给 A，A 增加、B 减少，但 B 仍比 A 高，按定义这样的调整就应当继续做下去，迭代到最后落在 $(0.5,\ 1.5,\ 0.5)$。要紧的是：最大-最小公平是一个**目标**，而 TCP 的 AIMD 是一个**机制**，两者相差十万八千里——AIMD 在单瓶颈、丢包同步的理想条件下确实收敛到最大-最小公平（本单元后面会用 Chiu-Jain 相图证明这一点），但真实互联网中各流 RTT 相异、丢包不同步，AIMD 会系统性地亏待长 RTT 流。目标与机制之间的这条缝隙，是本单元反复出现的主题。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 12：Max-min fair allocation（最大-最小公平分配）</span>

图示：把最大-最小公平的定义应用到 Slide 10 的拓扑上，得到的分配正是 $r_A = 0.5$，$r_B = 1.5$，$r_C = 0.5$——B 的瓶颈不是链路 2（容量 2，A、C 合计才占 1），而是自己那条路径上的其他约束，因此 B 在不损害任何比它更慢的流的前提下拿走了剩余容量。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 13：Max-min fairness: Single link（最大-最小公平：单链路情形）</span>

在单条链路上，这个定义直观而简单：所有流平分链路容量。但注意——一旦拓扑复杂起来（多链路、多瓶颈），"公平"的定义与计算都会变得微妙得多。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 14：Goals for congestion control（拥塞控制的目标）</span>

1. **高吞吐量**：让链路保持忙碌、让流保持快速；
2. **最大-最小公平**；
3. 对变化的网络条件**快速响应**；
4. **分布式控制**（distributed control，分布式控制）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 15：结束页</span>

本讲完。CS144, Stanford University。

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 4-1《Congestion Control - Basics》的配套 slides；公平性的机制化（AIMD）在 4-2、4-4 中展开，TCP 的实现则在 4-5 至 4-7 中展开。</span>
