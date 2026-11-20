# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Packet Switching: Queue models（分组交换：队列模型）

> <span style="color:#7f8c8d;">Source: 慕课/Unit3-Packet-Switching/ps_delay_model.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

> <span style="color:#7f8c8d;">提取说明：源 PDF 由 pdftotext 提取后存在 "ti"→"8" 的系统性乱码（如 determinis8c = deterministic、Sta8s8cal = statistical、LiPle/Li>le = Little、oEen = often），以下译文已按原意复原。</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1：标题页</span>

CS144: An Introduction to Computer Networks（CS144：计算机网络导论）
Packet Switching（分组交换）
Queue models（队列模型）
主讲人 Nick McKeown（斯坦福大学电气工程与计算机科学教授）
CS144, Stanford University（CS144，斯坦福大学）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 2：大纲</span>

1. simple deterministic queue model（简单确定性队列模型）；
2. small packets reduce end-to-end delay（小分组降低端到端时延）；
3. statistical multiplexing（统计复用）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 3：路由器队列的简单模型</span>

router queue（路由器队列）。图中标注：cumulative arrivals（累积到达）$A(t)$、queue occupancy（队列占用）$Q(t)$、cumulative departures（累积离开）$D(t)$、link rate（链路速率）$R$。

$A(t)$、$D(t)$ 的性质：两者都是 non-decreasing（非减）的；$A(t) \ge D(t)$。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 4~6：队列模型的图示展开</span>

这三页为同一模型逐步展开的图示 slide（幻灯片），其中第 3 页内容带 HIDE/显示切换动画：<span style="color:#00838f;">$A(t)$ 是截至时间 $t$ 累积到达的字节数；$D(t)$ 是截至时间 $t$ 累积离开的字节数</span>；$Q(t)$ 是 queue occupancy（队列占用）；$R$ 为 link rate（链路速率）。图轴为"累积字节数—时间"平面上的两条曲线，$D(t)$ 以斜率不超过 $R$ 增长（基于图示推断）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 7：占用与排队时延的正式定义</span>

<span style="color:#00838f;">queue occupancy（队列占用）：$Q(t) = A(t) - D(t)$。</span>

<span style="color:#00838f;">queueing delay（排队时延）$d(t)$：在队列按 FCFS（first-come first-served，先到先服务）服务的前提下，时刻 $t$ 到达的一个字节在队列中度过的时间</span>——图上即 $A(t)$ 与 $D(t)$ 之间的水平距离。

🎥 *CS144 教授在视频中解释: "在任何一个时间点，可能有一些字节到达，但还没有离开——它们是坐在桶这里的字节。桶的占用率将是 Q(t)"*（字幕来源：043 - p42 3-5 Packet Switching - Principles Simple Deterministic Queue Model (revised).zh.srt）

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这一页把 3-5a（本讲例题的精讲视频）用过的图解法正式化，两条"读图规则"请务必内化：**竖直距离 = queue occupancy（队列占用）$Q(t)$，水平距离 = queueing delay（排队时延）$d(t)$**。"水桶"类比帮助记忆：桶里现有的水是 $Q(t)$，一滴水从进桶到出桶的等待是 $d(t)$。这条几何对偶关系是 Little's law（利特尔法则）的图形版——"队列里平均有多少水"（竖直量）与"每滴水平均等多久"（水平量）由同一个三角形的形状锁定，只差一个"流量"（斜率）作换算因子。你在 ps_queue_props（下一讲，讲随机到达下的队列性质）会看到正式表述 $L=\lambda W$（其中 $W$ 即平均排队时延）；而在更远处，CDN（Content Delivery Network，内容分发网络）队列、工业产线、医院候诊都服从同一关系，因为它对到达过程几乎不设假设（只需 $A$、$D$ 非减且 $A\ge D$，即守恒）。这就是"好抽象"的标志：假设极少、适用极广。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 8：图示</span>

本页为图示 slide（幻灯片，文字极少），与下一页例题共用同一张 $A(t)/D(t)$ 图。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 9：例题</span>

<span style="color:#c0392b;">**例题：**</span>每一秒，一个 100 比特的 packet（分组）以 1000 b/s 的速率到达一个 queue（队列）。最大离开速率为 500 b/s。queue（队列）的平均占用是多少？

