# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

> <span style="color:#7f8c8d;">Source: src/实验讲义/Lab 4 - TCP连接.txt · CS144 Fall 2019 · 行内双语翻译版</span>

<span style="color:#2471a3;">**[lecture]**</span>

## <span style="color:#c0392b;">Lab 4：登顶——完整的 TCP（the summit, TCP in full）</span>

> <span style="color:#7f8c8d;">截止：11 月 6 日（周三）17:00。实验课：10 月 29 日与 11 月 5 日（周二）19:30–22:00，地点 STLC114。</span>

<span style="color:#2471a3;">**[section]**</span>

### <span style="color:#c0392b;">0 协作政策（Collaboration Policy）</span>

编程作业必须独立完成：你为编程作业提交的代码必须全部由你本人编写，作业随附发放给你的代码除外。请不要从 Stack Overflow、GitHub 或其他来源复制粘贴代码。如果你的代码以网络上或其他地方找到的示例为基础，请在提交的源代码注释中注明相应 URL。

与他人合作：不得向任何人展示你的代码，不得查看任何人的代码，也不得查看往年的解答。你可以与其他同学讨论作业，但不要抄任何人的代码。如果你与另一位同学讨论了作业，请在提交的源代码注释中写明其姓名。更多细节请参阅课程行政讲义，如有不明请在 Piazza 上提问。

Piazza：欢迎在 Piazza 上提问，但请不要发布任何源代码。

<span style="color:#2471a3;">**[section]**</span>

### <span style="color:#c0392b;">1 概览（Overview）</span>

在 Lab 0 中，你实现了流量控制字节流（flow-controlled byte stream）这一抽象：ByteStream。

在 Lab 1、2 和 3 中，你实现了在该抽象与互联网提供的抽象——不可靠数据报（unreliable datagram，IP 或 UDP）——之间互相转换的工具。

现在，你即将登顶：一个可用的 TCPConnection，它组合你的 TCPSender 与 TCPReceiver，能以至少 100 Mbit/s 的速度与其他实现对话。图 1 展示了总体设计。

#### <span style="color:#c0392b;">图 1：TCP 实现中的模块布局与数据流（Figure 1: The arrangement of modules and dataflow in your TCP implementation）</span>

图示内容（依据图中文字推断）：应用通过 CS144TCPSocket 对连接进行写入（"from Internet writes (to the socket)"）与读取（"reads (from the socket)"）。CS144TCPSocket 内部是 TCPConnection，后者包含 TCPSender 与 TCPReceiver 两个子模块：

- 出站方向：应用写入的数据进入 TCPSender 持有的 ByteStream（"outbound"）；TCPSender 产出 TCPSegment（字段：seqno、SYN、payload、FIN），并从 TCPReceiver 拿到（ackno, window_size）一并填入；TCPConnection 把段交给外层，封装为以 TCPSegment 为 payload 的 IPv4Datagram（附加端口号），发往 Internet。
- 入站方向：TCPSegment 到达后调用 TCPConnection 的 `segment_received(TCPSegment)`，交给 TCPReceiver（内部为 StreamReassembler + ByteStream（"inbound"））重组为入站字节流；同时把（ackno, window_size）回送给 TCPSender。
- 图中标注的关键调用：`segment_received(TCPSegment)`、`ack_received(ackno, window_size)`、`segments_out()`。

<span style="color:#2471a3;">**[section]**</span>

### <span style="color:#c0392b;">2 开始上手（Getting started）</span>

你的 TCPConnection 实现将使用与 Lab 0–3 相同的 Sponge 库，并附带新增的类与测试。我们提供了把 TCP 报文段读写进用户数据报有效载荷（"TCP-over-UDP"）与互联网数据报（"TCP/IP"）的支持代码。我们还提供了一个类（CS144TCPSocket），它包装你的 TCPConnection，使其表现得像普通的流套接字（stream socket）——就像你在 Lab 0 实现 webget 时用过的 TCPSocket 一样。到本实验结束时，你将稍微修改你的 webget，让它使用你自己的 TCP 实现。开始步骤如下：

