# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

> <span style="color:#7f8c8d;">Source: src/慕课/Unit7-Lower-Layers/wireless-csma.txt · CS144 Fall 2019 · 行内双语翻译版</span>

## <span style="color:#2471a3;">**[lecture]**</span> <span style="color:#c0392b">Wireless: CSMA/CA（无线：载波侦听多路访问/冲突避免）</span>

原文 Slide 1 为标题页：**Wireless: CSMA/CA**——carrier sense multiple access / collision avoidance（载波侦听多路访问/冲突避免）。CS144, Stanford University。本讲回答一个问题：当无线上"边发边听"的冲突检测行不通时，多个节点如何共享信道——答案是先退避、后确认的 CSMA/CA。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 2 · Link Layer Acknowledgments（链路层确认）</span>

图示：节点 $A$、$B$、$C$ 组成一条无线链路。Slide 标题点出无线链路层的关键机制——link-layer acknowledgment（链路层确认）：<span style="color:#00838f;">发送方发出 frame（帧）之后，必须等接收方显式回送一个 acknowledgment (ACK)（确认）短帧，才能认定本次发送成功。</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 有线以太网不搞逐帧确认：网卡发完就当成功，出错交给上层（比如 TCP）重传。无线为什么要多此一举？因为无线的误码率高得多：一个比特翻转就可能毁掉整帧，而如果只靠端到端重传（你在 CSAPP ch11 写过的 TCP 逻辑），一跳的瞬时衰落会迫使整个报文从源头重新走完全程。链路层确认把恢复范围限制在出错的那一跳——这正是分层（layering）思想的经典取舍：每一层只负责自己这一段的差错控制，代价是每个帧后面都要多付一个 ACK 报文的开销。还要注意：ACK 本身也要参与信道争用，它不是"免费"的，所以 802.11 把确认帧做得极短，只回一个层内的应答而不捎带任何数据。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 3 · CSMA/CA（载波侦听多路访问/冲突避免）</span>

CSMA/CA 的核心流程只有四步：

1. Pick random backoff（随机选择一个退避时间）；
2. Sense local channel, transmit after backoff（侦听本地信道，退避结束后发送）；
3. 若分组（packet）未被确认，则再次退避并重试（retry）；
4. 若分组被确认，则接受下一个待发送的分组。

相比以太网 CSMA/CD 的"边发边听"，这里没有冲突检测，取而代之的是"发送后等确认、没确认就退避重来"——把冲突的发现从发送方挪给了接收方的 ACK。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 4 · 802.11 CSMA/CA（802.11 的 CSMA/CA 算法）</span>

802.11 的具体算法：

- 选择一个随机初始等待时间 $t$；
- 周期性检查信道：若信道空闲（idle），就把 $t$ 递减；
- 当 $t = 0$ 时，尝试发送（transmit）；
- 若分组被成功接收（收到 ACK 确认），接受下一个待发送的分组；
- 若分组未被成功接收，把 $t$ 加倍（double $t$）；
- 若 $t \ge T$，则丢弃该分组（drop packet）。

图示：发送节点 $S$ 的时间线，先后标注 B1, TX（发送）与 B2, ACK（确认）——展示"退避计数 → 发送 → 等确认"的一次完整循环。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这几行伪代码里有三个设计细节值得从第一性原理推一遍。第一，"只在信道空闲时才递减 $t$"：信道一旦被别人占用，你的退避计时器就冻结（freeze），这保证先到者的剩余退避不会被后来者稀释，是公平性的来源。第二，失败后 $t$ 加倍就是 binary exponential backoff（二进制指数退避），与以太网同源：冲突越多说明争用越激烈，把随机窗口指数拉大，让各节点的重试在时间轴上铺得更开。第三，$t \ge T$ 就丢帧：重试不是无限的，超过阈值即放弃并交给上层处理，避免一个注定发不出去的帧把队列堵死。从概率论看，随机退避本质上是用随机化把 $N$ 个节点的冲突概率从 1 压到约 $1/N$ 量级，重试次数近似服从几何分布、期望有限——这与你在线性代数之外的马尔可夫链一讲里对独立重复试验期望的计算是同一套路。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 5 · Problem 1: Hidden Terminals（问题 1：隐藏终端）</span>

