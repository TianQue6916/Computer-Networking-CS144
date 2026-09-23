# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Flow Control II：Sliding Window（流量控制 II：滑动窗口）

> <span style="color:#7f8c8d;">Source: 慕课/Unit2-Transport/flow-sliding-window.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1：Flow Control II（流量控制 II）· Sliding Window（滑动窗口）</span>

本讲义共 11 页，是停等协议（stop-and-wait）的续篇：用滑动窗口（sliding window）同时解决流量控制与信道利用率问题。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 2：Stop and Wait（停等协议回顾）</span>

- 任意时刻至多一个分组在途（at most one packet in flight）
- 发送方发送一个分组
- 接收方收到数据后回送确认分组
- 发送方收到确认后发送新数据
- 超时时重发当前数据
- 使用 1-bit 计数器（1-bit counter）检测重复分组

<span style="color:#7f8c8d;">（Slide 2/11）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 3：Stop and Wait Problem（停等的问题）</span>

图示一条从 San Francisco（旧金山）到 Boston（波士顿）的跨美链路：

- 瓶颈带宽（bottleneck）为 10 Mbps
- 往返时延（RTT，round-trip time）为 50 ms

<span style="color:#7f8c8d;">（Slide 3/11）</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 把停等的低效算成具体数字：带宽时延积（bandwidth-delay product）$= 10\,\text{Mbps} \times 50\,\text{ms} = 5 \times 10^5$ bit，即每个 RTT 内链路本可以"装下"约 50 万比特；而停等每 RTT 只允许 1 个分组在途，假设分组为 1000 字节，信道利用率仅约 $\frac{8000}{5\times10^5 + 8000} \approx 1.6\%$——其余 98% 的时间链路都在空转。滑动窗口的解法在 CSAPP 里有精确类比：流水线。第 4 章讲 5 级流水线时，吞吐量的提升来自让多条指令的不同阶段同时占据硬件；滑动窗口让 SWS 个未被确认的分组同时占据链路，"keep the pipe full（保持管道充满）"与"保持流水线充满"是同一句话。SWS 就像流水线深度：深度至少要覆盖带宽时延积对应的分组数，否则传播时延仍然裸露在吞吐量里。这个"用并发隐藏时延"的思想在 TCP 拥塞控制（第 4 单元）中还会回来，届时窗口大小改由网络反馈动态调节。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 4：Sliding Window（滑动窗口）</span>

- 停等的一般化（generalization of stop-and-wait）：允许**多个未被确认的段（un-acked segments）**同时在途
- 对未确认段的数量设置上限，称为**窗口（window）**
- 这样可以让管道保持充满（keep the pipe full）

<span style="color:#7f8c8d;">（Slide 4/11）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 5：Example Execution（运行示例）</span>

本页是 sender（发送方）/receiver（接收方）时序对比图，横轴为时间：

- **Stop and Wait（停等）**：`data 0 → ACK 0 → data 1 → ACK 1`，严格串行；
- **Sliding Window（滑动窗口）**：`data 0 → data 1 → ...` 连续发出多个数据，确认 `ACK 0` 等随后陆续返回，在途分组重叠。

图示说明：滑动窗口在同一时间窗内有多个未确认分组，而停等只有一个。

<span style="color:#7f8c8d;">（Slide 5/11）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 6：Sliding Window Sender（滑动窗口发送方）</span>

- 每个段都有一个序列号（sequence number，SeqNo）
- 维护 3 个变量：
  - **发送窗口大小（send window size，SWS）**：允许的未确认段上限
  - **最后收到的确认（last acknowledgment received，LAR）**
  - **最后发送的段（last segment sent，LSS）**
- 维护不变式（invariant）：$(LSS - LAR) \le SWS$
- 收到新确认时推进 LAR
- 最多缓冲 SWS 个段

<span style="color:#7f8c8d;">（Slide 6/11）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 7：Sliding Window Receiver（滑动窗口接收方）</span>

- 维护 3 个变量：
  - **接收窗口大小（receive window size，RWS）**
  - **最后可接受的段（last acceptable segment，LAS）**
  - **最后收到的段（last segment received，LSR）**
- 维护不变式：$(LAS - LSR) \le RWS$
- 若收到的分组序号 $< LAS$，则发送确认：
  - 发送**累积确认（cumulative acks）**：若收到了 1、2、3、5，则确认到 3
  - **注意：TCP 的确认号是"下一个期望的数据"**（例如上例中 TCP 会回 ack 4）

