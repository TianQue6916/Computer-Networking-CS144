# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> TCP Congestion Control I — Slow start, congestion avoidance, triple duplicate acks（TCP 拥塞控制 I——慢启动、拥塞避免、三重重复确认）

> <span style="color:#7f8c8d;">Source: 慕课/Unit4-Congestion-Control/tcp1-tahoe-slow-start-notes.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1：封面与开场</span>

TCP 拥塞控制 I：慢启动（slow start，慢启动）、拥塞避免（congestion avoidance，拥塞避免）、三重重复确认（triple duplicate acks，三重重复确认）。

**讲稿**：在接下来的三讲里，我将解释 TCP 的拥塞控制是如何工作的。概括地说，TCP 使用加性增、乘性减（AIMD），但实现细节非常多。TCP 被视为网络领域的一项巨大成就：它是一个可靠、高性能的传输层，能在跨度极大的网络环境中良好运转。它当然不完美，但从有多少应用依赖它，就能看出它的强大。本讲将解释 TCP 如何用一个简单的有限状态机（finite state machine，有限状态机）来控制它在网络中的在途分组数量——这个状态机实现了 AIMD，同时处理连接的启动与严重的网络故障。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 2：Congestion Control Motivation（拥塞控制的动机）</span>

图示：从 San Francisco（旧金山）到 Boston（波士顿）的路径，途经多台路由器。

**讲稿**：回忆一下，拥塞控制是网络协议或系统为了不过载"两个端点之间的网络"而做出的努力。假设我们在旧金山与波士顿之间传输数据，两端的路径上有很多路由器，每台路由器都有自己的队列（queue，队列）与流量负载。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 3：Congestion Control Motivation（续）</span>

**讲稿**：如果我们以路径能支撑的速率以下发送分组，可以看到相当低的丢包率——从旧金山到波士顿的大多数分组都会到达，反方向亦然。但如果我们发送的分组多于路径所能支撑的，它们就会从队列中被丢弃。发送方需要以某种方式检测到分组丢了，并重传（retransmit，重传）它——这既耗时又降低性能。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 4：Congestion Control Motivation（续：一句话定义）</span>

<span style="color:#00838f;">拥塞控制：限制在途数据（outstanding data，在途数据）的量，使其不拥塞网络，从而提升整体性能。</span>

**讲稿**：所以拥塞控制的目的，是控制发送方在网络中保有多少在途分组。目标是"发送尽可能多的分组——但不多于网络所能支撑的"：超发会让队列塞满并丢弃分组。高层面上这看起来简单。之所以困难（我们将会看到），是因为在 TCP 的场景下，发送方对网络内部状态的信息极其有限，只能从这些有限的信号中推断丢包。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 5：TCP and AIMD（TCP 与 AIMD）</span>

- TCP 使用加性增、乘性减（AIMD）；
- 维护一个拥塞窗口（congestion window，cwnd）——对"能发送多少个未确认报文段"的估计；
- 每个 RTT 将拥塞窗口加 1 个报文段（segment，报文段）；
- 检测到丢包时，将拥塞窗口减半（或更多）；
- 关于"为什么"的一点历史（互联网崩溃了）；关于它如何达成并实现 AIMD 的解释。

**讲稿**：基本概括是：在稳态下，TCP 使用 AIMD。它维护一个叫拥塞窗口的变量，指定连接在网络中可以保有多少个未确认报文段。随着发送方收到确认（acknowledgement，确认），这些确认表明报文段已离开网络，于是它可以发送更多。每个往返时延（RTT，往返时延），TCP 把拥塞窗口加 1——这是 AIMD 的加性增。当 TCP 检测到丢包，它把拥塞窗口减半，或在某些情况下直接设为 1 个报文段。到目前为止，我们把拥塞控制当作一个清晰显然的问题、把 AIMD 当作一个简单而高效的算法来介绍。但这并不是一开始就被认识到的。我要讲一段 TCP 如何演化出 AIMD 的历史——这是个精彩的故事：互联网崩溃过、变得不可用了！为了应对这次崩溃，一些研究者提出了 AIMD。我会详细解释它如何工作。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 6：TCP History（TCP 简史）</span>

