# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Congestion — AIMD with a single flow（拥塞——单条流的 AIMD）

> <span style="color:#7f8c8d;">Source: 慕课/Unit4-Congestion-Control/congestion_-_aimd_single.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1：封面</span>

CS144：计算机网络导论。**Congestion（拥塞）：AIMD with a single flow（单条流的 AIMD）**。Nick McKeown，斯坦福大学电气工程与计算机科学教授。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 2：AIMD（回顾）</span>

AIMD：Additive Increase, Multiplicative Decrease（加性增、乘性减）：

- 若分组被正确接收：$W \to W + 1$；
- 若有分组被丢弃：$W \to \dfrac{W}{2}$。

图示：cwnd 随时间 $t$ 的锯齿波——线性爬升，遇丢包（Drops，丢包处）骤降一半（halved，减半）。

🎥 *CS144 教授在视频中解释: "注意，这是由终点主机完成的，没有网络任何明确的支持"*（字幕来源：055 - p54 4-3 Congestion Control - Dynamics of a single AIMD flow.zh.srt）

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 字幕里这句话点出了一个容易被忽略的事实：整个 AIMD 机制没有任何来自网络的显式帮助——路由器不发"减速"指令，TCP 完全靠端主机自己观测丢包来推断网络状态。这与第 3 单元"网络内部尽量简单"的尽力而为（best-effort，尽力而为）服务模型严丝合缝：IP 层只负责转发，拥塞信号只能以"包没到"这种最原始的形式从网络边缘渗出来。这种设计的代价是信息极度贫乏——发送方连"丢包是因为拥塞还是链路错误"都分不清（有线网络里几乎都是前者，无线网络里后者很常见，这也是无线环境下 TCP 性能差的重要原因）。把智能放在端系统，是端到端原则在本单元的又一次落地。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 3：Animation（动画演示）</span>

动画演示见：<http://guido.appenzeller.net/anims/>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 4：Single Flow Dynamics（单流动力学）</span>

纯图示 slide：单条流穿过一台路由器的示意图——发送方 A、路由器缓冲区、瓶颈链路、接收方 B。本讲将追踪这条流在"一条瓶颈链路 + 一个缓冲区"最简拓扑下的完整行为。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 5：Sending rate for a single flow（单流的发送速率）</span>

设瓶颈链路速率为 $R$，往返时延为 $RTT$，窗口大小为 $W$：

1. $R \times RTT > W$：窗口不足以填满"管道"（带宽-时延积），发送方断续；
2. $R \times RTT = W$：窗口恰好填满管道，发送连续。

图示：A、B 之间多个 ACK 沿反方向流动，滑动窗口随之推进。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 6：Sending rate for single flow（单流发送速率与缓冲区占用的对照）</span>

图示（上下两幅对照图）：

- 上幅：窗口大小 $W(t)$ 随时间的锯齿曲线；
- 下幅：对应的路由器缓冲区（Router buffer，路由器缓冲区）占用曲线；
- 阶段 A（Link rate > C，发送速率低于链路容量 $C$）：窗口尚小，缓冲区为空，瓶颈链路没有被填满；
- 阶段 B（Link rate = C，发送速率达到链路容量 $C$）：窗口超过带宽-时延积后，多余的在途分组只能在缓冲区里排队——缓冲区占用线性上升，直到塞满、丢包、窗口减半。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 7：How big should the buffer be?（缓冲区应该多大？）</span>

- 缓冲区 $B = RTT \times C$（恰好等于带宽-时延积）：锯齿的谷底恰好还能填满链路，**链路不空转**；
- 缓冲区 $B < RTT \times C$：窗口减半后管道填不满，链路出现空闲——容量被浪费。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> $B = RTT \times C$ 这条法则是第一性原理推理的好范本，值得自己推一遍：锯齿谷底时窗口减半，此时还能"恰好填满管道"（带宽-时延积那么多比特在路上）而不动用缓冲区；峰顶时窗口翻倍，多出来的那一份只能堆进缓冲区——所以缓冲区恰好需要一个 BDP 的容量，谷不空、峰不溢。但请注意硬币的反面：缓冲区**大于** BDP 并不会提高吞吐，只会让分组在队列里泡更久（排队时延拉长），这个"缓冲区越大时延越高"的问题在真实互联网上有个专名——bufferbloat（缓冲区膨胀），是 2010 年代重要的工程议题。还有一层：本页结论依赖"单流"假设；多条流共享缓冲区时，谁在谷底、谁在峰顶各不相同，$B = RTT \times C$ 的配置对多流而言只是个起点。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 8：Observations for single flow（单流情形的观察）</span>

1. 窗口按 AIMD 规律扩张与收缩；
2. ……以此探测"管道"能装下多少字节；
3. 锯齿波就是稳定的运行点（stable operating point，稳定运行点）；
4. 发送速率是恒定的；
5. ……前提是缓冲区足够大（$RTT \times C$）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "锯齿波是稳定运行点"这句话值得用 6.042J 的递归思想细品：每个周期从 $W_{max}/2$ 线性爬回 $W_{max}$，丢一包，回到 $W_{max}/2$——起点即终点，周期闭合，这就是一个不动点式的循环不变量。只要缓冲区 $\geq RTT \times C$，这个循环就自我维持：每周期恰好丢一包、恰好减半一次、恰好爬回原高度，周而复始。更妙的是观察 4：虽然窗口在锯齿般起伏，发送速率却恒定在 $C$——谷底时全部窗口在管道里（链路满、队列空），峰顶时管道满、队列满（链路仍满），两个极端链路都是满的，中间段链路也满。窗口的波动被缓冲区"吸收"了，这正是缓冲区的本职：把速率的波动转化为时延的波动。概率论里排队论的视角在此同样适用——服务率恒为 $C$、到达率被窗口约束在 $C$ 附近，队列长度在两个确定性边界之间游走，系统处在"临界负载"的边缘。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 9：结束页</span>

本讲完。CS144, Stanford University。

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 4-3《Congestion Control - Dynamics of a single AIMD flow》的配套 slides；配套数值例题见 4-3a，多流情形见 4-4。</span>