> <span style="color:#c0392b;">**解答：**</span>在每个重复的 1 s 周期里，queue（队列）先以 500 b/s 的净速率填充 0.1 s（100 比特到齐），再以 500 b/s 的速率排空 0.1 s。因此在最初的 0.2 s 内，平均 queue occupancy（队列占用）为 $0.5\times(0.1\times 500)=25$ 比特。每个周期里 queue（队列）有 0.8 s 为空，所以平均 queue occupancy（队列占用）为：
>
> ```math
> Q(t) \;=\; (0.2\times 25) + (0.8\times 0) \;=\; 5\ \text{bit}
> ```
>
> （即时间平均 $\bar Q = 5$ 比特。）

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 注意解答里那条容易被忽略的假设："queue（队列）以 500 b/s 的**净**速率填充"之所以成立，是因为题目说 queue（队列）逐比特被服务——比特一到就被以 500 b/s 抽走，净积压速率才是 $1000-500=500$ b/s；若必须整包收齐才开始服务（即 store-and-forward（存储转发）），前 0.1 s 收包期间 queue（队列）没有流出、以 1000 b/s 涨到 100 比特，随后用 0.2 s 排空，三角形就变成底 0.3 s、高 100 比特的形状，时间平均占用是 $0.5\times 0.3\times 100 = 15$ 比特，是逐比特服务情形（5 比特）的三倍。这再次印证 3-3 讲的 store-and-forward（存储转发）代价。另一个值得练熟的动作是用 Little's law（利特尔法则）验算：长期平均到达率 $\lambda=100\ \text{bit/s}$，由 3-5a (b) 知平均时延 $W=0.05\ \text{s}$（可独立验证：一个在时刻 $a$ 到达的比特，其等待时间恰为 $a$ 秒——它到达时 queue（队列）里已积压 $500a$ 比特，按 500 b/s 排空需 $a$ 秒，而 $a$ 在 $0$ 到 $0.1$ 秒上均匀取值、均值即 $0.05$ 秒），则 $L=\lambda W=5\ \text{bit}$，与图解法一致——一题两解互相印证，比单独记住答案可靠得多。等你学到随机到达（ps_queue_props），这张确定性三角形图仍然成立，只是 $A(t)$ 变成了随机路径，平均要改成对时间（或对到达）取期望。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 10：大纲回顾</span>

1. simple deterministic queue model（简单确定性队列模型）；2. small packets reduce end-to-end delay（小分组降低端到端时延）；3. statistical multiplexing（统计复用）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 11：为什么不把整个报文放进一个分组？</span>

packet switching（分组交换）：为什么不把整条 message（报文）作为一个 packet（分组）发送？考虑 A 经 R1、R2、R3 到 B 的路径（A、B 是两端主机，R1、R2、R3 是途中的 router（路由器））。把整条 $M$ 比特的 message（报文）作为一个 packet（分组）发送时，end-to-end delay（端到端时延）为（其中 $r_i$ 是第 $i$ 条 link（链路）的速率、$l_i$ 是第 $i$ 条 link（链路）的长度、$c$ 是信号传播速度，故 $\frac{M}{r_i}$ 是该跳的 transmission delay（传输时延）、$\frac{l_i}{c}$ 是该跳的 propagation delay（传播时延））：

```math
t \;=\; \sum_i\left(\frac{M}{r_i}+\frac{l_i}{c}\right)
```

把 message（报文）切成 $p$ 比特的小 packet（分组）后：

```math
t \;=\; \sum_i\left(\frac{p}{r_i}+\frac{l_i}{c}\right)\;+\;\left(\frac{M}{p}-1\right)\frac{p}{r_{\min}}
```

把 message（报文）拆成 packet（分组），允许在所有 link（链路）上并行传输（pipeline（流水线）），从而降低 end-to-end delay（端到端时延）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这两个公式是本单元最值得亲手推一遍的结论，关键在第二式多出的那项 $\left(\frac{M}{p}-1\right)\frac{p}{r_{\min}}$：它度量的正是"最慢一跳上的拖尾"——第 1 个小 packet（分组）一路领先，但后面还有 $\frac{M}{p}-1$ 个 packet（分组）要在瓶颈 link（链路，速率 $r_{\min}$）上逐个排队发出，这笔 pipeline（流水线）的"排空时间"省不掉；而收益在于每个小 packet（分组）的传输时延从 $\frac{M}{r_i}$ 缩到 $\frac{p}{r_i}$，且各 link（链路）能同时忙起来（对照 3-3 的 store-and-forward（存储转发）：不等整包收齐）。这正是 CSAPP（《深入理解计算机系统》，Computer Systems: A Programmer's Perspective）第 4 章 pipeline（流水线）思想在网络上的重演：拆小处理单元 → 提高并行度，但受限于最慢一级。那么 $p$ 越小越好吗？不是——每个 packet（分组）都有 header overhead（首部开销）$H$，有效带宽是 $\frac{p}{p+H}$，$p$ 太小时 header（首部）把信道吃光；以太网默认 MTU（Maximum Transmission Unit，最大传输单元）1500 字节就是这组权衡的历史折中。你看，一个看似纯理论的公式，两端各拴着一条工程约束。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 12：大纲回顾</span>

