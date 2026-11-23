> <span style="color:#7f8c8d;">Source: src/实验讲义/Lab 6 - IP路由器.txt · CS144 Fall 2019 · 行内双语翻译版</span>

# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#c0392b;">**Lab 6: building an IP router（Lab 6：构建一个 IP 路由器）**</span>

> <span style="color:#7f8c8d;">Fall 2019 · 截止时间：12 月 6 日（周五）晚 11 点 · 实验课：12 月 3 日（周二）晚 7:30–10 点，地点 STLC114</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">**0 Collaboration Policy（协作政策）**</span>

编程作业必须独立完成：除了我们随作业发给你的那部分代码之外，你为编程作业上交的所有代码都必须出自你手。请不要从 Stack Overflow、GitHub 或其他来源复制粘贴代码。如果你的代码借鉴了网上或其他地方找到的示例，请在提交的源代码中以注释形式注明 URL（URL）。

关于与他人合作：你不可以把自己的代码展示给任何人、查看任何人的代码，或查看往年的解答。你可以和其他同学讨论作业，但不能抄袭任何人的代码。如果你与另一位同学讨论过作业，请在提交的源代码注释中写上对方的名字。更多细节请参阅课程行政讲义（course administrative handout），有不清楚之处请在 Piazza 上提问。

Piazza：欢迎在 Piazza 上提问，但请不要发布任何源代码。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">**1 Overview（概述）**</span>

在本周的实验中，你将在已有的 NetworkInterface 之上实现一个 IP 路由器（IP router），为本课程画上句号。路由器有若干个网络接口，可以在其中任何一个接口上收到 Internet 数据报（Internet datagram）。路由器的职责是按照转发表（routing table）转发收到的数据报——转发表是一份规则列表，它告诉路由器，对任何一个到来的数据报：

- 从哪个接口发出去；
- 下一跳（next hop）的 IP 地址是什么。

你的任务是实现一个路由器，能为任何给定的数据报算出这两件事。（你不需要实现生成转发表的那些算法，例如 RIP、OSPF、BGP 或 SDN 控制器（SDN controller）——只需要实现"遵循转发表"的算法。）

你的路由器实现将使用 Sponge 库中的一个新 Router 类，配套的测试会在一个模拟网络中检验你路由器的功能。Lab 6 建立在你 Lab 5 实现的 NetworkInterface 之上，但不使用你在 Lab 0–4 中实现的 TCP 协议栈。IP 路由器不需要知道任何关于 TCP、ARP 或以太网的事情（只需要懂 IP）。我们预计你的实现大约需要 25–30 行代码。

图 1（Figure 1）中路由器的层次（自上而下）：

```text
Lab 6: Routing logic (longest-prefix match)
Internet datagrams ⇄ Network Interface ×3
Link-layer device (NIC) ── Network to London / São Paulo / Chicago
```

> 图 1：一台路由器包含若干网络接口，可以在其中任何一个接口上收到 IP 数据报。路由器把收到的任何数据报转发给下一跳——从合适的出站接口发出。转发表告诉路由器如何做这个决定。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">**2 Getting started（开始动手）**</span>

1. 确保你已提交（commit）Lab 5 的全部解答。请不要改动 `libsponge` 目录顶层之外的任何文件，也不要改 `webget.cc`（并且不要添加你的代码所依赖的额外文件），否则你在合并 Lab 6 起始代码时可能遇到麻烦。
2. 在实验作业仓库内运行 `git fetch`，获取实验作业的最新版本。
3. 运行 `git merge origin/lab6-startercode` 下载起始代码（原文写作 "Lab 5"，应为笔误，实际命令是 lab6 分支）。
4. 在 build 目录内编译源代码：`make`（可以用 `make -j4` 之类的命令用四个处理器并行编译）。
5. 在 build 目录外，打开并开始编辑 `writeups/lab6.md`。这是你实验报告的模板，将包含在你的提交里。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">**3 Implementing the Router（实现路由器）**</span>

在本实验中，你将实现一个 Router 类，它能够：

- 维护一张转发表（转发规则列表，或称路由（route））；
- 转发它收到的每一个数据报：发往正确的下一跳，从正确的出站 NetworkInterface 发出。

你的实现将添加到 `router.hh` 和 `router.cc` 两个骨架文件中。在开始写代码之前，请先阅读新 Router 类的文档。

下面是你要实现的两个方法，以及我们对每个方法的期望：

```cpp
void add_route(const uint32_t route_prefix, const uint8_t prefix_length,
               const optional<Address> next_hop, const size_t interface_num);
```

