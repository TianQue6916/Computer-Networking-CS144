# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Packet Switching: Queue models（分组交换：队列模型）

> <span style="color:#7f8c8d;">Source: 慕课/Unit3-Packet-Switching/ps_delay_model.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

> <span style="color:#7f8c8d;">提取说明：源 PDF 由 pdftotext 提取后存在 "ti"→"8" 的系统性乱码（如 determinis8c = deterministic、Sta8s8cal = statistical、LiPle/Li>le = Little、oEen = often），以下译文已按原意复原。</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1：标题页</span>

CS144: An Introduction to Computer Networks
Packet Switching（分组交换）
Queue models（队列模型）
Nick McKeown，斯坦福大学电气工程与计算机科学教授
CS144, Stanford University

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 2：大纲</span>

1. 简单确定性队列模型（simple deterministic queue model）；
2. 小分组降低端到端时延（small packets reduce end to end delay）；
3. 统计复用（statistical multiplexing）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 3：路由器队列的简单模型</span>

路由器队列（router queue）。图中标注：累积到达 $A(t)$、队列占用 $Q(t)$、累积离开 $D(t)$、链路速率 $R$。

$A(t)$、$D(t)$ 的性质：$A(t)$、$D(t)$ 非减（non-decreasing）；$A(t) \ge D(t)$。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 4~6：队列模型的图示展开</span>

这三页为同一模型逐步展开的图示 slide（其中第 3 页内容带 HIDE/显示切换动画）：<span style="color:#00838f;">$A(t)$ 是截至时间 $t$ 累积到达的字节数；$D(t)$ 是截至时间 $t$ 累积离开的字节数</span>；$Q(t)$ 是队列占用；$R$ 为链路速率。图轴为"累积字节数—时间"平面上的两条曲线，$D(t)$ 以斜率不超过 $R$ 增长（基于图示推断）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 7：占用与排队时延的正式定义</span>

<span style="color:#00838f;">队列占用：$Q(t) = A(t) - D(t)$。</span>

<span style="color:#00838f;">排队时延 $d(t)$：在队列按先到先服务（FCFS）服务的前提下，时刻 $t$ 到达的一个字节在队列中度过的时间</span>——图上即 $A(t)$ 与 $D(t)$ 之间的水平距离。

🎥 *CS144 教授在视频中解释: "在任何一个时间点，可能有一些字节到达，但还没有离开——它们是坐在桶这里的字节。桶的占用率将是 Q(t)"*（字幕来源：043 - p42 3-5 Packet Switching - Principles Simple Deterministic Queue Model (revised).zh.srt）

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这一页把 3-5a 用过的图解法正式化，两条"读图规则"请务必内化：**竖直距离 = 队列占用 $Q(t)$，水平距离 = 排队时延 $d(t)$**。"水桶"类比帮助记忆：桶里现有的水是 $Q(t)$，一滴水从进桶到出桶的等待是 $d(t)$。这条几何对偶关系是 Little's law 的图形版——"队列里平均有多少水"（竖直量）与"每滴水平均等多久"（水平量）由同一个三角形的形状锁定，只差一个"流量"（斜率）作换算因子。你在 ps_queue_props 会看到正式表述 $L=\lambda d$；而在更远处，CDN 队列、工业产线、医院候诊都服从同一关系，因为它对到达过程几乎不设假设（只需 $A$、$D$ 非减且 $A\ge D$，即守恒）。这就是"好抽象"的标志：假设极少、适用极广。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 8：图示</span>

本页为图示 slide（文字极少），与下一页例题共用同一张 $A(t)/D(t)$ 图。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 9：例题</span>

<span style="color:#c0392b;">**例题：**</span>每一秒，一个 100 比特的分组以 1000 b/s 的速率到达一个队列。最大离开速率为 500 b/s。队列的平均占用是多少？

> <span style="color:#c0392b;">**解答：**</span>在每个重复的 1 s 周期里，队列先以 500 b/s 的净速率填充 0.1 s（100 bit 到完），再以 500 b/s 的速率排空 0.1 s。因此在最初的 0.2 s 内，平均队列占用为 $0.5\times(0.1\times 500)=25$ bit。每个周期里队列有 0.8 s 为空，所以平均队列占用为：
>
> ```math
> Q(t) \;=\; (0.2\times 25) + (0.8\times 0) \;=\; 5\ \text{bit}
> ```
>
> （即时间平均 $\bar Q = 5$ bit。）

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 注意解答里那条容易被忽略的假设："队列以 500 b/s 的**净**速率填充"之所以成立，是因为题目说队列逐比特被服务——比特一到就被以 500 b/s 抽走，净积压速率才是 $1000-500=500$ b/s；若必须整包收齐才开始服务，三角形会变成底 0.2 s、高 100 bit 的形状，平均占用翻倍。这再次印证 3-3 说的存储转发代价。另一个值得练熟的动作是用 Little's law 验算：长期平均到达率 $\lambda=100\ \text{bit/s}$，由 3-5a (b) 知平均时延 $W=0.05\ \text{s}$，则 $L=\lambda W=5\ \text{bit}$，与图解法一致——一题两解互相印证，比单独记住答案可靠得多。等你学到随机到达（ps_queue_props），这张确定性三角形图仍然成立，只是 $A(t)$ 变成了随机路径，平均要改成对时间（或对到达）取期望。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 10：大纲回顾</span>