1. 确保你已提交（commit）Lab 3 的全部解答。请不要修改 libsponge 顶层目录之外的任何文件，也不要修改 webget.cc，否则在合并 Lab 4 起始代码时可能遇到麻烦。
2. 在实验作业仓库内运行 `git fetch`，获取实验作业的最新版本。
3. 运行 `git merge origin/lab4-startercode` 下载 Lab 4 的起始代码。
4. 在 build 目录内编译源代码：`make`（可以运行 `make -j4`，用四个处理器并行编译）。
5. 在 build 目录之外，打开并开始编辑 writeups/lab4.md。这是你实验报告（writeup）的模板，将随你的提交一起上交。

<span style="color:#2471a3;">**[section]**</span>

### <span style="color:#c0392b;">3 Lab 4：TCP 连接（The TCP connection）</span>

本周你将完成一个可与互联网上数十亿台计算机和移动设备互通的可用 TCP 实现。你已经完成了大部分工作：实现了发送端与接收端。本周的任务是把它们"接线"（wire up）成一个对象（TCPConnection），并处理一些连接全局性的日常事务。

回顾：TCP 可靠地传输一对流量控制字节流，每个方向一条。TCP 连接由两方参与，每一方同时扮演自己出站字节流的"发送端"（sender）与对端入站字节流的"接收端"（receiver）：

（图示：两端各有一个 TCPConnection，各自的 TCPSender 从出站 ByteStream 读取数据，图中两个方向的数据分别以 "Hello there!" 与 "General Kenobi!" 示意；TCPReceiver 把收到的段重组为入站字节流。）

这两方（上图中的 "A" 与 "B"）称为连接的"端点"（endpoints）或"对等方"（peers）。你的 TCPConnection 扮演其中一方。它负责收发报文段，确保 sender 与 receiver 对进出的段中"它们各自关心的字段"知情，并有机会参与填写。

**接收报文段（Receiving segments）。** TCPConnection 会从互联网收到 TCPSegment，然后：

- 若 ack 标志被置位，把入站段中 TCPSender 关心的字段——ackno 与 window size——告知 TCPSender；
- 把段交给 TCPReceiver，让它检查入站段中自己关心的字段：seqno、SYN、payload 与 FIN。

**发送报文段（Sending segments）。** TCPConnection 会通过互联网发送 TCPSegment：

- 每当 TCPSender 把一个段压入其出站队列时（它已设置自己负责的出站字段：seqno、SYN、payload 与 FIN）；
- 发送段之前，TCPConnection 会向 TCPReceiver 索取它负责的出站字段：ackno 与 window size。若存在 ackno（脚注 1），就设置 ack 标志，并把这两个字段填入 TCPSegment。

> <span style="color:#7f8c8d;">**脚注 1：** 记住 TCPReceiver::ackno() 返回的是一个 optional 值。</span>

于是，每个 TCPSegment 的整体结构如下，图中以不同颜色区分"sender 字段"与"receiver 字段"：

| 字段 | 英文名 |
|---|---|
| 源端口号（sport） | Source Port Number |
| 目的端口号（dport） | Destination Port Number |
| 序列号（seqno） | Sequence Number —— 由发送端负责填写 |
| 数据偏移（doff） | Data Offset |
| 确认号（ackno） | Acknowledgement Number —— 由接收端负责填写 |
| 标志位 | URG ACK PSH RST SYN FIN |
| 窗口大小（win） | Window Size —— 由接收端负责填写 |
| 校验和（cksum） | Checksum |
| 紧急指针（uptr） | Urgent Pointer |
| 选项/填充 | Options / Padding |
| 有效载荷 | Payload |

TCPConnection 的完整接口在类文档（class documentation）中，请花些时间通读。你的实现大部分工作是"把 TCPConnection 的公共 API 接线到 TCPSender 与 TCPReceiver 中相应的例程上"。你要尽可能把重活推给已实现的 sender 与 receiver。不过并非一切都这么简单，有些微妙之处涉及整个连接的"全局"行为。最难的部分，是决定何时彻底终止一个 TCPConnection、宣布它不再"active"（活动）。

