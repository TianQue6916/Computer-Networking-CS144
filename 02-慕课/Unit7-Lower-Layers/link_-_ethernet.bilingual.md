# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## Physical Links: Ethernet（物理链路：以太网）

> <span style="color:#7f8c8d;">Source: src/慕课/Unit7-Lower-Layers/link_-_ethernet.txt · CS144 Fall 2019 · 行内双语翻译版</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 1 · 标题页</span>

Physical Links（物理链路）——Ethernet（以太网）。主讲人：Nick McKeown，Stanford University 电气工程与计算机科学教授。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 2 · The Original Ethernet（最初的以太网）</span>

图示 slide：Bob Metcalfe（以太网共同发明人）绘制的原始草图，1972 年，Xerox PARC。草图勾勒了多台工作站挂在一条公共电缆上的最初构想——"以太网"这个名字也画在图上。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这张图是网络史上的著名文物：1972–1973 年，Bob Metcalfe 与 David Boggs 在 Xerox PARC 为 Alto 工作站设计联网系统，画下了这份备忘录草图；名字借自 19 世纪物理学家假想的传光介质"以太"（luminiferous ether）——寓意像以太一样无处不在的广播介质。1980 年 DEC/Intel/Xerox 三家发布 DIX 标准，1983 年 IEEE 802.3 成为正式标准；Metcalfe 后来创办 3Com，并因发明以太网获 2022 年图灵奖（2023 年颁发）。最有意思的对照是：草图画的是共享总线，今天的以太网却已是全双工、点对点的交换链路——四十多年里协议内核（CSMA/CD）被逐步退役，反而"帧格式"几乎一厘米未改地活了下来。协议的寿命取决于它的可扩展缝隙在哪里，这值得你在学每个协议时都想一想。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 3 · Ethernet Frame Format（以太网帧格式）</span>

图示按字节标注的帧结构：

| 字段 | 字节数 |
|---|---|
| Preamble（前导码） | 7 |
| SFD（帧起始定界符） | 1 |
| DA（目的地址） | 6 |
| SA（源地址） | 6 |
| Type（类型） | 2 |
| Data（数据） | 0–1500 |
| Pad（填充） | 0–46 |
| CRC（校验） | 4 |

1. **Preamble**：用于"训练"接收端的时钟恢复电路（与《时钟与时钟恢复》一讲呼应——发一串交替的 1/0 让锁相环先锁定节奏）；
2. **SFD（Start of Frame Delimiter，帧起始定界符）**：指示帧从哪里开始；
3. **DA（Destination Address，目的地址）**：48 位全球唯一地址，由网卡厂商分配；其中 1 位区分 unicast（单播）/multicast（多播），1 位区分 local/global（本地管理/全球管理）地址；
4. **Type**：指示封装数据所属的协议（如 IP = 0x0800）；
5. **Pad（填充）**：用零填充以保证最小帧长；
6. **CRC（Cyclic Redundancy Check，循环冗余校验）**：校验序列，用于检测比特错误。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这里有本课程的两条主线交汇。其一是两级地址体系：MAC 地址负责"同一段链路上的本地投递"，IP 地址负责"跨网络的端到端寻址"——你在 CSAPP ch11 写 socket 时只见过 IP+端口，因为操作系统把链路层整个藏了起来；把 IP 解析成 MAC 的正是前一单元讲的 ARP（地址解析协议）。其二是分用（demultiplexing）链条：帧到达后按 Type=0x0800 交给 IP，IP 按协议字段交给 TCP/UDP，TCP/UDP 按端口号交给进程——最后一级正是 CSAPP ch11 里端口分用进程的机制，链路层的 Type 只是这条链的第一环。另外，Pad 与最小帧长 64 字节联动（7+1+6+6+2+0+0+4 不足 64 就补零），这 64 字节正是上一讲 $P\ge 2LR/c$ 算出的冲突检测要求在标准里的落地。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 4 · The 10Mb/s Ethernet Standard（10 Mb/s 以太网标准）</span>

图示 IEEE 802.3 标准树：Ethernet MAC Protocol（以太网 MAC 协议）在树根，四种物理实现在枝头——10Base-5、10Base-2、10Base-T、10Base-F。

- 10Base-5：原始以太网，粗同轴电缆（thick coaxial cable）；
- 10Base-2：细同轴电缆（thin coaxial cable）版本；
- 10Base-T：话音级非屏蔽双绞线（Category-3 电话线，unshielded twisted-pair）；
- 10Base-F：单缆内两根光纤（optical fibers）。

