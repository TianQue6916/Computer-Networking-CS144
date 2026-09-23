# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Flow Control：Stop-and-Wait（流量控制：停等协议）

> <span style="color:#7f8c8d;">Source: 慕课/Unit2-Transport/flow-stop-and-wait.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1：Flow Control（流量控制）· Stop-and-Wait（停等）</span>

本讲义共 10 页，主题是流量控制的第一种基本方法——停等协议（stop-and-wait）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 2：Problem（问题）</span>

- 发送方 A：每秒可发送 500,000 个分组（packet，分组）
- 接收方 B：每秒只能接收 200,000 个分组

<span style="color:#7f8c8d;">（Slide 2/10）</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 先把这两个数字用你熟悉的语言算一遍。A 的发送速率是 B 的接收速率的 2.5 倍，如果不加任何约束，B 的接收缓冲区（buffer，缓冲区）将以每秒 30 万个分组的速度溢出。停等协议的思路是"釜底抽薪"：与其让发送方猜收多快，不如让它一次只发一个、等确认再发下一个——这样速率天然被压到接收方水平。但代价可以用概率论精确核算：设单次传输成功率 $1-p$（Bertsekas 第 3 章的 Bernoulli 模型），则每个分组需要的平均传输次数是几何分布的期望 $E[N] = \frac{1}{1-p}$，成功交付的平均时延被放大为 $\frac{T + \mathrm{RTT}}{1-p}$，其中 $T$ 是发送一个分组的传输时延。更重要的是下一讲会给出的对比：链路速率 10 Mbps、RTT 50 ms 时，带宽时延积高达 $5 \times 10^5$ bit，停等每往返只送出一个分组，管道几乎永远空着。CSAPP 第 4 章讲流水线时你见过同样的取舍：停等就是深度为 1 的流水线，吞吐量被时延彻底锁死。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 3：Flow Control（流量控制）</span>

<span style="color:#00838f;">**定义核心句：不要发送超过接收方处理能力的分组；由接收方向发送方提供反馈。**</span>（Don't send more packets than receiver can process; receiver gives sender feedback.）

两种基本方法（two basic approaches）：

1. 停等（stop and wait）——本视频
2. 滑动窗口（sliding window）——下一视频

<span style="color:#7f8c8d;">（Slide 3/10）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 4：Finite State Machines（有限状态机）</span>

本页是 FSM（finite state machine，有限状态机）图例页：圆圈表示状态（state），箭头表示状态转移（state transition），每条边上标注两行——引发转移的事件（event causing state transition）与转移时采取的动作（actions taken on state transition）。图中画出 State 1、State 2、State 3 三个状态及 `event/action` 形式的转移标注。

<span style="color:#7f8c8d;">（Slide 4/10）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 5：Stop and Wait（停等协议）</span>

- 任意时刻**至多有一个分组在途**（at most one packet in flight）
- 发送方发送一个分组
- 接收方收到数据后回送确认分组（acknowledgment packet，确认分组）
- 发送方收到确认后，才发送新数据
- 超时（timeout，超时）时，发送方重发当前数据

<span style="color:#7f8c8d;">（Slide 5/10）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 6：Receiver FSM（接收方状态机）</span>

接收方只有一个状态 **Wait for Packets（等待分组）**，自环转移标注为：

- 事件：`receive data`（收到数据）→ 动作：`send ack`（发送确认）+ `deliver data`（上交数据给应用层）

即接收方的全部行为：收到数据、回确认、把数据交给上层，然后继续等待。

<span style="color:#7f8c8d;">（Slide 6/10）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 7：Stop and Wait FSM（发送方状态机）</span>

发送方有两个状态：

| 状态 | 事件 → 动作 |
|---|---|
| **Wait for Data（等待数据）** | `software calls send`（应用层调用 send）→ `send packet`（发送分组）；`receive ack packet` → `-`（无动作） |
| **Wait for ACK（等待确认）** | `timeout`（超时）→ `resend`（重发） |

