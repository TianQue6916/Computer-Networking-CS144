> <span style="color:#7f8c8d;">Source: src/课堂讲义/第2周-TCP.txt · CS144 Fall 2019 · 行内双语翻译版</span>

# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## TCP：数据报之上的可靠字节流（TCP: Reliable Bytestreams over Datagrams）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">数据报（Datagrams）</span>

- IP 的服务是互联网数据报（Internet datagrams）：<span style="color:#00838f;">在两台主机（host）之间不可靠、尽力而为（best-effort）的交付</span>。
- UDP 的服务是用户数据报（user datagrams）：<span style="color:#00838f;">在两个用户态（非特权）程序之间不可靠、尽力而为的交付</span>。
- 这两者都不可靠！数据报可能：
  - 丢失（lost）
  - 被交付不止一次（delivered more than once）
  - 乱序交付（delivered out of order）
- 但你不必担心截断与损坏，因为有工具在手：
  - 首部校验和（header checksum）（IP 提供）
  - 数据校验和（data checksum）（UDP 提供）

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 把 IP 与 UDP 放在同一页对比是有意的：两者的区别只在交付范围——IP 交付给主机（host），UDP 交付给主机上的具体进程（用端口号区分），这正是你在 CSAPP 第 11 章用过的套接字（socket）抽象的底层语义。UDP 数据报里的"数据校验和"只保护 UDP 首部加数据，IP 首部校验和只保护 IP 首部——所以 UDP 能发现数据损坏，IP 只保证转发时首部没坏。注意校验和只是"检测"而非"纠正"：发现错了就整包丢弃，把恢复工作留给上层，这又回到端到端原则——网络层连"发现即重传"都不做。另外，"非特权（unprivileged）程序"这个措辞提醒我们：UDP 是应用程序就能直接使用的服务，不需要内核特殊权限；而直接构造原始 IP 包（raw socket）则往往需要管理员权限，这也是 CSAPP 网络编程部分强调的分层权限模型的现实投影。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">如果我们想要"可靠"的恰好一次（exactly-once）数据报呢？</span>

- 基本方法：一遍又一遍地发送该数据报，直到收到确认为止。
- 工具：
  - 确认（acknowledgment，ACK）（由接收方发给发送方）
  - 定时器与超时（timer and timeout）（放在发送方）
  - 重传（retransmission）（由发送方执行）

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "反复发送直到确认"看似笨拙，却是所有可靠协议的原子操作。用概率论可以立刻算出它的代价：设单次交付成功率为 $p$，则重传次数服从几何分布（geometric distribution），期望发送次数为 $1/p$。若丢包率 1%，平均发 1.01 次就够，完全可接受；但若丢包率 50%，期望就变成 2 次，且方差仍然很大——尾部延迟会很难看。这个简单模型还漏了一个致命问题：ACK 本身也可能丢！发送方分不清"数据没到"还是"数据到了但确认没回来"，因此真正的 exactly-once 必须让接收方用序列号去重——这正是下一页引入序列号的动机。在 CS144 的 lab 里你会亲手实现这套"确认—超时—重传"闭环，届时会发现真正难的不是发送，而是超时时间该设多长：设太短会重传已到达的包（浪费带宽），设太长则故障恢复慢，TCP 用自适应的 RTT 估计来折中。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">如果我们想要"可靠"的大消息呢？</span>

- 基本方法：把消息拆成带编号的段（segment），把每一段作为一个可靠数据报发送出去。
- 工具：
  - 确认（acknowledgment）（接收方 → 发送方）
  - 定时器与超时（timer and timeout）（发送方）
  - 重传（retransmission）（发送方）
  - 序列号（sequence number）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">TCP：数据报之上的可靠字节流</span>

TCP 的基本规则：

- 每个方向上各有一条字节流（ByteStream）。
- 每个字节都标记了它在序列中的位置。另外：
  - 流的开始（beginning of stream）也算作序列中的一个位置；
  - 流的结束（end of stream）同样算作一个。
- 接收方告诉发送方两件事：
  - 下一个需要重组（reassemble）的字节是什么；
  - 还愿意再接受多少字节。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">每个 TCP 段（segment）的格式</span>

- 发送方 → 接收方信息：
  - 段内第一个字节的索引（序列号，sequence no.）
  - SYN 位：这是流的开始吗？
  - 段的内容（contents）
  - FIN 位：这是流的结束吗？
- 接收方 → 发送方信息：
  - 重组所需的下一个字节的索引（确认号，ack no.）
  - 在此之外还愿意接受的字节数（窗口大小，window size）

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这页格式表与 CS144 的第一个编程作业直接对应：你要实现的正是把字节流切分、编号、重组的组件（stream reassembler，字节流重组器）。"SYN 占一个序列号、FIN 也占一个序列号"是初学者最容易踩的坑：TCP 的序列号空间不只是给数据字节的，流的两端边界各自消费一个序号，这样确认才能无歧义地覆盖"我收到了你的流结束标志"本身。接收方报告的两样东西承担两个正交功能：确认号（ack no.）回答可靠性的"我还缺什么"，窗口大小（window size）回答流量控制（flow control）的"别撑坏我的缓冲区"——它约束的是接收方缓冲区，与网络拥塞无关，拥塞控制另有拥塞窗口（cwnd）。你在 CSAPP 第 11 章写 echo 服务器时，内核 TCP 栈已经在替你完成这一切；CS144 的价值在于把这层黑盒拆开，让你亲手把 ACK、序列号与窗口拼回去。
