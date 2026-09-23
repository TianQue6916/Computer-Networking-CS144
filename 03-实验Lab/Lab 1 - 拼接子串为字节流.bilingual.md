# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Lab 1: stitching substrings into a byte stream（Lab 1：把子串拼接成字节流）

> <span style="color:#7f8c8d;">Source: 实验讲义/Lab 1 - 拼接子串为字节流.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

> <span style="color:#7f8c8d;">脚注（源文噪声说明）：原 PDF 经 pdftotext 提取后存在系统性噪声：列表项目符号丢失、下划线丢失（如 `stream reassembler.hh` 应为 `stream_reassembler.hh`、`make check lab1` 应为 `make check_lab1`、`-DCMAKE BUILD TYPE=RelASan` 应为 `-DCMAKE_BUILD_TYPE=RelASan`）、代码与正文混排。以下译文已按上下文复原，不逐一标注。</span>

---

**截止时间**：10 月 4 日（周五）下午 5 点。实验课：10 月 1 日（周二）晚 7:30–10:00，地点 STLC114。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">0 合作政策（Collaboration Policy）</span>

**编程作业必须是你自己的工作**：除了我们随作业发给你的代码之外，你提交的编程作业代码必须全部自己写。请不要从 Stack Overflow、GitHub 或其他来源复制粘贴代码。如果你的代码基于网上或其他地方找到的示例，请在提交的源代码注释中注明出处 URL。

**与他人合作**：不得把你的代码给任何人看，不得看任何人的代码，也不得看往年的解答。你可以和其他同学讨论作业，但不要抄任何人的代码。如果你和另一位同学讨论了作业，请在提交的源代码注释中写出对方姓名。更多细节请参阅课程管理讲义，有任何不清楚的地方请在 Piazza 上提问。

**Piazza**：欢迎在 Piazza 上提问，但请不要张贴任何源代码。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">1 概览（Overview）</span>

在 Lab 0 中，你使用互联网流套接字从一个网站抓取信息、发送了一封电子邮件，用的是 Linux 内置的传输控制协议（Transmission Control Protocol，TCP）实现。这套 TCP 实现设法产出了一对可靠、有序的字节流（一条从你到服务器，一条沿相反方向），尽管底层的网络只投递"尽力而为"（best-effort）的数据报（datagram）——我们的意思是：短小的数据分组，它们可能丢失、乱序、被改动或被重复投递。你也在一台计算机的内存里亲手实现过字节流（byte stream）抽象本身。在接下来的四周里，你将实现 TCP：在一对被不可靠数据报网络分隔的计算机之间，提供字节流抽象。

**我为什么要做这个？** 在一个不太可靠的服务之上提供另一种服务或抽象，是网络领域许多有趣问题的来源。过去 40 年里，研究者和工程实践者弄清了如何在互联网上传送各种各样的东西——即时消息与电子邮件、超链接文档、搜索引擎、声音与视频、虚拟世界、协作式文件共享、数字货币。TCP 自己的角色——用不可靠的数据报提供一对可靠字节流——正是这类问题的经典范例之一。有一种合理的观点认为：TCP 实现堪称这个星球上使用最广泛的非平凡计算机程序。

实验作业将要求你以模块化的方式逐步搭建出一个 TCP 实现：

1. 在 Lab 1 中，你将实现一个**字节流重组器**（stream reassembler）——一个把字节流的小碎片（称为子串（substring）或段（segment））按正确顺序重新拼接成连续字节流的模块。
2. 在 Lab 2 中，你将实现 TCP 中处理入站字节流的部分：TCPReceiver。这需要思考 TCP 如何表示每个字节在流中的位置——即"序列号"（sequence number）。TCPReceiver 负责告诉发送方：(a) 它已成功组装了多少入站字节流（这称为"确认"，acknowledgment），(b) 发送方现在还允许再发送多少字节（"流量控制"，flow control）。
3. 在 Lab 3 中，你将实现 TCP 中处理出站字节流的部分：TCPSender。当发送方怀疑它发出的某个段在路上丢失、没能到达接收方时，它该如何反应？它应该在什么时候重试、重传丢失的段？
4. 在 Lab 4 中，你将把前几个实验的成果组合成一个能工作的 TCP 实现：一个包含 TCPSender 和 TCPReceiver 的 TCPConnection。你将用它与世界各地的真实服务器对话。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">2 起步（Getting started）</span>

你的 TCP 实现将使用你在 Lab 0 中用过的同一个 Sponge 库，外加一些新的类和测试。开始步骤：