<span style="color:#7f8c8d;">（Slide 7/11）</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 发送方三个变量与不变式 $(LSS - LAR) \le SWS$ 把整个窗口算法压缩成一行代数：LAR 只能被 ACK 推动，LSS 只能被应用层新数据推动，不变式保证"未确认段数"永不超过 SWS——这正是 6.042J 状态机不变式证明方法的直接应用：先声明不变式，再对每条转移逐一验证它成立，协议正确性就机械地有了着落。接收方的累积确认是另一个精妙设计：收到 1、2、3、5 时只回"确认到 3"，含义是"3 及之前全部按序收齐"，一次 ACK 顶多个确认、节省上行带宽；代价是 4 丢失时 5 的到达信息被暂时浪费。幻灯片特别提醒：真实 TCP 的确认号语义是"下一个期望字节"（上例回 ack 4 而非 ack 3），这个差别是第 4 单元快速重传（fast retransmit）能靠"三个重复 ACK"判定丢包的基础——重复 ACK 意味着"我还在等同一个字节"。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 8：RWS, SWS, and Sequence Space（窗口大小与序列号空间）</span>

- 约束：$RWS \ge 1$，$SWS \ge 1$，$RWS \le SWS$
- 若 $RWS = 1$：即 **Go-Back-N（回退 N）协议**，需要 $SWS + 1$ 个序列号
- 若 $RWS = SWS$：需要 $2 \cdot SWS$ 个序列号
- 一般情形需要 $RWS + SWS$ 个序列号：
  - RWS 个序号留给"状态未知"的分组（其确认可能已发、也可能丢失）
  - SWS 个在途分组不得使序列号空间溢出（overflow）

<span style="color:#7f8c8d;">（Slide 8/11）</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> $RWS + SWS$ 这个下界值得自己推一遍。设序列号共 $k$ 个比特、$2^k$ 个取值：发送方有 SWS 个分组在途，确认可能已发出也可能还在路上；接收方还要预留 RWS 个序号给"可能仍在途的旧分组"，防止新分组的序号与旧分组撞车后无法区分。两个需求相加即得 $2^k \ge SWS + RWS$。两个极端情形都落在公式上：$RWS = 1$ 时退化为 Go-Back-N——接收窗口为 1 意味着失序分组一律丢弃、只认下一个按序字节，所以 $SWS + 1$ 个序号就够；$RWS = SWS$ 时是选择性重传（selective repeat），需要 $2\,SWS$ 个。真实 TCP 用 32 位序号、按字节计序，空间大到回绕（wrap-around）几乎不会与活跃旧分组相遇，再加三次握手随机化的初始序号（ISN）把新旧连接的序号域隔开。这里"序号回绕取模、留出安全余量防歧义"的处理，与 6.042J 中模运算与鸽笼原理的论证方式是同一类思维。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 9：TCP Flow Control（TCP 流量控制）</span>

- 接收方使用首部中的 **window（窗口）字段**通告自己的 RWS
- 发送方只能发送到 $LAR + \text{window}$ 为止的数据

本页还给出了 TCP 首部（TCP header）示意图，字段包括：source port（源端口）、destination port（目的端口）、data sequence number（数据序列号）、acknowledgment sequence number（确认序列号）、offset、res、N C E、U A P R S F（六个标志位）、window（窗口）、checksum（校验和）、urgent（紧急指针）、options（选项）。

<span style="color:#7f8c8d;">（Slide 9/11）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 10：Sliding Window Example（滑动窗口数值示例）</span>

本页为一张数值轴图，横轴标出序列号 0、5、10、15、20、25 的刻度，示意窗口随确认到达而在序列号空间中向前滑动的形态：窗口左端（LAR）被确认推动、右端（LSS）被新数据推动，两者间距不超过 SWS。

<span style="color:#7f8c8d;">（Slide 10/11）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 11：Sliding Window Flow Control（滑动窗口流量控制）总结</span>

- 允许一个"窗口"数量的未确认分组在途（in flight）
- 确认到达时，窗口向前推进（advance window）
- 所需序列号空间的大小取决于窗口大小

<span style="color:#7f8c8d;">（Slide 11/11）</span>

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 2-8《Sliding window》的配套 slides；下一讲 2-9 讨论两种重传策略（Go-Back-N 与 Selective Repeat）。</span>
