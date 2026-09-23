# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Nick's Review topics（Nick 的期末复习专题）：1. 分组交换；2. 路由、生成树与 Dijkstra —— Nick McKeown

> <span style="color:#7f8c8d;">Source: 课堂讲义/期末复习-Nick McKeown.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

> <span style="color:#7f8c8d;">说明：本文为期末复习课 slides，分两大部分。原稿各页角落的 "CS144, Stanford University" 水印与独立页码不再重复出现。本 PDF 的数学符号在 pdftotext 提取中大面积丢失（希腊字母、上下标几乎全部残缺），凡公式均按残存数字与 CS144 课程的标准记号复原，并在相应位置以灰色脚注声明；图示连线关系已散失，仅按残留数字描述。例题解答按原文以引用块呈现。</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">复习主题一：分组交换</span>

CS144: An Introduction to Computer Networks — Packet Switching（复习主题一：分组交换）。本部分提纲：

1. 端到端时延（End-to-end delay）；
2. 排队时延（Queueing delay）；
3. 简单确定性排队模型（Simple deterministic queue model）；
4. 速率保证（Rate guarantees）；
5. 时延保证（Delay guarantees）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">传播时延</span>

<span style="color:#00838f;">传播时延（propagation delay，传播时延）$t_l$：一个比特以传播速度 $c$ 越过一条链路所需的时间。</span>

```math
t_l = \frac{l}{c}
```

（按原文复原，$l$ 为链路长度。）

例：一个比特在光纤中走 1,000 km（传播速度 $2\times10^8$ m/s）需要 5 ms。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">分组化（传输）时延</span>

<span style="color:#00838f;">分组化时延（packetization delay，分组化时延）$t_p$：从分组的第一个比特被发送出去，到最后一个比特被发送出去所经过的时间。</span>

```math
t_p = \frac{p}{r}
```

（按原文复原，$p$ 为分组长度（比特），$r$ 为链路速率（bit/s）。）

- 例 1：一个 64 字节的分组要放上 100 Mb/s 的链路，需要 5.12 ms；
- 例 2：一个 1 kbit 的分组要放上 1 kb/s 的链路，需要 1.024 s。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">跨一条链路发送一个分组的总时间</span>

Total time to send a packet across a link（跨一条链路发送一个分组的总时间）：<span style="color:#00838f;">从第一个比特被发出，到最后一个比特到达所经历的时间。</span>

```math
t = t_l + t_p = \frac{l}{c} + \frac{p}{r}
```

例：一个 100 bit 的分组以 10 Mb/s 越过 1 km 链路，共需 $10 + 5 = 15$ ms。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">路由器队列的简单模型</span>

Simple model of a router queue（路由器队列的简单模型）——图示页：分组以速率 $r$ 进入路由器的队列，队列以出口速率向外发送。下一页把它画成"累积曲线"。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">队列的累积曲线模型</span>

Simple model of a queue（队列的简单模型）。横轴为时间，纵轴为累计字节数（cumulative number of bytes），三条曲线：

- $A(t)$：累计到达（cumulative arrivals），单调不减；
- $D(t)$：累计离开（cumulative departures），单调不减；
- <span style="color:#00838f;">队列占用（queue occupancy）：$Q(t) = A(t) - D(t)$，即同一时刻"已到但未走"的字节数。</span>

<span style="color:#00838f;">排队时延（queueing delay）$d(t)$：在先到先服务（FCFS，first-come-first-served）的假设下，时刻 $t$ 到达的字节在队列里度过的时间——几何上就是 $A(t)$ 与 $D(t)$ 曲线之间的水平距离。</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">例题一：存储转发</span>

Example (store & forward)（例题：存储转发）。纵轴为累计比特数：

> 每秒到达一个 500 bit 的分组，分组以 10,000 b/s 的速率进入队列（即用 0.05 s 收完整个分组）。出口最大发送速率为 1,000 b/s。队列的平均占用是多少？

图中曲线：$A(t)$ 每 1 s 阶跃上涨一段，$D(t)$ 从 0.05 s 起以斜坡爬升，$Q(t)$ 是两者的差，在 0.05 s 达到峰值 500 bit，在 0.55 s 清零，直到 1.0 s 下一分组到来。