1. 确保你已经 commit 了 Lab 0 的全部解答。请不要修改 libsponge 目录顶层之外的任何文件，也不要修改 webget.cc——否则合并 Lab 1 初始代码时可能出问题。
2. 在实验作业仓库内运行 `git fetch`，获取实验作业的最新版本。
3. 运行 `git merge origin/lab1-startercode` 下载 Lab 1 的初始代码。
4. 在你的 build 目录中编译源码：`make`（可以运行 `make -j4`，编译时使用四个处理器）。
5. 在 build 目录之外，打开并开始编辑 writeups/lab1.md 文件。这是你实验报告的模板，将包含在你的提交里。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">3 把子串按序拼接（Putting substrings in sequence）</span>

在本实验和下一个实验中，你将实现一个 TCP 接收端（TCP receiver）：接收数据报并把它们转换成供用户读取的可靠字节流的模块（就像你的 webget 程序在 Lab 0 中从 Web 服务器读取字节流那样）。

TCP 发送端把自己的字节流切分成短小的段（segment）——每段是不超过约 1460 字节的子串——以便每段都能装进一个数据报。但网络可能重排这些数据报，或者丢弃它们，或者把它们投递不止一次。接收端必须把这些段重新组装成它们最初所处的那个连续字节流。

在本实验中，你将编写负责这种重组的数据结构：一个 StreamReassembler。它接收子串（由一个字节串和该串第一个字节在整个流中的索引组成），并供养出一个所有数据都排好序的 ByteStream。

接口如下：

```cpp
// Construct a `StreamReassembler` that will store up to `capacity` bytes.
StreamReassembler(const size_t capacity);

// Receive a substring and write any newly contiguous bytes into the stream.
//
// `data`: the segment
// `index` indicates the index (place in sequence) of the first byte in `data`
// `eof`: the last byte of this segment is the last byte in the entire stream
void push_substring(const string &data, const uint64_t index, const bool eof);

// Access the reassembled byte stream
ByteStream &stream_out();

// The number of bytes in the substrings stored but not yet reassembled
size_t unassembled_bytes() const;

// Is the internal state empty (other than the output stream)?
bool empty() const;
```

重组器的完整（公共）接口由 stream_reassembler.hh 头文件中的 StreamReassembler 类描述。你的任务就是实现这个类。你可以向 StreamReassembler 类添加任何你想要的私有成员和成员函数，但不能改变它的公共接口。

**我为什么要做这个？** 因为它是 TCP 抵御段乱序（segment reordering）能力的核心，而且它会让后续处理收到的段容易得多。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> StreamReassembler 要解决的问题是：收到的子串可能乱序、重复、留下空洞（gap），你要在容量受限的前提下把它们拼回连续流。这与 CSAPP ch9 虚拟内存一章的缓冲管理是同构问题：页可以乱序换入、页表负责定位、物理页框是有限资源——这里段对应页，索引对应虚拟地址，capacity 对应物理内存总量。设计的关键抉择是"重组区放哪"：一种做法是为乱序数据单独开辟暂存区，另一种（更优雅的）做法是把未就绪字节也放进 ByteStream 的存储里，只推进一个"第一个未重组索引"指针。无论哪种，都要保证 total 内存占用（已重组未读取 + 暂存未重组）不超过 capacity——这正对应虚拟内存管理里"预留 + 提交"的记账思想。数据结构选择上，用位图标记"哪些字节已就绪"（CSAPP ch2 的位操作技巧在这里能派上用场：用 `1UL << (i % 64)` 置位、用与或判断区间是否连续），或者用 `std::map<uint64_t, string>` 按首索引组织不连续块，都是常见做法。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">3.1 常见问题（FAQs）</span>

- **整个流的第一个字节的索引是多少？** 零。
- **我的实现应该多高效？** 我们暂时不指定具体的效率标准，但请不要把这当成一个挑战去构造一个在空间或时间上极其低效的数据结构——这个数据结构将是你 TCP 实现的地基。
- **不一致的子串该如何处理？** 你可以假设它们不存在。也就是说，你可以假设存在唯一的底层字节流，所有子串都是它的（准确的）切片。
- **我可以使用什么？** 你可以使用标准库中任何你觉得有用的部分。特别地，我们希望你自己实现至少一个数据结构。<span style="color:#7f8c8d;">（译按：原文为 "we expect you to use at least one data structure"，直译为"我们希望你至少使用一个数据结构"。）</span>
- **字节应该在什么时候写入流？** 尽可能快。唯一不该出现在流里的字节的情形是：它前面还有尚未"推入"（push）的字节。
- **子串可以重叠吗？** 可以。
- **我需要给 StreamReassembler 添加私有成员吗？** 需要。由于段可能以任意顺序到达，你的数据结构必须"记住"那些还没准备好进入流的子串——也就是说，直到它们前面的所有索引都被填上为止。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "字节应尽快写入流"这条 FAQ 其实给定了重组器的核心不变量（invariant）：ByteStream 中可读部分必须是流的严格前缀，任何字节一旦前面的字节都已就绪就立刻推入，不得滞留。这与 CSAPP ch9 里页表按需调页（demand paging）形成有趣对照：虚拟内存允许页乱序驻留、访问时再检查，因为内存管理单元（MMU）可以在访问时刻兜底；而字节流语义不允许"读者先看到后面的字节"，所以重组器必须在写入时刻维护全局有序。重叠子串的处理（同一段数据收到两次）则考验你的幂等设计：无论一个字节到达多少次，它的效果必须与到达一次相同——用"已写入标记"或按区间去重即可。至于数据结构，`std::set`/`std::map` 的有序性天然支持"找第一个未就绪索引之后的块"，复杂度 $O(\log N)$，比线性扫描更可取；这也算把 CLRS 里二叉搜索树的有序性质用在了真实系统上。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">4 开发与调试建议（Development and debugging advice）</span>

