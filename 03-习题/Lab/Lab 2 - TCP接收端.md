# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Lab 2: the TCP receiver（Lab 2：TCP 接收端）

> <span style="color:#7f8c8d;">Source: 实验讲义/Lab 2 - TCP接收端.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

> <span style="color:#7f8c8d;">脚注（源文噪声说明）：原 PDF 经 pdftotext 提取后存在系统性噪声：上标丢失（原文 "232 - 2" 实为 $2^{32} - 2$、"232 + 17" 实为 $2^{32} + 17$、"233 + 17" 实为 $2^{33} + 17$、"234 + 17" 实为 $2^{34} + 17$，译文已按上下文复原）、下划线丢失（如 `wrapping integers.hh` 应为 `wrapping_integers.hh`、`length in sequence space()` 应为 `length_in_sequence_space()`）、箭头符号丢失（"Convert absolute seqno seqno" 实为 "absolute seqno → seqno"）、代码与正文混排。以下译文已按上下文复原，不逐一标注。</span>

---

**截止时间**：10 月 11 日（周五）下午 5 点。实验课：10 月 8 日（周二）晚 7:30–10:00，地点 STLC114。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">0 合作政策（Collaboration Policy）</span>

**编程作业必须是你自己的工作**：除了我们随作业发给你的代码之外，你提交的编程作业代码必须全部自己写。请不要从 Stack Overflow、GitHub 或其他来源复制粘贴代码。如果你的代码基于网上或其他地方找到的示例，请在提交的源代码注释中注明出处 URL。

**与他人合作**：不得把你的代码给任何人看，不得看任何人的代码，也不得看往年的解答。你可以和其他同学讨论作业，但不要抄任何人的代码。如果你和另一位同学讨论了作业，请在提交的源代码注释中写出对方姓名。更多细节请参阅课程管理讲义，有任何不清楚的地方请在 Piazza 上提问。

**Piazza**：欢迎在 Piazza 上提问，但请不要张贴任何源代码。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">1 概览（Overview）</span>

在 Lab 0 中，你实现了流量控制的字节流抽象（ByteStream）。

在 Lab 1 中，你创建了一个模块，接收一系列摘自同一条字节流的子串，并把它们重新组装回原始的 ByteStream，同时把内存消耗限制在给定的数量（capacity）之内。

现在，在 Lab 2 中，你将实现 TCP 中处理入站字节流的部分：TCPReceiver。当你编写 StreamReassembler 和 ByteStream 时，其实已经完成了这里大部分"算法性"的工作；本周主要是把这些类连接到 TCP 的格式上。这将涉及思考 TCP 如何表示每个字节在流中的位置——即"序列号"（sequence number）。TCPReceiver 负责告诉发送方：(a) 它已成功组装了多少入站字节流（这称为"确认"，acknowledgment），(b) 发送方现在允许发送的字节范围（"流量控制窗口"，flow control window）。

下周，在 Lab 3 中，你将实现 TCP 中处理出站字节流的部分：TCPSender。最后，在 Lab 4 中，你将把前几个实验的成果组合成一个能工作的 TCP 实现：一个包含 TCPSender 和 TCPReceiver 的 TCPConnection。你将用它与世界各地的真实服务器对话。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">2 起步（Getting started）</span>

你的 TCPReceiver 实现将使用你在 Lab 0 和 Lab 1 中用过的同一个 Sponge 库，外加一些新的类和测试。开始步骤：

1. 确保你已经 commit 了 Lab 1 的全部解答。请不要修改 libsponge 目录顶层之外的任何文件，也不要修改 webget.cc——否则合并初始代码时可能出问题。
2. 在实验作业仓库内运行 `git fetch`，获取实验作业的最新版本。
3. 运行 `git merge origin/lab2-startercode` 下载 Lab 2 的初始代码。
4. 在你的 build 目录中编译源码：`make`（可以运行 `make -j4`，编译时使用四个处理器）。
5. 在 build 目录之外，打开并开始编辑 writeups/lab2.md 文件。这是你实验报告的模板，将包含在你的提交里。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">3 Lab 2：TCP 接收端</span>

TCP 是一个在不可靠数据报之上可靠地传送一对流量控制字节流（每个方向一条）的协议。两方参与 TCP 连接，每一方同时扮演"发送端"（发送自己出站的字节流）和"接收端"（接收一条入站字节流）两种角色。这两方被称为连接的"端点"（endpoint），或"对等方"（peer）。

