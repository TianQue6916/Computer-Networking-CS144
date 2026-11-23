# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Flow Control：Stop-and-Wait（流量控制：停等协议）

> <span style="color:#7f8c8d;">Source: 慕课/Unit2-Transport/flow-stop-and-wait.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1：Flow Control（流量控制）· Stop-and-Wait（停等）</span>

本讲义共 10 页，主题是 flow control（流量控制）的第一种基本方法——stop-and-wait（停等协议）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 2：Problem（问题）</span>

- 发送方 A（sender A）：每秒可发送 500,000 个 packet（分组）
- 接收方 B（receiver B）：每秒只能接收 200,000 个 packet（分组）

<span style="color:#7f8c8d;">（Slide 2/10）</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 先把这两个数字用你熟悉的语言算一遍。发送方 A 的发送速率是接收方 B 的接收速率的 2.5 倍，如果不加任何约束，接收方 B 的 buffer（缓冲区）将以每秒 30 万个 packet（分组）的速度 overflow（溢出）。stop-and-wait（停等协议）的思路是“釜底抽薪”：与其让 sender（发送方）猜 receiver（接收方）收多快，不如让它一次只发一个 packet（分组）、等 ACK（确认）再发下一个——这样速率天然被压到 receiver（接收方）的水平。但代价可以用概率论精确核算：设单次传输成功率 $1-p$（这是 Bernoulli（伯努利）模型，即每次传输独立地以概率 $1-p$ 成功，见 Bertsekas《Data Networks》第 3 章），则每个 packet（分组）需要的平均传输次数是 geometric distribution（几何分布，即独立重复试验直到首次成功所需的次数）的期望 $E[N] = \frac{1}{1-p}$，成功交付的平均时延被放大为 $\frac{T + \mathrm{RTT}}{1-p}$，其中 $T$ 是发送一个 packet（分组）的传输时延，RTT（round-trip time，往返时延）是一次往返所需的时间。更重要的是下一讲会给出的对比：链路速率 10 Mbps、RTT（往返时延）50 ms 时，bandwidth-delay product（带宽时延积，等于链路速率 × 往返时延，表示一条链路上能同时“在途”的比特数上限）高达 $5 \times 10^5$ bit，stop-and-wait（停等）每往返只送出一个 packet（分组），pipeline（流水线）几乎永远空着。CSAPP（《深入理解计算机系统》，Computer Systems: A Programmer's Perspective）第 4 章讲流水线时你见过同样的取舍：stop-and-wait（停等）就是深度为 1 的流水线，吞吐量被时延彻底锁死。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 3：Flow Control（流量控制）</span>

<span style="color:#00838f;">**定义核心句：不要发送超过 receiver（接收方）处理能力的 packet（分组）；由 receiver（接收方）向 sender（发送方）提供 feedback（反馈）。**</span>

两种基本方法（two basic approaches）：

1. stop-and-wait（停等）——本视频
2. sliding window（滑动窗口）——下一视频

<span style="color:#7f8c8d;">（Slide 3/10）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 4：Finite State Machines（有限状态机）</span>

本页是 FSM（finite state machine，有限状态机）图例页：圆圈表示 state（状态），箭头表示 state transition（状态转移），每条边上标注两行——引发 transition（转移）的 event（事件）与 transition（转移）时采取的 action（动作）。图中画出 State 1、State 2、State 3 三个 state（状态）及 `event/action` 形式的转移标注。

<span style="color:#7f8c8d;">（Slide 4/10）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 5：Stop and Wait（停等协议）</span>

- 任意时刻**至多有一个 packet（分组）in flight（在途，即已发出但尚未被确认）**
- 发送方（sender）发送一个 packet（分组）
- 接收方（receiver）收到数据后回送 acknowledgment packet（确认分组）
- 发送方（sender）收到 ACK（确认）后，才发送新数据
- 超时（timeout）时，发送方（sender）重发当前数据

<span style="color:#7f8c8d;">（Slide 5/10）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 6：Receiver FSM（接收方状态机）</span>

接收方（receiver）只有一个 state（状态）**Wait for Packets（等待分组）**，自环 transition（转移）标注为：

- 事件（event）：`receive data`（收到数据）→ 动作（action）：`send ack`（发送确认）+ `deliver data`（上交数据给应用层）

即接收方（receiver）的全部行为：收到数据、回 ACK（确认）、把数据交给上层，然后继续等待。

<span style="color:#7f8c8d;">（Slide 6/10）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 7：Stop and Wait FSM（发送方状态机）</span>

发送方（sender）有两个 state（状态）：

| state（状态） | event（事件）→ action（动作） |
|---|---|
| **Wait for Data（等待数据）** | `software calls send`（应用层调用 send）→ `send packet`（发送分组）；`receive ack packet`（收到确认分组）→ `-`（无动作） |
| **Wait for ACK（等待确认）** | `timeout`（超时）→ `resend`（重发） |