1. simple deterministic queue model（简单确定性队列模型）；2. small packets reduce end-to-end delay（小分组降低端到端时延）；3. statistical multiplexing（统计复用）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 13：统计复用的基本思想</span>

statistical multiplexing（统计复用）：基本思想——看速率随时间的曲线。一条 flow（流）的速率曲线波动剧烈（图示：一条 flow（流）的速率随时间起伏）；两条 flow（流）叠加后波动趋于平缓；很多 flow（流）叠加后，aggregate rate（聚合速率）围绕其平均值小幅波动（图示：1、2、10、100、1000 条 flow（流）的平均速率，基于图示推断）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 14~15：分组缓冲吸收突发</span>

statistical multiplexing（统计复用，属于 packet switching（分组交换））：$N$ 个输入各自以速率 $R$ 向同一输出发送 packet（分组）（1 号、2 号……N 号，各带 data（数据）和 hdr（header，首部）两部分），进入 packet buffer（分组缓冲区），输出 link（链路）速率为 $R$；queue（队列）长度记为 $X(t)$（或 $Q(t)$）。

- 因为 packet buffer（分组缓冲区）吸收了临时的 burst（突发），输出 link（链路）不需要运行在 $N\cdot R$ 的速率上；
- 但 packet buffer（分组缓冲区）大小有限（为 $B$），所以会发生 loss（丢包）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "输出链路不必跑在 $N\cdot R$"是 packet switching（分组交换）对 circuit switching（电路交换）的经济学胜利，它成立的统计学根基你恰好正在学：**law of large numbers（大数定律）与 central limit theorem（中心极限定理）**。若 $N$ 条 flow（流）各自独立、均值 $\mu$、方差 $\sigma^2$，aggregate rate（聚合速率）的均值是 $N\mu$，但**相对**波动（标准差/均值）按 $1/\sqrt{N}$ 衰减——flow（流）越多，聚合流量越像一条平稳的河，峰值超出均值的幅度增长远慢于 $N$ 本身。于是按"略高于 $N\mu$"配置 link（链路）、再给个 buffer（缓冲区）$B$ 吸收残差波动，就能承载"按峰值预留"根本不敢想的用户数。这正是 3-1（本单元开头讲网络历史的那一讲）里"packet switching（分组交换）高效使用带宽"的数学版本，也是电话网工程师早已懂得的常识（他们叫它"话务量（traffic）统计复用"，A. K. Erlang（丹麦数学家、排队论奠基人）在 1909 年就算过）。代价写在第二行：buffer（缓冲区）有限就有 loss（丢包），loss（丢包）又反过来成为端到端 congestion control（拥塞控制）的输入信号——因果链在此接上了下一单元。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 16~17：复用增益</span>

statistical multiplexing（统计复用）：速率 A 与速率 B 的两条 flow（流）各自峰值 $C$，聚合后峰值小于 $2C$（图示：Rate A、Rate B 的波形与聚合 Rate A+B，其中 $R<2C$）。statistical multiplexing gain（统计复用增益）定义为：

```math
\text{gain} \;=\; \frac{2C}{R}
```

（两条峰值各为 $C$ 的 flow（流），聚合后只需一条速率 $R<2C$ 的 link（链路）承载，增益即"按峰值预留"与"按聚合预留"所需容量之比。）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 18：总结</span>

- 常常，我们可以用一个简单的 deterministic queue model（确定性队列模型）来理解网络中 packet（分组）的动态。
- 我们把 message（报文）拆成 packet（分组），因为它让我们能 pipeline（流水线化）传输、降低 end-to-end delay（端到端时延）。
- statistical multiplexing（统计复用）让我们在一条 link（链路）上高效承载许多 flow（流）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 19：结束页</span>

\<end\>（本讲结束）

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 3-5《Simple Deterministic Queue Model》的配套 slides（幻灯片）；例题的精讲版见 3-5a（本讲例题的精讲视频），随机到达情形见 ps_queue_props（下一讲，讲随机到达下的队列性质）。</span>
