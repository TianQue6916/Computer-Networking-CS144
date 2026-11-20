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

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这一页的设计问题是「要交付一条按序、可靠的字节流，接收方最少要记住多少状态」。Go-Back-N（回退 N）的答卷是零缓冲加一个整数：接收方只维护 expectedseqnum（期望序号）这一个变量，序号等于它就收下并加一，不等于它的一律丢弃并重发上一个 ACK（确认），无论更小（重复）还是更大（失序）。发送方却要维护 base（窗口头）、nextseqnum（下一个可发序号）与整份 send buffer（发送缓冲），因为重传需要原文。这个不对称的全部来源是 cumulative acknowledgment（累积确认）的语义：ACK 编号只有「我下一个想要的是 $x$」这一个含义，于是「小于 $x$ 的都收到」「大于 $x$ 的都不要」两条规则把接收方所有可能的记忆折叠进了一个标量。
>
> 为什么折叠是安全的？因为序号本身是模 $2^k$ 的等价类，$a$ 与 $a+2^k$ 在接收方眼里无法区分。你在 6.042J 学过的 equivalence relation（等价关系）在这里就是它：同一个同余类里的整数被视为同一个序号，代表元就是那 $k$ 个比特。CSAPP 第 2 章讲的 unsigned wraparound（无符号回绕）是同一件事的硬件版本，模 $2^w$ 的加法里溢出不是错误而是定义。所以「序号会绕回来」不是要修的缺陷，而是 sliding window（滑动窗口）必须远小于序列号空间的根因，精确的界见本讲义 Slide 6 之后那条注。
>
> 代价也要算清：一次丢包会让「正确到达但注定被丢弃」的分组白白占用链路。窗口 $N=4$、分组 1 丢失时，2、3、4 三次传输全部作废，超时后 1 到 4 重发，等于用 7 次传输交付 4 个分组。这就是 state-compression-for-bandwidth（状态压缩换带宽）这个模板：把接收方的内存挡在门外，用重传带宽买回来。你在 CSAPP 第 6 章见过的 cache（缓存）用一位 valid（有效位）加 tag（标签）把一整块内存行折叠成一个槽位，是同一个思路的存储版本，区别在于 cache（缓存）丢弃后由下一级存储补齐，Go-Back-N（回退 N）丢弃后由重传补齐。
>
> 费曼式说法三句话：接收方只需要知道「下一个该给我的是几号」；来的是这个号就收货，不是就退货并喊一声「我还在等几号」；发送方听到这句喊声，就知道该从那儿重发。这三句话与正式定义的缺口只有一处，就是「喊一声」并不免费，ACK 也要占用反向链路的分组；反向链路一旦丢包，发送方只能靠定时器自己推，这正是 per-packet timer（每分组定时器）存在的理由。还有一处幂等性值得一提：ACK 丢失会让发送方误以为分组丢了而重发，接收方却因为序号小于 expectedseqnum（期望序号）而直接丢弃重复包——重复不会造成二次交付，这也是 $O(1)$ 状态能成立的一半原因。
>
> 再补一笔存储账：Go-Back-N（回退 N）把 $N$ 个分组的副本全部压在发送方（重传要原文），Selective Repeat（选择性重传）在接收方也存一份，于是两端各 $N$ 个。「谁付内存、谁付带宽」在本讲义里始终是同一个旋钮的两端。
>
> 回到正文：Slide 3 图上那些「丢弃」箭头不是画得潦草，而是这个 $O(1)$ 接收方的定义性行为。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 4：Selective Repeat（选择性重传）</span>

本页同为 sender/receiver 时序图：发送方发出窗口内多个分组；某分组丢失后，接收方**缓存失序到达的后续分组**，发送方仅对丢失的那一个分组设置定时器并重传，接收方收齐后按序上交。