> <span style="color:#c0392b;">**解答：**</span> 在每个重复的 1 s 周期里，队列先以 10,000 b/s 被填充 0.05 s（存储转发：整个分组收完才开始转发，故 $Q$ 在 0.05 s 涨到 500 bit），然后以 1,000 b/s 用 0.5 s 排空。前 0.55 s 内 $Q(t)$ 是一个底 0.55 s、高 500 bit 的三角形，平均占用即 $\tfrac{1}{2}\times500=250$ bit；每个周期里队列空 0.45 s，所以整周期平均占用为 $(0.55\times250)+(0.45\times0)=137.5$ bit。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">例题二：直通（cut through）</span>

Example ("cut through")（例题："直通"）。同样的到达参数：每秒一个 500 bit 分组、入口速率 10,000 b/s、出口速率 1,000 b/s。区别：不必等收完整个分组，边收边发。

> <span style="color:#c0392b;">**解答：**</span> 在每个重复的 1 s 周期里，队列在前 0.05 s 内以 10,000 b/s 充入、同时以 1,000 b/s 排出，净增速 9,000 b/s，涨到 $500-50=450$ bit；随后以 1,000 b/s 排空 0.45 s。前 0.5 s 内平均占用为 225 bit（三角形 $\tfrac12\times0.5\times450$ 的均值）；每周期空 0.5 s，故平均占用 $= 0.5\times225+0.5\times0=112.5$ bit。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这两道例题里的"平均占用"是时间平均：$\bar Q=\frac{1}{T}\int_0^T Q(t)\,dt$，几何上就是曲线下面积除以周期——$Q(t)$ 分段线性，积分退化为算三角形面积；这正是概率论里"连续随机变量期望 $E[X]=\int x f(x)\,dx$"的确定性版本，把 $Q(t)$ 看成周期平稳过程，时间平均即期望。更妙的是可以用利特尔法则（Little's law，$L=\lambda W$）交叉验证：长期到达率 $\lambda=500$ bit/s，存储转发下 $L=137.5$ bit，则每比特平均滞留 $W=137.5/500=0.275$ s；直接算也吻合——$s$ 时刻到达的比特要等输出曲线追上输入曲线，等待为 $0.05+9s$，对爆发内均匀到达取平均得 $0.05+9\times0.025=0.275$ s。直通（cut-through）省下的 25 bit 占用，本质是把"攒整包"的串行等待流水线化了——代价是没法在转发前做整包差错检测（CRC 要收完整包才算得出）。存储换检测、直通换时延，这是一对经典工程权衡。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">如何给流一个"加权公平"的链路份额？</span>

How do I give flows a weighted fair share of a link? (instead of strict priority)（如何给各条流一个加权的公平份额，而不是严格优先级？）——引出 WFQ。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">加权公平排队（WFQ）的机制图</span>

Weighted Fair Queueing（加权公平排队）。图示页：两条流（权重 3/4 与 1/4）竞争一条链路，图里把比特逐个编号交错排列。与之前一样（As before），分组按"逐比特方案（bit-by-bit scheme）中它们本会完成（complete）的顺序"发送——即：先想象一个每流轮流发一个比特的理想调度器，再按这个理想方案里各分组的**完成时间**给真实分组排序。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">WFQ 的两条性质</span>

Weighted Fair Queueing (WFQ)（加权公平排队）：