本周，你将实现 TCP 的"接收端"部分，负责接收 TCP 段（segment，即实际的数据报有效载荷）、重组字节流（包括流的结束，如果发生的话），并计算应当回送给发送方的用于确认与流量控制的信号。

**我为什么要做这个？** 这些信号对 TCP 在不可靠数据报网络上提供"流量控制的可靠字节流"服务至关重要。在 TCP 中，确认（acknowledgment）的意思是："接收方需要下一个序号为多少的字节，才能组装出更多的 ByteStream？"这告诉发送方它需要发送（或重发）哪些字节。流量控制（flow control）的意思是："接收方有兴趣且愿意接收哪个索引范围？"

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 注意"确认"与"流量控制"这对信号其实是同一条回路的两个侧面，都来自同一个等式：接收方已连续收到的最高序号 + 可用缓冲 = 窗口右边界。确认回答"我从哪里开始缺数据"（窗口左边界），流量控制回答"我还能吞下多少"（窗口宽度）——前者来自重组进度，后者来自 ByteStream 的剩余容量，Lab 0 的 `remaining_capacity()` 在这里直接变成 TCP 首部里的 window 字段。这种"进度 + 容量"合成的思想你在 CSAPP 的生产者-消费者问题里见过：生产者发送速率被"消费者读走的字节数"反向钳制，缓冲区满则逻辑上停发。TCP 用窗口通告把这一机制放到不可靠网络上分布式地实现，而应用层读写（`write` 进 outbound 流、`read` 出 inbound 流）就是那个"生产者/消费者"。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">3.1 序列号（Sequence Numbers）</span>

作为热身，我们需要实现 TCP 表示字节流中每个字节索引的方式。上周你创建的 StreamReassembler 重组的子串中，每个字节有一个 64 位索引：从零开始，流中每个字节依次加一。而在 TCP 中，每个字节在流中的索引用一个 32 位的"序列号"（sequence number，seqno）表示，这带来了一些复杂性：

1. **开始与结束各占一个序列号位置**：除了确保收到数据的所有字节之外，TCP 还必须确保流的开始和结束也被收到。因此，TCP 中 SYN（流开始）和 FIN（流结束）标志各被分配一个序列号，各占一个序列号位置。流中的每个数据字节也各占一个序列号位置。
2. **32 位回绕序列号**：在 StreamReassembler 中，索引总是从零开始且有 64 位——足够多，我们不必担心用尽所有 64 位索引。而在 TCP 中，传输的序列号是 32 位的，如果流足够长就会回绕（wrap around）。（TCP 中的流可以任意长——能通过 TCP 发送的 ByteStream 长度没有上限。所以回绕相当常见。）
3. **TCP 序列号不从零开始**：为了提高安全性并避免不同连接之间的混淆，TCP 力图确保序列号无法被猜测、不易重复。所以流的序列号不从零开始。流中的第一个序列号通常是一个随机的 32 位数，称为初始序列号（Initial Sequence Number，ISN）。它是代表 SYN（流开始）的序列号。此后的序列号就正常行事——例如第一个数据字节的序列号是 ISN+1，第二个数据字节的序列号是 ISN+2，等等。

这些序列号是"真实"序列号：每个 TCP 段首部中实际传输的 32 位数值。有时谈论"绝对序列号"（absolute sequence number，总是从零开始、不回绕）的概念，以及"流索引"（stream index，你在 StreamReassembler 中用的那种：流中第一个实际数据字节从零开始编号）也很有帮助。

为了让这些区别具体化，考虑只包含三个字母的字符串 `cat` 的字节流。如果 SYN 的 seqno 恰好是 $2^{32} - 2$，那么每个字节的 seqno、绝对 seqno 和流索引如下：

| 元素 | seqno | absolute seqno | stream index |
|---|---|---|---|
| SYN | $2^{32} - 2$ | 0 | — |
| `c` | $2^{32} - 1$ | 1 | 0 |
| `a` | 0 | 2 | 1 |
| `t` | 1 | 3 | 2 |
| FIN | 2 | 4 | — |

这个图示展示了 TCP 中涉及的三种不同索引方式：

