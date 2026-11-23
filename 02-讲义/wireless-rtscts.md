# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

> <span style="color:#7f8c8d;">Source: src/慕课/Unit7-Lower-Layers/wireless-rtscts.txt · CS144 Fall 2019 · 行内双语翻译版</span>

## <span style="color:#2471a3;">**[lecture]**</span> <span style="color:#c0392b">Wireless: RTS/CTS（无线：请求发送/允许发送）</span>

原文 Slide 1 为标题页：**Wireless: RTS/CTS**——request-to-send / clear-to-send（请求发送/允许发送）。本讲先看 RTS/CTS 四步握手如何修补 CSMA/CA 的三大问题，再算一笔账：这个修补机制的 overhead（开销）到底值不值。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 2 · RTS/CTS（请求发送/允许发送的四步握手）</span>

图示：$A$、$B$、$C$ 三个节点，时间轴分四步（1–4），每一步 $A$、$B$ 交替收发，节点 $C$ 在每一步旁边监听：

1. $A$ 向 $B$ 发送 RTS（request-to-send，请求发送）短帧；
2. $B$ 回应 CTS（clear-to-send，允许发送）短帧；
3. $A$ 发送数据帧（data）；
4. $B$ 回送 ACK（确认）。

<span style="color:#00838f;">标题 RTS/CTS 表明：正式的数据发送之前，先用两个短帧"打招呼、拿许可"。</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这套握手是对 hidden terminal（隐藏终端）的定点手术。回忆隐藏终端的结构：$C$ 听不见 $A$，但听得见 $B$。所以 $A$ 发的 RTS 本身帮不了 $C$（它压根收不到），真正起作用的是 $B$ 发出的 CTS——$C$ 听得见 $B$。CTS 里携带 duration（持续时间）字段，明说"接下来的若干微秒信道归 $A \to B$ 使用"，$C$ 据此设置一个倒计时定时器（802.11 里称为 NAV，network allocation vector，网络分配向量），期间保持静默。妙处在于**信息被搬运了**：接收方 $B$ 把自己的视角广播给所有听得见它的节点，等于替它们代做了一次"接收方状态"的侦听——上一讲暴露终端缺的那块信息，这里用两个短帧补上了。代价也摆在明面上：每次传输都要先付 RTS 加 CTS 两个短帧及帧间隙的成本，这笔账在最后两页算。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 3 · Problems with CSMA/CA（CSMA/CA 的问题）</span>

回顾 CSMA/CA 的三大问题：

- 问题 1：hidden terminals（隐藏终端）；
- 问题 2：exposed terminals（暴露终端）；
- 问题 3：是冲突还是低信噪比（collision or low SNR?）。

这一页是对上一讲的小结，也是 RTS/CTS 登场的理由：三者之中，RTS/CTS 直接解决前两个——CTS 让隐藏终端闭嘴（问题 1），RTS/CTS 期间邻居精确知道信道被占用多久、不必过度退避（问题 2）。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 4 · Problem with RTS/CTS: Overhead（RTS/CTS 的问题：开销）</span>

图示：$A$、$B$、$C$ 三个节点与 802.11b 的速率档。Slide 抛出问题：How long do these packets take?（这些分组的传输要花多长时间？）

以 802.11b 的四个速率档对比（纯 CSMA 与启用 RTS/CTS）：

| Bitrate（比特率） | 1 Mbps | 2 Mbps | 5.5 Mbps | 11 Mbps |
|---|---|---|---|---|
| CSMA | 0.79 | 1.44 | 3.36 | 5.89 |
| RTS/CTS | 0.76 | 1.35 | 2.89 | 4.42 |
| Overhead（开销） | 4.0% | 6.6% | 14.1% | 25.1% |

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 5 · Problem with RTS/CTS: Overhead（RTS/CTS 的问题：开销，续）</span>

同一张表再列一次，结论落在 Overhead（开销）一行：启用 RTS/CTS 的代价随比特率升高，从 4.0% 一路涨到 25.1%。

| Bitrate（比特率） | 1 Mbps | 2 Mbps | 5.5 Mbps | 11 Mbps |
|---|---|---|---|---|
| CSMA | 0.79 | 1.44 | 3.36 | 5.89 |
| RTS/CTS | 0.76 | 1.35 | 2.89 | 4.42 |
| Overhead（开销） | 4.0% | 6.6% | 14.1% | 25.1% |

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 表中 CSMA 与 RTS/CTS 两行数值随比特率同步增长，量级与 802.11b 的有效吞吐量（throughput，单位 Mbps）吻合；Overhead 一行可以用前两行直接验证：$(5.89 - 4.42) / 5.89 \approx 25.1\%$，$(3.36 - 2.89) / 3.36 \approx 14.1\%$，即启用 RTS/CTS 后吞吐量的相对损失。为什么速率越高损失越大？因为 RTS、CTS、ACK 这些控制短帧必须以所有网卡都听得懂的最低基础速率（1 Mbps）发送，还各带一段低速前导码，成本近乎固定；数据率从 1 Mbps 提到 11 Mbps 时，分子不变、分母变小，开销占比自然从 4.0% 涨到 25.1%。所以现实中的 802.11 默认关闭 RTS/CTS（或把触发阈值设得很高），只在帧很长、或确知存在隐藏终端时才启用——让短帧白白支付四分之一的代价显然不划算。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这套机制并不是 802.11 的发明。1975 年 Tobagi 与 Kleinrock 研究分组无线网时提出 busy tone（忙音）思路：接收方在一个独立频道上发忙音，告诉所有邻居"我正在收帧"；1990 年 Phil Karn 为业余分组无线电设计了 MACA（Multiple Access with Collision Avoidance，冲突避免多路访问）协议，去掉额外的频道，改用 RTS/CTS 一对短帧完成同样的信息广播——Karn 论文标题里的 "collision avoidance"，正是本讲 CSMA/CA 中 "CA" 的出处。802.11 后来把 MACA 的握手与 CSMA 的退避、链路层确认缝合起来，才得到今天的完整协议。把这条线连起来看：每一次改进，都是在"获取接收方视角的信息"与"支付控制开销"之间重新定价，本讲的表格就是这个定价过程的账本。
