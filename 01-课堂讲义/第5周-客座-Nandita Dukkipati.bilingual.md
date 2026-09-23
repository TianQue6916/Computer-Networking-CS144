> <span style="color:#7f8c8d;">Source: src/课堂讲义/第5周-客座-Nandita Dukkipati.txt · CS144 Fall 2019 · 行内双语翻译版</span>

# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## 现实世界中的拥塞控制（Congestion Control in the Real World）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">标题页</span>

- 主讲：Nandita Dukkipati（nanditad@google.com）——人名与邮箱保持原文。
- CS144 讲座，2019 年 10 月 25 日。
- 页面水印为 "Confidential + Proprietary"（Google 内部机密+专有），为幻灯片原注，正文略。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">今日讲座（Today's Talk）</span>

1. 现实世界的经验（Real World Exp.）
2. 面向低时延网络的基于时延的拥塞控制算法（外链）——TIMELY
3. 拥塞控制的多种不同方式——拥塞控制分类法（Congestion-Control Taxonomy）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">拥塞控制的多种信号方式与分类</span>

纯图示页：展示"有许多种不同的方式来通告拥塞（signal congestion）"。

**拥塞控制分类法（Congestion Control Taxonomy）：**

| 拥塞信号 | 代表算法 |
|---|---|
| 丢包（Loss） | New Reno、Cubic TCP |
| 时延（Delay） | TCP Vegas、TIMELY、BBR |
| 显式反馈（Explicit Feedback） | DCTCP、XCP、RCP |

- 参考文献（保留原文）：[1] TIMELY: RTT-based Congestion Control for the Datacenter, SIGCOMM 2015；[2] DCTCP: Data Center TCP (DCTCP), SIGCOMM 2010；[3] BBR: Congestion-based Congestion Control, ACM Queue, 2016；[4] TCP Vegas: new techniques for congestion detection and avoidance, SIGCOMM 1994。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这张分类表值得用"信号的信息量"来统一理解。丢包（loss）是最原始的信号：路由器缓冲区满了才会丢，等它发生时拥塞已经造成排队时延，甚至已经连累了别的包——而且一次丢包只携带 1 bit 信息（"发生了"），TCP New Reno/Cubic 只能据此做乘性减半。时延（delay）是先行信号：队列刚开始积压时延就上升，丢包还没发生，且用硬件时间戳测出的 RTT 有多比特分辨率——Vegas/TIMELY/BBR 都据此调速率。显式反馈（explicit feedback）则是路由器直接"开口说话"：ECN 标记、XCP/RCP 在包头写明拥塞程度，信息量最大。三者构成一个"信号越早、越丰富、但对基础设施改动越大"的光谱。你学过的 AIMD（加性增、乘性减）属于第一类：它在丢包信号下也能收敛到公平分享，但收敛慢且必然周期性丢包——数据中心受不了这种代价，这正是后面 TIMELY/DCTCP 出场的动机。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">显式拥塞通告（Explicit Congestion Notification, ECN）</span>

- 按照规范："如果队列增长过大，交换机将标记 CE 位。"
- 即：<span style="color:#00838f;">按照 IETF ECN 标准，当队列增长过大时，交换机会在分组上设置"已遭遇拥塞"（Congestion Experienced, CE）位。</span>
- <span style="color:#00838f;">交换机把标记告知接收方，接收方再反过来把拥塞标记告知发送方。</span>
- 图示：TCP 发送方 ↔（网络中的交换机标记 CE 位）↔ TCP 接收方；接收方通过 TCP 首部中的 ECN Echo 字段回告发送方。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">数据中心 TCP（Datacenter TCP, DCTCP）</span>

- 数据中心的 DCTCP（SIGCOMM 2010）使用 ECN 标记。
- 交换机在队列超过 65 KB 时在 IP 首部标记 CE 位。
- 接收方把标记反射回发送方（通过 TCP 标志位）。
- 发送方根据每个 RTT 内被标记分组的比例放慢速度。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">速率控制协议（Rate Control Protocol, RCP）</span>

RCP 的动机：

- <span style="color:#00838f;">用流完成时间（flow completion time）作为拥塞控制的首要指标。</span>
- 应当对"短流"也表现良好——短流没有机会进行持续的增量速率更新。
- 交换机内不做复杂的每分组计算。

图示（两页）：RCP 沿途使用路径上"最小支持速率"（min. supported rate）；RCP 算法示意——分组在每一跳被写入当前链路可支持的速率，取沿途最小值作为该流的发送速率。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">显式反馈是拥塞控制的游戏规则改变者</span>

端到端拥塞控制难以处理的问题：

- 辨别偶发的丢包/时延与持续性拥塞。
- 区分前向路径与反向路径上的拥塞。
- 弄清网络路径上到底哪一跳在贡献时延。
- 应用一次性发起海量连接，例如 $O(10^4)$ 条。
- 协调局域网（LAN）与广域网（WAN）的拥塞。
- ……以及其他若干问题。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">TIMELY：面向数据中心的基于 RTT 的拥塞控制</span>

