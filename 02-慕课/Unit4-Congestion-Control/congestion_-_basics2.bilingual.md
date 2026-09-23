# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Congestion — Basic Approaches（拥塞——基本途径）

> <span style="color:#7f8c8d;">Source: 慕课/Unit4-Congestion-Control/congestion_-_basics2.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1：封面</span>

CS144：计算机网络导论。**Congestion（拥塞）：Basic Approaches（基本途径）**。Nick McKeown，斯坦福大学电气工程与计算机科学教授。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 2：Outline（大纲）</span>

- 把拥塞控制放在哪里？
  - 例子：在每个路由器上做公平排队（Fair Queueing，公平排队）
  - 在网络中（in the network）
  - 在终端主机（at the end host）
- 滑动窗口（sliding window，滑动窗口）与 AIMD

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 3：Example: FQ at every router（例子：在每个路由器上做公平排队）</span>

图示：路由器为每条流（per-flow，每流）维护独立的队列（Per-flow queues，每流队列），由一个公平排队调度器（FQ scheduler，公平排队调度器）轮流从各队列取分组发送。上一讲的"公平共享"目标如果放在网络内部实现，这就是最直接的做法：把第 3 单元学过的 WFQ（加权公平排队，WFQ）调度器装进每台路由器，各流自然各得其份。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 4：Network-based（基于网络的做法）</span>

纯图示 slide：在网络内部解决拥塞——路由器负责排队、调度、标记或丢弃分组，向发送方反馈拥塞状态。这种方案把智能放在网络里。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 5：End-host based（基于终端主机的做法）</span>

纯图示 slide：与上一页相对——网络里的路由器尽量保持简单（尽力而为转发），由端主机自己探测网络状态、调节发送速率。这种方案把智能放在网络边缘。TCP 选择的正是这一条路。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "把公平性装进每一台路由器"还是"让端主机自己收敛"，是本讲最重要的一处分岔。前者（FQ/WFQ 路由器）看起来更彻底，代价是每台路由器都要为每条流维护状态并做复杂的调度——回顾第 1 单元的端到端原则（end-to-end principle，端到端原则）：网络核心应当保持简单，复杂功能放在边缘。互联网最终选了后者，不是因为它更优雅，而是因为它可部署：只要改端主机的 TCP 代码，几十万台路由器一台都不用动。这个决策的历史回响很深——后来网络内拥塞反馈的各种尝试（如 ECN 显式拥塞通知）都因为同样的部署难题而推进缓慢。做系统设计时，"谁需要改代码"往往比"哪种方案在纸面上更优"更能决定胜负。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 6：TCP Congestion Control（TCP 拥塞控制）</span>

TCP 在终端主机上实现拥塞控制：

- 对端主机可观测到的事件做出反应（例如丢包 packet loss，丢包）；
- 利用了 TCP 本来就有的、用于流量控制的滑动窗口；
- 试图搞清楚：**同一时刻在网络里安全地保有多少个"在途"（outstanding，在途未确认）分组**。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 7：Sliding Window（滑动窗口）</span>

图示（字节流上的窗口划分，从左到右）：

- 已确认数据（Data ACK'd，已确认数据）；
- **窗口大小（Window Size，窗口大小）**覆盖的范围；
  - 已发送但未确认的在途数据（Outstanding Data sent, but un-ack'd，已发送未确认数据）；
  - 允许立即发送的数据（Data OK to send，可发送数据）；
- 尚不允许发送的数据（Data not OK to send yet，暂不可发送数据）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 8：TCP Sliding Window（TCP 滑动窗口与"管道"）</span>

设瓶颈链路速率为 $R$，往返时延为 $RTT$（round-trip time，往返时延），窗口大小为 $W$：

1. $R \times RTT > W$：窗口太小，"管道"填不满——发送方每发出一个窗口就得停下来等确认，链路出现空闲；
2. $R \times RTT = W$：窗口恰好填满管道，发送方可以连续发送。

