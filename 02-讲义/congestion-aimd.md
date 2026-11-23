# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Congestion Control IV — Why AIMD（拥塞控制 IV——为什么是 AIMD）

> <span style="color:#7f8c8d;">Source: 慕课/Unit4-Congestion-Control/congestion-aimd.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1：封面</span>

拥塞控制 IV：为什么是 AIMD（Why AIMD）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 2：Congestion Control（拥塞控制：各方的诉求）</span>

- **服务提供商（Service Provider，服务提供商）**：最大化链路利用率（link utilization，链路利用率）；
- **用户（User，用户）**："我要拿到我应得的那一份"（公平份额）；
- 我们希望网络**收敛（converge，收敛）**到每个人都拿到 $1/N$ 的状态；
- 并避免拥塞崩溃（congestion collapse，拥塞崩溃）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这页把拥塞控制摆成了一个多目标优化问题，四句话背后各有数学内容。服务商要的是 $\max \sum_i r_i$（效率），用户要的是 $r_i$ 之间按某种准则平等（公平），而这两者在 Slide 10 那个例子里已经见过冲突：总吞吐 2.75 的分配比 2.5 的更"高效"却更不公平。"收敛到 $1/N$"给出了我们真正想要的解：公平与效率同时在最大-最小公平点上达成。而"避免拥塞崩溃"则是硬约束：1986 年的互联网真的发生过崩溃——链路近乎满负荷传输的却大多是重传的废包，有效吞吐趋近于零。后面你会看到 AIMD 用一个极简的规则同时照顾了这四条诉求，这种"一条规则满足多个目标"的 elegance，正是它统治互联网三十余年的原因。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 3：Congestion Window Size（拥塞窗口应有多大？）</span>

图示：一条从 San Francisco（旧金山）到 Boston（波士顿）的跨大陆路径。

<span style="color:#00838f;">**最优的拥塞窗口大小等于带宽-时延积（bandwidth-delay product，带宽-时延积，BDP）**</span>：即链路速率 $C$ 与往返时延 $RTT$ 的乘积 $C \times RTT$——恰好填满整条"管道"所需的比特数。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 带宽-时延积值得算一个具体数字找感觉：跨美大陆 RTT 约 70 ms，10 Gb/s 链路的 BDP 是 $10^{10} \times 0.07 = 7 \times 10^8$ 比特，约 87.5 MB——这就是说，在途窗口得攒上近九千万字节，才能让这条管道不空转。这个数字解释了本单元后面好几件事：为什么长肥网络（长 RTT 或高带宽）上的 TCP 恢复特别慢（乘性减一下子砍掉几十 MB 的窗口，加性增每个 RTT 才补回一个报文段）；为什么路由器缓冲区至少要配到 BDP（4-3 节会推导 $B = RTT \times C$）；以及为什么窗口不满时发送方"发出一个窗口就卡住"（Slide 8 的 $R \times RTT > W$ 情形）。BDP 是贯穿本单元的量纲基准：任何关于窗口、缓冲区的数值，都应先跟它比一比。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 4：Chiu-Jain Plot（Chiu-Jain 相图）</span>

纯图示 slide：以两条流 A、B 的速率（bps）为横纵坐标的平面（相图，phase plot），纵横坐标分别为 Flow A rate（bps）与 Flow B rate（bps）。AIMD 在这个平面上的运动轨迹将从下一页起逐层画出。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 5：Chiu-Jain Plot——公平线（Fair: A = B）</span>

在相图上叠加**公平线（fairness line，公平线）** $A = B$：从原点出发的 45° 对角线。落在这条线上的分配对两条流完全平等。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 6：Chiu-Jain Plot——公平线与效率线</span>

再叠加**效率线（efficiency line，效率线）** $A + B = C$：斜率为 $-1$ 的直线，线上每一点恰好把瓶颈容量 $C$ 用满。公平线与效率线的交点就是同时满足两个目标的理想点。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 7：Chiu-Jain Plot——过载区与轻载区</span>

效率线 $A + B = C$ 把相图分成两半：

- 右上方为**过载区（overload，过载区）**：$A + B > C$，需求超出容量，队列增长、丢包逼近；
- 左下方为**轻载区（underload，轻载区）**：$A + B < C$，链路没有被喂饱，容量在浪费。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 8：Chiu-Jain Plot——两个区域（标注完整版）</span>

同上一页的完整标注版：公平线 $A = B$、效率线 $A + B = C$、过载区与轻载区同图呈现。AIMD 的目标：不管初始点在哪，状态都应被推向公平线与效率线的交点。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 9：Chiu-Jain Plot——AIMD 的轨迹（收敛）</span>

图示：从某个不公平的初始分配出发，AIMD 的完整轨迹（标注时间点 $t_1 \sim t_6$）：加性增阶段沿 45° 方向平移进入过载区，丢包触发乘性减、状态沿指向原点的方向缩回一半（落在轻载区），再加性增……每经过一轮"增—减"，状态与公平线的距离都在缩短，最终轨迹紧贴公平线与效率线的交点往复振荡。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这组相图是本单元理论含量最高的部分，值得把收敛性的几何论证吃透。AIMD 的两个动作在平面上有固定形状：加性增是沿 45° 方向的**平移**（两流各加 1，相对差距 $A-B$ 不变），乘性减是朝原点的**缩放**（两流各减半，比值 $A/B$ 不变）。缩放不改变公平性、平移不改变差距，而平移会让点朝"大流增加的绝对量占比更小"的方向相对靠拢——每轮减半后再平移，与公平线的距离就以几何方式收缩，这就是 AIMD 收敛到公平的全部秘密。从概率论的角度看，这个不断"增-随机丢包-减"的过程是一个马尔可夫链（Markov chain，马尔可夫链）：加性增是确定的漂移项，丢包时刻是随机转移，链的平稳分布集中在效率线附近——你看到的锯齿正是这条马尔可夫链的样本路径。Chiu 与 Jain 在 1989 年的论文中严格证明了这个收敛性；但注意它的前提是"两流同时收到丢包信号"（同步假设），真实网络中 RTT 各异的流并不同步，这正是 AIMD 在实践中偏离最大-最小公平的根源。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 9 之后的收尾</span>

本讲完。CS144, Stanford University。

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 4-8《Congestion Control - AIMD》的配套 slides；Chiu-Jain 相图出自 D. Chiu 与 R. Jain, "Analysis of the Increase and Decrease Algorithms for Congestion Avoidance in Computer Networks", 1989。</span>