<span style="color:#7f8c8d;">（Slide 4/6）</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> Selective Repeat（选择性重传）把接收方从一个整数升级成「一个窗口加一块缓冲」，代价是多了一个纯逻辑问题：同一个序号 $x$ 既可能是一个从未见过的新分组，也可能是一个 ACK（确认）丢失后重传的旧分组，接收方必须能判定。判定唯一可用的信息是窗口的位置，于是要求 sender window（发送窗口）与 receiver window（接收窗口）在模 $2^k$ 的序号空间里不交叠：$2N \le 2^k$，即 $N \le 2^{k-1}$。
>
> 反例最省事。取 $k=3$（序号 0 到 7），故意让 $N=5 > 4$：发送窗口是 $\{0,1,2,3,4\}$，接收窗口是 $\{5,6,7,0,1\}$。现在让发送方把整窗发完、接收方全部收到、5 个 ACK（确认）在回程全丢。发送方超时重传 0 到 4，接收方收到「0」时无法判断它是新分组的 0 还是旧分组的 0，因为自己的窗口里也含 0，两种解释都自洽，于是它把重复数据当新数据按序上交，in-order delivery（按序交付）被破坏。若 $N \le 4$，发送窗口 $\{0,1,2,3\}$ 与接收窗口 $\{4,5,6,7\}$ 恰好互补而不重叠，任何序号只有一种自洽解释。「恰好互补」这四个字值得记住：$N = 2^{k-1}$ 正是那个把模空间对半切开、让两端窗口刚好拼满全部序号的值。
>
> 这条约束用的是你在 6.042J 见过的计数论证，不是概率论证：两个窗口是模空间里的两张「牌」，两张牌不交叠的充要条件是它们的总大小不超过牌堆大小，于是 $2N \le 2^k$。同一个模板可以立刻复用：Go-Back-N（回退 N）的接收窗口恒为 1，所以只需 $N \le 2^k - 1$。可见 Go-Back-N（回退 N）的宽松并非因为它更聪明，而是因为它主动放弃了缓存，从根上消灭了歧义。命名模板：window non-overlap condition（窗口不交叠条件）。凡是「用有限位宽编号、两端各自维护一个区间」的协议，第一件该算的事就是这个不等式。
>
> 跨课程连接：CSAPP 第 2 章讲 fixed-width representation（定长表示）时给过一句判词，$w$ 位只能表示 $2^w$ 个不同的值，任何更大的范围都必须付出额外比特或额外约束，这里付出的就是约束。再往远处看半步：真实 TCP 的序号是 32 位，在 long fat pipe（长肥管道，带宽高而时延大）上 $4\,\mathrm{GiB}$ 的序号空间可以在一个往返内被消耗干净，于是 RFC 7323（TCP 高性能扩展）用 PAWS（Protect Against Wrapped Sequences，防序号回绕）配合 timestamp（时间戳）拒绝绕回来的旧报文，同一个歧义只是把额外比特换成了 64 位时间戳。
>
> 费曼式说法三句话：给每个分组编号；让发送方记「发出去了但还没被确认的号」这一段，让接收方记「我愿意收的号」这一段；两段在号池里不准重叠，否则老号与新号会被认错。与正式定义的缺口是「号池会重复使用」这一条：如果序号永不重复（比如位宽大到一条连接用不完），这条不等式根本不必存在。换个角度更清楚：Go-Back-N（回退 N）能用 $2^k - 1$ 这么松的界，是因为它的接收窗口只有 1，不缓存就不存在「老号与新号并存」的场景；一旦允许缓存，两个窗口就必须在号池里互补，界立刻收紧一半。这也解释了 RFC 7323 为什么要在序号之外再挂一个 64 位时间戳——让「号」以外的第二个维度去承担区分新旧的工作，等于把不等式重新放宽。
>
> 回到正文：Slide 6 说「接收窗口为 $N$ 时两种策略都能实现」，那个「能」字背后就是这条要付账的不等式。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 5：First Example（第一个例子）</span>

- 发送方（sender）窗口为 $N$
- 接收方（receiver）窗口为 1
- 这是 Go-Back-N 还是 Selective Repeat？

**解答：** 接收窗口为 1 意味着接收方不缓存任何失序分组——窗口头部一旦丢失，后续分组无论是否到达都注定被丢弃。此时选择性重传毫无收益，两者行为完全等价于 Go-Back-N；逐分组定时器也退化为整窗超时重传。

> <span style="color:#1e8449;">**[solution]**</span> 结论：**Go-Back-N**（选择性重传在接收窗口为 1 时退化为无效）。

