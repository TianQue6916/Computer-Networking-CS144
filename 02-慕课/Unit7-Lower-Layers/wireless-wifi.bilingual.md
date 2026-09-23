# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

> <span style="color:#7f8c8d;">Source: src/慕课/Unit7-Lower-Layers/wireless-wifi.txt · CS144 Fall 2019 · 行内双语翻译版</span>

## <span style="color:#2471a3;">**[lecture]**</span> <span style="color:#c0392b">802.11 Format and Overhead（802.11 的帧格式与开销）</span>

原文 Slide 1 为标题页：**802.11 Format and Overhead**（802.11 的格式与开销）。本讲拆开 802.11（即 WiFi）的 PHY 与 MAC 帧结构，看它如何用"以时间计价"的虚拟载波侦听把多种物理层统一起来，最后提醒：别被 600 Mbps 的标称速率骗了。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 2 · Example 802.11n（例：802.11n 的 MCS 速率表）</span>

以 802.11n 为例，MCS Index（MCS = modulation and coding scheme，调制与编码方案索引）从 0 到 7（均为单空间流）：

| MCS Index（索引） | Spatial Streams（空间流） | Modulation（调制） | Coding（编码率） | Data Rate（数据率，Mbps）<br>20 MHz，800 ns GI | 20 MHz，400 ns GI | 40 MHz，800 ns GI | 40 MHz，400 ns GI |
|---|---|---|---|---|---|---|---|
| 0 | 1 | BPSK | 1/2 | 6.5 | 7.2 | 13.5 | 15.0 |
| 1 | 1 | QPSK | 1/2 | 13.0 | 14.4 | 27.0 | 30.0 |
| 2 | 1 | QPSK | 3/4 | 19.5 | 21.7 | 40.5 | 45.0 |
| 3 | 1 | 16-QAM | 1/2 | 26 | 28.9 | 54.0 | 60.0 |
| 4 | 1 | 16-QAM | 3/4 | 39 | 43.3 | 81.0 | 90.0 |
| 5 | 1 | 64-QAM | 2/3 | 52 | 57.8 | 108.0 | 120.0 |
| 6 | 1 | 64-QAM | 3/4 | 58.5 | 65.0 | 121.5 | 135.0 |
| 7 | 1 | 64-QAM | 5/6 | 65 | 72.2 | 135.0 | 150.0 |

其中 GI = guard interval（保护间隔），BPSK（二进制相移键控）、QPSK（正交相移键控）、$n$-QAM（quadrature amplitude modulation，正交幅度调制）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 教你怎么读这张表。每个符号（symbol）携带的比特数：BPSK 为 1，QPSK 为 2，16-QAM 为 4，64-QAM 为 6——QAM 的数字就是星座图上的点数，$2^n$ 个点塞进同一幅图，点距越小越容易被噪声挤错位。编码率（coding）1/2 表示每 2 个比特里有 1 个是纠错冗余，5/6 则冗余很少。于是数据率大致等于：符号率 × 每符号比特数 × 编码率 × 空间流数，再乘上信道宽度（40 MHz 比 20 MHz 翻倍）与保护间隔占比（400 ns GI 比 800 ns GI 约多 11%）的修正。规律一目了然：速率越高的行，对信噪比（SNR，signal-to-noise ratio）要求越苛刻——这是一场"速率换鲁棒性"的交换。接收质量好就升档、变差就降档，正是 802.11 速率自适应（rate adaptation）机制的工作空间，也呼应前几讲"冲突还是低信噪比"的困境：这一列选择本质上是在为信道质量下注。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 3 · 802.11 (WiFi)（基本挑战）</span>

802.11（WiFi）的基本挑战：support wide range and extensible bitrates（支持宽范围且可扩展的比特率）。

也就是说，同一套 MAC 既要在 1 Mbps 的老网卡上工作，也要跟得上 600 Mbps 的新物理层，还得为未来留出扩展空间。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 4 · 802.11 (WiFi)（基本挑战与协议栈位置）</span>

同一挑战的完整表述：support a wide range of and extensible bitrates（支持宽范围且可扩展的比特率）。

图示：OSI 七层参考模型（Application 应用层、Presentation 表示层、Session 会话层、Transport 传输层、Network 网络层、Link 链路层、Physical 物理层），802.11 只占据最底下的两层——link（链路层）与 physical（物理层）。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 5 · 802.11b PHY（802.11b 的物理层帧结构）</span>

802.11b 的 PHY（物理层）帧头结构（PLCP 前导码与首部）：