<span style="color:#7f8c8d;">（Slide 7/10）</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这两张 FSM 图值得和 2-6 节的有限状态机讲法互相印证：圆圈是状态，箭头上写"事件/动作"。发送方只有两个状态，恰恰刻画了"至多一个分组在途"的全部语义——协议的复杂度直接反映在状态数上。这套模型与 CSAPP 第 11 章写 echo 服务器时的阻塞语义是同构的：你调用 `write()` 之后若不同时读回对端响应，进程就停在原地；这与发送方停在 Wait for ACK 状态、等待 `receive ack packet` 事件，是同一件事的软件视角与协议视角。6.042J 讲状态机时强调"状态是对未来行为的完整概括"——停等发送方只需记住"上一包是否已被确认"这一个比特（bit），就足以决定下一个动作。这也解释了为什么下一页敢用 1-bit 序列号：状态空间越小，协议正确性越容易验证，这正是 FSM 方法论的价值所在。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 8：Example Executions（运行示例）</span>

本页用三组 sender（发送方）/receiver（接收方）时序图对比三种情形，横轴为时间，`data` 为数据分组、`ACK` 为确认分组：

1. **No Loss（无丢包）**：`data → ACK → data → ACK`，交替进行，一切正常。
2. **Data Loss（数据丢失）**：第一个 `data` 在网络中丢失（图中以 ✗ 标出），发送方等不到 ACK，`timeout` 后重发同一数据，成功。
3. **ACK Loss（确认丢失）**：`data` 送达、接收方回的 `ACK` 丢失，发送方超时后重发数据——问题来了：接收方已经交付过这份数据，重传的这份是"新数据"还是"旧数据的重放"？这正是下一页要解决的重复（duplicates）问题。

<span style="color:#7f8c8d;">（Slide 8/10）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 9：Duplicates（重复分组）</span>

- 在数据分组和确认分组中都使用 **1-bit 计数器（1-bit counter）**：
  - 接收方可据此判断收到的是新数据还是重复数据
- 一些简化假设（simplifying assumptions）：
  - 网络不会复制分组（network does not duplicate packets）
  - 分组不会被延迟超过多个超时周期（packets not delayed multiple timeouts）

<span style="color:#7f8c8d;">（Slide 9/10）</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 为什么丢一个 ACK 就非得引入序号？做一个思想实验：接收方发出 ACK 后它丢了，发送方超时重传"同一包"数据——接收方收到的字节内容与上次完全相同，若没有任何标记，它无法区分这是新数据还是旧数据的重放。解决办法是让相邻两次传输交替携带 0 和 1，接收方比较计数器即可分辨重复；这个方案就是著名的交替比特协议（alternating bit protocol），由 Bartlett、Scantlebury 与 Wilkinson 于 1970 年前后在一条不可靠拨号链路上提出。注意幻灯片明确列出的两条简化假设——网络不复制分组、分组延迟不超过多个超时周期——正是为了让"1-bit 空间"勉强够用：一旦允许任意延迟与复制，新旧分组可能在同一时刻在途，1 bit 无法消歧。这为滑动窗口一节"序列号空间需要 $SWS + RWS$ 个取值"的结论埋下伏笔：窗口越大，在途分组的"身份"越多，序列号空间必须随之扩大。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 10：ACK Delay（确认延迟示例）</span>

本页时序图演示 1-bit 计数器的工作方式：发送方与接收方交替执行 `data 0 → ACK 0 → data 1 → ACK 1 → data 0 → ACK 0`，其中出现若干次超时重传——数据与确认的计数值（0、1 交替）使得即便发生超时重发，接收方也能识别出重复并正确接收后续数据。

<span style="color:#7f8c8d;">（Slide 10/10）</span>

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 2-7《Stop and wait》的配套 slides；下一讲 Flow Control II 介绍滑动窗口。</span>