<span style="color:#7f8c8d;">（Slide 5/6）</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> Slide 5 的结论值得升格成一个命题，因为它的证明骨架可以反复用。定义 Selective Repeat（选择性重传）相对 Go-Back-N（回退 N）的收益 $G$ 为该策略能保住、而 Go-Back-N（回退 N）会丢弃的失序分组数，于是有界 $G \le \min(W_r - 1,\, N - 1)$，其中 $W_r$ 是 receiver window（接收窗口）大小、$N$ 是 sender window（发送窗口）大小。第一项来自接收方容量：窗口为 $W_r$ 时最多缓存 $W_r - 1$ 个失序分组，第 $W_r$ 个位置留给「下一个该到的」，不算缓存。第二项来自发送方：一个窗口里最多只有 $N - 1$ 个分组可能失序。取 $W_r = 1$ 立刻得 $G \le 0$，于是两种策略在所有可达状态上的后续行为逐点相同，「等价」不是比喻而是收益函数取零。
>
> 逐字核对机制：$N=4$、$W_r=1$、分组 1 丢失。Selective Repeat（选择性重传）的接收方期望的是 1，所以 2 到达时「不在接收窗口内」（窗口只有 $\{1\}$），不缓存、直接丢弃并重发对 1 的 ACK（确认）；发送方超时后重传 2、3、4。这与 Go-Back-N（回退 N）的行为完全一致。你直觉上认为「能不能存决定一切」是对的，只需补一句：能存多少是一个数，不是一种理念。
>
> 命名模板：degenerate-case check（退化检验）。把一个设计参数推到边界值，检验机制是否坍缩成更简单的特例，并用一个不等式写出坍缩的充要条件。这个模板你有三处已学的实例可以对照：6.006 的 Dijkstra（单源最短路算法）依赖非负权假设，假设一破就从正确退化到失效（CLRS 第 24 章）；Bellman-Ford（贝尔曼-福特算法）在无负环时仍正确（CLRS 第 24.1 节），属于可退化的一侧；CSAPP 第 6 章的 direct-mapped cache（直接映射缓存）就是 associativity（相联度）为 1 的特例，conflict miss（冲突性缺失）飙升正是 $W_r=1$ 要付的同一种代价。
>
> 效率视角把两页的内容合并成一个可比量。设 packet loss rate（丢包率）为 $p$、各分组独立丢失、ACK（确认）不丢，则 Go-Back-N（回退 N）一次窗口传输（发出 $N$ 个分组）平均推进的分组数是 $E = \sum_{i=1}^{N}(i-1)\,p\,q^{i-1} + N q^{N}$，其中 $q = 1-p$；而接收方有缓冲的 Selective Repeat（选择性重传）每个分组平均只要 $1/q$ 次传输。$p=0.1$ 时，$N=2$ 的 Go-Back-N（回退 N）效率约 85.5%，$N=4$ 约 77.4%，$N=8$ 只有 64.1%，Selective Repeat（选择性重传）恒为 90%。第二项 $N q^{N}$ 是「整窗全部到达」那条分量的概率质量，$N$ 增大时它指数衰减，效率损失全部由第一项承担，这就是窗口越大、接收方能不能存越值钱的原因。
>
> 顺带把边界的两侧都验一遍，这才算完整的退化检验。$W_r = N$ 时 $G \le N - 1$，这正是收益的上界本身，再大也没有意义：接收方存得比发送方能发得更多，纯属浪费内存。$W_r = 0$ 不是合法状态（窗口至少为 1），所以 $W_r = 1$ 恰好是唯一的下界。两端一夹，Selective Repeat（选择性重传）的收益函数在 $[1, N]$ 上从 0 单调升到 $N - 1$，而 Slide 5 与 Slide 6 就是这条线段的两端点。还有一处工程细节值得现在就记下：$W_r = 1$ 时「逐分组定时器」也退化成整窗超时，因为只有一个未确认分组值得等——定时器的数量是随窗口走的，这一点在 Slide 6 之后那条注里会变成真实 TCP 的取舍。
>
> 回到正文：判定 Slide 5 与 Slide 6 的关键，从头到尾只有接收窗口这一个数。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 6：Second Example（第二个例子）</span>

- 发送方（sender）窗口为 $N$
- 接收方（receiver）窗口为 $N$
- 这是 Go-Back-N 还是 Selective Repeat？

**解答：** 接收方窗口为 $N$ 时可以缓存最多 $N$ 个失序分组并逐个确认，此时两种策略都可以实现，但只有**选择性重传**能兑现接收方缓存的全部价值：一次丢包只补那一个洞，其余 $N-1$ 个分组的传输成果全部保留。

> <span style="color:#1e8449;">**[solution]**</span> 结论：应当采用 **Selective Repeat**（接收方有缓冲能力时，整窗回退是在浪费带宽与已缓存的分组）。