- **序列号（Sequence Numbers）**：从 ISN 开始；包含 SYN/FIN；32 位，回绕；记作 "seqno"
- **绝对序列号（Absolute Sequence Numbers）**：从 0 开始；包含 SYN/FIN；64 位，不回绕；记作 "absolute seqno"
- **流索引（Stream Indices）**：从 0 开始；不含 SYN/FIN；64 位，不回绕；记作 "stream index"

在绝对序列号和流索引之间转换很容易——加减一即可。遗憾的是，在序列号和绝对序列号之间转换则要难一些，把两者搞混会产生棘手的缺陷（bug）。为了系统性地防止这些缺陷，我们将用一个自定义类型表示序列号：WrappingInt32<span style="color:#7f8c8d;">¹</span>，并编写它与绝对序列号（用 `uint64_t` 表示）之间的转换函数。

我们已为你定义了该类型并提供了一些辅助函数（见 wrapping_integers.hh），但你将在 wrapping_integers.cc 中实现这些转换：

1. `WrappingInt32 wrap(uint64_t n, WrappingInt32 isn)`
   把绝对 seqno 转换为 seqno。给定一个绝对序列号（n）和一个初始序列号（isn），生成 n 对应的（相对）序列号。
2. `uint64_t unwrap(WrappingInt32 n, WrappingInt32 isn, uint64_t checkpoint)`
   把 seqno 转换为绝对 seqno。给定一个序列号（n）、初始序列号（isn）和一个绝对检查点序列号（checkpoint），计算与 n 对应、且最接近 checkpoint 的绝对序列号。

（注意：需要 checkpoint 是因为任何一个 seqno 都对应许多个绝对 seqno。例如 seqno "17" 对应绝对 seqno 17，但也对应 $2^{32} + 17$、$2^{33} + 17$、$2^{34} + 17$ 等等。checkpoint 帮助消除歧义：它是一个最近解包得到的绝对 seqno，使用这个类的用户知道它接近于该 seqno 所想要的解包结果。在你的 TCP 实现中，你将用最后一个重组字节的索引作为 checkpoint。）

提示：最干净/最容易的实现会使用 wrapping_integers.hh 中提供的辅助函数。wrap/unwrap 操作应当保持偏移量不变——相差 17 的两个 seqno 对应的两个绝对 seqno 也相差 17。所以，试着：(a) 对 checkpoint 做 wrap，(b) 在回绕空间中计算 n 与 checkpoint 的差，(c) 在未回绕空间中加上该差，(d) 处理下溢（负数）结果。

你可以运行 WrappingInt32 的测试来检验你的实现。在 build 目录中运行 `ctest -R wrap`。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 回绕序列号的比较与运算是 Lab 2–4 里最"CSAPP"的部分，直接对应 ch2 的整数表示。要点有三：其一，seqno 空间是模 $2^{32}$ 的算术——这与无符号整数溢出回绕是同一个数学结构，CSAPP 讲过无符号运算是模 $2^w$ 加法，这里的"加 17 再回绕"就是模运算本身。其二，**回绕安全比较**不能用普通的 `<`：当序号从 $2^{32}-1$ 跳到 0 时，直接比较会误判"新序号小于旧序号"。正确做法是把两个 32 位数相减、把差取模回绕到 $[-2^{31}, 2^{31}-1]$ 区间（在 C++ 中通常写成 `int32_t(a - b)` 的有符号解释），差为正则 a 更新——这正是 RFC 1982 定义的时代次序关系，也是 CSAPP "符号位只是解释而非本质"的活例子：同一串比特，用无符号解释是回绕，用有符号解释就是"前后差"。其三，unwrap 的 checkpoint 机制之所以必要，是因为模运算把"距离"压扁了：$2^{32}$ 之外的绝对值全都塌缩成同一个 seqno，必须用"最近一次已知位置"挑出唯一解——直觉上就是"我只知道我在 17 附近，17 加上回绕差的合理取值就是答案"。Hint 给的"wrap checkpoint → 求回绕差 → 加回 → 处理下溢"四步法实现时请严格照做。

> <span style="color:#7f8c8d;">脚注 1：WrappingInt32 是一个包装类型（wrapper type）的例子：它包含一个内部类型（此处是 `uint32_t`），但提供一组不同的函数/运算符。</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">3.2 "可接受字节窗口"是什么？</span>

恭喜你把回绕逻辑做对了！在深入 TCPReceiver 实现之前，我们先讨论接收端流量控制窗口的概念。