同一套 MAC 协议可以骑在完全不同的物理介质上——"协议与介质解耦"正是分层带来的好处。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 5 · 10Base-T（双绞线以太网）</span>

"Twisted pair Ethernet"（双绞线以太网）。图示：A、B、C、D 四台主机通过 Hub（集线器）星型连接，集线器再经 Router（路由器）连出。

- 跑在已有的话音级 "Category-3" 双绞电话线上——不用重新布线；
- "星型"（star）布线与楼宇里的配线间（wiring closet）天然契合；
- 集中式管理使网络更容易运维；
- 直接促成了 1980 年代中期以太网的爆发式增长。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 6 · Increasing the data rate（提升速率的难题）</span>

演进路线：10 Mb/s → 100 Mb/s → 1 Gb/s → 10 Gb/s。

问题：<span style="color:#00838f;">CSMA/CD 要求 $P/R \ \ge\ 2L/c$。</span>当 $R$ 提高 10 倍、100 倍时，是让 $P$ 更大（帧更长），还是让 $L$ 更小（链路更短）？

解决：

- 把 100 Mb/s 与 1 Gb/s 以太网的 $L$ 限制到 100 米；
- 采用 Ethernet switching（以太网交换）。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 7 · Faster and Faster（越来越快）</span>

图示：标准树继续生长——100Base-TX / 100Base-FX（100 Mb/s "Fast" Ethernet，快速以太网）与 1000Base-T / 1000Base-FX（1 Gb/s Ethernet，千兆以太网），树根仍是以太网 MAC 协议。

- 100Base-TX：Cat 5 电缆 + RJ45 接头；全双工（full duplex），每个方向一对线各跑 100 Mb/s；物理层用 4B5B 编码（见《时钟与时钟恢复》）；最大距离 100 m；
- 1000Base-T：Cat 5 电缆 + RJ45；四对线同时在两个方向使用；复杂编码、5 电平信令（5-level signaling）；最大距离 100 m。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 8 · Ethernet Switching（以太网交换的兴起）</span>

四个因素共同把业界推向交换机：

- 10Base-T 意味着配线间里放的是 hub/repeater（集线器/中继器）——大家仍共享一个冲突域；
- 100 Mb/s 与 1 Gb/s 以太网带来 100 米的距离限制；
- 需要把以太网切分成小段，以缩小 collision domain（冲突域）；
- 交换硬件的成本大幅下降。

→ 于是有了 Ethernet Switching（以太网交换）。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 9 · Hubs to Switches（从集线器到交换机）</span>

图示同一拓扑的更新：A、B、C、D 改接 Switch（交换机）的独立端口，交换机再连 Router。

区别：集线器把收到的信号原样复制到所有端口（所有主机仍共享一个冲突域，CSMA/CD 照旧）；交换机则每个端口一个独立冲突域，可以同时转发多对通信。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 10 · Ethernet switch at Stanford（斯坦福的以太网交换机）</span>

图示 slide：一台早期以太网交换机的设备照片（摄于斯坦福），展示交换机从协议概念变成实际商品。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 11 · Ethernet Switch（以太网交换机：转发与学习）</span>

Forwarding and Learning（转发与学习）：

1. 检查每个到达帧的首部；
2. 若以太网 DA（目的地址）在 forwarding table（转发表）中，则把帧转发到正确的输出端口；
3. 若 DA 不在表中，则向除入端口之外的所有端口 broadcast（广播）该帧；
4. 转发表中的表项通过"查看到达包的以太网 SA（源地址）"来学习（learning）。

Topology（拓扑）：运行 Spanning Tree Protocol（STP，生成树协议）构造无环（loop free）拓扑。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 交换机的"自学习"本质上是在线构建一张 (MAC 地址 → 端口) 的哈希表：每来一帧就记录"源地址从哪个口进来"（学习），查目的地址决定转发还是泛洪——查表平均 $O(1)$，正是 CLRS 散列表的直接应用；表项带老化时间，对应哈希表的过期删除（主机搬家、表就失效重学）。而 STP 用 6.042J 的图论语言说最清楚：把交换机看成顶点、链路看成边，带环的图里广播帧会沿环无限复制（广播风暴），STP 在图上删掉若干条边、造出一棵 spanning tree（生成树）——保持全图连通（任意两点仍有唯一路径）同时消灭所有环，$n$ 个顶点恰剩 $n-1$ 条边。注意它求的不是 CLRS 里按权重最优的 MST（最小生成树）：STP 只要任意一棵以"根交换机"为中心的生成树即可，但两者共享同一套图论骨架。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 12 · 组网层级图示</span>