这个方法向转发表中添加一条路由。你需要在 Router 类里添加一个私有数据成员来保存这些信息。此方法需要做的就是保存这条路由，供以后使用。

一条路由的各个部分是什么含义？

路由是一条"匹配-动作"（match-action）规则：它告诉路由器，如果一个数据报正发往某个特定网络（一段 IP 地址范围），并且这条路由被选为"最具体的匹配路由"，那么路由器就应该把数据报转发给某个特定接口上的特定下一跳。

**"匹配"部分**：这个数据报是不是发往本网络的？`route_prefix` 与 `prefix_length` 共同指定一段 IP 地址范围（一个网络），它可能包含数据报的目的地址。`route_prefix` 是一个 32 位的数值 IP 地址；`prefix_length` 是 0 到 32 之间（含两端）的一个数，它告诉路由器 route_prefix 的最高多少位是有效的。

<span style="color:#2471a3;">**[example]**</span> 举例来说，要表达一条通往网络 "18.47.0.0/16" 的路由（它匹配任何前两个字节为 18 和 47 的 32 位 IP 地址），route_prefix 应为 305070080（即 $18 \times 2^{24} + 47 \times 2^{16}$），prefix_length 应为 16。任何发往 "18.47.x.y" 的数据报都会匹配这条路由。

**"动作"部分**：如果这条路由匹配并且被选中，要做什么？如果路由器直接连接在目标网络上，`next_hop` 将是一个空的 optional——此时下一跳就是数据报自己的目的地址。但如果路由器是经由别的路由器连接到目标网络的，`next_hop` 将包含路径上下一个路由器的 IP 地址。`interface_num` 给出路由器的某个 NetworkInterface 的编号，应当用这个接口把数据报发给下一跳。你可以用 `interface(interface_num)` 方法访问这个接口。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 最长前缀匹配（longest-prefix match）值得从算法角度多想一步。本实验你只需要"查表"——这一层叫数据面（data plane）；而转发表本身是由路由协议算出来的——这一层叫控制面（control plane），其核心正是你在 CLRS 里学过的最短路问题：OSPF（开放最短路优先）在每个路由器上运行 Dijkstra 算法，以链路代价为边权计算到所有目的网络的最短路树；RIP 这类距离向量（distance vector）协议则是分布式版的 Bellman–Ford，靠邻居之间反复交换距离估计值逐步收敛。复杂度上，Dijkstra 用二叉堆实现是 $O((V+E)\log V)$，Bellman–Ford 是 $O(VE)$——CLRS 第 24 章有完整证明。于是整幅图景是：控制面用图论算法偶尔算一次表，数据面对每个到达的数据报查一次表。计算慢而少，查找快而多——这是整个 Internet 转发体系的基本分工，也解释了为什么本实验的接口把"填表"（add_route）与"查表"（route_one_datagram）分成两个方法。

```cpp
void route_one_datagram(InternetDatagram &dgram);
```

真正见真章的地方在这里。这个方法要把一个数据报路由到下一跳、从合适的接口发出。它需要实现 IP 路由器的"最长前缀匹配"（longest-prefix match）逻辑，以找到应遵循的最佳路由。也就是说：

- Router 在转发表中搜索所有匹配该数据报目的地址的路由。所谓"匹配"，是指目的地址的最高 prefix_length 位与 route_prefix 的最高 prefix_length 位完全相同。
- 在所有匹配的路由中，路由器选择 prefix_length 值最大的那一条。<span style="color:#00838f;">这就是最长前缀匹配（longest-prefix match）路由。</span>
- 如果没有任何路由匹配，路由器丢弃该数据报。
- 路由器把数据报的 TTL（time to live，生存时间）减一。如果 TTL 原本就是零，或减完之后变成零，路由器应丢弃该数据报。
- 否则，路由器通过合适的接口（`interface(interface_num).send_datagram()`）把修改后的数据报发给合适的下一跳。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> TTL（生存时间）是给路由环路（routing loop）准备的保险丝。回想 6.042J 的图论视角：只要转发表因路由暂时不一致或配置错误而成环，数据报就可能沿环无限转圈；距离向量协议更有著名的"计数到无穷"（count-to-infinity）问题——链路一旦断开，两个路由器可能互相把对方当作中转，距离估计一路膨胀。工程上无法要求每台路由器时刻杜绝环路，于是退而求其次：让每个数据报自带一个"最多还能经过几台路由器"的计数器，每过一台减一，减到零就丢弃——环路的危害从"永久吞噬带宽"降级为"最多 64 或 255 跳的浪费"。注意讲义要求的判断顺序：先把 TTL 减一；若原本就是零、或减完为零，都必须丢弃——若放行 TTL 为零的数据报，当 255 台路由器构成环时它还能再活 255 跳。这种"给无界危害加上界"的思路，与概率论里给重试过程设上限、从而保证期望成本有限的论证异曲同工。