思路是：TCPReceiver 将告诉 TCPSender 它有兴趣且愿意接受的连续字节索引范围（或称"窗口"，window）。（并且，让我们说清楚：你的 TCPReceiver 会把你的 StreamReassembler 作为成员，用它干重活。本周主要是把 StreamReassembler 连接到 TCP 上。）

接收端这样做是为了流量控制。TCPReceiver 的 StreamReassembler 会忽略任何超出其内存用量上限（capacity）的字节。通过告诉发送方"允许/不允许发送哪些范围的数据"，接收端可以防止发送方白费力气，并让发送方的发送速度不超过应用从重组后的 ByteStream 中读取的速度。（记住 ByteStream 本身可以任意长——太字节、艾字节、尧字节都行——但接收端的内存用量，包括 StreamReassembler 和它正在重建的 ByteStream，都受 capacity 约束。只有当应用读取并弹出（pop）接收端 ByteStream 中的一些数据之后，发送方才获准发送流的更多部分。）

所以"窗口"指的是接收端当前可接受的一个字节索引（或序列号）范围。

窗口的下边缘（有时称"左"边缘）称为 ackno：它是 TCPReceiver 尚不知道的第一个字节的索引（因此也是它希望得知的字节）。举例来说，假设接收端收到了一个序列号为 24 的 SYN，然后收到序列号为 25、26、27 和 35 的四个数据字节。ackno 将是 28：它是接收端尚无法重组的第一个字节的序列号。

窗口的上边缘（"右"边缘）是 TCPReceiver 不愿意接受的第一个索引。

窗口大小（window size）是上边缘减去下边缘。

你的接收端负责在任意时刻计算 ackno（窗口下边缘）和窗口大小（窗口上下边缘之差）。它们最终会被放进从接收端发往发送端的 TCPSegment。实践中，你的接收端通告的窗口大小等于其 capacity 减去 StreamReassembler 的 ByteStream 中持有的字节数。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 上面"SYN=24，数据 25、26、27、35 → ackno=28"的例子浓缩了自动重传请求（ARQ）的全部直觉：35 号字节虽然先到了，但它落在 28–34 这个空洞之后，重组器只能扣住不放，确认号钉在第一个缺失字节上——这就是"累积确认"（cumulative acknowledgment）的形状。接收窗口宽度的计算公式"capacity 减去 ByteStream 中已重组未读走的字节数"也值得记住：它意味着窗口不是静态字段，而是随应用读取实时膨胀的活物。注意这里有个与真实 RFC 行为的简化差异：本课规定重组器直接丢弃超出容量的字节，而真实 TCP 只保证不**通告**超出容量的窗口、但对越过窗口右边缘的"坏"段有自己的接受性判定（见下一节 window size 为零的特例）。另外注意"接收端永不收缩窗口右边缘"是一条隐含约定，Lab 3 的发送端重传逻辑将依赖它。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">3.3 实现 TCP 接收端</span>

在本实验的其余部分，你将实现 TCPReceiver。它将：(1) 从对等方接收段，(2) 用你的 StreamReassembler 重组 ByteStream，并计算 (3) 确认号（ackno）和 (4) 窗口大小。

首先，请复习 TCP 段的格式。这是两个端点互相发送的数据报有效载荷的结构。ackno 和窗口大小都是 TCPSegment 结构中的字段。请查阅 TCPSegment（https://cs144.github.io/doc/lab2/class_tcp_segment.html）与 TCPHeader（https://cs144.github.io/doc/lab2/struct_tcp_header.html）的文档。你可能会对 `length_in_sequence_space()` 方法感兴趣——它计算一个段占用多少个序列号（包括 SYN 和 FIN 各占一个序列号位置，以及有效载荷的每个字节各占一个位置）。

接下来，谈谈你的 TCPReceiver 将提供的接口：

