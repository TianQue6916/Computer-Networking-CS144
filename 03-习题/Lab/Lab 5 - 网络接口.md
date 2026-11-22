> <span style="color:#7f8c8d;">Source: src/实验讲义/Lab 5 - 网络接口.txt · CS144 Fall 2019 · 行内双语翻译版</span>

# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#c0392b;">**Lab 5: down the stack (the network interface)（Lab 5：走下协议栈——网络接口）**</span>

> <span style="color:#7f8c8d;">Fall 2019 · 截止时间：12 月 4 日（周三）下午 5 点 · 提前提交加分：11 月 22 日（周五）下午 5 点前提交可获 15% bonus（成绩 ×1.15）· 实验课：11 月 19 日与 12 月 3 日（周二）晚 7:30–10 点，地点 STLC114</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">**0 Collaboration Policy（协作政策）**</span>

编程作业必须独立完成：除了我们随作业发给你的那部分代码之外，你为编程作业上交的所有代码都必须出自你手。请不要从 Stack Overflow、GitHub 或其他来源复制粘贴代码。如果你的代码借鉴了网上或其他地方找到的示例，请在提交的源代码中以注释形式注明 URL（URL）。

关于与他人合作：你不可以把自己的代码展示给任何人、查看任何人的代码，或查看往年的解答。你可以和其他同学讨论作业，但不能抄袭任何人的代码。如果你与另一位同学讨论过作业，请在提交的源代码注释中写上对方的名字。更多细节请参阅课程行政讲义（course administrative handout），有不清楚之处请在 Piazza 上提问。

Piazza：欢迎在 Piazza 上提问，但请不要发布任何源代码。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">**1 Overview（概述）**</span>

在本周实验中，你将走下协议栈（down the stack），实现一个网络接口（network interface）：它是"周游世界的 Internet 数据报（Internet datagram）"与"只走一跳的链路层以太网帧（Ethernet frame）"之间的桥梁。这个组件可以"塞在"你前几个实验写出的 TCP/IP 实现的下面，但它还会在另一种场景中使用：当你在 Lab 6 里构建路由器（router）时，路由器将在多个网络接口之间转发数据报。图 1 展示了网络接口如何同时融入这两种场景。

你的网络接口实现将使用你在 Lab 0–4 中用过的同一个 Sponge 库，外加一些新的类和测试。不过：应大家的强烈要求，本实验的大部分（虽非全部）都可以在不依赖你之前写的 TCPConnection 的情况下完成。

在过去几个实验里，你写出的 TCP 实现已能与其他任何讲 TCP 的计算机成功交换 TCP 报文段（segment）。那么这些报文段究竟是怎么被送到对端 TCP 实现的？正如我们讨论过的，有几种方案：

- **TCP-in-UDP-in-IP**。TCP 报文段可以装在用户数据报（user datagram）的有效载荷（payload）里传输。在普通（用户态）环境下，这是最容易实现的方案：Linux 提供了一个接口（"Internet 数据报套接字"，UDPSocket），让应用程序只需提供用户数据报的载荷和目标地址，内核（kernel）会负责构造 UDP 首部（header）、IP 首部和以太网首部，然后把分组发给合适的下一跳（next hop）。内核保证每个套接字（socket）都拥有本地与远程地址、端口号（port number）组合的唯一性，而且由于往 UDP 和 IP 首部里写内容的正是内核自己，它可以在不同应用程序之间提供隔离。

  图 1（Figure 1）中 Sponge 库各组件的层次关系（自上而下）：

  ```text
  Application (e.g. webget.cc) ── reliable byte streams
  Labs 1-4: TCPConnection (TCPReceiver / TCPSender)
  StreamReassembler / ByteStream
  TCP segments
  TLS encryption fits here (bytestream-to-bytestream)
  Internet Protocol ── Internet datagrams
  Lab 5: Network Interface ── Ethernet frames
  Lab 6: Routing logic (longest-prefix match)
  Link-layer device (NIC) ── Network to London / São Paulo / Chicago
  ```

  > 图 1：网络接口架起 Internet 数据报世界与链路层帧世界之间的桥梁。该组件既可作为主机 TCP/IP 协议栈的一部分（左图），也可作为 IP 路由器的一部分（右图）。