接下来是若干 FAQ，以及你需要处理的边界情况细节。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 实验预告只需 100–150 行，这不是低估，而是分层设计的胜利：TCPSender 与 TCPReceiver 已经把"可靠传输"的全部重活干完，TCPConnection 只是一层薄胶水。你写它的体验会类似把两个独立模块接成一台状态机：难的不是代码量，而是决定"哪一层该知道什么"。这与 CSAPP ch11 的 RIO 包是同一种思想——Robust I/O 从不重新实现 read/write，它只是封装了 EINTR 重试与缓冲管理；同样，TCPConnection 不重新实现重组与重传，只维护一张"字段分工表"（seqno/SYN/payload/FIN 归发送端填，ackno/window size 归接收端填），并把每个公共方法转发给正确的下属。实现时有个自检标准：一旦你发现自己在 tcp_connection.cc 里写重组或重传逻辑，几乎可以肯定是放错了层。这种"每层只补齐全局信息、不重复下层功能"的纪律，在真实的内核协议栈里同样成立。

<span style="color:#2471a3;">**[section]**</span>

### <span style="color:#c0392b;">4 常见问题与特殊情况（FAQs and special cases）</span>

<span style="color:#2471a3;">**[question]**</span> **你们期望写多少代码？**

总体上，我们预计实现（在 tcp_connection.cc 中）总共需要约 100–150 行代码。完成时，测试套件将广泛测试你与自己的实现、以及与 Linux 内核 TCP 实现之间的互操作性（interoperability）。

<span style="color:#2471a3;">**[question]**</span> **我该如何入手？**

最好的入手方式大概是先把一些"普通"方法接线到 TCPSender 与 TCPReceiver 中相应的调用上。这可能包括 `remaining_outbound_capacity()`、`bytes_in_flight()`、`unassembled_bytes()` 之类。然后你可以选择实现"写者"方法：`connect()`、`write()`、`end_input_stream()`。其中一些方法可能需要对出站 ByteStream（由 TCPSender 持有）做些操作，并把这一点告诉 TCPSender。

你可以在尚未实现完所有方法之前就运行测试套件（`make check`）；测试失败信息能给你提示，告诉你下一步该处理什么。

<span style="color:#2471a3;">**[question]**</span> **应用如何从入站流读取？**

TCPConnection::inbound_stream() 已在头文件中实现。

<span style="color:#2471a3;">**[question]**</span> **TCPConnection 需要什么花哨的数据结构或算法吗？**

不需要，真的不需要。重活全由你已实现的 TCPSender 与 TCPReceiver 完成。这里的工作就是把一切接线好，并处理一些无法轻易归入 sender 与 receiver 的、连接范围的全局微妙问题。

<span style="color:#2471a3;">**[question]**</span> **TCPConnection 实际上如何发送一个段？**

与 TCPSender 类似——压入 `segments_out` 队列。对你的 TCPConnection 而言，压入即视为已发送。很快拥有者就会来弹出它（通过公共访问器方法 `segments_out()`）并真正发送。

<span style="color:#2471a3;">**[question]**</span> **TCPConnection 如何感知时间流逝？**

与 TCPSender 类似——tick() 方法会被周期性调用。请不要用任何其他方式获取时间——tick 方法是你感知时间流逝的唯一途径。这能保证确定性与可测试性。

<span style="color:#2471a3;">**[question]**</span> **如果入站段设置了 rst 标志，TCPConnection 该怎么做？**

这个标志（"reset"，重置）意味着连接即刻死亡。若你收到带 RST 的段，应把入站与出站 ByteStream 都置为 error（错误）状态，之后任何对 TCPConnection::active() 的调用都应返回 false。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 收到或发出 RST 就立即处死连接，这条语义在 CSAPP ch11 有一个你踩过的对偶。在 webget 里，如果对端进程崩溃，你继续 `write()` 时数据先进入本机内核缓冲、调用照常成功；等对端 TCP 回送 RST 后，下一次写才会失败，进程收到 SIGPIPE 信号或 EPIPE 错误——这就是为什么 RIO 包的 `rio_writen` 必须检查错误码。本 Lab 把这一切搬到用户态：你要自己把双向 ByteStream 置为 error 状态、让 `active()` 返回 false，并保证析构一个仍 active 的连接时主动发 RST，否则对端会拿着未完成的流一直等。实现上推荐写一个私有的"处死连接"助手函数，把"置错、发 RST、标记非 active"三步收拢，避免两条触发路径行为不一致。对比 §5 的正常关闭，你会看到 RST 的存在意义：对端重启、端口被滥用、连续 8 次重传失败这类错误没有协商余地，慢慢收尾只会白白占用内存与端口号。

