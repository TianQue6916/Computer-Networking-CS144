# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## Physical Links: CSMA/CD and Ethernet（物理链路：CSMA/CD 与以太网）

> <span style="color:#7f8c8d;">Source: src/慕课/Unit7-Lower-Layers/link_-_csmacd.txt · CS144 Fall 2019 · 行内双语翻译版</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 1 · 标题页</span>

Physical Links（物理链路）——CSMA/CD and Ethernet（CSMA/CD 与以太网）。主讲人：Nick McKeown，Stanford University 电气工程与计算机科学教授。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 2 · The Link Layer（链路层的位置）</span>

图示：主机 A 与主机 B 各自拥有 Application（应用层）/Transport（传输层）/Network（网络层）/Link（链路层）四层协议栈，中间经过两台 "Network–Link" 设备（路由器）相连。

要点：链路层只负责"一段"——把分组从一个节点送到相邻的下一个节点；完整的端到端路径由多段链路串接而成，每一段可以用完全不同的链路技术。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 3 · Why is Ethernet often referred to as "Layer 2"?（为什么以太网常被叫"第二层"？）</span>

图示对照两套模型，以 http/ASCII 应用为例：

- **4-layer Internet model（四层互联网模型）**：Application（应用层）→ Transport：TCP → Network：IP → Link：Ethernet；
- **7-layer OSI Model（七层 OSI 模型）**：Application / Presentation / Session / Transport / Network / Link / Physical（应用/表示/会话/传输/网络/数据链路/物理）。

答案：以太网帧承载的是四层模型中的链路层功能，而工业界沿用了 OSI 的编号习惯——链路层在 OSI 里排第 2 层，所以以太网交换机常被称为"二层设备"。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 4 · The origins of Ethernet（以太网的起源）</span>

过渡标题页：以太网的起源（图示内容在《Ethernet》一讲中详细展开）。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 5 · Sharing a "medium"（共享"介质"）</span>

- 以太网是多台主机共享一条公共电缆（medium，介质）的典型例子；
- 要共享介质，就必须决定**谁**在**何时**可以发送；
- 这类规则有一个通称：Medium Access Control（介质访问控制）协议，简称 MAC 协议；
- 下面看几个例子。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 6 · Examples of MAC Protocols（MAC 协议谱系）</span>

图示一条光谱，从 Simple·Random（简单·随机）到 Complex·Deterministic（复杂·确定性）：

- **Aloha**（分组交换无线网络，Packet-Switched Radio Network）——最随机的极端；
- **CSMA/CD**（Carrier Sense Multiple Access / Collision Detection，载波侦听多路访问/冲突检测）——Ethernet（以太网），IEEE 802.3；
- **Token Passing（令牌传递）**——Token Ring（令牌环），IEEE 802.5——最确定性的极端。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 7 · Goals of MAC Protocols（MAC 协议的设计目标）</span>

Medium Access Control 协议在一群用户之间仲裁（arbitrate）对公共共享信道的访问：

1. 共享信道的高利用率（high utilization）——信道别闲着；
2. 对各 end host（终端主机）公平（fair）；
3. 实现简单、成本低；
4. 对错误健壮、容错（fault tolerant）。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 8 · Aloha Network (1968)（Aloha 网络，1968）</span>

图示夏威夷群岛地图：Kauai（可爱岛）、Oahu（欧胡岛）、Molokai（莫洛凯岛）、Maui（茂宜岛）、Hawaii（夏威夷大岛）之间用无线信道互联。Aloha 是随机接入协议的鼻祖，诞生于夏威夷大学的科研项目。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 9 · Aloha（Aloha 协议规则）</span>

图示：两个频率——Frequency 0 与 Frequency 1（上行与下行各用一个频段）。

Original Aloha MAC protocol（原始 Aloha 协议）只有两条规则：

1. 有数据就发；
2. 如果这次发送与其他站点"冲突"（collide），稍后重试。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 10 · Aloha Protocol（Aloha 的评价与改进）</span>

- Aloha 协议非常简单；
- （相当）能容忍单台主机故障——协议逻辑分布在各主机上，没有中心节点；
- 低负载（low-load）下可以期待时延很小；
- 高负载（high-load）下，大量时间被"浪费"在互相冲突的分组上。

改进性能的三个方向：