- **TCP-in-IP**。在实际使用中，TCP 报文段几乎总是直接放进一个 Internet 数据报，IP 首部与 TCP 首部之间不再插入 UDP 首部——这就是人们说 "TCP/IP" 时的含义。这种方案实现起来稍难一些。Linux 提供了一个叫 TUN 设备（TUN device）的接口，让应用程序提交一个完整的 Internet 数据报，其余工作由内核完成（写以太网首部、真正通过物理以太网卡发送等）。但现在应用程序必须自己构造完整的 IP 首部，而不仅仅是载荷。

  这一步你已经做过了：在 Lab 4 中，我们给了你一个表示 Internet 数据报、并知道如何解析与序列化自身的对象（`tcp_helpers/ipv4_datagram.{hh,cc}`），以及把 TCP 报文段封装进 IP 的逻辑（现在位于 `tcp_helpers/tcp_over_ip.cc`）。CS144TCPSocket 就是用这些工具把你的 TCPConnection 连接到一个 TUN 设备上的。

- **TCP-in-IP-in-Ethernet**。在上面那种方案里，协议栈的一部分仍然依赖 Linux 内核。每当你的代码向 TUN 设备写入一个 IP 数据报，Linux 都得构造一个以该数据报为载荷的链路层（以太网）帧。这意味着 Linux 必须根据下一跳的 IP 地址，弄清下一跳的以太网目的地址。如果它还没有这个映射，就会广播一个询问："Who claims the following IP address? What's your Ethernet address?"（谁认领下面这个 IP 地址？你的以太网地址是什么？）然后等待应答。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 如果你在 CSAPP ch10–ch11 里体会过"Unix 一切皆文件"的哲学，那么 TUN/TAP 设备就是这条哲学的漂亮延伸：普通文件、管道、套接字在应用眼里都只是文件描述符（file descriptor），TUN/TAP 也不例外——打开它得到一个 fd，向它 write 一段字节，内核就当作一个 IP 数据报（TUN）或原始以太网帧（TAP）交给你处理，read 则是相反方向。你在 Lab 4 用过的 CS144TCPSocket，本质上就是把 TUN 设备的 fd 包成一个 Socket 对象——与你写 webget 时包装 TCP fd 的思路一脉相承。区别只在字节流的"语义"：CSAPP 里的 fd 帮你屏蔽了磁盘块、网卡缓冲等细节，而 TUN/TAP 把"内核网络协议栈的最底一层"也开放给用户态。这门课让你做的事，等于把内核协议栈一层层搬进自己的代码——这正是整套实验设计的精髓，也是理解 ch11 中 socket 接口为何如此设计的最好反衬。

<span style="color:#00838f;">**网络接口（network interface）就是完成这些功能的组件**：它把出站 IP 数据报翻译成链路层（例如以太网）帧，并做反向翻译。</span>（在真实系统中，网络接口的名字通常是 `eth0`、`eth1`、`wlan0` 之类。）在本周的实验中，你将实现一个网络接口，并把它装在你 TCP/IP 协议栈的最底部。你的代码将产出原始以太网帧，并通过一个叫 TAP 设备（TAP device）的接口交给 Linux——它与 TUN 设备类似，但更底层：它交换的是原始链路层帧，而不是 IP 数据报。

大部分工作量在于查找（并缓存）每个下一跳 IP 地址对应的以太网地址。完成这件事的协议叫地址解析协议（Address Resolution Protocol，ARP）。

我们为你提供了单元测试来充分考验你的网络接口。然后，在本实验的最后，你将对 `webget` 稍作修改，让它使用你的 TCP 实现、运行在你自己的网络接口之上——这样整套东西产出的就是原始以太网帧，并且仍然能与 Internet 上的真实 Web 服务器对话。在 Lab 6 中，你将在 TCP 之外的语境下复用同一个网络接口，作为 IP 路由器的一部分。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">**2 Getting started（开始动手）**</span>

1. 确保你已提交（commit）Lab 4 的全部解答。请不要改动 `libsponge` 目录顶层之外的任何文件，也不要改 `webget.cc`（并且不要添加你的代码所依赖的额外文件），否则你在合并 Lab 5 起始代码时可能遇到麻烦。
2. 在实验作业仓库内运行 `git fetch`，获取实验作业的最新版本。
3. 运行 `git merge origin/lab5-startercode` 下载 Lab 5 的起始代码。
4. 在 build 目录内编译源代码：`make`（可以用 `make -j4` 之类的命令用四个处理器并行编译）。
5. 在 build 目录外，打开并开始编辑 `writeups/lab5.md`。这是你实验报告的模板，将包含在你的提交里。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">**3 The Address Resolution Protocol（地址解析协议）**</span>