<span style="color:#7f8c8d;">（Slide 6/6）</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> Slide 6 的解答说「接收方有缓冲能力时就该选 Selective Repeat（选择性重传）」，但真实 TCP 既不是纯 Go-Back-N（回退 N）也不是纯 Selective Repeat（选择性重传），它的答案是被工程成本逼出来的折中。设计问题先问清：per-packet timer（每分组定时器）在实现层要花什么？
>
> 窗口为 $N$ 时，逐分组定时器意味着一个连接同时维持 $N$ 个超时事件；一万个连接、每连接 64 个 in flight（在途）分组，就是六十四万个定时器事件，还要在 ACK（确认）回来时逐个定位并取消。所以主流实现反其道而行：整个连接只维护一个 retransmission timer（重传定时器），它盯住最早未确认的那个字节，一个推进了窗口头的 ACK（确认）到达就把它重启。2011 年的 RFC 6298（Computing TCP's Retransmission Timer，计算 TCP 的重传定时器）规定的正是这套单定时器加平滑 RTT（往返时延）的算法：维护 $SRTT$ 与 $RTTVAR$ 两个状态做加权滑动平均，取 $RTO = SRTT + \max(G, 4\,RTTVAR)$，其中 $RTO$ 是 retransmission timeout（重传超时）、$G$ 是 clock granularity（时钟粒度）。这是在时间维度上又做了一次状态压缩，和 Slide 3 那条注里接收方的做法同源，只是被压掉的东西从「每个分组一个序号位」换成了「每个分组一个闹钟」。
>
> 第二件事是换信号。等定时器到期要一个 $RTO$（通常数百毫秒），而在 cumulative acknowledgment（累积确认）下，接收方每收到一个失序分组就立刻重发同一个 ACK（确认），于是发送方攒够 3 个 duplicate ACK（重复确认）就能断定窗口头那个丢了。这就是 fast retransmit（快速重传）：RFC 5681（TCP 拥塞控制规范）的 §3.2 把它规定为收到 3 个重复确认即重传窗口头并进入 fast recovery（快速恢复），其间不得有推进 SND.UNA（发送未确认序号）的 ACK 插入。把时间到期换成数量攒够，代价是接收方多花一次立即确认的带宽，收益是少等一个 $RTO$，用反方向的一点比特买掉数百毫秒的等待。
>
> 第三件事是把 Go-Back-N（回退 N）的浪费补回来。SACK（Selective Acknowledgment，选择性确认）选项由 RFC 2018（TCP 选择性确认选项）定义，那是 1996 年 10 月 Mathis（马西斯）等人的工作：接收方在 ACK（确认）里报告若干「已收到但不相邻的数据块」，发送方据此只补空洞。option space（选项空间）只有 40 字节，一块要 8 字节加 2 字节头，所以最多 4 块；同时启用 timestamp（时间戳）选项时被挤到 3 块。RFC 6675（基于 SACK 的丢失恢复算法）后来把「有了选择性确认之后到底重传哪个」整理成一套可实现的重传算法。历史线索很短很干净：RFC 793（1981 年的 TCP 规范）给出累积确认，早期 RFC 2001 的快速重传后来由 RFC 5681（TCP 拥塞控制规范）接管，RFC 2018（1996 年）补上选择性确认，RFC 6298（2011 年）固定定时器算法，2022 年的 RFC 9293（TCP 整合规范）把这些合并成一份规范并取代 RFC 793 及其七份修订。Kurose 与 Ross 的判词最省字：TCP 的错误恢复是 Go-Back-N（回退 N）与 Selective Repeat（选择性重传）的 hybrid（混合体）。
>
> 跨课程连接：定时器本身是 CSAPP 第 11 章那个「不用线程怎么同时等很多事」的问题；作业系统普遍用 timer wheel（时间轮）把海量定时器摊平成摊销常数时间的插入与到期，而 6.006 里你学过的 binary heap（二叉堆）实现 priority queue（优先队列）是 $\Theta(\log n)$ 的通用解。协议的定时维度与算法课的复杂度考量，在这一页合流。
>
> 回到正文：Slide 6 的结论在理论上正确，工程上要分三步才兑现，省掉逐分组定时器、把丢失检测从时间信号换成数量信号、再用 SACK（选择性确认）把整窗回退的浪费补掉。

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 2-9《Reliable comm --- Retransmission strategies》的配套 slides；两个例子的判定关键都在接收窗口大小。</span>