下面这段话体现了 Internet 设计中的一种美（至少是一次成功的抽象）：路由器从不考虑 TCP，不考虑 ARP，也不考虑以太网帧。路由器甚至不知道链路层长什么样。路由器只思考 Internet 数据报，并且只通过 NetworkInterface 这个抽象与链路层交互。至于"链路层地址是怎么解析的？""链路层是否有一套与 IP 不同的自己的编址方案？""链路层帧的格式是什么？""数据报的载荷是什么含义？"这类问题，路由器一概不关心。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这段话值得逐字品味：路由器不认识 TCP、不认识 ARP、甚至不知道链路层的存在——这是分层（layering）与封装（encapsulation）思想最纯粹的一次展示。你在 CSAPP ch11 已经体验过同款抽象：写 Web 服务器时，你通过套接字（socket）接口读写字节流，完全不必知道 TCP 如何做可靠传输、IP 如何选路、以太网帧头长什么样——socket 接口就是应用与传输层之间的一条窄缝。本实验里，NetworkInterface 扮演了类似角色：它是路由器与链路层之间唯一的窄缝，向上只暴露收发数据报的方法。抽象的代价是多一层间接调用，收益是每层可以独立演进——Internet 上的链路技术换了一茬又一茬（以太网、Wi-Fi、光纤、蜂窝），IP 层及其以上几乎不动，正因为谁都不需要知道谁的内部细节。这就是"窄腰"（narrow waist）设计的力量。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">**4 Testing（测试）**</span>

运行 `make check_lab6` 即可测试你的实现。它会在一个特定的模拟网络中测试你的路由器，该网络如图 2 所示。

```text
NetworkInterface "default_router" 171.67.76.1
NetworkInterface "applesauce" 10.0.0.2
NetworkInterface "cherrypie" 192.168.0.2
NetworkInterface "default" 171.67.76.46
route: 0.0.0.0/0 via 171.67.76.1
NetworkInterface "eth0" 10.0.0.1
route: 10.0.0.0/8 (direct)
NetworkInterface "eth1" 172.16.0.1
route: 172.16.0.0/16 (direct)
NetworkInterface "eth2" 192.168.0.1
route: 192.168.0.0/24 (direct)
route: 198.178.229.0/0 (direct)
NetworkInterface "uun3" 198.178.229.1
Router
route: 143.195.0.0/17 via 143.195.0.1
route: 143.195.128.0/18 via 143.195.0.1
route: 143.195.192.0/19 via 143.195.0.1
NetworkInterface "hs4" 143.195.0.2
route: 128.30.76.255/16 via 128.30.0.1
NetworkInterface "mit5" 128.30.76.255
NetworkInterface "dm42" 198.178.229.42
NetworkInterface "dm43" 198.178.229.43
NetworkInterface "hs_router" 143.195.0.1
```

> 图 2：`apps/network_simulator` 工具所用（也是 `make check_lab6` 所运行）的模拟测试网络。（趣味知识：uun 网络是 David Mazières 于 1993 年分到的 Internet 地址块。用 whois 工具或相关网站可以查到每个 IP 地址段的控制者。）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">**5 Q&A（问答）**</span>

- <span style="color:#00838f;">**问：我该用什么数据结构来记录转发表？**</span> 答：由你决定！但不必走火入魔。每个数据报需要 $O(N)$ 的工作量（其中 N 是转发表条目数）是完全可以接受的。如果你想做得更高效，我们建议你先做出一个能工作的实现再优化，并为你最终选择的实现写好文档和注释。
- <span style="color:#00838f;">**问：如何把以 Address 对象形式出现的 IP 地址，转换成可以写进 ARP 报文的原始 32 位整数？**</span> 答：使用 `Address::ipv4_numeric()` 方法。
- <span style="color:#00838f;">**问：如何把一个原始 32 位整数形式的 IP 地址转换回 Address 对象？**</span> 答：使用新推出的 `Address::from_ipv4_numeric()` 方法。
- <span style="color:#00838f;">**问：如何比较一个 32 位 IP 地址的最高 N 位（$0 \le N \le 32$）与另一个 32 位 IP 地址的最高 N 位？**</span> 答：这大概是本作业"最刁钻"的部分——把这段逻辑写对。值得单独写一个小测试程序（一段独立的 C++ 短程序）或给 Sponge 添加一个测试，来验证你对相关 C++ 运算符的理解，并复核你的逻辑。
  请回忆：在 C 和 C++ 里，把 32 位整数移位 32 位会产生未定义行为（undefined behavior）。一个好习惯是用净化器（sanitizer）做一次干净的编译（`cmake -DCMAKE_BUILD_TYPE=RelASan ..`），在提交之前尽量捕捉代码里的未定义行为。
  你可以在 build 目录里直接运行 `./apps/network_simulator` 来运行路由器测试。