<span style="color:#2471a3;">**[question]**</span> **我什么时候该发送置位 rst 的段？**

有两种情况需要中止（abort）整个连接：

1. sender 连续重传太多次仍未成功（超过 TCPConfig::MAX_RETX_ATTEMPTS，即 8 次）。
2. TCPConnection 析构函数被调用时连接仍处于 active 状态（active() 返回 true）。

发送一个置位 RST 的段，与收到一个的效果类似：连接死亡、不再 active()，两个 ByteStream 都置为 error 状态。

<span style="color:#2471a3;">**[question]**</span> **等等，可我该怎么造一个能置 RST 标志的段？序列号是多少？**

任何出站段都需要正确的序列号。你可以调用 TCPSender 的 `send_empty_segment()` 方法，强制它生成一个带正确序列号的空段；也可以调用它的 `fill_window()` 让它填充窗口（若它有未发送的信息——比如流中的字节或 SYN/FIN——就生成段）。

<span style="color:#2471a3;">**[question]**</span> **ack 标志的用途是什么？难道不是总有 ackno 吗？**

几乎每个 TCPSegment 都有 ackno 且置位 ack 标志。例外只在连接刚开始时——接收方还没有任何东西可确认。

对出站段，你要尽可能设置 ackno 与 ack 标志：即每当 TCPReceiver 的 ackno() 方法返回一个有值的 `std::optional<WrappingInt32>`（可用 has_value() 测试）。

对入站段，只有当 ack 字段被置位时才查看 ackno；若置位，就把该 ackno（与 window size）交给 TCPSender。

<span style="color:#2471a3;">**[question]**</span> **收到一个段时，如果 TCPReceiver 抱怨该段与窗口没有重叠、不可接受（segment_received() 返回 false），我该怎么办？**

此时 TCPConnection 需要确保向对端回发一个段，附上当前的 ackno 与 window size。这有助于纠正一个"糊涂了"的对端。

<span style="color:#2471a3;">**[question]**</span> **好，那如果 TCPConnection 收到一个段，而 TCPSender 抱怨 ackno 无效（ack_received() 返回 false）呢？**

同样的答案！

<span style="color:#2471a3;">**[question]**</span> **如果 TCPConnection 收到一个段，一切都很顺利呢？我还需要回复吗？**

如果该段占用了任何序列号，你就必须确保它被确认——至少要有一个段发回对端，带上合适的序列号、新的 ackno 与 window size。你可能不需要专门做什么，因为 TCPSender 常常会在 ack_received() 中决定发送新段（因为窗口腾出了更多空间）。但即使 TCPSender 没有更多数据要发，你也需要确保入站段以某种方式被确认。

<span style="color:#2471a3;">**[question]**</span> **那如果 TCPConnection 对每个段都确认呢，哪怕它不占用任何序列号？**

不是好主意！两个对等方会陷入无限互发 ACK 的"乒乓"（ping-pong）。

<span style="color:#2471a3;">**[question]**</span> **我怎么解读这些"状态"名字（如 "stream started" 或 "stream ongoing"）？**

查看 libsponge/tcp_helpers/tcp_state.hh 与 tcp_state.cc 文件。

<span style="color:#2471a3;">**[question]**</span> **如果 TCPReceiver 想通告的窗口大小超出 TCPSegment::header().win 字段能装下的范围，我该发送什么窗口大小？**

发送你能装下的最大值。`std::numeric_limits` 类可能对你有帮助。

<span style="color:#2471a3;">**[question]**</span> **TCP 连接最终何时才算"完事"？active() 什么时候可以返回 false？**

请看下一节。