1. 发送前先监听信道有无活动（carrier sense，载波侦听）；
2. 尽快检测到冲突并停止发送；
3. 冲突后根据负载随机选择等待时间。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> Aloha 是"用概率换协议简洁性"的开山之作：Norman Abramson 于 1968 年在夏威夷大学启动该项目，1971 年 ALOHAnet 建成运行。它的吞吐上界可以用你学过的泊松过程一步算出：把各站发起发送建模为速率 $G$ 的泊松流，纯 Aloha 的冲突窗口是"本包开始前后各一个包长"共约 $2T$，期间无其他到达的概率为 $P(N(0)=0)=e^{-2G}$，故吞吐 $S=G\cdot e^{-2G}$，在 $G=1/2$ 处取最大值 $1/(2e)\approx18.4\%$；时隙化（slotted Aloha）把窗口砍半，得 $S=G\cdot e^{-G}$，上界 $1/e\approx36.8\%$。两个数字说明：完全放开的随机接入有一半以上信道时间注定被冲突烧掉，这正是后面 CSMA/CD 每一条改进（先听再说、边发边听、随机退避）要往回抢的时间。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 11 · CSMA/CD Protocol（CSMA/CD 协议）</span>

所有主机在同一信道上收发；分组长度可变（variable size）。当一台主机有包要发：

1. **Carrier Sense（载波侦听）**：发送前先确认线路安静；
2. **Collision Detection（冲突检测）**：尽快检测冲突；一旦检测到冲突，立即停止发送，等待一个随机时间后回到第 1 步——等待时间按 binary exponential backoff（二进制指数退避）逐次增长。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 二进制指数退避是分布式系统里"随机化打破对称"的教科书案例：第 $i$ 次冲突后，站点从 $\{0,1,\dots,2^{i}-1\}$ 中等概率抽一个数，等待相应个时隙再重试。冲突越多、窗口越大，期望等待指数增长；一旦成功，窗口立即重置——负载轻时几乎不等待，负载重时自动退让，这就是"自适应"。用概率论的话说，均匀随机化让两台主机的重发时刻几乎不可能再次重合，比任何确定性规则都更能化解同时重发的僵局（这正是 Bertsekas 随机过程里"随机化消除对称性"思想的应用）。顺带预告：WiFi 用的是 CSMA/CA（冲突避免）——无线收发器在同一频率上无法边发边听，"检测冲突"不可行，只能改成尽量避免，这是无线一讲的主题。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 12 · CSMA/CD operation（CSMA/CD 的运行图示）</span>

图示：A、B、C、D 四台主机挂在同一条总线上。演示两个站点几乎同时开始发送，信号在总线上叠加成垃圾数据，所有站点都看到冲突；先停止的站点随后按退避规则等待重试。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 13 · CSMA/CD Packet size requirement（最小分组长度·图示）</span>

图示：A、B、C、D 总线，标注两个关键量——总线的传播时延 $L/c$（信号走完长度为 $L$ 的总线需要 $L/c$ 秒，$c$ 是信号在介质中的传播速度）与发送速率 $R$。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 14 · CSMA/CD Packet size requirement（最小分组长度·推导）</span>

图示：A 在最左端开始发送，信号向最右端的 D 传播；若 D 恰好在信号到达前的瞬间也开始发送，冲突信号还要再花 $L/c$ 才能传回 A。

<span style="color:#00838f;">要让一台终端主机在自己发完一个包**之前**就能检测到冲突，必须要求发送时间不短于往返传播时延：</span>

```math
P/R \ \ge\ 2L/c \quad\Longleftrightarrow\quad P \ \ge\ \frac{2LR}{c}
```

其中 $P$ 为分组大小，$R$ 为发送速率。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 15 · CSMA/CD Packet size requirement（最小分组长度·算例）</span>

例：$R = 10\,\mathrm{Mb/s}$，$L = 10{,}000\,\mathrm{m}$，$c = 2\times10^{8}\,\mathrm{m/s}$。

```math
P_{\min} = \frac{2LR}{c} = \frac{2\times10^{11}}{2\times10^{8}} = 1{,}000\ \text{bits}
```

即分组至少要 1000 比特，发送方才能保证在发完之前检测到最坏位置的冲突。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这页的物理直觉是"传输时延与传播时延的赛跑"：冲突的坏消息以 $c$ 的速度传播，你只有还在发送，才能听到关于自己这次发送的坏消息。把结果换个单位看：1000 比特在 10 Mb/s 上正好是 100 微秒，而往返传播 2L/c 是 100 微秒——两者必须咬合。这正是本课程 Unit 2 讲过的时延带宽积（delay × bandwidth product）思想：管道越长越快，"正在路上"的比特就越多，协议要为之付出的最小帧就越大。实际以太网标准取最小帧 64 字节（512 比特）、时隙时间 51.2 微秒，为最大网络直径（约 2.5 km，含中继器）和检测裕量留足预算。再往后看：速率提高 10 倍、100 倍而最小帧不变，就只能把 $L$ 压到 100 米——这直接引出下一讲的百兆/千兆标准与以太网交换。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 16 · &lt;end&gt;</span>