- 1974：三次握手（three-way handshake，三次握手）；
- 1978：TCP 与 IP 分裂为 TCP/IP；
- 1983：1 月 1 日，ARPAnet 切换到 TCP/IP；
- 1986：互联网开始出现拥塞崩溃（congestion collapse，拥塞崩溃）；
- 1987-8：Van Jacobson 修复 TCP，发表奠基性 TCP 论文（Tahoe）；
- 1990：加入快速恢复（fast recovery，快速恢复）（Reno）。

**讲稿**：这是 TCP 早期历史的简表。70 年代中后期，Vint Cerf 等人开发了用于建立连接的三次握手，并把 TCP 拆分为我们今天熟知的两层：网络层的 IP 与传输层的 TCP。1983 年 1 月 1 日，整个 ARPAnet 切换到 TCP/IP。三年后的 1986 年，互联网开始遭受"拥塞崩溃"：随着 TCP 流开始灌满链路，路由器丢弃分组，而 TCP 对这些被丢分组的反应很糟——它把大部分时间花在重传那些其实已经成功到达的分组上，继续浪费这些饱和链路的容量。这使丢包率居高不下，新报文段频繁被丢，应用看到的速率极低。网络在拼命发送废包——"拥塞崩溃"一词由此而来：网络极度拥塞，应用却看不到任何有效工作量。1987-8 年，Van Jacobson 深入研究了发生的事情并修复了 TCP，发表了那篇奠基性论文，其算法是今天所有 TCP 实现的基础。这个版本的 TCP 称为 TCP Tahoe，以它随附发布的那个 BSD UNIX 版本命名。两年后，TCP 又加入了一些改进，称为 TCP Reno。现代 TCP 为适应现代网速在 Reno 之上增加了更多复杂性，但核心仍是 TCP Reno。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 1986 年的拥塞崩溃是网络史上最值得铭记的一次"系统级翻车"，注意年份：距离 TCP/IP 全面部署（1983）只有三年。崩溃的机理是一个正反馈回路：链路饱和 → 丢包 → 重传挤占带宽 → 更饱和 → 更多丢包，而当时的 TCP 没有任何机制打破这个回路——它只认流量控制（接收方收得下就发）。据后来 Van Jacobson 的测量，LBL 到 UC Berkeley 的一条链路吞吐曾跌到 32 bit/s 量级，有效吞吐几乎归零。修复它的 Van Jacobson 不是 TCP 的原作者，而是一位把测量、模型与工程结合起来的研究者——他的 1988 年论文《Congestion Avoidance and Control》是本单元三讲所有图（Slide 10、14 的散点图）的出处。这段历史给系统学习者的启示：协议设计时没出事不等于设计对了，负载增长会把你从未测试过的运行区域暴露出来——拥塞控制属于那种"网络变大变忙之后才涌现的问题"。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 7：Three Questions（三个问题）</span>

- 什么时候发送**新数据**？
- 什么时候发送**重传数据**？
- 什么时候发送**确认（acknowledgment，确认）**？

**讲稿**：我们可以把 TCP 归结为三个问题：什么时候发送新数据？什么时候重传数据？什么时候发送确认？

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 8：Three Questions（续）</span>

**讲稿**：本讲解释第一个问题的答案：TCP 什么时候发送新数据？后两个问题的答案将在下一讲（RTT 估计与自时钟）中解释。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 9：TCP Pre-Tahoe（Tahoe 之前的 TCP）</span>

- 端点只知道流量控制（flow control，流量控制）窗口大小；
- 连接建立后，立即发送一整个窗口的分组；
- 为每个分组启动一个重传定时器（retransmit timer，重传定时器）；
- 问题：**如果窗口远大于网络所能支撑的怎么办？**

**讲稿**：回忆 TCP 首部有一个"窗口"字段，连接的一方用它告诉对方自己流量控制窗口的大小。TCP 规范要求发送方发送的数据不得超过"最后被确认的字节 + 流量控制窗口"。流量控制保证发送方不发送接收方处理不了的数据。Tahoe 之前的 TCP 在三次握手完成后就立即发送整个窗口的报文段——比如收到 40 KB 的流量控制窗口且恰有 40 KB 要发，就立刻把 40 KB 全发出去，然后为每个分组启动重传定时器；定时器到点而报文段未被确认就重传。确认陆续到来时，只要确认号与窗口字段表明接收方能处理更多数据，发送窗口就前进。**如果窗口远大于网络所能支撑的，这就成了大问题**：假设瓶颈链路只能容纳几个分组的队列，握手一完成发送方就发出 30 多个分组，前几个填满瓶颈队列，其余全部被丢。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 10：TCP in 1986（1986 年的 TCP 实测行为）</span>