在开始写代码之前，请阅读：

- `NetworkInterface` 对象的公开接口。
- Wikipedia 上对 ARP 的总结，以及 ARP 的原始规范（RFC 1982）。
- `EthernetFrame` 与 `EthernetHeader` 对象的文档/实现。
- `IPv4Datagram` 与 `IPv4Header` 对象的文档与实现（它们能解析并序列化 Internet 数据报，序列化结果可以赋给以太网帧的载荷）。
- `ARPMessage` 对象的文档与实现（它知道如何解析与序列化 ARP 报文，序列化后同样可以作为以太网帧的载荷）。

你在本实验中的主要任务，是实现 `NetworkInterface` 的三个主要方法（位于 `network_interface.cc` 文件），维护一个从 IP 地址到以太网地址的映射。<span style="color:#00838f;">这个映射是一个缓存（cache），或称"软状态"（soft state）：NetworkInterface 出于效率考虑把它保存在身边，但如果必须从零开始，映射也会自然地重新生成，不会造成问题。</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "软状态"（soft state）是分布式系统里一个非常值得咀嚼的设计词汇：它与"硬状态"相对，指丢了也能靠协议自动重建的临时信息。你在 CSAPP ch11 里其实已经见过现成的例子——内核对套接字状态与地址解析结果的管理：即使进程崩溃重启，这些信息也能重新建立，通信可以照常继续。ARP 正是同样的思路。顺带澄清一处：正文提到的"RFC 1982"，实际指的是 1982 年由 David Plummer 发表的 ARP 原始规范 RFC 826。它一开始就假设局域网里任何机器都可能随时掉电重启——只要缓存条目过期后重新广播询问一遍，映射就会自然恢复，所以根本不需要专门的一致性协议来维护它。反过来想，如果把 IP→MAC 映射做成需要人工配置或专门同步协议维护的"硬状态"，局域网的鲁棒性会大打折扣。这个"缓存 + 过期 + 可重建"三件套，在 DNS 缓存与 DHCP 租约中还会反复出现。

**方法一**

```cpp
void NetworkInterface::send_datagram(const InternetDatagram &dgram,
                                     const Address &next_hop);
```

当调用方（例如你的 TCPConnection 或一台路由器）想把一个出站 Internet（IP）数据报发给下一跳时，会调用这个方法。¹ 把这个数据报翻译成以太网帧并（最终）发送出去，是你的接口的职责。

> <span style="color:#7f8c8d;">¹ 请不要把数据报的最终目的地——记录在数据报自身首部里的目的地址字段——与下一跳混为一谈。在本实验中你只关心下一跳的地址。</span>

- 如果目的以太网地址已知：立即发送。创建一个以太网帧（`type = EthernetHeader::TYPE_ipv4`），把载荷设为序列化后的数据报，并设置源地址与目的地址。
- 如果目的以太网地址未知：广播一个 ARP 请求（ARP request）询问下一跳的以太网地址，并把该 IP 数据报排入队列，待收到 ARP 应答后再发送。
- 例外：你不会想用 ARP 请求淹没（flood）网络。如果网络接口在过去五秒内已经就同一 IP 地址发送过 ARP 请求，就不要再发第二个请求——只需等待第一个请求的应答。同样，把数据报排入队列，直到你学到目的以太网地址为止。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 把"五秒内同一 IP 只发一次 ARP 请求、其余数据报排队等待"这条规则放大来看，是一个典型的"请求合并 + 速率限制"设计。设想一个突发（burst）场景：服务器瞬间要向同一网关发 1000 个数据报，而 ARP 缓存恰好过期。没有限制的话，1000 份重复的 ARP 广播会同时涌向整个局域网——广播帧会被网段上每一块网卡处理，成本随主机数线性放大；从概率论排队模型看，这就是到达率瞬间超过服务率、队列爆炸的过载情形。有了合并机制，1000 个数据报只触发一次广播，随后大家在队列里等第一个应答回来，你甚至可以用利特尔法则（Little's law：平均队长 = 平均到达率 × 平均等待时间）粗估这段等待期内的积压量。这种"等一个应答、惠及一串请求"的结构，在 DNS 缓存和 TCP 拥塞控制里都会以不同面目重现。

**方法二**