- 128 bit 的 sync（同步字段），由 "1111...0000" 的交替图样填充；
- 之后依次为：SFD（start frame delimiter，帧起始定界符）16 bit、signal（速率指示）8 bit、service（业务字段）8 bit、length（长度）16 bit、CRC（循环冗余校验）16 bit；
- 这些字段之后才接 link frame（链路层帧）；
- 整个 PHY 前导部分会被加扰处理（scrambled by PHY）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 为什么前导码要做得这么"笨"——128 位同步加一堆固定字段？因为它是所有接收方都必须听得懂的"公共语言"：无论后面数据部分跑 1 Mbps 还是 600 Mbps，前导码永远按最低速率、最鲁棒的方式发送。先说慢话，再说快话。其中 signal 字段就是"换挡提示"：它告诉接收方"接下来的数据部分用什么速率解"，service 与 length 则帮接收方算出数据部分要持续多少微秒。128 位同步图样给接收机留足了时钟恢复（clock recovery，从信号里还原出发送方的比特节拍）与自动增益调整的时间。这套设计与 TCP 三次握手是同一种工程直觉：任何高速通信之前，都需要一段双方保证能听懂的慢速协商。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 6 · 802.11 MAC（802.11 的 MAC 帧格式）</span>

802.11 MAC 帧按字节的布局：

- frame control（帧控制）2 字节；
- duration（持续时间）2 字节；
- addr 1（地址 1）6 字节、addr 2（地址 2）6 字节、addr 3（地址 3）6 字节；
- seq no.（sequence number，序列号）2 字节；
- addr 4（地址 4）6 字节；
- network data（网络层数据，即有效载荷）；
- FCS（frame check sequence，帧校验序列）4 字节。

对比上一页的 802.11b PHY 前导码（128 bit sync 与 16/8/8/16/16 bit 的 SFD/signal/service/length/CRC 字段）可见：MAC 帧整体被包在 PHY 帧结构里面——这就是封装（encapsulation）在链路层内部的又一次上演。四个地址字段是为无线场景准备的：一帧的收发可能涉及发送方、接收方，以及两端各自的上游/下游节点（比如经接入点转发的帧），有线以太网只需两个地址就够了。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 7 · Virtual Carrier Sense（虚拟载波侦听）</span>

同一 MAC 帧结构，这次聚焦 duration（持续时间）字段：<span style="color:#00838f;">任何收到该帧的节点，读出 duration 里"信道将被占用多少微秒"，据此设置 NAV（network allocation vector，网络分配向量）倒计时——即使它完全解不出帧的数据内容，也能在倒计时归零前保持静默。这就是 virtual carrier sense（虚拟载波侦听）。</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 关键设计是：MAC 的控制信息以**时间**（微秒）而非**字节**为单位。为什么？因为信道争用争的本质是"时间片"而不是"字节量"：同样的字节数，在 1 Mbps 的 PHY 上占用 8 倍于 8 Mbps 的信道时间；如果 duration 写字节数，接收方还得先猜对方的速率再换算，多一步就多一处错。直接写微秒，让"这次发送还要占线多久"变成一个所有邻居（不管用什么 PHY 速率）都能直接使用的数字。这正是 802.11 一个 MAC 帧格式能同时架在 b/g/n 多代物理层之上的前提：MAC 与 PHY 之间的接口被"时间"抽象隔开了。虚拟载波侦听也与 RTS/CTS 一讲呼应——CTS 里的 duration 同样用这个机制让隐藏终端闭嘴，物理层侦听听不见的，靠"听懂控制字段"来补。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 8 · Virtualizing a Link（虚拟化一条链路）</span>

Slide 只展示 MAC 帧本身：frame control、duration、addr 1–3、seq no.、addr 4、data、FCS。

要点：借 duration 字段与 NAV 倒计时，一帧交换期间所有邻居都表现得像"这条链路此刻独占"——802.11 在一片共享的空气里，虚拟出了一条点对点链路（virtual link）。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 9 · 802.11 Overhead（802.11 的开销）</span>

图示：OSI 协议栈旁边摆着 1 Mbps 与 600 Mbps 两个速率的对比，以及同一套 PHY 前导码结构（128 bit sync、SFD/signal/service/length/CRC 字段、link frame、scrambled by PHY）。

信息点：无论数据部分跑到多高的速率（600 Mbps），前导码与 PLCP 首部始终按最低的 1 Mbps 发送——这部分是每个帧都要付的固定开销。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "别被 600 Mbps 骗了"这句警告，值得算一笔具体的账。有效吞吐量（throughput）不等于标称速率，实际等于标称速率乘以一个小于 1 的效率因子，而效率被三样东西吃掉：每帧一次的低速前导码、帧间的退避与帧间隙、以及 PHY/MAC 各层首部。前导码以 1 Mbps 发是"向后兼容"的代价——所有老网卡必须听得懂帧头，这个代价不随技术进步而缩小，反而随着数据率提高变得**相对**更重。这与你刚在 RTS/CTS 一讲看到的表格是同一个现象：开销固定、速率越高、占比越大。这也在提醒带宽（bandwidth）与吞吐量（throughput）的区分——前者是链路的物理上限，后者是扣除一切协议成本后真正落到应用手里的部分，两者的差距在无线链路上格外可观。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 10 · 802.11 Summary（802.11 小结）</span>

- 基本的 MAC 帧格式，要能架在多种物理层（physical layers）之上；
- 需要向后兼容（backwards compatibility）——因此用时间（time），而不是字节（bytes）来计量；
- MAC 控制（虚拟载波侦听）以 duration（持续时间）表述——由此把链路虚拟化；
- 在帧中嵌入额外的地址（addr 1–4）；
- 别被 600 Mbps 迷惑！
