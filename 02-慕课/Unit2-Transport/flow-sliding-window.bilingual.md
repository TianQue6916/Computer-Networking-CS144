# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Flow Control II：Sliding Window（流量控制 II：滑动窗口）

> <span style="color:#7f8c8d;">来源：慕课/Unit2-Transport/flow-sliding-window.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1：Flow Control II（流量控制 II）· Sliding Window（滑动窗口）</span>

本讲义共 11 页，是 stop-and-wait（停等协议）的续篇：用 sliding window（滑动窗口）同时解决 flow control（流量控制）与 channel utilization（信道利用率）两个问题。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 2：Stop and Wait（停等协议回顾）</span>

- 任意时刻至多一个分组 in flight（在途）
- 发送方发送一个分组
- 接收方收到数据后回送 acknowledgment（确认）分组
- 发送方收到 acknowledgment（确认）后发送新数据
- 超时时重发当前数据
- 使用 1-bit counter（1 位计数器）检测重复分组

<span style="color:#7f8c8d;">（Slide 2/11）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 3：Stop and Wait Problem（停等的问题）</span>

图示一条从 San Francisco（旧金山）到 Boston（波士顿）的跨美 link（链路）：

- bottleneck bandwidth（瓶颈带宽）为 10 Mbps（兆比特每秒）
- round-trip time（往返时延，RTT）为 50 ms（毫秒）

<span style="color:#7f8c8d;">（Slide 3/11）</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 把 stop-and-wait（停等协议）的低效算成具体数字：bandwidth-delay product（带宽时延积）$= 10\,\text{Mbps} \times 50\,\text{ms} = 5 \times 10^5$ bit，即每个 round-trip time（往返时延，RTT）内链路本可以"装下"约 50 万比特；而 stop-and-wait（停等协议）每个 RTT 只允许 1 个分组在途，假设分组为 1000 字节，channel utilization（信道利用率）仅约 $\frac{8000}{5\times10^5 + 8000} \approx 1.6\%$——其余 98% 的时间链路都在空转。sliding window（滑动窗口）的解法在 CSAPP（《深入理解计算机系统》，Computer Systems: A Programmer's Perspective）里有精确类比：pipelining（流水线）。第 4 章讲 5 级流水线时，吞吐量的提升来自让多条指令的不同阶段同时占据硬件；sliding window（滑动窗口）让 SWS（send window size，发送窗口大小）个未被确认的分组同时占据链路，"keep the pipe full（保持管道充满）"与"保持流水线充满"是同一句话。SWS 就像流水线深度：深度至少要覆盖 bandwidth-delay product（带宽时延积）对应的分组数，否则 propagation delay（传播时延）仍然裸露在吞吐量里。这个"用并发隐藏时延"的思想在 TCP（Transmission Control Protocol，传输控制协议）的 congestion control（拥塞控制）（第 4 单元）中还会回来，届时窗口大小改由 network feedback（网络反馈）动态调节。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 4：Sliding Window（滑动窗口）</span>

- 这是对 stop-and-wait（停等协议）的 generalization（一般化）：允许多个 un-acked segment（未被确认的段）同时在途
- 对 un-acked segment（未被确认的段）的数量设置上限，这个上限称为 window（窗口）
- 这样就能让管道保持充满，也就是 keep the pipe full（保持管道充满）

<span style="color:#7f8c8d;">（Slide 4/11）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 5：Example Execution（运行示例）</span>

本页是 sender（发送方）/receiver（接收方）时序对比图，横轴为时间：

- **Stop and Wait（停等）**：`data 0 → ACK 0 → data 1 → ACK 1`，严格串行；
- **Sliding Window（滑动窗口）**：`data 0 → data 1 → ...` 连续发出多个数据，确认分组 `ACK 0`（acknowledgment，确认）等随后陆续返回，在途分组重叠。

图示说明：sliding window（滑动窗口）在同一时间窗内有多个未被确认的分组，而 stop-and-wait（停等协议）只有一个。

<span style="color:#7f8c8d;">（Slide 5/11）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 6：Sliding Window Sender（滑动窗口发送方）</span>

- 每个 segment（段）都有一个 sequence number（序列号，SeqNo）
- 维护 3 个变量：
  - **send window size（发送窗口大小，SWS）**：允许的 un-acked segment（未被确认的段）数量上限
  - **last acknowledgment received（最后收到的确认，LAR）**
  - **last segment sent（最后发送的段，LSS）**
- 维护 invariant（不变式）：$(LSS - LAR) \le SWS$
- 收到新 acknowledgment（确认）时推进 LAR
- 最多缓冲 SWS 个段

<span style="color:#7f8c8d;">（Slide 6/11）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 7：Sliding Window Receiver（滑动窗口接收方）</span>

- 维护 3 个变量：
  - **receive window size（接收窗口大小，RWS）**
  - **last acceptable segment（最后可接受的段，LAS）**
  - **last segment received（最后收到的段，LSR）**
- 维护 invariant（不变式）：$(LAS - LSR) \le RWS$
- 若收到的分组序号 $< LAS$，则发送 acknowledgment（确认）：
  - 发送 cumulative acks（累积确认）：若收到了 1、2、3、5，则确认到 3
  - **注意：TCP（Transmission Control Protocol，传输控制协议）的确认号是"下一个期望的数据"**（例如上例中 TCP 会回 ACK 4）