```cpp
// Construct a `TCPReceiver` that will store up to `capacity` bytes
TCPReceiver(const size_t capacity);  // implemented for you in .hh file

// Handle an inbound TCP segment
//
// returns `true` if any part of the segment was inside the window
bool segment_received(const TCPSegment &seg);

// The ackno that should be sent to the peer
//
// returns empty if no SYN has been received
//
// This is the beginning of the receiver's window, or in other
// words, the sequence number of the first byte in the stream
// that the receiver hasn't received.
std::optional<WrappingInt32> ackno() const;

// The window size that should be sent to the peer
//
//
// Formally: this is the size of the window of acceptable indices
// that the receiver is willing to accept. It's the difference between
// (a) the sequence number of the end of the window (the first index that
//     won't be accepted by the receiver) minus
// (b) the sequence number of the beginning of the window (the ackno).
//
//
// Operationally: it's the capacity minus the number of bytes that the
// TCPReceiver is holding in the byte stream.
size_t window_size() const;

// number of bytes stored but not yet reassembled
size_t unassembled_bytes() const;  // implemented for you in .hh file

// Access the reassembled byte stream
ByteStream &stream_out();  // implemented for you in .hh file
```

TCPReceiver 是围绕你的 StreamReassembler 构建的。我们已在 .hh 文件中为你实现了构造函数以及 `unassembled_bytes()` 和 `stream_out()` 方法。其余方法需要你做的事如下：

#### <span style="color:#c0392b;">3.3.1 segment_received()</span>

这是主要的工作方法！每当从对等方收到一个新段，`TCPReceiver::segment_received()` 就会被调用。

这个方法需要：

- **在必要时设置初始序列号（ISN）**。第一个到达的、SYN 标志置位的段的序列号就是初始序列号。你要把它记下来，以便持续在 32 位回绕 seqno/ackno 与其绝对值之间转换。（注意 SYN 只是首部中的一个标志位；同一个段也可能携带数据，甚至可能同时置位 FIN。）
- **把任何数据或流结束标志推给 StreamReassembler**。如果某个 TCPSegment 的首部中 FIN 标志置位，那意味着流在有效载荷的最后一个字节处结束——它等价于一个 EOF。
- **判断段的任何部分是否落在窗口内**。如果段的任何部分落在窗口内，这个方法应返回 `true`，否则返回 `false`。<span style="color:#7f8c8d;">²</span> 我们的意思是：一个段占据一段序列号范围——从它的序列号开始，长度等于它的 `length_in_sequence_space()`（这反映了 SYN 和 FIN 各占一个序列号、有效载荷每个字节各占一个序列号的事实）。如果该段占据的序列号中有任何一个落在接收端的窗口之内，它就是可接受的（方法应返回 `true`）。

有一些特殊情况：

- 如果 ISN 尚未设置，则一个段可接受当且仅当它置位了 SYN 位。
- 如果窗口大小为零，那么在确定窗口的第一个和最后一个字节时，应把窗口大小当作一个字节来处理。<span style="color:#7f8c8d;">³</span>
- 如果段的序列号空间长度为零（例如一个没有有效载荷、也没有 SYN 或 FIN 标志的裸确认），那么在确定它占据的第一个和最后一个序列号时，应把它的长度当作一个字节来处理。

> <span style="color:#7f8c8d;">脚注 2：如果返回 `false`，说明 TCPSender 不知怎么搞糊涂了，需要就窗口问题被纠正。</span>

> <span style="color:#7f8c8d;">脚注 3：这个可接受性定义是对 RFC 793 第 25–26 页定义的一个轻微改写。</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "窗口大小为零按一字节处理、零长度段按一字节处理"这两条特殊规则看似别扭，其实是为了让**闭区间判定**自洽：段的可接受性判定本质上是两个半开区间的交叠测试（段的 $[seqno, seqno+length)$ 与窗口的 $[ackno, ackno+win)$ 是否相交），而零宽区间在数学上与任何区间都不相交——若不人为把零放宽成一，一个"恰好顶在窗口左边缘"的空段或零窗口下的探测段就会被永远拒之门外，连接就此卡死。这与你写 CSAPP 作业时处理 `memcpy` 零长度、空区间边界是同一类思维：边界条件不是麻烦，是规格说明的一部分。另外，把"SYN 也可能捎带数据甚至 FIN"记在心里——TCP 首部只有 20 字节，设计者舍不得为纯握手浪费一次往返，所以首段常常是 SYN+数据一体，这是判断你实现是否通用（而非只对黄金测试样例成立）的关键。

#### <span style="color:#c0392b;">3.3.2 ackno()</span>

返回一个 `optional<WrappingInt32>`，包含接收端尚不知道的第一个字节的序列号。这是窗口的左边缘：接收端有兴趣接收的第一个字节。如果 ISN 尚未设置，返回空的 optional。