<span style="color:#2471a3;">**[question]**</span> **这份 PDF 发布之后又新增了 FAQ，我去哪里读？**

请定期查看网站（https://cs144.github.io/lab_faq.html ）与 Piazza。

<span style="color:#2471a3;">**[section]**</span>

### <span style="color:#c0392b;">5 TCP 连接的结束：达成共识需要下功夫（consensus takes work）</span>

TCPConnection 的一个重要职能，是判断 TCP 连接何时真正"完事"。此时，实现要释放对本地端口号的独占占用、停止为入站段回发确认、把连接视为历史，并让 active() 方法返回 false。

连接有两种结束方式。**非正常关闭（unclean shutdown）**：TCPConnection 发送或收到一个置位 RST 标志的段。此时出站与入站 ByteStream 都应处于 error 状态，active() 可以立即返回 false。

**正常关闭（clean shutdown）**则是在不出错的前提下到达"完事"（active() = false）。这更复杂，但它是一件美好的事，因为它尽可能确保两条 ByteStream 都被完整、可靠地交付给接收对端。下一节（§5.1）给出正常关闭何时发生的实用结论，你可以直接跳过去。

很好，你还在读。由于两将军问题（Two Generals Problem），不可能保证两个对等方都达成正常关闭，但 TCP 已经非常接近。下面是做法。从一个对等方（某个 TCPConnection，我们称之为"本地"对等方）的视角，它与"远端"对等方的连接要正常关闭，有四个前提：

- **前提 #1**：入站流已被完全组装，且已结束。
- **前提 #2**：出站流已被本地应用结束，且已完全发送（包括"流已结束"这一事实本身，即带 FIN 的段）给远端对等方。
- **前提 #3**：出站流已被远端对等方完全确认。
- **前提 #4**：本地 TCPConnection 确信远端对等方能满足前提 #3。这是最烧脑的部分。它可以通过两种互相替代的方式发生：

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 两将军问题值得展开：两支军队驻扎在山的两侧，只能派可能被截获的信使约定共同进攻时刻；哪怕将领 A 收到了 B 的确认，B 也无法知道 A 收到了确认，于是"我确认你已确认"可以无限回归下去。用 6.042J 的语言说：在"消息可丢失"的信道模型上，不存在保证双方达成共识且必然终止的协议。你在 CSAPP ch11 调 `close()` 时从没想过这些，是因为所有不确定性都被内核的 TIME_WAIT 状态默默消化了；本 Lab 让你在用户态亲手收尾连接，才会发现"关好一个连接"比三次握手更难。TCP 的答案是工程妥协而非理论突破：被动关闭方（选项 B）能 100% 确定共识达成，主动关闭方（选项 A）只能等 10×RTO 假装确定——把"不可能保证"换成"足够确信，且错误后果可容忍"。这与你学概率论时"用置信水平替代确定性"的思路一脉相承。

**选项 A：两条流都结束后 linger（悬停等待）。** 前提 #1 至 #3 已满足，且远端对等方"看起来"已经收到了本地对等方对整条流的确认。本地对等方无法确知这一点——TCP 不保证 ack 的可靠送达（它不确认 ack）。但本地对等方相当确信远端已收到它的 ack，因为远端似乎没有在重传任何东西，而且本地对等方已等待了一段时间来确保这一点。

具体而言：<span style="color:#00838f;">当前提 #1 至 #3 满足，且本地对等方自上次收到远端任何段以来已经过了至少 10 倍初始重传超时（cfg.rt_timeout）的时间，连接即告完成。</span>这叫两条流都结束后的"linger"，用以确保远端没有在重传任何我们需要确认的东西。它的确意味着 TCPConnection 需要多存活一段时间（脚注 2）——即使 TCPSender 与 TCPReceiver 早已彻底完成工作、两条流都已结束，也要继续独占本地端口号，并可能为入站段回发 ack。