<span style="color:#7f8c8d;">（Slide 7/11）</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 发送方三个变量与 invariant（不变式）$(LSS - LAR) \le SWS$ 把整个窗口算法压缩成一行代数：LAR 只能被 ACK（acknowledgment，确认）推动，LSS 只能被 application layer（应用层）的新数据推动，invariant（不变式）保证"未确认段数"永不超过 SWS——这正是 6.042J（MIT 的《Mathematics for Computer Science》课程）里 state machine invariant（状态机不变式）证明方法的直接应用：先声明 invariant（不变式），再对每条转移逐一验证它成立，协议正确性就机械地有了着落。接收方的 cumulative acks（累积确认）是另一个精妙设计：收到 1、2、3、5 时只回"确认到 3"，含义是"3 及之前全部按序收齐"，一次 ACK 顶多个确认、节省上行带宽；代价是 4 丢失时 5 的到达信息被暂时浪费。幻灯片特别提醒：真实 TCP 的确认号语义是"下一个期望字节"（上例回 ACK 4 而非 ACK 3），这个差别是第 4 单元 fast retransmit（快速重传）能靠"三个重复 ACK"判定丢包的基础——重复 ACK 意味着"我还在等同一个字节"。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 8：RWS, SWS, and Sequence Space（窗口大小与序列号空间）</span>

- 约束：$RWS \ge 1$，$SWS \ge 1$，$RWS \le SWS$
- 若 $RWS = 1$：即 Go-Back-N（回退 N）协议，需要 $SWS + 1$ 个 sequence number（序列号）
- 若 $RWS = SWS$：需要 $2 \cdot SWS$ 个 sequence number（序列号）
- 一般情形需要 $RWS + SWS$ 个 sequence number（序列号）：
  - 其中 RWS 个序号留给"状态未知"的分组（其 acknowledgment（确认）可能已发、也可能丢失）
  - 而 SWS 个在途分组不得使 sequence number space（序列号空间）overflow（溢出）

<span style="color:#7f8c8d;">（Slide 8/11）</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 关于 $RWS + SWS$ 这个下界，值得自己推一遍。设序列号共 $k$ 个比特、$2^k$ 个取值：发送方至多有 SWS 个分组同时在途，这些分组的 acknowledgment（确认）可能已发出、也可能还在路上甚至丢失；接收方因此还要预留 RWS 个序号给"可能仍在途的旧分组"，防止新分组的序号与旧分组撞车后无法区分。两段序号域加起来不能超出 $2^k$，即得 $2^k \ge SWS + RWS$。两个极端情形都落在公式上：$RWS = 1$ 时退化为 Go-Back-N（回退 N）——接收窗口为 1 意味着失序分组一律丢弃、只认下一个按序字节，所以 $SWS + 1$ 个序号就够；$RWS = SWS$ 时是 selective repeat（选择性重传）——与 Go-Back-N（回退 N）相反，失序分组会被缓存而不是丢弃、只重传真正丢失的分组，需要 $2\,SWS$ 个。真实 TCP 用 32 位序号、按字节计序，空间大到 wrap-around（回绕）几乎不会与活跃旧分组相遇，再加 three-way handshake（三次握手）随机化的 ISN（initial sequence number，初始序列号）把新旧连接的序号域隔开。这里"序号回绕取模、留出安全余量防歧义"的处理，与 6.042J（即 MIT 的《Mathematics for Computer Science》课程）中 modular arithmetic（模运算）与 pigeonhole principle（鸽笼原理）的论证方式是同一类思维。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 9：TCP Flow Control（TCP 流量控制）</span>

- 接收方使用 header（首部）中的 window（窗口）字段通告自己的 RWS
- 发送方只能发送到 $LAR + \text{window}$ 为止的数据

本页还给出了 TCP header（TCP 首部）示意图，字段包括：source port（源端口）、destination port（目的端口）、data sequence number（数据序列号）、acknowledgment sequence number（确认序列号）、offset（数据偏移）、res（reserved，保留位）、N C E（NS、CWR、ECE 三个拥塞标志位）、U A P R S F（URG、ACK、PSH、RST、SYN、FIN 六个标志位）、window（窗口）、checksum（校验和）、urgent pointer（紧急指针）、options（选项）。

<span style="color:#7f8c8d;">（Slide 9/11）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 10：Sliding Window Example（滑动窗口数值示例）</span>

本页为一张数值轴图，横轴标出 sequence number（序列号）0、5、10、15、20、25 的刻度，示意 window（窗口）随 acknowledgment（确认）到达而在 sequence number space（序列号空间）中向前滑动的形态：窗口左端（即 LAR）被 acknowledgment（确认）推动、右端（即 LSS）被新数据推动，两者间距不超过 SWS。

<span style="color:#7f8c8d;">（Slide 10/11）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 11：Sliding Window Flow Control（滑动窗口流量控制）总结</span>

- 允许 window（窗口）数量的 un-acked segment（未被确认的段）in flight（在途）
- 当 acknowledgment（确认）到达时，advance window（窗口向前推进）
- 所需 sequence number space（序列号空间）的大小取决于 window size（窗口大小）

<span style="color:#7f8c8d;">（Slide 11/11）</span>

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 2-8《Sliding window》的配套 slides（幻灯片）；下一讲 2-9 讨论两种 retransmission strategy（重传策略）：Go-Back-N（回退 N）与 Selective Repeat（选择性重传）。</span>