1. 可以证明：WFQ 下一个分组的离开时间，比"逐比特调度"下至多晚 $L_{\max}/R$ 秒——其中 $L_{\max}$ 是最大分组的长度，$R$ 是出链路的数据速率。
2. 在极限意义下，各流恰好获得其加权的公平份额。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这条 $L_{\max}/R$ 的界值得拆开看。理想方案是"逐比特交错"——每条流每次只发一个比特，没有谁的头能挡住谁；真实网络必须按整个分组发送，于是当一个最大分组 $L_{\max}$ 正在上线时，轮到别人也得等它发完，最多堵 $L_{\max}/R$ 秒——这个界就是"最坏的头阻塞时间"，与流的数量无关，所以流越多、公平性越好（每流份额的相对误差越小），第 2 条"极限下公平"由此而来。工程实现上，调度器要按"理想完成时间"给所有分组排序、每次取出最小者——这正是 CLRS 第 6 章优先队列（priority queue，用二叉堆实现，插入与取最小都是 $O(\log n)$）的标准应用场景；而没有理想调度器可对照时，完成时间还要用"虚拟时间"递推出来。对比你在拥塞控制章节见过的 FIFO 尾丢弃：FIFO 只有一条队列，谁突发谁占便宜；WFQ 给每流一条队列，把"公平"从机制上固定下来——这也是后面做时延保证的地基。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">WFQ 的系统视图</span>

Weighted Fair Queueing (WFQ)（加权公平排队·系统图）。图示页：分组从不同的入口端口到达路由器 → 被分类（classify）进各自的流 → 进入调度器（scheduler）。设链路速率为 $R$，共有 $N$ 条流，流 $i$ 的权重为 $\phi_i$（$\sum_{i=1}^{N}\phi_i = 1$；符号按原文残片复原），则<span style="color:#00838f;">流 $i$ 被保证至少获得 $\phi_i R$ 的速率。</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">时延保证：直觉（一）</span>

Delay guarantees: Intuition（时延保证：直觉）。图示页：从 $A$ 到 $B$ 依次经过 4 条链路（长度/速率记为 $l_1,r_1$；$l_2,r_2$；$l_3,r_3$；$l_4,r_4$）与 3 台路由器（排队时延记为 $\tau_1,\tau_2,\tau_3$；符号按原文残片复原）。

端到端时延（end-to-end delay）是各段之和：

```math
D \;=\; \sum_{i=1}^{4}\Big(\frac{l_i}{c}+\frac{p}{r_i}\Big) \;+\; \sum_{i=1}^{3}\tau_i
```

（原文公式符号残缺，按"四段传播 + 四跳传输 + 三台路由器排队"复原。）

下列量是固定的（或在我们控制之下）：$p$、$c$、$l_i$ 与 $r_i$。<span style="color:#00838f;">只要知道 $\tau_1$、$\tau_2$、$\tau_3$ 的上界，就知道端到端时延的上界。</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">时延保证：直觉（二）</span>

Delay guarantees: Intuition（时延保证：直觉·续）。同一拓扑的重复图示，强调单个分组的端到端时延是各跳时延的求和（同上式）；最后一跳 $B$ 处不再有排队项。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">给端到端时延定界</span>

Bounding end-to-end delay（给端到端时延定界）。两步配置：

1. 为这条流分配一个大小为 $b$ 的队列；
2. 给它指定 WFQ 服务速率 $R$（原文记号残缺，此处沿用 $R$）。

于是，长度为 $p$ 的单个分组的端到端时延满足：

```math
D \;\le\; 4\,\frac{l}{c} \;+\; 4\,\frac{p}{R} \;+\; 3\,\frac{b}{R}
```

（原文符号残缺，按"4 段传播 + 每跳传输 $p/R$ + 3 台路由器各最多积压 $b$ 个比特、以 $R$ 排空"复原，与前两页的端到端方程一致。）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">漏桶调节器</span>

The leaky bucket regulator（漏桶调节器）。图示页：一个深度为 $\sigma$ 的"桶"（原文图示记号残缺，按课程标准记号 $(\sigma,\rho)$ 复原），到达的分组先进桶、再以速率 $\rho$ 匀速漏出进入网络。

<span style="color:#00838f;">任意长度为 $t$ 的时间段内能发送的比特数被限制在 $\sigma + \rho t$ 以内</span>——因此它也叫 "(σ, ρ) regulator"（(σ, ρ) 调节器）。图中画出调节器入口 $In(t)$ 与出口 $Out(t)$ 的累积曲线：出口是入口被"削顶填谷"后的平缓版本。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">漏桶：限制"突发性"与一个很酷的定理</span>

Limiting the "burstiness"（限制"突发性"）：