#### <span style="color:#c0392b;">3.3.3 window_size()</span>

窗口大小的定义请参见第 3.2 节。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">4 开发与调试建议（Development and debugging advice）</span>

1. 在文件 tcp_receiver.cc 中实现 TCPReceiver 的公共接口（以及任何你想要的私有方法或函数）。你可以在 tcp_receiver.hh 中向 TCPReceiver 类添加任何你喜欢的私有成员。
2. 你可以（编译之后）用 `make check_lab2` 测试你的代码。
3. 请重读 Lab 0 文档中"关于使用 Git"的一节，记住让代码留在分发时的 Git 仓库的 master 分支上。做小的提交，提交信息要清楚说明改了什么、为什么改。
4. 请努力让你的代码对负责风格评分的课程助理具有可读性。为变量使用合理、清晰的命名约定。用注释解释复杂或微妙的代码片段。使用"防御式编程"——显式检查函数前置条件或不变量，一旦有问题就抛出异常。在设计中使用模块化——识别共同的抽象与行为，并尽可能把它们抽取出来。大段重复代码和巨型函数会让你的代码难以理解。
5. 请同时遵守 Lab 0 文档中描述的"现代 C++"风格。cppreference 网站（https://en.cppreference.com）是极好的资源，虽然做这些实验用不到任何精深的 C++ 特性。（你有时可能需要用 `move()` 函数来传递一个无法拷贝的对象。）
6. 如果你遇到了段错误，说明有什么东西真的出问题了！我们希望你以安全编程实践来书写代码，让段错误极其罕见（不用 `malloc()`、不用 `new`、不用指针、在不确定处放置会抛异常的安全检查等）。话虽如此，为了调试，你可以用 `cmake .. -DCMAKE_BUILD_TYPE=RelASan` 配置你的 build 目录，启用编译器的"消毒器"（sanitizers）来检测内存错误和未定义行为，并在它们发生时给你漂亮的诊断信息。你也可以使用 valgrind 工具。你还可以用 `cmake .. -DCMAKE_BUILD_TYPE=Debug` 配置并使用 GNU 调试器（gdb）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">5 提交（Submit）</span>

1. 提交时，请只修改 libsponge 顶层的 .hh 和 .cc 文件。在这些文件中，欢迎按需添加私有成员，但请不要修改任何类的公共接口。
2. 交任何作业之前，请按顺序运行：
   (a) `make format`（规范代码风格）
   (b) `git status`（检查未提交的变更——如果有，就 commit！）
   (c) `make`（确保代码能编译）
   (d) `make check_lab2`（确保自动化测试通过）
3. 在 writeups/lab2.md 中写一份报告。这个文件应当是一份约 20 至 50 行的文档，每行不超过 80 个字符以便阅读。报告应包含以下小节：
   (a) **Program Structure and Design（程序结构与设计）**。描述你的代码所体现的高层结构与设计选择。你不需要详细讨论你从初始代码继承来的内容。请把它当作一个突出重要设计方面、并为评分助教提供更详细说明的机会。我们强烈鼓励你用小标题和大纲让这份报告尽可能易读。请不要简单地把你的程序翻译成一段英文散文。
   (b) **Implementation Challenges（实现难点）**。描述你的代码中最麻烦的部分并解释原因。回顾你如何克服这些困难、是什么最终帮你理解了那个困扰你的概念。你如何确保代码维持了你的假设、不变量和前置条件？你觉得哪些地方容易、哪些地方困难？你如何调试和测试你的代码？
   (c) **Remaining Bugs（遗留缺陷）**。指出并尽你所能解释代码中仍然存在的任何缺陷（或未处理的边界情况）。
4. 在报告中，也请填入本作业花费的小时数以及任何其他意见。
5. 准备好提交时，请按照 https://cs144.github.io/submit 上的说明操作。请确保在提交之前已经 commit 所有你打算提交的内容。
6. 如有任何问题，请尽快在周二晚间的实验课上告知课程组，或在 Piazza 上发帖。祝你好运！

---

> <span style="color:#7f8c8d;">脚注：本讲义为 Lab 2（TCPReceiver）原始 handout。黄金测试通过 `make check_lab2` 运行；三种序号空间（seqno/absolute seqno/stream index）的转换与窗口计算是 Lab 3（发送端使用 ackno/window_size）与 Lab 4（连接关闭）的直接前置。</span>