> <span style="color:#7f8c8d;">**脚注 2：** 在生产级 TCP 实现中，linger 定时器（也叫 time-wait 定时器，或两倍的最大报文段生存期 MSL）通常约为 60 或 120 秒。在连接实际上已经结束后还要把端口号保留这么久，可能挺难熬——尤其当你想启动一个绑定同一端口号的新服务器时：没人想等两分钟。SO_REUSEADDR 套接字选项基本上让 Linux 无视这一保留，调试或测试时很方便。</span>

**选项 B：被动关闭（passive close）。** 前提 #1 至 #3 已满足，且本地对等方 100% 确定远端对等方能满足前提 #3。既然 TCP 不确认确认，这怎么可能？因为远端对等方是先结束自己流的那一方。

这条规则为什么成立？这是最烧脑的地方，你不读下文也能完成本实验，但它很有趣，直指两将军问题以及"在不可靠网络上追求可靠性的内在限制"。它成立的原因是：在收到并组装出远端对等方的 FIN 之后（前提 #1），本地对等方发送过一个序列号大于以往任何段的新段（至少，为满足前提 #2 它必须发送自己的 FIN 段），而该段同时携带确认远端 FIN 位的 ackno。远端对等方确认了那个段（为满足前提 #3），这意味着远端对等方必然也看到了本地对等方对其 FIN 的确认。这就保证了远端对等方必然能满足它自己的前提 #3。这一切意味着本地对等方无需 linger 就能满足前提 #4。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 选项 B 的论证结构值得单独拆开看，它是典型的不变量归纳推理，而非枚举测试。链条是：本地收到并组装出对端的 FIN（前提 #1）→ 本地此后必然发送过某个序列号大于历史所有段的段（至少有为满足前提 #2 而发的自己的 FIN），且该段携带确认对端 FIN 的 ackno → 对端要满足它自己的前提 #3 就必须 ACK 这个段 → 而能 ACK 就意味着对端必然读到了那个 ackno。这里用的是"严格递增序列号"这一单射式论证手法，与 6.042J 中用单调性与鸽笼原理建立"某事件必然发生过"的证明同构：序列号空间的全序把不可靠信道上"互相确认"的无限回归压缩成了一轮。结论可以一句话复述：若对端先结束流，本地就无需 linger——这正是 5.1 节 linger_after_streams_finish 置 false 的条件。写加分题（extra credit）时，试试把这条链用自己的话讲清楚。

呼！我们说了这很烧脑。实验报告加分题（Extra credit）：你能想出更好的解释方式吗？

底线是：如果 TCPConnection 的入站流在它发送过任何 FIN 段之前就结束了，那么 TCPConnection 无需在两条流结束后 linger。

#### <span style="color:#c0392b;">5.1 TCP 连接的结束（实用总结）</span>

实际上，这一切意味着你的 TCPConnection 有一个成员变量 linger_after_streams_finish，通过 state() 方法暴露给测试装置。该变量初始为 true。如果入站流在 TCPConnection 到达其出站流 EOF 之前就结束，这个变量需要被置为 false。

在任何前提 #1 至 #3 都已满足的时刻，若 linger_after_streams_finish 为 false，连接即告"完成"（active() 应返回 false）。否则你需要 linger：只有自最后一个段被收到以来经过足够时间（10 × cfg.rt_timeout）之后，连接才算完成。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这一节的 linger 机制解释了一个你在 CSAPP ch11 实验中可能撞过的坑：服务端进程重启后 `bind()` 报 "Address already in use"。根因就在这里——连接"完成"后，TCPConnection 还要占着本地端口号等 10 × cfg.rt_timeout（生产实现里对应 TIME_WAIT 状态，通常 60–120 秒，参见脚注 2 的 SO_REUSEADDR），期间内核不会把端口分给新套接字。为什么非要占着？因为 linger 期间对端最后的重传段仍可能到达，必须有持有该端口的实体回应 ACK，否则对端永远无法确认"我的 FIN 被收到了"，前提 #3 在它那边就悬而未决。所以占端口不是实现缺陷，而是"必须有人守到最后"的必然代价。理解了这一点，linger_after_streams_finish 为什么默认为 true、又在什么条件下置 false，就顺理成章了。

<span style="color:#2471a3;">**[section]**</span>

### <span style="color:#c0392b;">6 性能（Performance）</span>