> Cool theorem（很酷的定理）：若到达队列的流是 $(\sigma,\rho)$ 约束的，且队列以速率 $R>\rho$ 服务（缓冲 $b\ge\sigma$），则离开的流同样是 $(\sigma,\rho)$ 约束的——这意味着到达下一台路由器的流也自动 $(\sigma,\rho)$ 约束。

> Cool theorem（续）：若 $R>\rho$，则该流的所有分组穿过第一台路由器的时延 $\le b/R$。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">把所有环节拼起来</span>

Putting it all together（把所有环节拼起来）：主机 $A$ 先用漏桶 $(\sigma,\rho)$ 整形自己的发送；沿途每台路由器为该流分配缓冲 $b$、WFQ 速率 $R$。

> 若 $R>\rho$，则长度为 $p$ 的每个分组的端到端时延满足 $D \le 4\,\frac{l}{c} + 4\,\frac{p}{R} + 3\,\frac{b}{R}$（复原说明同前）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这条定理是"确定性时延保证"故事的核心，关键的后半句两行就能证完：$(\sigma,\rho)$ 约束下，任意窗口 $T$ 内最多流入 $\sigma+\rho T$、排出 $RT$，积压上界为 $\sigma-(R-\rho)T\le\sigma\le b$——队列永不溢出、时延 $\le b/R$。前半句（离开流仍受约束）保证这个性质能**逐跳传递**：只要每一跳都配 $R>\rho$，同一套论证沿路径复制即可，不需要任何概率假设。这套不依赖分布、只封顶"任意窗口到达总量"的方法叫网络演算（network calculus）——用概率论的话说，它放弃均值方差等统计量，直接对随机变量的样本路径取上确界，最保守也最强。它与 Jeff Mogul 讲座里 B4 的 Bandwidth Enforcer 是同一思想的两种落地：B4 在端点测量并整形流量、为每条流分配确定性隧道带宽，从而敢把广域网跑到 100% 利用率——整形器把"随机的到达"改造成"可控的到达"，确定性界限才成立。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">一道综合例题</span>

An Example（一道例题）：

> **问（Q）：** 在下面的网络中，我们想给一个应用流提供 10 Mb/s 的速率，并保证 1,000 字节分组的端到端时延小于 4.7 ms。漏桶调节器应取什么参数？各路由器需要什么服务速率与缓冲区大小？（设传播速度 $c = 2\times10^8$ m/s。）
>
> 拓扑：主机 $A$ —[10 km, 1 Gb/s]→ 路由器 —[100 km, 100 Mb/s]→ 路由器 —[10 km, 1 Gb/s]→ 主机 $B$；主机 $A$ 处装漏桶 $(\sigma,\rho)$ 调节器。

> <span style="color:#c0392b;">**答（A）：**</span> 时延的固定分量为传播加传输：$\dfrac{120\times10^{3}\ \text{m}}{2\times10^{8}\ \text{m/s}} + 8000\ \text{bit}\times\Big(\dfrac{1}{10^{9}}+\dfrac{1}{100\times10^{6}}+\dfrac{1}{10^{9}}\Big)\ \text{s} = 0.6+0.096 \approx 0.7\ \text{ms}$（120 km 为三段链路总长；8000 bit 为分组长度）。
>
> 剩下 $4.7-0.7=4$ ms 留给两台路由器中的排队。把时延均分给两台路由器、每台 2 ms，意味着每台的队列不必大于 $2\ \text{ms}\times10\ \text{Mb/s}=20{,}000$ bit（即 2500 字节）。因此主机 $A$ 的漏桶调节器应取 $\rho=10$ Mb/s、$\sigma\le20{,}000$ bit；每台路由器的 WFQ 应设为 $R\ge10$ Mb/s，且该流的队列容量至少 2500 字节。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">复习主题二：路由、生成树与 Dijkstra</span>

Nick's Review topics（复习主题二）：Routing, spanning trees and Dijkstra（路由、生成树与 Dijkstra）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">生成树</span>

Spanning Tree（生成树）。图示页：路由器 R1—R8 与叶节点 A、B、C、D 组成一张带环的图，其中加粗的一棵"Spanning tree X"（生成树 X）覆盖全图。