1. 你可以（编译之后）用 `make check_lab1` 测试你的代码。
2. 请重读 Lab 0 文档中"关于使用 Git"的一节，记住让代码留在分发时的 Git 仓库的 master 分支上。做小的提交，提交信息要清楚说明改了什么、为什么改。
3. 请努力让你的代码对负责风格评分的课程助理（CA）具有可读性。为变量使用合理、清晰的命名约定。用注释解释复杂或微妙的代码片段。使用"防御式编程"——显式检查函数前置条件或不变量，一旦有问题就抛出异常。在设计中使用模块化——识别共同的抽象与行为，并尽可能把它们抽取出来。大段重复代码和巨型函数会让你的代码难以理解。
4. 请同时遵守 Lab 0 文档中描述的"现代 C++"风格。cppreference 网站（https://en.cppreference.com）是极好的资源，虽然做这些实验用不到任何精深的 C++ 特性。（你有时可能需要用 `move()` 函数来传递一个无法拷贝的对象。）
5. 如果你遇到了段错误（segmentation fault），说明有什么东西真的出问题了！我们希望你以安全编程实践来书写代码，让段错误极其罕见（不用 `malloc()`、不用 `new`、不用指针、在不确定处放置会抛异常的安全检查等）。话虽如此，为了调试，你可以用 `cmake .. -DCMAKE_BUILD_TYPE=RelASan` 配置你的 build 目录，启用编译器的"消毒器"（sanitizers）来检测内存错误和未定义行为，并在它们发生时给你漂亮的诊断信息。你也可以使用 valgrind 工具。你还可以用 `cmake .. -DCMAKE_BUILD_TYPE=Debug` 配置并使用 GNU 调试器（gdb）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">5 提交（Submit）</span>

1. 提交时，请只修改 libsponge 顶层的 .hh 和 .cc 文件。在这些文件中，欢迎按需添加私有成员，但请不要修改任何类的公共接口。
2. 交任何作业之前，请按顺序运行：
   (a) `make format`（规范代码风格）
   (b) `make`（确保代码能编译）
   (c) `make check_lab1`（确保自动化测试通过）
3. 在 writeups/lab1.md 中写一份报告。这个文件应当是一份约 20 至 50 行的文档，每行不超过 80 个字符以便阅读。报告应包含以下小节：
   (a) **Program Structure and Design（程序结构与设计）**。描述你的代码所体现的高层结构与设计选择。你不需要详细讨论你从初始代码继承来的内容。请把它当作一个突出重要设计方面、并为评分助教提供更详细说明的机会。我们强烈鼓励你用小标题和大纲让这份报告尽可能易读。请不要简单地把你的程序翻译成一段英文散文。
   (b) **Implementation Challenges（实现难点）**。描述你的代码中最麻烦的部分并解释原因。回顾你如何克服这些困难、是什么最终帮你理解了那个困扰你的概念。你如何确保代码维持了你的假设、不变量和前置条件？你觉得哪些地方容易、哪些地方困难？你如何调试和测试你的代码？
   (c) **Remaining Bugs（遗留缺陷）**。指出并尽你所能解释代码中仍然存在的任何缺陷（或未处理的边界情况）。
4. 在报告中，也请填入本作业花费的小时数以及任何其他意见。
5. 准备好提交时，请按照 https://cs144.github.io/submit 上的说明操作。请确保在提交之前已经 commit 所有你打算提交的内容。
6. 如有任何问题，请尽快在周二晚间的实验课上告知课程组，或在 Piazza 上发帖。祝你好运！

---

> <span style="color:#7f8c8d;">脚注：本讲义为 Lab 1（StreamReassembler）原始 handout。黄金测试（golden test）通过 `make check_lab1` 运行；重组器的设计要点（容量、重叠、EOF 语义）将在 Lab 2 的 TCPReceiver 中被直接复用。</span>