完成你的 TCP 实现、并通过 `make check` 运行的所有测试之后，请先 commit（提交）！然后测量系统性能，把它提升到至少 100 Mbit/s（兆比特每秒）。

在 build 目录中运行 `./apps/tcp_benchmark`。如果一切顺利，你会看到类似这样的输出：

```
user@computer:~/sponge/build$ ./apps/tcp_benchmark

CPU-limited throughput                  : 1.78 Gbit/s
CPU-limited throughput with reordering: 1.21 Gbit/s
```

要在本实验获得满分，你的性能需要在两行上都达到至少 "0.10 Gbit/s"（即 100 Mbit/s）。你可能需要剖析（profile）代码、推理哪里慢，也可能需要改进某些关键模块（例如 ByteStream 或 StreamReassembler）的实现才能达标。

请在实验报告中报告你达到的速度数字（有/无乱序两种）。

如果愿意，欢迎尽量优化代码，但请不要以牺牲 CS144 其他部分（包括本实验的其他部分）为代价。性能快于 100 Mbit/s 不加分——超出最低限度的改进只为你自己的满足与学习。如果你的实现在不改变任何公共接口的前提下比我们的参考实现（脚注 3）还快，我们很乐意向你请教你是怎么做到的。

> <span style="color:#7f8c8d;">**脚注 3：** 我们的参考实现运行在 2011 年的 Intel Core i7-2600K CPU @ 4.40GHz 上，系统为 Ubuntu 19.04、Linux 5.0.0-31-generic #33-Ubuntu（开启针对 Meltdown/Spectre 等漏洞的默认缓解措施），编译器为 g++ 8.3.0、默认（"Release"）编译选项。CPU 受限吞吐量（第一行）为 7.18 Gbit/s，第二行（带乱序）为 6.84 Gbit/s。</span>

<span style="color:#2471a3;">**[section]**</span>

### <span style="color:#c0392b;">7 再访 webget（webget revisited）</span>

是时候绕场庆祝了！还记得你在 Lab 0 写的 webget.cc 吗？它用的是 Linux 内核提供的 TCP 实现（TCPSocket）。我们希望你把它切换成使用你自己的 TCP 实现，其余一律不改。我们认为你只需要：

- 把 `#include "socket.hh"` 替换为 `#include "tcp_sponge_socket.hh"`。
- 把 TCPSocket 类型替换为 CS144TCPSocket。
- 在 get_URL() 函数的末尾，添加一次 socket.wait_until_closed() 调用。

**为什么要做这个？** 通常，即使用户进程已经退出，Linux 内核也会负责等待 TCP 连接到达"正常关闭"（并放弃其端口保留）。但你的 TCP 实现全部在用户空间，除了你的程序，没有别人来跟踪连接状态。加上这个调用，套接字就会一直等待，直到你的 TCPConnection 报告 active() = false。

重新编译，运行 `make check_webget`，确认你已功德圆满：你在自己的完整 TCP 实现之上写了一个基本的 Web 抓取器，而且它仍能成功与真实 Web 服务器对话。如果遇到问题，试试手动运行程序：`./apps/webget cs144.keithw.org /hasher/xyzzy`。你会在终端得到一些可能有用的调试输出。

<span style="color:#2471a3;">**[section]**</span>

### <span style="color:#c0392b;">8 开发与调试建议（Development and debugging advice）</span>