- <span style="color:#00838f;">Spanning（生成/覆盖）：它到达所有的叶子；</span>
- <span style="color:#00838f;">Tree（树）：它没有环。</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Dijkstra 最短路优先算法</span>

Dijkstra's shortest path first algorithm（Dijkstra 最短路优先算法）——"链路状态算法"（Link State Algorithm，链路状态算法）的一个实例：

1. 交换链路状态（exchange link state）：每台路由器把与自己相连的链路状态**泛洪（flood）**给所有其他路由器——周期性地发，以及每当链路状态变化时（whenever link state changes）。
2. 运行 Dijkstra：每台路由器**独立地**运行 Dijkstra 最短路优先算法。

于是每台路由器都得到一棵到所有其他路由器的最小代价生成树（min-cost spanning tree）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">例图（标注版）×2</span>

Example Annotated Graph（示例标注图）。图示页：叶节点 A、B、C、D 挂在路由器 R1—R8 上，每条链路旁标注代价（残留数字有 1、2、4、5、3 等）。同一张图连续出现两页——第二页把其中一条边的标注由 3 改为 2（对标注的修正），并给出结论：最小代价生成树（Minimum cost spanning tree）——本例中很简单（In this case, simple）。

> <span style="color:#7f8c8d;">说明：pdftotext 把图中的连线关系全部丢失，仅剩零散的边权数字，无法一一复原对应关系；算法过程以下一页的"距离标签 + 加入顺序"为准。</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">例：以 R8 为根的生成树</span>

An example: spanning tree rooted on R8（例：以 R8 为根的生成树）。图示页：同一张图，画出以 R8 为根的最短路径树；下一页开始演示时，树上节点旁将依次标出距离标签 0—7（R8 为 0，最远的节点为 7）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">算法演示（表格式推进）</span>

The algorithm（算法）。两页 slide 以同一张表推进：表格三列——"Shortest Path Set"（已确定最短路的节点集合）、"Candidate Set"（候选集合）、"Add"（本轮加入者）。以 R8 为根，按可辨识的残片，推进顺序为：

| 轮次 | 加入节点 | 距离标签 |
|---|---|---|
| 0 | R8（根） | 0 |
| 1 | R7 | 1 |
| 2 | R6 | 2 |
| 3 | R4 | 3 |
| 4 | R3 | 4 |
| 5 | R2 | 5 |
| 6 | R1 | 6 |
| 7 | R5 | 7 → 候选集合为空（Empty），Done（完成） |

每轮的机制：从候选集合中挑**路径代价最小**的节点加入最短路径集合；再把刚加入节点的邻居作为新的候选路径放入候选集合（原文残片中候选集合始终含 R3、R5 等路径项，直到最后清空）。第一页演示到加入 R7 为止，第二页演示其余各轮直至完成。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 用你已有的算法语言给这套流程定个位：这就是 CLRS 第 24 章的 Dijkstra——维护"已确定集合 S"与"候选优先队列 Q"，每轮从 Q 中提取距离最小者加入 S，并对它的邻居做松弛（relaxation，松弛：若经新点到达邻居更近则更新距离）。用二叉堆实现复杂度是 $O\big((V+E)\log V\big)$，用数组扫描则是 $O(V^2)$。Dijkstra 的正确性前提是**边权非负**，链路代价天然满足；若有负权边就得换 Bellman-Ford（$O(VE)$，还能报告负环）。链路状态路由的妙处在于"复制算法而非分布式化算法"：泛洪让每台路由器拿到同一张图，然后各跑一份自己为根的 Dijkstra——对比距离向量（各节点只与邻居交换估计、本质是分布式 Bellman-Ford），它把"全局一致视图"这个不变量做实了，收敛快、环路少。生成树本身回到 6.042J 的定义：$n$ 个顶点、$n-1$ 条边、连通无环；最短路径树是它的特例——每条树边都落在"根到某点"的最短路上。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">复习到此结束</span>

两大部分（分组交换、路由）复习完毕——祝考试顺利。