图示：A、B 两端之间多个 ACK 沿反方向回流，窗口沿字节流向前滑动。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 9：TCP Congestion Control（TCP 拥塞控制：发送窗口的取值）</span>

TCP 通过调节窗口大小来改变网络中在途分组的数量：

<span style="color:#00838f;">发送窗口 $=$ $\min\{$ 通告窗口（Advertised window，通告窗口），拥塞窗口（Congestion Window，拥塞窗口）$\}$</span>

图示：发送方（Transmitter，发送方）与接收方（Receiver，接收方）——其中拥塞窗口记作 **cwnd**。

**那么，cwnd 的取值该如何决定？**

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这里有一个 CSAPP 视角的关键区分，很多初学者会混：拥塞窗口 **cwnd 不是 TCP 首部里的字段**。首部里的 window 字段（第 2 单元讲过）承载的是接收方通告的流量控制窗口，它要随报文段发给对方；而 cwnd 是**发送方内核里 TCP 套接字数据结构（如 Linux 的 `tcp_sock`）中的一个私有变量**，对方根本看不见，纯粹由发送方根据自己观测到的确认与超时悄悄维护。你在 CSAPP 第 11 章写的 echo 服务器只调用了 socket API，从没接触过 cwnd，因为它完全活在内核协议栈里。于是"能发多少"就是两个约束取小值：对方能收多少（rwnd，来自网络里的对端）与网络能传多少（cwnd，来自自己的推测）——流量控制保护接收端，拥塞控制保护网络，二者在发送窗口 $= \min\{\text{rwnd},\ \text{cwnd}\}$ 处汇合。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 10：AIMD（加性增、乘性减）</span>

AIMD：Additive Increase, Multiplicative Decrease（加性增、乘性减）：

- 若分组被正确接收：<span style="color:#00838f;">$W \to W + 1$</span>（每个 RTT 窗口加 1）；
- 若有分组被丢弃：<span style="color:#00838f;">$W \to \dfrac{W}{2}$</span>（窗口减半）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 11：Leads to the AIMD "sawtooth"（由此得到 AIMD"锯齿波"）</span>

图示：cwnd 随时间 $t$ 变化的曲线——线性爬升，到达丢包点（Drops，丢包处）后骤降一半（halved，减半），周而复始，形成锯齿波（sawtooth，锯齿波）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 为什么偏偏是"加性增、乘性减"这个不对称组合？先用 6.042J 的不变量思想看公平性：设两条流速率分别为 $a$、$b$，加性增把 $(a, b)$ 变成 $(a+1, b+1)$，二者之差 $a - b$ 不变；乘性减把它变成 $(a/2, b/2)$，二者之**比** $a/b$ 不变。增若不加在慢流头上就永远追不平，减若不同时砍快流也拦不住贪婪——AIMD 的妙处在于：增保持差距、减压缩差距。具体地，每轮"减半再增 1"之后，差距变为 $a/2 - b/2 + 1$（按与差同向计），当 $a > b$ 时这一项小于原来的 $a - b$，即每一轮锯齿都在把大流拉回小流，最终收敛到公平线——下一讲的 Chiu-Jain 相图会把这个论证画成几何图形。从概率论角度再补一刀：乘性减让窗口快速离开危险区（响应快、防崩溃），加性增让探测缓慢而谨慎（避免自己制造拥塞），"快退慢进"正是让窗口序列形成稳定随机游走的条件。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 12：Summary（总结）</span>

- 抉择：在网络中，还是在终端主机？**TCP 从终端主机控制拥塞。**
  - 对端主机可观测的事件做出反应（例如丢包）；
  - 利用了 TCP 用于流量控制的滑动窗口；
  - 试图搞清楚同一时刻在网络里安全地保有多少个在途分组；
  - 依据 AIMD 调节窗口大小。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 13：结束页</span>

本讲完。CS144, Stanford University。

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 4-2《Congestion Control - Basics 2》的配套 slides；AIMD 单流动力学见 4-3，多流与吞吐量公式见 4-4。</span>