图示：纵轴为分组序列号（kB），横轴为发送时间（秒）的散点/折线图（出自 Van Jacobson 与 Karels《Congestion Avoidance and Control》，经许可使用）。

**讲稿**：这正是当年实际观测到的行为。这张图出自确立 TCP 拥塞控制机制的奠基性论文。横轴是时间（秒），纵轴以报文段序列号表示 TCP 正在发送哪些段：向右上方延伸的线意味着 TCP 在发送更多数据；跳落并向右的线意味着重传，因为 TCP 在发送更旧的序列号；纵坐标相同、横坐标不同的两点就是重传。图中直线表示可用带宽，约 20 kb/s——那时的网络慢得多！如果 TCP 表现良好，我们会看到黑线贴合这条灰线、填满可用带宽。但我们看到的不是这样：TCP 一上来就发送大量报文段，远超网络的处理能力；然后等待近一秒，直到超时触发一个又一个重传；窗口前进后又猛发一批，其中又有丢失。这种锯齿状的模式意味着 TCP 几乎每一波突发都在丢包，整条线的斜率远低于 20 kb/s 的容量。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 11：Three Improvements（三项改进）</span>

- 拥塞窗口（congestion window，拥塞窗口）；
- 超时估计（timeout estimation，超时估计）；
- 自时钟（self-clocking，自时钟）。

**讲稿**：TCP Tahoe 为恰当地控制拥塞增加了三项改进：拥塞窗口、更好的超时估计、自时钟。本讲余下部分将逐一介绍。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 12：Three Improvements（续）</span>

**讲稿**：TCP Tahoe 加入的第一项改进，叫做拥塞窗口。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 13：Congestion Window (TCP Tahoe)（拥塞窗口）</span>

- 流量控制窗口只关乎端点；
- 让 TCP 为**网络**估计一个拥塞窗口；
- 发送窗口 $= \min($流量控制窗口，拥塞窗口$)$；
- 把拥塞控制分成两个状态：
  - **慢启动（slow start，慢启动）**：连接启动或分组超时时进入；
  - **拥塞避免（congestion avoidance，拥塞避免）**：稳态运行。

**讲稿**：回忆流量控制关心的是端点能处理什么：TCP 不会发送超过流量控制窗口的数据。但如果端点能处理的比网络能承载的还多呢？流量控制窗口只是发送量的上界；为了性能，也许该发得更少。所以 TCP Tahoe 为网络估计一个称为拥塞窗口的量，发送窗口取二者的最小值——既不超过对方能处理的，也不超过网络能承载的。为了管理这个拥塞窗口，TCP 把拥塞控制分成两个状态：慢启动与拥塞避免。稳态下 TCP 处于拥塞避免状态，遵循 AIMD 策略；连接启动或分组超时时，TCP 进入慢启动状态，不遵循 AIMD 策略。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 再次强调那个 CSAPP 视角的关键区分：拥塞窗口 cwnd **不是 TCP 首部字段**——首部里能随报文段传输的只有接收方通告的流量控制窗口；cwnd 是发送方内核中 TCP 套接字结构（如 Linux 的 `tcp_sock`）里的一个私有变量，由协议栈根据自己观测的确认与超时维护，对端看不见、应用层也摸不着。你在 CSAPP 第 11 章用 `socket()/connect()/accept()` 写客户端服务器时，从没碰过 cwnd，因为它深埋在内核协议栈里。两个窗口在发送方汇合成一条不等式：在途字节 $\leq \min(\text{rwnd},\ \text{cwnd})$。"为网络的状态建模"这个思路也有更深一层的意味：网络不向端点报告任何状态，cwnd 就是端点在头脑中（准确说在内核里）为网络维护的一个**信念变量**，丢包与超时则是更新这个信念的贝叶斯证据——后面 RTT 估计一讲会把这个"从噪声信号估计隐状态"的思想做得更精细。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 14：Slow Start Benefits（慢启动的好处）</span>