纯图示 slide：从边缘到骨干的组网层级——Switch 与 Switch 相连，向上经 Router（路由器）、再经 BGP Router（边界网关协议路由器）接入 Public Internet（公共互联网）。对应各层分工：交换机工作在链路层，路由器工作在网络层，BGP 负责自治系统（AS）之间的域间路由。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 13 · Summary（总结）</span>

- MAC 协议分为随机（Random）与确定性（Deterministic）两大类；
- CSMA/CD 是一个简单、随机接入的协议，用在最早的 10 Mb/s 以太网中；
- 过去 20 年间，100 Mb/s、1 Gb/s、10 Gb/s 的以太网标准相继出现；
- 链路长度的限制与对更大容量的需求，意味着 CSMA/CD 最终被以太网交换（Ethernet switching）取代。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 14 · &lt;end&gt;</span>

原讲义的视频分段结束标记，无正文内容。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 15 · Performance of CSMA/CD（性能分析·引入）</span>

> <span style="color:#7f8c8d;">本节 5 页（Slide 15–19）出自 CS244a（Winter 2008）Handout 10（p.15–19），与《CSMA/CD and Ethernet》一讲末尾内容相同；原 slides 的公式与曲线以图片嵌入、文本提取丢失，以下推导依据原 slide 的文字定义补全。</span>

我们要分析 CSMA/CD 网络的性能。指标是 efficiency（效率）$\eta$：定义为"用于发送有用的、成功的数据"的时间占比——花在制造与检测冲突上的时间越多，协议效率越低。

简化假设：时间分槽（slotted），一个时隙（slot）长为 $2\times\text{PROP}$（PROP 为最大端到端传播时延）；任一时隙内每台主机以概率 $p$ 决定发送（含首次发送与重传）。

分析分两步：先找使吞吐量（实为 goodput，有效吞吐）最大的 $p$，再用最优 $p$ 求效率。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 16 · Performance of CSMA/CD: Maximizing goodput（最大化有效吞吐）</span>

设共 $N$ 台主机。某个时隙"恰好一台主机发送"的概率为

```math
q \;=\; N\,p\,(1-p)^{N-1}
```

对 $q$ 求极值，最优 $p^{*}=1/N$，此时 $q_{\max}=\left(1-\tfrac{1}{N}\right)^{N-1}\ \to\ e^{-1}$（当 $N\to\infty$）。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 17 · Performance of CSMA/CD: Finding the overhead（求浪费的开销）</span>

定义 $A$ 为一个包成功发送之前平均被浪费的时隙数。每个时隙以概率 $q$ 成功（几何分布），故 $A = 1/q - 1$。

原 slide 的硬币类比：一枚正面概率为 0.4 的硬币，掷出首个正面平均需 $1/0.4=2.5$ 次——即平均 1.5 次失败之后跟 1 次成功。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 18 · Performance of CSMA/CD: Finding the efficiency（求效率）</span>

一个成功周期 = $A$ 个浪费时隙（每个 $2\times\text{PROP}$）+ 一次成功发送（$P/R$）：

```math
\eta \;=\; \frac{P/R}{P/R \;+\; 2\,\text{PROP}\cdot A}
\qquad\xrightarrow{\ p=1/N,\ N\to\infty\ }\qquad
\eta \;=\; \frac{P/R}{P/R \;+\; 2\,(e-1)\,\text{PROP}}
```

取最优 $p$ 后，平均浪费时隙数趋于 $A\to e-1\approx1.72$。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 把效率公式放到以太网演进的时间线上看，它恰好解释了 Slide 13 的结论。共享 CSMA/CD 里每一份吞吐都要缴"协调税"：$2(e-1)\,\text{PROP}$ 这一项——几何分布（Bertsekas 首次成功的期望 $1/q$）给出的 $e-1$ 说明即使 $p$ 调到最优，平均仍有约 1.7 个时隙被冲突浪费；而 $\text{PROP}=L/c$ 随速率提升相对 $P/R$ 越来越大，效率随之塌陷。交换机把每条链路变成全双工点对点，冲突与退避整个消失，"效率"问题被连根拔除——这就是 Slide 8"交换硬件降价"四个字背后的定量动因。概率模型的价值正在于此：一个 $e^{-1}$ 就能预言一项技术二十年的兴衰。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 19 · Performance of CSMA/CD: From simulation and more precise models（模拟与更精细的模型）</span>

图示 slide：仿真与更精细模型得到的效率曲线。结论与简化分析趋势一致：主机越多效率越低、"传输时延/传播时延"之比越大效率越高；精细模型在边界情形上做了修正。
