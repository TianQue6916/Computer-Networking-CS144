# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

> <span style="color:#7f8c8d;">Source: src/慕课/Unit7-Lower-Layers/wireless-mac.txt · CS144 Fall 2019 · 行内双语翻译版</span>

## <span style="color:#2471a3;">**[lecture]**</span> <span style="color:#c0392b">Wireless Media Access Control（无线媒体访问控制）</span>

原文 Slide 1 为标题页：**Wireless Media Access Control**（无线媒体访问控制）。MAC = media access control（媒体访问控制）。本讲先复习 MAC 的目标与以太网 CSMA/CD 的算法，再展示这套算法为什么在无线上一条都走不通。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 2 · Basic MAC Goals（MAC 的基本目标）</span>

<span style="color:#00838f;">对信道（channel）的使用权进行仲裁（arbitrate），使得：</span>

- 任意单个节点应能使用 100% 的信道；
- 多个节点应各自获得公平份额（fair share）；
- 争用（contention）之下仍保持高利用率（utilization）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这三句话其实是三张考卷，任何 MAC 协议都要同时应试。第一句测"效率上限"：只有一个节点时不应被协议开销拖累，要能吃满带宽；第二句测"公平"：$N$ 个节点长期看应平分信道，不能有人饿死；第三句测"高负载表现"：大家都抢的时候信道不能空转。张力在于：集中式调度（如基站轮询）公平好做，但需要专门的协调信道、时延也大；分布式随机接入（CSMA 系列）没有单点、时延小，但公平与利用率只能靠随机化逼近。用概率论的期望语言说：固定分配在轻载时空闲浪费的期望大，随机接入在重载时冲突浪费的期望大——好的 MAC 就是在两种浪费之间找平衡点。后面以太网与 802.11 的全部算法，都是这套平衡术的具体化。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 3 · Ethernet CSMA/CD（以太网 CSMA/CD）</span>

以太网 CSMA/CD（载波侦听多路访问/冲突检测）的完整算法。

发送时（on transmission）：

- 置 $n = 0$；
- 若信道空闲（idle），发送；
- 若信道忙（busy），等到信道空闲并持续 96 bit times（比特时间）后，发送。

传输过程中（during transmission）：

- 若未检测到冲突（collision），等待 96 比特时间，然后接受下一个待发送的帧（frame）；
- 若检测到冲突：
  - 发送 jam signal（干扰信号）；
  - 随机选择一个等待时间 $t$，取值范围 $(0, 2^n) \times 512$ bit times（比特时间）；
  - $n$ 自增 1；
  - 到时刻 $t$ 再检查信道。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 两个常数都值得抠一抠，它们不是拍脑袋定的。96 比特时间是 interframe gap（帧间隙），10 Mbps 下即 9.6 µs，给网卡电路留一点恢复时间，也保证相邻两帧不会粘连。512 比特时间（一个 slot time，时隙）则被物理定律锁死：要检测冲突，发送方必须"还在发送时"就听到冲突的回传，即最短帧的传输时延必须不小于信号在最远两点间的往返传播时延。10 Mbps 以太网最大网段的往返传播约 51.2 µs，恰好对应 512 比特；64 字节的最小帧长也由此而来——帧再短，冲突还没传回来你就发完了，检测无从谈起。退避窗口 $(0, 2^n)$ 随失败次数指数扩大，就是 exponential backoff（指数退避）：你在本课程前面见过的拥塞控制与重传机制，与它是同一家族的随机化思想。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 4 · Problem with CSMA/CD in Wireless（CSMA/CD 用于无线的问题）</span>

Slide 把以太网同一套算法原样重列一遍——这不是笔误，而是让你逐条对照：这套流程在无线上一条都执行不了。

发送时（on transmission）：置 $n = 0$；若信道空闲，发送；若信道忙，等到空闲持续 96 比特时间后发送。

传输过程中（during transmission）：若未检测到冲突，等待 96 比特时间后接受下一帧；若检测到冲突：发送 jam signal（干扰信号）；随机选 $t \in (0, 2^n) \times 512$ bit times；$n$ 自增；到时刻 $t$ 再查信道。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 问题出在最核心的 collision detection（冲突检测）这一步。从第一性原理算一笔账：发送方自己的信号在近处能量极强，而要从远端传来的微弱信号（已按 $r^2$ 以上衰减）里分辨出一个同时发送者，等于要在自己震耳欲聋的喇叭声里听清对面的一句低语——所需的接收动态范围远超无线网卡的能力。更麻烦的是冲突的"现场"在接收方：$A$ 发给 $B$ 时，冲突发生在 $B$ 的天线处，$A$ 未必听得见。所以无线改走另一条路线：检测不到，就干脆**避免**加**事后确认**——CSMA/CA 先用随机退避避开冲突，再用链路层 ACK 兜底验证。这也解释了为什么同一个 MAC 缩写，在有线与无线语境下是两套完全不同的协议。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 5 · Not a Shared Medium（并非共享介质）</span>

图示：$A$、$B$、$C$ 三个节点。无线"链路"（link）不是一根所有节点共听同一状态的总线——每个位置听到的信道都不一样，"信道"这个概念只相对于某一个接收者才有意义。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这张图是本讲的题眼。以太网的隐喻是"一根导线加若干抽头"，载波侦听的结果人人相同；无线更像"每个接收者一座独享的剧场"，同一台发射机在每处留下的声压都不同。你在 CSAPP 里接触过的总线（bus）仲裁，以及集线器（hub）时代"所有主机共听碰撞"的以太网，都建立在"全局信道状态一致"这个假设之上；无线把假设抽走之后，hidden terminal（隐藏终端）、exposed terminal（暴露终端）之类的错位全都是它的推论。下一讲开始登场的东西——CSMA/CA 的随机退避、链路层确认、再到 RTS/CTS 握手——全部是为了在"没有共享介质"的世界里，重新造出一个"仿佛共享"的抽象，让上层协议还能按老习惯写。