- **慢启动**：窗口从最大报文段大小 MSS（Maximum Segment Size，最大报文段大小）起步；每收到一个已确认分组的确认，窗口加一个 MSS；
- 以**指数级**增长探测网络容量；
- "慢"是相对此前的做法而言（对比图中 Van Jacobson 与 Karels 论文的实测散点图）。

**讲稿**：理解拥塞避免与慢启动的方式，是看它们各自把拥塞窗口增加多少。TCP 进入慢启动时，拥塞窗口是 MSS——一个分组。每收到一个**新的**确认（确认了此前未被确认的数据），TCP 把拥塞窗口加 1 个 MSS。慢启动的这条策略意味着拥塞窗口**指数增长**：发送方从 MSS 起步，发 1 个段；收到确认后窗口变 2 个 MSS，发 2 个新段；收到这些段的确认后窗口变 4 个 MSS（每个确认加 1），再发 4 个新段。"慢启动"这个名字看似误导——指数增长可比加性增快多了——但它之所谓"慢"，是与旧 TCP"立刻发一整个窗口"的做法相比。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "每收到一个确认窗口加 1 MSS"怎么会变成指数增长？这里藏着一个漂亮的不变量推理，正是 6.042J 递归思想的用武之地：稳态下发送方的节奏是"收到一个确认，发一个新段"（自时钟，下一讲展开），于是在途分组数恒等于窗口 $W$。一个 RTT 内，$W$ 个在途分组全部确认归来，每个确认带来 +1 MSS 的增量，所以窗口从 $W$ 涨到 $2W$——**每 RTT 翻倍**，即 $W(t) = 2^t \cdot MSS$，几何级数。对比 AIMD 的"每 RTT 加 1"，从 1 MSS 爬到 1000 MSS 慢启动只需约 10 个 RTT，AIMD 却要 1000 个 RTT。这就是为什么要分成两个状态：启动阶段用指数猛冲（尽快逼近容量），接近容量后换加性增小心试探。注意 slide 原文 "Exponentially grow congestion window"（源文本误作 "congestion"的笔误）说的就是这条指数曲线——它会在遇到 ssthresh 阈值或丢包时被截断，下面 FSM 一页把这些规则画成了一张状态机图。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 15：Congestion Avoidance（拥塞避免）</span>

- **慢启动**：每收到一个确认，窗口加 MSS——指数增长；
- **拥塞避免**：每收到一个确认，窗口加 $\dfrac{MSS^2}{\text{cwnd}}$——效果：每个 RTT 增加 1 个 MSS，线性（加性）增长。

**讲稿**：在拥塞避免状态，TCP 增长窗口的速度慢得多，接近 AIMD：每收到一个确认，窗口增加 $MSS^2 / \text{cwnd}$。在无丢包的前提下，这使 TCP 每个 RTT 恰好把拥塞窗口增加 1 个 MSS。我们想要每 RTT 增加 1 个 MSS；此刻在途的有 $\text{cwnd}/MSS$ 个报文段——若有 $N$ 个在途段，每个确认就应为我们想要的增量贡献 $1/N$，即每个确认增加 $MSS \cdot (MSS/\text{cwnd})$。这就是 AIMD 的加性增部分。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> $\text{cwnd} \mathrel{+}= MSS^2/\text{cwnd}$ 这个写法乍看古怪，其实是一道简单的除法分配题：目标是"每 RTT 增加一个 MSS"，而一个 RTT 内会回来 $\text{cwnd}/MSS$ 个确认（在途段数），把总增量 MSS 均摊到每个确认头上，就是 $MSS \div \frac{\text{cwnd}}{MSS} = \frac{MSS^2}{\text{cwnd}}$。这解释了为什么内核实现里累加的是一个小数增量（并配合字节计数舍入），而不是整数段数——整数实现会让小窗口时每 RTT 的增量四舍五入成 0 或 2，破坏线性斜率。还有一个隐含的不变量：该增量公式让"每确认增量 × 每轮确认数 = MSS"恒成立，与窗口大小无关——这正是"线性增长斜率与窗口无关"的代数保证。你在 CLRS 做摊还分析（amortized analysis，摊还分析）时见过同款思路：把一次大操作的成本摊到多个小操作上，保证每个操作的摊还成本恒定。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 16：State Transitions（状态转移）</span>