（本节为讲座第二部分：低时延网络中基于时延的拥塞控制算法 TIMELY。）

**数据中心拥塞控制的高层目标：**

- 低时延的 RPC（Low-Latency for RPCs）：为短 RPC 和长 RPC 都提供低的完成时间。
- 端到端网络性能隔离（performance isolation）：在路径上每个拥塞点都做到隔离。
- 带宽效率（bandwidth efficiency）：即使在高负载突发汇聚（heavy incast）下也提供高吞吐。
- CPU 效率（CPU efficiency）：使用占用更少 CPU 的机制（例如不需要太多锁）。

**时延作为拥塞信号（Delay as a Congestion Signal）：**

- 可以用网卡（NIC）硬件时间戳精确测量。
- 是一种快速、多比特的信号，与排队（queuing）相关性极好。
- 是端到端拥塞信号，与拥塞控制想提供的时延保证直接挂钩。
- 是可分解的（decomposable）——可以在不同的拥塞点上提供隔离。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "时延是可分解的信号"这一条与排队论直接呼应。NIC 硬件时间戳让 RTT 测量精确到微秒级，而利特尔法则 $Q=\lambda d$ 告诉你：在到达率 $\lambda$ 已知时，平均排队时延 $d$ 与平均队列占用 $Q$ 一一对应——测出时延就等于测出了交换机队列的积压程度。更妙的是"可分解"：一次 RPC 的总 RTT 可以按发送网卡、本地排队、交换结构、远端处理等环节拆开（讲义后面的 SNAP 延迟来源图正是这个分解），从而把拥塞归因到具体瓶颈点并只对它做性能隔离；而丢包信号不可分解——你只知道"有包丢了"，不知道丢在哪一跳。这其实是第 3 周公式 $D=\sum l_i/c+\sum p/r_i+\sum Q_i(t)$ 的实测版：前三项固定，可变项只有排队，把总时延减去固定部分就得到排队时延的估计。TIMELY 正是据此把 RTT 与目标值比较来调节速率。

**TIMELY 框架与算法：**

- 图示：RTT 测量引擎（Measurement Engine）→ 速率计算引擎（Rate Computation Engine）→ "速率旋转木马"整形引擎（Rate Carousel's Shaping Engine）；基于测得 RTT 与目标 RTT（Target RTT）的距离做增/减（Increase/Decrease），按计算出的速率步调发送分组（paced packets）。

**可预测的低时延（Predictable Low-Latency）：**

- TIMELY 根据测得的时延调整发送速率。
- 发送方维护一个拥塞窗口（congestion-window, cwnd）来决定发送速率：若测得时延低于目标值，则增大 cwnd；否则减小 cwnd。
- 用加性增、乘性减（AIMD）来保证发送者之间的公平性。
- 目标时延（target-delay）允许对时延做细粒度控制：实验与生产数据表明 RTT 会围绕指定目标值徘徊。
- **要点 1（Takeaway 1）：TIMELY 把时延控制在指定目标值附近。**

**大规模部署中的 RTT：**

- RTT 中位数约 59 µs；第 99 百分位约 158 µs；第 99.9 百分位约 251 µs。
- 即使尾部 RTT 也远低于传统拥塞控制算法。

**高汇聚（High Fan-In）流量模式：**

- 当大量流向同一台主机发送时，TIMELY 的 cwnd 可以降到 1 以下。
- 这让 TIMELY 既兑现低时延与零丢包的承诺，又仍提供最大吞吐。
- cwnd < 1 通过用时间轮（timing wheel）对分组做步调发送（pacing）来实现。
- TIMELY 在基于速率（rate-based）与基于窗口（window-based）的协议之间无缝切换。
- **要点 2（Takeaway 2）：TIMELY 支持极小的 cwnd 以高效应对突发，同时维持高吞吐、低时延与近零丢包。**

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "cwnd < 1"表面上违反窗口的定义——一个窗口至少发一个包吧？它暴露的是窗口语义的离散化缺陷：若 1000 条流同时涌向一台主机（incast），链路容量摊到每条流只有千分之一，理想窗口是 0.001 个包，但传统 TCP 最多只能"每 RTT 发 1 个包"，1000 条流合起来仍是 1000 个包的突发，队列瞬间爆掉。TIMELY 的解法是 pacing（步调发送）：用时间轮把"每 RTT 0.001 个包"实现为"每 1000 个 RTT 发 1 个包"，相当于把速率定义从离散窗口推广到连续时间。这与第 3 周 WFQ 的虚拟时间思想同源：都是把"份额"从整包粒度细化到比特粒度。"在基于速率与基于窗口之间无缝切换"点破了二者的数学本质——窗口 $W$ 与速率 $W/\text{RTT}$ 本来就是同一约束的两种写法，难点只在工程实现。

**大规模汇聚（Large Incast：1000 条流同时发往 1 台）：**