```cpp
optional<InternetDatagram> NetworkInterface::recv_frame(const EthernetFrame &frame);
```

当一个以太网帧从网络抵达时，此方法被调用。代码应当忽略所有不以本网络接口为目的地的帧（也就是说，以太网目的地既不是广播地址（broadcast address）、也不是保存在 `ethernet_address` 成员变量中的接口自身以太网地址）。

- 如果入站帧是 IPv4：把载荷解析为 InternetDatagram，若成功（即 `parse()` 方法返回 `ParseResult::NoError`），就把得到的 InternetDatagram 返回给调用方。
- 如果入站帧是 ARP：把载荷解析为 ARPMessage，若成功，则记住发送方 IP 地址与以太网地址之间的映射，保存 30 秒。（从请求和应答中都要学习映射。）此外，如果这是一个询问我们 IP 地址的 ARP 请求，就发送一个相应的 ARP 应答。

**方法三**

```cpp
void NetworkInterface::tick(const size_t ms_since_last_tick);
```

随时间流逝被调用。让所有已过期的 IP 到以太网映射失效。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 为什么映射缓存 30 秒，而不是 30 分钟或半秒钟？这是缓存一致性与效率之间的权衡：以太网地址很少变，但网卡确实会被更换、主机的 IP 也可能被 DHCP 重新分配，缓存太久就会把帧发给错误的网卡；缓存太短则每次通信都要先付一次广播询问的时延。30 秒是个经验值——足够覆盖一次典型交互，又能在合理时间内纠正错误。另一个值得注意的实现细节是 `tick()` 的建模方式：课程框架不给你真实的定时器线程，而是每次喂给你"距上次 tick 过去了多少毫秒"，由你自己推进内部时钟。这与你 Lab 3 实现 TCPSender 的 RTO 超时定时器时面对的模型完全一致：时间不是事件驱动的中断，而是一个由测试者显式驱动的离散时钟。好处是测试完全可复现——你可以精确断言"第 29 秒条目还在、第 31 秒已失效"。

运行 `ctest -V -R "^arp"` 即可测试你的实现。该测试不依赖你的 TCP 实现。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">**4 webget, revisited（重访 webget）**</span>

还记得你在 Lab 0 写的 `webget.cc` 吗（当时用的是 Linux 提供的 TCP 实现 TCPSocket）？还记得你在 Lab 4 里怎么把它改成使用你自己的 TCP-in-IP 实现（CS144TCPSocket）吗？正如上文讨论的，协议栈的一部分仍然依赖 Linux 内核：在 IP 与链路层（以太网）之间做转换的网络接口。

我们希望你在不改变其他任何东西的前提下，把它切换成使用你自己的网络接口。你需要做的只是把 CS144TCPSocket 类型替换为 `FullStackSocket`。

它使用的就是图 1（左侧）所示的 TCP-in-IP-in-Ethernet 协议栈：你的 `webget.cc` 应用程序，架在你实现的 TCPConnection 之上；其下是 `tcp_helpers/tcp_over_ip.cc` 中的 TCP-in-IP 代码；再往下是你的 NetworkInterface。

重新编译，然后运行 `make check_lab5`，确认你已经"全栈在手"（full-stack）：你已经在自己完整的 TCP 实现和自己实现的网络接口之上，写出了一个基本的网页抓取器，而且它仍然能成功地与真实 Web 服务器对话。

如果遇到问题，试着手动运行程序：`./apps/webget cs144.keithw.org /hasher/xyzzy`，并试着用 wireshark 抓取它发送和接收的内容。你可以运行 `sudo tcpdump -i tap10 -w /tmp/packets.tap` 来保存它收发的分组，然后在 wireshark 里打开 `/tmp/packets.tap` 文件。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">**5 Q&A（问答）**</span>

