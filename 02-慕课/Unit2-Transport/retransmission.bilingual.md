# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Retransmission Strategies（重传策略）

> <span style="color:#7f8c8d;">Source: 慕课/Unit2-Transport/retransmission.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1：Retransmission Strategies（重传策略）</span>

本讲义共 6 页，讨论滑动窗口（sliding window）之上丢包发生后的两种重传策略：Go-Back-N（回退 N）与 Selective Repeat（选择性重传）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 2：Question（问题）</span>

<span style="color:#2471a3;">**[question]**</span>（习题原文保留如下）

```text
Have a window of packets in flight for reliable transport
Using cumulative acknowledgments
Per-packet timer
How does the protocol behave?
Go-back-N: one loss will lead to entire window retransmitting
Selective repeat: one loss will lead to only that packet retransmitting
```

题意讲解（行内双语）：设可靠传输有一个窗口的分组在途（in flight），使用**累积确认（cumulative acknowledgments）**与**每分组定时器（per-packet timer）**，协议将如何表现？——**Go-Back-N（回退 N）**：一次丢包导致整个窗口重传；**Selective Repeat（选择性重传）**：一次丢包只重传那一个分组。

<span style="color:#7f8c8d;">（Slide 2/6）</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这两种策略的差异本质是"用谁的资源换谁的效率"。用概率论的期望语言核算：设丢包率为 $p$ 且一次丢包使窗口内其余在途分组失效，Go-Back-N 平均要多传 $O(N)$ 个分组，而选择性重传只需重传 1 个；当 $N$ 大、$p$ 不可忽略时，两者的有效吞吐量差距是 $N$ 倍量级的。但 Go-Back-N 换来的是接收方极简：它只认"下一个按序字节"，失序分组直接丢弃，不需要任何缓冲区。真实 TCP 早期正是带着这个选择演化的——Phil 在单元总结里概括"基础 TCP 机制使用 Go-Back-N"，后来加入的快速重传（fast retransmit，收到 3 个重复 ACK 就重传窗口头部）和 SACK 选项，才把选择性重传的能力逐步"补丁"回 TCP。这提示协议设计很少一步到位：1980 年代主机内存昂贵，先选接收方零缓存的简单方案，再随硬件成本下降增强——与 CSAPP 中缓存层次随工艺演化的逻辑一样，都是成本约束下的工程折中。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 3：Go-Back-N（回退 N）</span>

本页为 sender（发送方）/receiver（接收方）时序图，标题为 Sliding Window（滑动窗口）：发送方发出窗口内多个分组；当窗口头部某分组丢失后，接收方丢弃其后所有失序到达的分组，发送方超时后**从丢失处起把整个窗口的分组全部重传一遍**。

<span style="color:#7f8c8d;">（Slide 3/6）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 4：Selective Repeat（选择性重传）</span>

本页同为 sender/receiver 时序图：发送方发出窗口内多个分组；某分组丢失后，接收方**缓存失序到达的后续分组**，发送方仅对丢失的那一个分组设置定时器并重传，接收方收齐后按序上交。

<span style="color:#7f8c8d;">（Slide 4/6）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 5：First Example（第一个例子）</span>

- 发送方（sender）窗口为 $N$
- 接收方（receiver）窗口为 1
- 这是 Go-Back-N 还是 Selective Repeat？

**解答：** 接收窗口为 1 意味着接收方不缓存任何失序分组——窗口头部一旦丢失，后续分组无论是否到达都注定被丢弃。此时选择性重传毫无收益，两者行为完全等价于 Go-Back-N；逐分组定时器也退化为整窗超时重传。

> <span style="color:#1e8449;">**[solution]**</span> 结论：**Go-Back-N**（选择性重传在接收窗口为 1 时退化为无效）。

<span style="color:#7f8c8d;">（Slide 5/6）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 6：Second Example（第二个例子）</span>

- 发送方（sender）窗口为 $N$
- 接收方（receiver）窗口为 $N$
- 这是 Go-Back-N 还是 Selective Repeat？

**解答：** 接收方窗口为 $N$ 时可以缓存最多 $N$ 个失序分组并逐个确认，此时两种策略都可以实现，但只有**选择性重传**能兑现接收方缓存的全部价值：一次丢包只补那一个洞，其余 $N-1$ 个分组的传输成果全部保留。

> <span style="color:#1e8449;">**[solution]**</span> 结论：应当采用 **Selective Repeat**（接收方有缓冲能力时，整窗回退是在浪费带宽与已缓存的分组）。

<span style="color:#7f8c8d;">（Slide 6/6）</span>

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 2-9《Reliable comm --- Retransmission strategies》的配套 slides；两个例子的判定关键都在接收窗口大小。</span>