- 吞吐量：TIMELY 能提供线速吞吐（NIC 速率 = 20G）。
- RTT：TIMELY 的 RTT 中位数为 172 µs，而 DCTCP 的中位数为 3310 µs；第 99 百分位上 TIMELY 为 370 µs，DCTCP 为 4900 µs。
- 重传：DCTCP 有 5.5% 的重传，TIMELY 约 0%。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">现实世界的经验（Real World Experience）</span>

**为 TIMELY 专设的交换机队列（Special Switch Queues for TIMELY）：**

- TIMELY 与传统 TCP 不兼容（non-TCP-compatible），需要自己的网络队列。
- 单独的队列提供与传统 TCP 流量的隔离。
- 如果 TIMELY 与 TCP 共享队列，时延、丢包、吞吐都会被 TCP 影响——因为 TCP 通常需要更多缓冲区。
- **要点 3（Takeaway 3）：TIMELY 跑在自己专用的网络队列上，这使它得以兑现低时延与近零丢包的承诺。**

- 图示：在相近吞吐下，TIMELY 的丢弃数显著更低（TCP discards vs TIMELY discards、TCP bps vs TIMELY bps；并对比 Swift）——某速率下 TIMELY 几乎零丢包。

**受 CPU 约束的网络（CPU constrained Networks）：**

- 拥塞不限于交换结构（fabric），也可以发生在端点（endpoints）。
- 网络栈没有无限的 CPU：绕过内核/用户态收包（OS-bypass/user-space packet-processing）是 CPU 高效的替代方案，但包处理引擎也可能过载、队列也会积压。
- <span style="color:#00838f;">端点拥塞的行为与交换结构拥塞不同：它更易变、更瞬态——不仅取决于每秒到达的包数，还取决于主机自身属性（如缓存局部性 cache locality）。</span>
- **要点 4（Takeaway 4）：用两个拥塞窗口而不是一个。** 一个跟踪交换结构（fabric）中的可用容量，另一个跟踪端点的可用容量：$cwnd = \min(f_{cwnd},\ e_{cwnd})$。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "两个拥塞窗口"是这场讲座最有洞察的设计：拥塞不只发生在网络交换结构，也发生在端点的软件栈——网卡 DMA、内核协议栈、CPU 缓存都可能成为瓶颈，而且端点拥塞更易变（还与缓存局部性等主机属性有关）。用 $cwnd=\min(f_{cwnd}, e_{cwnd})$ 取两者较小值，等于分别跟踪两种瓶颈资源——这与 CSAPP 的性能分析思想一致：系统性能取决于最慢的组件，必须分别建模 CPU 瓶颈与 I/O 瓶颈，不能混为一谈。后面 SNAP 用微内核方式做主机网络栈、Carousel 每核一个 shaper，都是在把"端点拥塞"这个新维度工程化。给读者的启示：拥塞控制本质上是一般化的资源分配问题，"哪里会堵"比"怎么退避"更重要——发现新的瓶颈类型往往比优化老算法收益更大。

**SNAP 中的时延来源（Delay sources in SNAP）：**

- 图示（"Traffic Roundabout"）：一次往返的 7 段时延：
  1. 本地网卡发送时延（Local NIC Tx Delay）；
  2. 前向交换结构时延（Forward Fabric Delay，含交换机队列）；
  3. 远端网卡接收时延（Remote NIC Rx Delay）；
  4. 远端处理时延（Remote Processing Delay）；
  5. 远端网卡发送时延（Remote NIC Tx Delay）；
  6. 反向交换结构时延（Reverse Fabric Delay，含交换机队列）；
  7. 本地网卡接收时延（Local NIC Rx Delay）。
- 背景注：SNAP 是支持 RDMA 的高性能主机网络栈（微内核方式）：*Snap: a Microkernel Approach to Host Networking*。

**实践中的流量整形器（Traffic Shapers in Practice）：**

- 分组来源：主机 OS 或虚拟机（Guest VM）的套接字缓冲区（socket buffers）。
- 图示：分组经分类器（classifier）与调度器（scheduler），用多个令牌桶队列做预过滤（pre-filtering with multiple token bucket queues），以 Rate1/Rate2/Rate3 三种速率整形后送往 NIC。

**Carousel 的设计原则（Design Principles of Carousel）：**

1. 单一、$O(1)$、按时间索引的队列，按分组时间戳排序。
2. 施加背压（backpressure）。
3. 每核一个整形器（one shaper per core）。

**一个分组在 Carousel 中的生命周期（Life of a Packet in Carousel）：**

- 根据整形速率计算最早出发时间（compute earliest departure time）→ 把分组入队到时间轮（timing wheel）→ 到点出队并交付完成通知（dequeue packet and deliver completion）。

**大规模评测（Large Scale Evaluation）：**

- Carousel 最多节省 8.2% 的整体 CPU 利用率（在 72 核机器上相当于 5.9 个核）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">结束语与致谢</span>

- 拥塞控制是一个有着广泛应用领域的**系统问题**：资源分配（resource allocation）、效率（efficiency）、隔离（isolation）；它直接影响应用的性能。
- 致谢：感谢 Google 的多位同事对这项工作与幻灯片内容的合作，特别是 Gautam Kumar。
