# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Packet Switching - Principles: Delay Guarantees（分组交换原理：时延保证）

> <span style="color:#7f8c8d;">Source: 慕课/Unit3-Packet-Switching/3-10_delay_guarantees_notes.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

> <span style="color:#7f8c8d;">提取说明：本源文本由 pdftotext 提取时噪声较多（部分 slide 正文残缺、序号错位），以下按可辨认内容整理，无法辨认处已标注。</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1~2：回顾与问题</span>

（Slide 1 为标题页；Slide 2 正文在文本提取中残缺，仅存句首 "If we kn…"，结合下一页可推断本页在提出问题：**如果我们知道分组经过哪个队列、缓冲区多大、以什么速率被服务，我们就能知道分组会遇到的最大时延。**）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 3：WFQ 给出的下界</span>

如果在路由器里，我知道分组走哪个队列、缓冲区的大小、以及队列被服务的速率，那么我就知道一个分组会遇到的最大时延。WFQ 给我一个下界——即每流服务速率的下界（再次画出表达式与缓冲区大小 $B$）：

```math
r_i \;=\; \frac{w_i}{\sum_j w_j}\,R
```

<span style="color:#00838f;">知道服务速率下界 $r_i$ 与缓冲区上界 $B$，就得到该队列排队时延的上界 $\dfrac{B}{r_i}$。</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 4：路由器的分类—排队—服务模型</span>

这提示了如下路由器模型：我们把到达的分组**分类（classify）**，放进正确的队列；设定队列大小；再以正确的速率服务该队列。这样，任何到达路由器的分组都有有界时延。如果我们把时延的各分量加对，就能按我们的方程把它做成交端到端（end to end）的保证。这对"一路走完"的分组有效。可如果分组到达得太快、被丢弃了怎么办？

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 5：控制到达过程 A(t)</span>

回顾 $A(t)$ 与 $D(t)$ 曲线（下一页 slide 的内容，此处先画草图）：$d(t)$ 是 $D(t)$ 与 $A(t)$ 之间的水平距离（这是一个 FIFO 队列）。$D(t)$ 本身界得很好，但如果到达过程太大，$Q(t)$ 就会增长，分组将被**丢弃（DROPPED）**——这就没什么用了。

如果我们能设法控制 $A(t)$、确保这不会发生呢？这就是接下来要看的。如果做得到，我们就能提供穿过路由器的时延保证。

设想我们能声明：<span style="color:#00838f;">在任何长度为 $T$ 的区间内，到达队列的比特不超过 $B + r_1 T$</span>。换言之：

```math
A(t+T) - A(t) \;\le\; B + r_1\,T
```

这种做法的优点：

- 队列永不溢出，因此我们知道时延是有保证的；
- 我们给了到达过程 $A(t)$ 相当大的自由度。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "队列永不溢出"不是一句口号，可以两行证明，建议你亲手推一遍：把约束记作 $A(t+T)-A(t)\le\sigma+\rho T$（本页的 $\sigma=B$、$\rho=r_1$），队列以速率 $R\ge\rho$ 排空，则任意长度为 $T$ 的窗口内，最多积压 $\sigma+\rho T-RT=\sigma-(R-\rho)T$ 个比特——它随 $T$ 递减，取 $T$ 为最长拥塞窗口，积压上界就是 $\sigma$ 本身；所以只要 $\sigma\le B$，缓冲区就永远不会溢出。这个论证的妙处在于它的"确定性"：不假设任何分布，只封顶"任意窗口内的到达总量"，对一个学过概率论的人来说，这相当于放弃了均值/方差这些统计量，直接抓住随机变量的样本路径上界——最保守、也最强。这正是漏桶（leaky bucket）约束：$\sigma$ 是桶深（允许的突发），$\rho$ 是漏孔速率（长期平均速率）。3-5a (d) 里那个"输出永远恒定 500 bit/s"的队列，就是 $\rho$ 型整形的最简单实例。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 6~8：图示</span>

这三页为纯图示 slide，配合讲解演示 $A(t)$、$D(t)$、$Q(t)$ 曲线与受约束到达过程（文字极少，基于上下文推断）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 9：图示</span>

