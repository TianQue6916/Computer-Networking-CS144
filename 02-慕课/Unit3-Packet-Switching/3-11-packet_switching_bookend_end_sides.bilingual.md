# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Packet Switching (recap)（分组交换：单元收束）

> <span style="color:#7f8c8d;">Source: 慕课/Unit3-Packet-Switching/3-11-packet_switching_bookend_end_sides.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1：标题页</span>

CS144: An Introduction to Computer Networks（计算机网络导论）
Unit 3: Packet Switching（第 3 单元：分组交换）
CS144, Stanford University

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 2：你学到了什么</span>

- 排队时延（queueing delay）与端到端时延（end-to-end delay）；
- 为什么流媒体应用要使用播放缓冲区（playback buffer）；
- 一个简单的确定性队列模型；
- 速率保证（rate guarantees）；
- 时延保证（delay guarantees）；
- 分组如何被交换与转发。

🎥 *CS144 教授在视频中解释: "你需要完全理解数据包延迟的主要三个组成部分：数据包化延迟、传播延迟和队列延迟，并且你理解导致它们的物理过程"*（字幕来源：051 - p50 3-11 Packet Switching (recap).zh.srt）

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这张清单值得换个角度读：它其实是**一个模型、两次应用**。核心模型只有一样——3-5 的确定性队列图（$A(t)$ 与 $D(t)$ 两条曲线，竖直距离是占用、水平距离是时延）；第一次应用在端主机（播放缓冲区设计：给到达曲线预留一段竖直缓冲换播放平滑），第二次应用在网络内部（速率/时延保证：给到达曲线加漏桶约束、给服务速率加 WFQ 下界）。学完整个单元后你应该能体会教授强调的"草图习惯"的价值：所有这些结论都不需要解方程，一张手绘图就够——这和 CSAPP 用"存储器层次"一张图统摄缓存/内存/磁盘是同一种功力：找到那个能反复复用的最小抽象。如果你能脱稿给别人讲清楚"为什么小分组更快"和"为什么缓冲区既防丢包又加时延"，这个单元就算真正过关了。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 3：这些知识将帮你做什么</span>

1. 端到端时延：固定分量；
2. 端到端时延：可变分量；
3. 分组交换机是如何工作的；
4. 为实时应用设计播放缓冲区；
5. 解释如何实现速率保证与时延保证。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 第 1、2 条把"固定分量 vs 可变分量"单列出来，是本单元最有迁移价值的心智框架：固定分量（$\sum p/r_i+\sum l_i/c$）由物理与配置决定，砸钱升级链路、挪近机房就能改善；可变分量（排队时延）由他人流量决定，只能靠调度（WFQ）、整形（漏桶）或缓冲（播放缓冲区）去管理——分不清这两类，就会犯"买了千兆带宽却还是卡顿"的典型错误。第 3 条指向 ps_switching 两讲的交换机内部结构（查表、交换、队列位置）；第 5 条则是前面机制的合成应用。至于这套知识往哪里去：理解了"分组逐跳独立转发、队列随机积压"，你才算拿到学习拥塞控制（为什么丢包是信号）、路由（转发表从哪来）的门票——Unit 3 正是这两大主题的物理与数学基础。

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 3-11《Packet Switching (recap)》的配套 bookend slides；Unit 3 各小节的技术细节见本单元其余 12 份讲义。</span>