<span style="color:#7f8c8d;">（Slide 7/10）</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这两张 FSM（有限状态机）图值得和 2-6 节的 FSM（有限状态机）讲法互相印证：圆圈是 state（状态），箭头上写“event（事件）/action（动作）”。sender（发送方）只有两个 state（状态），恰恰刻画了“至多一个 packet（分组）in flight（在途）”的全部语义——协议的复杂度直接反映在 state（状态）数上。这套模型与 CSAPP（《深入理解计算机系统》，Computer Systems: A Programmer's Perspective）第 11 章写 echo（回显）服务器时的阻塞语义是同构的：你调用 `write()` 之后若不同时读回对端响应，进程就停在原地；这与 sender（发送方）停在 Wait for ACK（等待确认）state（状态）、等待 `receive ack packet`（收到确认分组）event（事件），是同一件事的软件视角与协议视角。6.042J（MIT 的《计算机科学中的数学》课程）讲 state machine（状态机）时强调“state（状态）是对未来行为的完整概括”——stop-and-wait（停等）的 sender（发送方）只需记住“上一包是否已被确认”这一个 bit（比特），就足以决定下一个 action（动作）。这也解释了为什么下一页敢用 1-bit sequence number（序列号）：state（状态）空间越小，协议正确性越容易验证，这正是 FSM（有限状态机）方法论的价值所在。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 8：Example Executions（运行示例）</span>

本页用三组 sender（发送方）/receiver（接收方）时序图对比三种情形，横轴为时间，`data` 为数据分组、`ACK` 为确认分组：

1. **无丢包（No Loss）**：`data → ACK → data → ACK`，交替进行，一切正常。
2. **数据丢失（Data Loss）**：第一个 `data` 在网络中丢失（图中以 ✗ 标出），sender（发送方）等不到 ACK（确认），`timeout`（超时）后重发同一数据，成功。
3. **确认丢失（ACK Loss）**：`data` 送达、receiver（接收方）回的 `ACK`（确认）丢失，sender（发送方）超时后重发数据——问题来了：receiver（接收方）已经交付过这份数据，重传的这份是“新数据”还是“旧数据的重放”？这正是下一页要解决的 duplicate（重复）问题。

<span style="color:#7f8c8d;">（Slide 8/10）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 9：Duplicates（重复分组）</span>

- 在 data packet（数据分组）和 acknowledgment packet（确认分组）中都使用 **1-bit counter（计数器）**：
  - 接收方（receiver）可据此判断收到的是新数据还是 duplicate（重复）数据
- 一些 simplifying assumptions（简化假设）：
  - 网络不会 duplicate（复制）packet（分组）
  - 分组（packet）不会被 delayed（延迟）超过多个 timeout（超时）周期

<span style="color:#7f8c8d;">（Slide 9/10）</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 为什么丢一个 ACK（确认）就非得引入 sequence number（序列号）？做一个思想实验：receiver（接收方）发出 ACK（确认）后它丢了，sender（发送方）超时重传“同一包”数据——receiver（接收方）收到的字节内容与上次完全相同，若没有任何标记，它无法区分这是新数据还是旧数据的重放。解决办法是让相邻两次传输交替携带 0 和 1，receiver（接收方）比较 counter（计数器）即可分辨 duplicate（重复）；这个方案就是著名的 alternating bit protocol（交替比特协议），由 Bartlett、Scantlebury 与 Wilkinson 于 1970 年前后在一条不可靠拨号链路上提出。注意幻灯片明确列出的两条简化假设——网络不复制分组、分组延迟不超过多个超时周期——正是为了让“1-bit 空间”勉强够用：一旦允许任意延迟与复制，新旧分组可能在同一时刻 in flight（在途），1 bit（比特）无法消歧。这为 sliding window（滑动窗口）一节“sequence number（序列号）空间需要 SWS（sender window size，发送方窗口大小）与 RWS（receiver window size，接收方窗口大小）之和个取值”的结论埋下伏笔：窗口越大，在途分组的“身份”越多，sequence number（序列号）空间必须随之扩大。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 10：ACK Delay（确认延迟示例）</span>

本页时序图演示 1-bit counter（计数器）的工作方式：sender（发送方）与 receiver（接收方）交替执行 `data 0 → ACK 0 → data 1 → ACK 1 → data 0 → ACK 0`，其中出现若干次 timeout（超时）重传——数据与确认的计数值（0、1 交替）使得即便发生超时重发，receiver（接收方）也能识别出 duplicate（重复）并正确接收后续数据。

<span style="color:#7f8c8d;">（Slide 10/10）</span>

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 2-7《Stop and wait》的配套 slides；下一讲 Flow Control II 介绍滑动窗口。</span>