原讲义的视频分段结束标记，无正文内容。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 17 · Performance of CSMA/CD（性能分析·引入）</span>

> <span style="color:#7f8c8d;">本节 5 页（Slide 17–21）出自 CS244a（Winter 2008）Handout 10（p.15–19）；原 slides 的公式与曲线以图片嵌入、文本提取丢失，以下推导依据原 slide 的文字定义补全。</span>

我们要分析一个 CSMA/CD 网络的性能。指标是 efficiency（效率）$\eta$：定义为"用于发送有用的、成功的数据"的时间占比——花在制造和检测冲突上的时间越多，协议效率越低。

为使分析可行，做两个简化假设：

- 时间分槽（slotted）：一个时隙（slot）长为 $2\times\text{PROP}$，其中 PROP 为最大端到端传播时延；
- 任一时隙内，每台主机以概率 $p$ 决定发送（包括首次发送与重传）。

分析分两步：先找出使吞吐量最大化的 $p$（实为 goodput，有效吞吐），再用最优 $p$ 求效率。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 18 · Performance of CSMA/CD: Maximizing goodput（最大化有效吞吐）</span>

设共 $N$ 台主机。某个时隙"恰好一台主机发送、其余都不发"的概率为

```math
q \;=\; N\,p\,(1-p)^{N-1}
```

这是一个"一荣俱损"的折中：$p$ 太大则冲突频发，$p$ 太小则信道闲置。对 $q$ 求极值得最优 $p^{*}=1/N$，此时

```math
q_{\max} = \left(1-\tfrac{1}{N}\right)^{N-1} \ \xrightarrow{\ N\to\infty\ }\ e^{-1}
```

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 19 · Performance of CSMA/CD: Finding the overhead（求浪费的开销）</span>

定义 $A$ 为一个包成功发送之前平均被浪费的时隙数。每个时隙以概率 $q$ 成功——这正是 geometric distribution（几何分布）：首次成功平均需要尝试 $1/q$ 个时隙，其中失败

```math
A \;=\; \frac{1}{q} - 1
```

个时隙。

原 slide 给出的硬币类比：一枚正面概率为 0.4 的硬币，掷出首个正面平均需要 $1/0.4=2.5$ 次——即平均 1.5 次失败之后跟 1 次成功，与公式一致。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 20 · Performance of CSMA/CD: Finding the efficiency（求效率）</span>

一个成功周期 = $A$ 个浪费的时隙（每个长 $2\times\text{PROP}$）+ 一次成功的发送（长 $P/R$），故

```math
\eta \;=\; \frac{P/R}{P/R \;+\; 2\,\text{PROP}\cdot A}
\qquad\xrightarrow{\ p=1/N,\ N\to\infty\ }\qquad
\eta \;=\; \frac{P/R}{P/R \;+\; 2\,(e-1)\,\text{PROP}}
```

取最优 $p$ 后，平均浪费的时隙数趋于 $A\to e-1\approx1.72$。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这套推导用的全是你已经会的概率工具：$A=1/q-1$ 就是 Bertsekas 里几何分布"首次成功所需试验次数期望为 $1/q$"的直接应用；而 $\left(1-\frac{1}{N}\right)^{N-1}\to e^{-1}$ 是 6.042J 与微积分里复利极限 $\left(1+\frac{1}{n}\right)^{n}\to e$ 的镜像——含义是：哪怕把 $p$ 调到最优，随机协调平均仍要浪费约 1.7 个时隙，这是"无中心调度"的固有税。效率公式还暴露了共享介质的死穴：$\eta$ 由 $P/R$ 与 $2(e-1)\,\text{PROP}$ 之比决定——包越短、链路越长越快，效率塌得越快。这个 $\text{PROP}$ 正是上一页最小帧长公式里的 $L/c$：同一个物理量，在"正确性"里决定帧长下限，在"效率"里决定吞吐上限，一路铺垫到交换机取代共享介质。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 21 · Performance of CSMA/CD: From simulation and more precise models（模拟与更精细的模型）</span>

图示 slide：给出仿真与更精细模型得到的效率曲线。结论与上述简化分析的趋势一致：效率随主机数量增加而下降、随"传输时延/传播时延"之比增大而上升；精细模型在此基础上修正了边界情形。