- <span style="color:#00838f;">**问：你预计需要写多少代码？**</span> 答：总体上，我们预计实现（在 `network_interface.cc` 中）总共需要大约 100–150 行代码。
- <span style="color:#00838f;">**问：如何领取提前提交的 15% 加分？**</span> 答：我们想鼓励你在感恩节假期之前完成本实验（部分原因也是给 12 月 6 日截止的 Lab 6 留出足够时间），但同时我们想给需要或想要更多时间的同学留出弹性。实验于 12 月 4 日下午 5 点截止，但我们为提前提交提供 15% 的加分。领取办法：1. 在 11 月 22 日下午 5 点前提交；2. 在 11 月 22 日下午 5 点前给助教邮件列表（cs144-aut1920-staff@lists.stanford.edu）发邮件，写明 "I would like to take the extra-credit option on Lab 5."（我想选择 Lab 5 的加分选项）；3. 我们将批改你在 11 月 22 日下午 5 点前的最后一次提交，无论分数多少都乘以 1.15；4. 如果你不想领取加分，就正常提交（包括使用任何 late day）。放弃加分选项不会受到惩罚（例如，CS144 不按曲线给分）。
- <span style="color:#00838f;">**问：NetworkInterface 实际上是怎么把以太网帧发出去的？**</span> 答：与 TCPSender 和 TCPConnection 的做法类似。对 NetworkInterface 而言，把任何出站帧 push 进 `frames_out` 队列，帧就会被该对象的所有者弹出并发送。
- <span style="color:#00838f;">**问：我应该用什么数据结构记录下一跳 IP 地址与以太网地址之间的映射？**</span> 答：由你决定！
- <span style="color:#00838f;">**问：如何把以 Address 对象形式出现的 IP 地址，转换成可以写进 ARP 报文的原始 32 位整数？**</span> 答：使用 `Address::ipv4_numeric()` 方法。
- <span style="color:#00838f;">**问：如果 NetworkInterface 发出了 ARP 请求却一直收不到应答怎么办？应该超时后重发吗？应该用 ICMP 向原始发送方报告错误吗？**</span> 答：在真实生活中，是的，这两件事都会做，但在本实验中不必担心。（真实系统里，如果接口无法收到 ARP 请求的应答，它最终会向 Internet 上的原始发送方回送一个 ICMP "host unreachable"（主机不可达）报文。）
- <span style="color:#00838f;">**问：如果一个 InternetDatagram 正排队等待得知下一跳的以太网地址，而这个信息迟迟不来怎么办？应该在某个超时后丢弃数据报吗？**</span> 答：同样，真实生活中答案是肯定的"要"，但本实验不必担心。
- <span style="color:#00838f;">**问：如何运行本实验的测试套件？**</span> 答：`make check_lab5`（两个测试）。也可以用 `make check` 运行整个测试套件（160 个测试）。
- <span style="color:#00838f;">**问：如果这份 PDF 发布之后还有更多 FAQ，去哪里看？**</span> 答：请定期查看网站（https://cs144.github.io/lab_faq.html）和 Piazza。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">**6 Submit（提交）**</span>

1. 提交时请只修改 `libsponge` 顶层的 `.hh` 和 `.cc` 文件。在这些文件里，欢迎按需添加私有成员，但请不要改变任何类的公开接口。
2. 请不要添加额外文件——自动评分程序不会看它们，你的代码还可能因此编译失败。
3. 在交任何作业之前，请按顺序运行：(a) `make format`（统一代码风格）；(b) `git status`（检查是否有未提交的改动——有的话就提交！）；(c) `make`（确保代码能编译）；(d) `make check_lab5`（确保自动测试全部通过）。
4. 在 `writeups/lab5.md` 中写一份报告。这份文件应是一份大约 20–50 行、每行不超过 80 个字符的文档，以便阅读。报告应包含以下小节：
   - (a) **Program Structure and Design（程序结构与设计）**：描述你代码体现的高层结构与设计取舍。不必详细讨论你从起始代码继承了什么。请借此机会突出重要的设计方面，并为评分助教提供更详细的说明。强烈建议使用小标题和提纲让这份报告尽量易读，请不要把程序简单翻译成一段英文。
   - (b) **Implementation Challenges（实现难点）**：描述你觉得最棘手的代码部分并解释原因；反思你如何克服这些困难、是什么最终帮你理解了困扰你的概念；你如何尝试保证代码维持你的假设（assumption）、不变量（invariant）与前置条件（precondition）？哪些方面觉得容易或困难？你如何调试和测试代码？
   - (c) **Remaining Bugs（遗留缺陷）**：尽你所能指出并解释代码中仍然存在的 bug（或未处理的边界情况）。
5. 请同时填写完成本作业所花的小时数以及任何其他评论。
6. 准备好提交时，请遵循 https://cs144.github.io/submit 上的说明。请确保在提交之前已 commit 所有打算提交的内容。
7. 有任何问题请尽快在周二晚实验课上告知课程团队，或在 Piazza 上发帖。祝你好运！