1. 简单确定性队列模型；2. 小分组降低端到端时延；3. 统计复用。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 11：为什么不把整个报文放进一个分组？</span>

分组交换：为什么不把整条报文（message）作为一个分组发送？考虑 A 经 R1、R2、R3 到 B 的路径。把整条 $M$ 比特报文作为一个分组发送时，端到端时延为：

```math
t \;=\; \sum_i\left(\frac{M}{r_i}+\frac{l_i}{c}\right)
```

把报文切成 $p$ 比特的小分组后：

```math
t \;=\; \sum_i\left(\frac{p}{r_i}+\frac{l_i}{c}\right)\;+\;\left(\frac{M}{p}-1\right)\frac{p}{r_{\min}}
```

把报文拆成分组，允许在所有链路上并行传输（流水线），从而降低端到端时延。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这两个公式是本单元最值得亲手推一遍的结论，关键在第二式多出的那项 $\left(\frac{M}{p}-1\right)\frac{p}{r_{\min}}$：它度量的正是"最慢一跳上的拖尾"——第 1 个小分组一路领先，但后面还有 $\frac{M}{p}-1$ 个分组要在瓶颈链路（速率 $r_{\min}$）上逐个排队发出，这笔流水线的"排空时间"省不掉；而收益在于每个小分组的传输时延从 $\frac{M}{r_i}$ 缩到 $\frac{p}{r_i}$，且各链路能同时忙起来（对照 3-3 的存储转发：不等整包收齐）。这正是 CSAPP 第 4 章流水线思想在网络上的重演：拆小处理单元 → 提高并行度，但受限于最慢一级。那么 $p$ 越小越好吗？不是——每个分组都有首部开销 $H$，有效带宽是 $\frac{p}{p+H}$，$p$ 太小时首部把信道吃光；以太网默认 MTU 1500 字节就是这组权衡的历史折中。你看，一个看似纯理论的公式，两端各拴着一条工程约束。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 12：大纲回顾</span>

1. 简单确定性队列模型；2. 小分组降低端到端时延；3. 统计复用。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 13：统计复用的基本思想</span>

统计复用（Statistical Multiplexing）：基本思想——看速率随时间的曲线。一条流的速率曲线波动剧烈（图示：一条流的速率随时间起伏）；两条流叠加后波动趋于平缓；很多流叠加后，聚合速率围绕其平均值小幅波动（图示：1、2、10、100、1000 条流的平均速率，基于图示推断）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 14~15：分组缓冲吸收突发</span>

统计复用（分组交换）：$N$ 个输入各自以速率 $R$ 向同一输出发送分组（1 号、2 号……N 号，各带 Data/Hdr 首部），进入分组缓冲区（packet buffer），输出链路速率为 $R$；队列长度记为 $X(t)$（或 $Q(t)$）。

- 因为缓冲区吸收了临时的突发（burst），输出链路不需要运行在 $N\cdot R$ 的速率上；
- 但缓冲区大小有限（为 $B$），所以会发生丢包（losses）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "输出链路不必跑在 $N\cdot R$"是分组交换对电路交换的经济学胜利，它成立的统计学根基你恰好正在学：**大数定律与中心极限定理**。若 $N$ 条流各自独立、均值 $\mu$、方差 $\sigma^2$，聚合速率的均值是 $N\mu$，但**相对**波动（标准差/均值）按 $1/\sqrt{N}$ 衰减——流越多，聚合流量越像一条平稳的河，峰值超出均值的幅度增长远慢于 $N$ 本身。于是按"略高于 $N\mu$"配置链路、再给个缓冲 $B$ 吸收残差波动，就能承载"按峰值预留"根本不敢想的用户数。这正是 3-1 历史里"分组交换高效使用带宽"的数学版本，也是电话网工程师早已懂得的常识（他们叫它"话务量统计复用"，Erlang 在 1909 年就算过）。代价写在第二行：缓冲有限就有丢包，丢包又反过来成为端到端拥塞控制的输入信号——因果链在此接上了下一单元。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 16~17：复用增益</span>

统计复用：速率 A 与速率 B 的两条流各自峰值 $C$、聚合后为 $C$（图示：Rate A、Rate B 的波形与聚合 Rate A+B，其中 $R<2C$）。统计复用增益（statistical multiplexing gain）定义为：

```math
\text{gain} \;=\; \frac{2C}{R}
```

（两条峰值各为 $C$ 的流，聚合后只需一条速率 $R<2C$ 的链路承载，增益即"按峰值预留"与"按聚合预留"所需容量之比。）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 18：总结</span>

- 常常，我们可以用一个简单的确定性队列模型来理解网络中分组的动态。
- 我们把报文拆成分组，因为它让我们能流水线化（pipeline）传输、降低端到端时延。
- 统计复用让我们在一条链路上高效承载许多流。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 19：结束页</span>

\<end\>（本讲结束）

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 3-5《Simple Deterministic Queue Model》的配套 slides；例题的精讲版见 3-5a，随机到达情形见 ps_queue_props。</span>