- **两个目标**：用慢启动快速找到网络容量；接近容量时，用拥塞避免非常小心地探测；
- **三个信号**：确认不断增多——传输顺利；重复确认（duplicate acknowledgments，重复确认）——有分组丢失/延迟；超时（timeout，超时）——出了大问题。

**讲稿**：我们有这两个状态。第一个，慢启动，让 TCP 快速找到可用网络容量——例如网络能支撑 40 个分组的拥塞窗口时，等 40 个 RTT 才爬到这个值太慢了。但接近网络容量时，我们想用拥塞避免的加性增更小心地探测。那么如何在两个状态间转移？TCP 有三个信号可用：确认不断增多说明传输顺利；由于 TCP 使用累积确认（cumulative acknowledgments，累积确认），重复确认意味着某个报文段丢失或延迟、但其后的段仍在到达；而超时则说明出了大问题。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 17：TCP Tahoe FSM（TCP Tahoe 状态机）</span>

图示：两状态状态机（慢启动 ⇄ 拥塞避免）：

- 慢启动中：收到 ack → $\text{cwnd} \mathrel{+}= MSS$；当 $\text{cwnd} > \text{ssthresh}$ → 转入拥塞避免；
- 拥塞避免中：收到 ack → $\text{cwnd} \mathrel{+}= \dfrac{MSS^2}{\text{cwnd}}$；
- 任一状态：超时（timeout）或三重重复确认（triple dup ack）→ $\text{cwnd} = 1$，$\text{ssthresh} = \text{cwnd}/2$ → 回到慢启动。

**讲稿**：这就是 TCP 状态机的样子。连接从慢启动状态开始，每收到一个确认就把拥塞窗口 cwnd 加一个 MSS；拥塞窗口一直这样增长，直到越过称为 ssthresh（slow start threshold，慢启动阈值）的门限。当拥塞窗口超过这个阈值，TCP 转入拥塞避免状态，以"每 RTT 一个 MSS"的保守速度增长。发生超时或三重重复确认时，TCP 转回慢启动状态。TCP 推断：三个重复确认（即同一确认号出现 4 次）是"下一个报文段丢了"的充分证据——接收方仍在收到后续段，只因缺了一段而无法推进确认号。转回慢启动时，TCP 把 ssthresh 设为拥塞窗口的一半；这个值决定了此后转用 AIMD 的分界。所以一次丢包后，TCP 先进入慢启动，再进入 AIMD。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 18：TCP Tahoe Behavior（TCP Tahoe 的行为曲线）</span>

图示：窗口大小随时间的曲线——标出 timeout（超时点）、duplicate acks（重复确认点）与 ssthresh 阶梯线。

**讲稿**：这张图给出 TCP Tahoe 拥塞窗口随时间变化的一个例子。窗口从 1 个 MSS 起步、指数增长。第一次下降对应一次超时：窗口回到 MSS，再次指数爬升，直到达到原峰值的一半——此时开始加性增长；继续增长直到某段丢失、出现三个重复确认（窗口不因重复确认而增长）；随后再次跌回 1 个 MSS，按慢启动指数增长直到 ssthresh，然后又开始加性增长。仔细看会发现：两次回到慢启动时 ssthresh 是同一个值——TCP 在同一个窗口大小处转入拥塞避免。在这个简化图里，这是因为 ssthresh 按整数个 MSS 计算且两种情况都向下取整到了同一个值。注意，TCP Tahoe 并不是严格按 AIMD 管理拥塞的：AIMD 是管理稳态（或稳定网络）的优秀算法，但实践中 TCP 要面对宽泛得多的条件——启动、丢失成批分组的瞬时网络故障、可用带宽的突变。最后回忆那三个问题：何时发送新数据、何时重传、何时确认。本讲回答了第一个：TCP 在其发送窗口——拥塞窗口与流量控制窗口的较小值——允许时发送新数据；拥塞窗口是发送方根据观测到的确认与超时维护的值。

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 4-5《Congestion Control - TCP Tahoe》的配套 slides；散点图均出自 Van Jacobson & Karels, "Congestion Avoidance and Control" (1988)，经许可使用。RTT 估计与自时钟见 4-6，快速重传/快速恢复见 4-7。</span>