本页为图示 slide（文字极少），衔接下一页的端到端机制（基于上下文推断）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 10：端到端时延保证与 RSVP</span>

把各环节拼起来：来自 $A$ 的流量受 $(\sigma, \rho)$ 约束；每台路由器运行 WFQ，速率与缓冲区大小预先确定。显示分组被分类进正确的队列以及它的服务速率：分组进入路由器 $Q_1$，以 $R_1$ 被服务。我们只要求 $\sigma \le B$ 且 $\rho \le R_1$。第二台路由器同理……依此类推。于是，用我们的端到端时延方程，就能算出整条路径上的时延。

你可能想问：$\sigma$、$\rho$、$R_1$、$B$ 这些值是怎么告诉路由器的？端到端时延承诺最初由谁建立？IETF 有一个名为 RSVP 的协议（IETF RFC 2205）就是干这个的；教科书和维基百科上有很好的介绍。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这页展示的整套机制（每流预留带宽与缓冲、路由器维护每流状态）属于互联网历史上的"综合服务（IntServ）"路线，RSVP（RFC 2205，1997 年）就是它的信令协议。它最终没有成为互联网的主流，原因值得每个学分布式系统的人记住：核心路由器要为**每一条**预留流维护状态，而互联网核心的流数量是百万级且动态生灭，状态规模与刷新开销随网络规模爆炸——工程上不可扩展。它输给的对手是"区分服务（DiffServ）"：不逐流预留，只把分组按类打标（前面 3-9 的 ToS 字段），核心路由器按类聚合处理，几乎不保存状态；再加上端到端拥塞控制兜底，互联网用"尽力而为 + 端点自律"拿到了 99% 的效果。这是"端到端原则 vs 网络内智能"之争最著名的一次实战——回头看 3-2 说的"中间节点几乎没有状态"，你就能体会为什么无状态的设计赢了。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 11：例题解答（源文本残缺，按可见数字整理）</span>

<span style="color:#c0392b;">**解答：**</span>固定时延为分组化时延加传播时延：

```math
\left(\frac{1000\times 8}{10^9}+\frac{1000\times 8}{10^8}+\frac{1000\times 8}{10^9}\right) + \frac{120\times 10^3}{2\times 10^8} \;=\; 0.096\,\text{ms} + 0.6\,\text{ms} \;=\; 0.696\,\text{ms}
```

端到端时延预算为 5 ms，因此排队时延必须小于 $5-0.696=4.304\ \text{ms}$。我们**选择**把时延在两台路由器之间平均分配：即每台 $2.152\ \text{ms}$。因此缓冲区需满足

```math
B \;>\; 10\,\text{Mb/s}\times 2.152\,\text{ms} \;=\; 21520\ \text{bit} \;=\; 2690\ \text{byte}
```

源 $A$ 需要以漏桶参数发送：突发 $\sigma = 21520$ bit，速率 $\rho = 10\,\text{Mb/s}$。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这道例题的源文本提取残缺（只留下算式与结果碎片），上面的完整数字是按可见算术复原的，值得指出它的隐含设定：一个 1000 字节分组的流以 $\rho=10\,\text{Mb/s}$ 跨越两台路由器、三条速率分别为 $10^9/10^8/10^9$ b/s 的链路、总传播距离 120 km，总时延预算 5 ms。解题骨架是"先扣固定分量，再给随机分量做预算"：分组化三项 $8+80+8=96\ \mu s$、传播 $0.6\ \text{ms}$，固定分量共 0.696 ms；剩余 4.304 ms 全部留给排队，并人为均分给两台路由器。注意"均分"是设计选择而非必然——3-10a 的同型例题按题设均分三台，你也可以按链路速率比例分配。最后一步 $B>\rho\times d_q$ 把"时延预算"翻译回"缓冲区大小"：速率乘时间等于比特数，这是排队论里最常用的量纲变换，务必形成肌肉记忆。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 12~13：结束</span>

本讲结束（图示总结 slide，文字极少）。

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 3-10《Delay Guarantees》的配套 notes（源 PDF 文本提取噪声较多）；完整数字的例题见 3-10a，WFQ 速率下界见 3-9。</span>
