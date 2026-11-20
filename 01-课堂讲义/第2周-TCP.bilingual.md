> <span style="color:#7f8c8d;">Source: src/课堂讲义/第2周-TCP.txt · CS144 Fall 2019 · 行内双语翻译版</span>

# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## TCP: Reliable Bytestreams over Datagrams（TCP：数据报之上的可靠字节流）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Datagrams（数据报）</span>

- IP 的服务是 Internet datagrams（互联网数据报）：<span style="color:#00838f;">在两个 host（主机）之间不可靠的、best-effort（尽力而为）的交付</span>。
- UDP 的服务是 user datagrams（用户数据报）：<span style="color:#00838f;">在两个 user-space（用户态）、unprivileged（非特权）程序之间不可靠的、best-effort（尽力而为）的交付</span>。
- 这两者都不可靠！数据报可能：
  - lost（丢失）
  - delivered more than once（被交付不止一次）
  - delivered out of order（乱序交付）
- 但你不必担心截断与损坏，因为有工具在手：
  - header checksum（首部校验和）（IP 提供）
  - data checksum（数据校验和）（UDP 提供）

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 把 IP 与 UDP 放在同一页对比是有意的：两者的区别只在交付范围——IP 交付给 host（主机），UDP 交付给 host（主机）上的具体进程（用 port number（端口号）区分），这正是你在 CSAPP（Computer Systems: A Programmer's Perspective，中译《深入理解计算机系统》，本课程的先修教材）第 11 章用过的 socket（套接字，即程序用来收发网络数据的端点，由 IP 地址与端口号标识）抽象的底层语义。UDP 数据报里的 data checksum（数据校验和）只保护 UDP 首部加数据，IP 的 header checksum（首部校验和）只保护 IP 首部——所以 UDP 能发现数据损坏，IP 只保证转发时首部没坏。注意 checksum（校验和）只是"检测"而非"纠正"：发现错了就整包丢弃，把恢复工作留给上层，这又回到 end-to-end principle（端到端原则，即把复杂的可靠性功能放到通信两端去实现、让网络核心保持简单）——网络层连"发现即重传"都不做。另外，"unprivileged（非特权）程序"这个措辞提醒我们：UDP 是应用程序就能直接使用的服务，不需要内核特殊权限；而直接构造 raw IP packet（原始 IP 包）往往需要 raw socket（原始套接字）和相应的管理员权限，这也是 CSAPP 网络编程部分强调的分层权限模型的现实投影。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">如果我们想要"可靠"的 exactly-once（恰好一次）数据报呢？</span>

- 基本方法：一遍又一遍地发送该数据报，直到收到 acknowledgment（确认）为止。
- 工具：
  - acknowledgment（确认，ACK）（由接收方发给发送方）
  - timer and timeout（定时器与超时）（放在发送方）
  - retransmission（重传）（由发送方执行）

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "反复发送直到确认"看似笨拙，却是所有可靠协议的原子操作。用概率论可以立刻算出它的代价：设单次交付成功率为 $p$，则发送次数服从 geometric distribution（几何分布，即独立重复发送直到第一次成功所需的次数），期望发送次数为 $1/p$。若丢包率 1%，平均发 1.01 次就够，完全可接受；但若丢包率 50%，期望就变成 2 次，且方差仍然很大——尾部延迟会很难看。这个简单模型还漏了一个致命问题：ACK（确认）本身也可能丢！发送方分不清"数据没到"还是"数据到了但确认没回来"，因此真正的 exactly-once（恰好一次）必须让接收方用 sequence number（序列号）去重——这正是下一页引入序列号的动机。在 CS144 的 lab 里你会亲手实现这套"确认—超时—重传"闭环，届时会发现真正难的不是发送，而是超时时间该设多长：设太短会重传已到达的包（浪费带宽），设太长则故障恢复慢，TCP 用自适应的 RTT（round-trip time，往返时间，即一个包从发出到收到它的确认所经历的时间）估计来折中。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">如果我们想要"可靠"的大消息呢？</span>

- 基本方法：把消息拆成带编号的 segment（段），把每一段作为一个可靠数据报发送出去。
- 工具：
  - acknowledgment（确认）（接收方 → 发送方）
  - timer and timeout（定时器与超时）（发送方）
  - retransmission（重传）（发送方）
  - sequence number（序列号）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">TCP：数据报之上的可靠字节流</span>

TCP 的基本规则：

- 每个方向上各有一条 ByteStream（字节流）。
- 每个字节都标记了它在序列中的位置。另外：
  - beginning of stream（流的开始）也算作序列中的一个位置；
  - end of stream（流的结束）同样算作一个。
- 接收方告诉发送方两件事：
  - 下一个需要 reassemble（重组）的字节是什么；
  - 还愿意再接受多少字节。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">每个 TCP segment（段）的格式</span>

- 发送方 → 接收方信息：
  - segment（段）内第一个字节的索引（sequence no.（序列号））
  - SYN（同步）位：这是流的开始吗？
  - segment（段）的 contents（内容）
  - FIN（结束）位：这是流的结束吗？
- 接收方 → 发送方信息：
  - 重组所需的下一字节的索引（ack no.（确认号））
  - 在此之外还愿意接受的字节数（window size（窗口大小））

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这页格式表与 CS144 的第一个编程作业直接对应：你要实现的正是把字节流切分、编号、重组的组件——stream reassembler（字节流重组器）。"SYN（同步）占一个 sequence number（序列号）、FIN（结束）也占一个 sequence number（序列号）"是初学者最容易踩的坑：TCP 的序列号空间不只是给数据字节的，流的两端边界各自消费一个序号，这样 acknowledgment（确认）才能无歧义地覆盖"我收到了你的流结束标志"本身。接收方报告的两样东西承担两个正交功能：ack no.（确认号）回答可靠性的"我还缺什么"，window size（窗口大小）回答 flow control（流量控制）的"别撑坏我的缓冲区"——它约束的是接收方缓冲区，与网络拥塞无关，congestion control（拥塞控制）另有 congestion window（拥塞窗口，cwnd）。你在 CSAPP 第 11 章写 echo server（回显服务器，即把收到的字节原样发回的服务器）时，内核 TCP 栈已经在替你完成这一切；CS144 的价值在于把这层黑盒拆开，让你亲手把 ACK（确认）、sequence number（序列号）与 window（窗口）拼回去。