1. 请在 tcp_connection.cc 文件中实现 TCPConnection 的公共接口（以及任何你想要的私有方法或函数）。你可以在 tcp_connection.hh 中为 TCPConnection 类添加任意私有成员。
2. 我们预计总共约 100–150 行代码。你不需要任何花哨的数据结构或算法。
3. 编译后可以用 `make check` 测试你的代码。这会运行一个相当全面的测试套件（159 个测试）。许多测试确认：你的 TCP 实现能在各种丢包与双向数据传输组合下，与 Linux 的 TCP 实现或与你自己的实现无错地传输文件（脚注 4）。
4. 请重读 Lab 0 文档与在线 FAQ 中"使用 Git"一节，并记住让代码保持在其发布时的 Git 仓库的 master 分支上。做小的提交，写清楚"改了什么、为什么改"的好提交信息。
5. 请努力让你的代码对将按代码风格评分的 CA（课程助理）可读。为变量使用合理清晰的命名约定。用注释解释复杂或微妙的代码片段。使用防御式编程——显式检查函数前置条件或不变量，任何出错之处都抛出异常。在设计中使用模块化——识别共同的抽象与行为，并尽可能将其抽出。成块的重复代码和巨大的函数会让你的代码难以读懂。
6. 也请遵守 Lab 0 文档描述的"现代 C++"风格。cppreference 网站（https://en.cppreference.com ）是极好的资源，不过做这些实验你不需要任何高深的 C++ 特性。（有时你可能需要用 move() 函数来传递不可拷贝的对象。）
7. 如果你遇到了段错误（segmentation fault），那说明真的出了问题！我们希望你用一种让段错误极其罕见的安全编程风格来写代码（不用 malloc()、不用 new、不用指针、在不确定处写抛异常的安全检查等）。尽管如此，调试时你可以用 `cmake .. -DCMAKE_BUILD_TYPE=RelASan` 配置 build 目录，启用编译器的"消毒器"（sanitizers）来检测内存错误与未定义行为，并在其发生时给出漂亮的诊断。你也可以使用 valgrind 工具。也可以用 `cmake .. -DCMAKE_BUILD_TYPE=Debug` 配置并使用 GNU 调试器（gdb）。记住这些设置只用于调试——它们会显著拖慢程序的编译与执行。要回到"Release"模式，最可靠/最省心的办法就是删掉 build 目录重建一个。

> <span style="color:#7f8c8d;">**脚注 4：** 在测试名中，"c" 表示你的代码是客户端（先发送 SYN 的对等方），"s" 表示你的代码是服务器。字母 "u" 表示测试 TCP-over-UDP，"i" 表示测试 TCP-over-IP（TCP/IP）。字母 "n" 表示与 Linux 的 TCP 实现互操作。"S" 表示你的代码在发送数据，"R" 表示你的代码在接收数据，"D" 表示双向都在传数据。测试名末尾，小写 "l" 表示接收（入站段）方向有丢包，大写 "L" 表示发送（出站段）方向有丢包。</span>

<span style="color:#2471a3;">**[section]**</span>

### <span style="color:#c0392b;">9 提交（Submit）</span>

1. 提交时，请只改动 libsponge 顶层的 .hh 与 .cc 文件。在这些文件内，你可以按需添加私有成员，但请不要改动任何类的公共接口。
2. 上交任何作业之前，请按顺序运行：
   - (a) `make format`（规范化代码风格）
   - (b) `git status`（检查未提交的改动——如果有，就提交！）
   - (c) `make`（确保代码能编译）
   - (d) `make check`（确保自动测试通过）
3. 在 writeups/lab4.md 中写一份报告。该文件应是一份约 20–50 行、每行不超过 80 字符的文档，便于阅读。报告应包含以下小节：
   - (a) Program Structure and Design（程序结构与设计）。描述你的代码体现的高层结构与设计取舍。无需详细讨论你从起始代码继承了什么。请利用这个机会突出重要设计点，并为评分 TA 提供这些部分的更多细节。强烈建议用小标题和提纲让报告尽可能易读。请不要简单地把你的程序翻译成一段英文。
   - (b) Implementation Challenges（实现难点）。描述你觉得最棘手的代码部分并解释原因。反思你如何克服这些困难、是什么最终帮你理解了困扰你的概念。你如何设法让代码维持你的假设、不变量与前置条件？这件事你觉得容易还是困难？你如何调试和测试代码？
   - (c) Remaining Bugs（遗留缺陷）。尽量指出并解释代码中仍存在的任何缺陷（或未处理的边界情况）。
4. 也请填写作业花费的小时数以及任何其他意见。
5. 准备好提交时，请按照 https://cs144.github.io/submit 的说明操作。提交前请确保已提交（commit）所有打算提交的内容。
6. 如有问题，请尽快在周二晚实验课上告知课程 staff，或在 Piazza 上提问。祝好运！