图示：$A$、$B$、$C$ 三个节点。$A$ 与 $C$ 相距太远、互相听不到（互为 hidden terminal，隐藏终端），但两者都在 $B$ 的覆盖范围内。于是 $A$ 和 $C$ 各自做载波侦听（carrier sense）时都觉得"信道空闲"，同时向 $B$ 发送，两个帧就在 $B$ 处发生 collision（冲突）——而在 $A$、$C$ 自己看来一切正常，冲突对它们完全不可见。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 麻烦的根源在于：载波侦听发生在发送方，而冲突发生在接收方，两者看到的是不同的世界。信号强度随距离至少按 $r^2$ 衰减，于是"我听得到你"的半径小于"我能干扰到你"的半径——$A$ 和 $C$ 虽然互相听不见，却都在 $B$ 的接收范围内。以太网 CSMA/CD 能工作的隐含前提是"所有人听到的信道状态一致"（一根总线人人共听），无线把这个前提整个砸掉了。术语上也要辨析一下：collision avoidance（冲突避免）里的"避免"，指的正是用退避加确认来绕开这种**检测不到**的冲突，而不是像 CD 那样先检测再处理；这一页的图就是后面 RTS/CTS 讲的伏笔——既然 $C$ 听不见 $A$，那就想办法让 $B$ 说话。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 6 · Problem 2: Exposed Terminals（问题 2：暴露终端）</span>

图示：$A$、$B$、$C$、$D$ 四个节点。$B$ 正在向 $A$ 发送；此时 $C$ 想向 $D$ 发送。$C$ 能侦听到 $B$ 的载波（$C$ 在 $B$ 的信号范围内），于是按规则退避等待——但 $D$ 并不在 $B$ 的干扰范围内，$C \to D$ 的发送根本不会影响 $A$ 的接收。$C$ 被不必要的退避卡住，白白浪费了并发传输的机会：这就是 exposed terminal（暴露终端）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 暴露终端与隐藏终端是一对镜像错误。隐藏终端是"该让没让"（假空闲）：以为没人发送，其实接收方正被干扰；暴露终端是"不该让却让了"（假忙）：听到有人在发，就条件反射地闭嘴。两者的共同根源是同一句话：发送方侦听到的信道状态，不等于接收方的信道状态。用 6.042J 图论的语言说，理想的信道调度应该作用在"接收方冲突图"上——两个发送节点之间连一条边，当且仅当它们会干扰同一个接收者；调度问题就是在这个图上挑出互不冲突的发送集合。而载波侦听只画得出"以发送方为中心"的局部图，信息天然不全，这两类错误都是缺信息的推论。后面 RTS/CTS 的本质，就是让接收方 $B$ 主动广播自己的视角（CTS），把缺的那块信息补上。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 7 · Problem 3: Collision or Low SNR?（问题 3：冲突还是低信噪比？）</span>

接收方收到一个坏帧时，无法区分两种原因：是两个发送方同时发送造成的 collision（冲突），还是只有一个发送方但信号太弱——low SNR（低信噪比；SNR = signal-to-noise ratio，信噪比）。Slide 只提出这一问而没有给答案，恰恰说明它没有现成答案。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这不是抬杠，而是 802.11 真实的设计困境：两种原因的最优反应完全不同。若是冲突，正确反应是退避重试，等下一个空闲窗口；若是信号太弱，退避毫无用处，正确反应是降低速率、换成更鲁棒的调制编码，用更多冗余换可靠性。MAC 层收不到能区分两者的信号，就一律按冲突处理、加倍退避——结果一个"离接入点太远"的节点会陷入"越失败越退避、越退避越慢"的恶性循环。用概率论的语言，这是一个二假设判别问题：$H_1$ = 冲突，$H_2$ = 低 SNR，而观测只有"帧对了没有"这一个粗糙读数，判别出错在所难免。工程上只能借助 RSSI（接收信号强度指示）这类旁证来猜，这正是后面 SWAN 测试床专门去测量 PRR 与 RSSI 关系的动机之一。