- <span style="color:#00838f;">**问：如果路由器没有通往目的地的路由，或者 TTL 减到了零，难道不应该向数据报的源头发送一个 ICMP 错误报文吗？**</span> 答：真实生活中，是的，那会很有帮助。但本实验不要求——丢弃数据报就足够了。（即便在真实世界里，也不是每台路由器都会在这些情况下给源头回送 ICMP 报文。）
- <span style="color:#00838f;">**问：如何运行本实验的测试套件？**</span> 答：`make check_lab6`（两个测试）。也可以用 `make check` 运行整个测试套件（161 个测试）。
- <span style="color:#00838f;">**问：如果这份 PDF 发布之后还有更多 FAQ，去哪里看？**</span> 答：请定期查看网站（https://cs144.github.io/lab_faq.html）和 Piazza。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 讲义说每个数据报 $O(N)$ 的线性扫描完全可接受——先把功能做对、再谈优化，这是工程上极其正确的顺序。但值得算一笔账：真实 Internet 骨干路由器的 BGP 转发表在 2019 年约有 80 万条，若每秒转发上亿个分组，线性扫描意味着每秒要做数万亿次比较，完全不可行。所以真实路由器使用更聪明的结构：把前缀组织成 trie（前缀树/基数树——把每个 32 位 IP 看成一串从根到叶的二进制选择，每层走 0 或 1），最长前缀匹配就是"记录沿途最后匹配到的节点"，单次查找只需 O(32) 步，与表项数无关。你也许会问：CLRS 里哈希表不是能把查找降到 $O(1)$ 吗？问题在于前缀匹配找的是"最长匹配的键"而非精确键，哈希表帮不上忙，树形结构才是正解。学有余力的话，不妨用 trie 实现一版，再拿讲义的模拟网络对比性能。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">**6 Submit（提交）**</span>

1. 提交时请只修改 `libsponge` 顶层的 `.hh` 和 `.cc` 文件。在这些文件里，欢迎按需添加私有成员，但请不要改变任何类的公开接口。
2. 请不要添加额外文件——自动评分程序不会看它们，你的代码还可能因此编译失败。
3. 在交任何作业之前，请按顺序运行：(a) `make format`（统一代码风格）；(b) `git status`（检查是否有未提交的改动——有的话就提交！）；(c) `make`（确保代码能编译）；(d) `make check_lab6`（确保自动测试全部通过）。
4. 在 `writeups/lab6.md` 中写一份报告。这份文件应是一份大约 20–50 行、每行不超过 80 个字符的文档，以便阅读。报告应包含以下小节：
   - (a) **Program Structure and Design（程序结构与设计）**：描述你代码体现的高层结构与设计取舍。不必详细讨论你从起始代码继承了什么。请借此机会突出重要的设计方面，并为评分助教提供更详细的说明。强烈建议使用小标题和提纲让这份报告尽量易读，请不要把程序简单翻译成一段英文。
   - (b) **Implementation Challenges（实现难点）**：描述你觉得最棘手的代码部分并解释原因；反思你如何克服这些困难、是什么最终帮你理解了困扰你的概念；你如何尝试保证代码维持你的假设（assumption）、不变量（invariant）与前置条件（precondition）？哪些方面觉得容易或困难？你如何调试和测试代码？
   - (c) **Remaining Bugs（遗留缺陷）**：尽你所能指出并解释代码中仍然存在的 bug（或未处理的边界情况）。
5. 请同时填写完成本作业所花的小时数以及任何其他评论。
6. 准备好提交时，请遵循 https://cs144.github.io/submit 上的说明。请确保在提交之前已 commit 所有打算提交的内容。
7. 有任何问题请尽快在周二晚实验课上告知课程团队，或在 Piazza 上发帖。
8. 最后：拍拍自己的肩膀吧。你们是第一届经历全新"模块化"CS144 实验的学生，我们真心感谢你们对这些实验的耐心以及所有有益的反馈。未来的学生将受益于你们的经验。谢谢大家，祝寒假愉快！——Keith、Nick、Sarah、Nick、Will、Alex、Emily 和 Sadjad
