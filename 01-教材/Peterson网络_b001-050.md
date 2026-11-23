<!-- 书页 1–36（PDF 1–40；PDF 1–4 为封面/空白/目录/前言，书页 < 1）｜来源：Peterson网络_p001-020.bilingual.md、Peterson网络_p021-040.bilingual.md｜PDF 页 = 书页 + 4｜机械重组，内容未改 -->
# Computer Networks: A Systems Approach（计算机网络：系统方法）· 行内双语

> **原书**：Larry L. Peterson, Bruce S. Davie, *Computer Networks: A Systems Approach*, Release Version 6.1（第 6 版开源发行版），2019-11-26。
> **本卷覆盖**：源文件 `Peterson网络_p001-020.pdf`，PDF 第 1–20 页 = 封面、空白页、目录（书页 i–ii）、PREFACE（书页 1–4）、Chapter 1 Foundation 起首至 §1.2.3 statistical multiplexing 前段（书页 5–16）。
> **切块约定**：每块以 `<!-- ===== pNNN ===== -->` 开头，NNN 为 **PDF 页码**；块内 `> 来源注` 一行给出该页对应的印刷页码与页眉、页脚信息。

**阅读约定**

1. 译文以中文为主体，术语首次出现写 `English（中文短注）`，此后正文只留 English 原词。
2. 网络四层术语与 CS144 讲义双语版保持一致，一律不意译：**application layer（应用层）/ transport layer（传输层）/ network layer（网络层）/ link layer（链路层）**。其余核心词同样保留：packet / frame / datagram / message / segment / socket / ACK / window / congestion control / flow control / routing / forwarding / MTU / ARP / DNS / TCP / UDP / IP / switch / router / host / channel / multiplexing / encapsulation / header / payload。
3. 行内公式用 `$...$`，块级公式用 ```math 围栏；不使用行间美元定界符，也不使用反斜杠方括号定界符。
4. 类型标签配色：<span style="color:#1f6feb;">**[Key Takeaway]**</span> 原书自带小结框；<span style="color:#117864;">**[definition]**</span> 定义；<span style="color:#6c3483;">**[example]**</span> 例子；<span style="color:#943126;">**[note]**</span> 原书脚注与插框。**自写内容一律绿色 `[译注]`，绝不占用上述任何颜色。**
5. 本卷不含习题（原书 Exercises 集中在各章末，本 20 页范围内无 `[exercise]` 块）。
6. 插图以 `[Figure N：它演示的是什么]` 保留；若该页只有图题、文字层无法复原拓扑与箭头顺序，明写「无法复原」并只给正文可核的部分，不编造。
7. 原文自身错误以 `†` 标记，并在该处给出判定依据。

---

<!-- ===== p001 ===== -->
> 来源注：PDF 第 1 页 = 封面（全页无页码）；页眉无，页脚为日期 `Nov 26, 2019`。

## 封面

**Computer Networks: A Systems Approach**（计算机网络：系统方法）

Release Version 6.1

Peterson and Davie

Nov 26, 2019

> **版本坐标**：这是 6th Edition 的 **Release Version 6.1**，即作者以「living document（活文档）」形式持续修订的开源发行版。与商业印刷的第 6 版相比，6.1 修订的是细节与勘误，章号与节号未重排——这一点很关键：CS144 的 reading 按节号引用（如 §5.2 Reliable Byte Stream），若版本间节号漂移，课程 reading 就会失效。

<span style="color:#1e8449;">**[译注]** 封面上只有三样东西可锚：书名、版本号、日期。值得在这里停下的是 **"A Systems Approach"** 这个方法名前缀——它不是副标题装饰，而是对全书组织方式的承诺：书不按「协议清单」组织，而按「设计问题 → 需求 → 机制 → 取舍」组织。判据在于目录：第 1 章是 Foundation（需求与架构），其后每章都先给 Requirements 再给具体协议；若换成协议大全式写法，章序会变成 OSI 自下而上的 physical → link → network，而本书把 End-to-End Protocols（第 5 章）放在 Congestion Control（第 6 章）之前，正是因为「端到端可靠字节流」这个需求先于「怎么控制拥塞」这个优化问题。

**消失测试**：抽掉「systems approach」这条组织原则，第 1 章会退化为术语表（layer / protocol / service 三个词的定义），读者拿到 §5.2 TCP 时无法回答「为什么 TCP 的 window 既是 flow control 又是 congestion control 的载体」——因为这个问题只能在「需求 → 机制」的框架里提出。

**落点**：§1.2 Requirements（本卷 p013 起）；§1.3 Architecture（本卷 p020 起，续见下一卷）。CS144 课程的对应位置是 Lecture 1–2 的「the Internet is a system of systems」，以及 Lab1–Lab3 逐层加代码的顺序。</span>

---

<!-- ===== p002 ===== -->
> 来源注：PDF 第 2 页 = 空白页（版式留白，位于封面与目录之间）；文字层为空。

本页为空白页，无内容。

---

<!-- ===== p003 ===== -->
> 来源注：PDF 第 3 页 = 目录第 1 页（书页 i）；页脚 `i`；页眉 `TABLE OF CONTENTS`。

## TABLE OF CONTENTS（目录，第 1 页）

**1 Foundation**（基础）— 5
　1.1 Applications — 6
　1.2 Requirements — 9
　1.3 Architecture — 21
　1.4 Software — 31
　1.5 Performance — 37

**2 Direct Links**（直连链路）— 47
　2.1 Technology Landscape — 48
　2.2 Encoding — 51
　2.3 Framing — 54
　2.4 Error Detection — 59
　2.5 Reliable Transmission — 65
　2.6 Multi-Access Networks — 77
　2.7 Wireless Networks — 83
　2.8 Access Networks — 95

**3 Internetworking**（网络互联）— 103
　3.1 Switching Basics — 104
　3.2 Switched Ethernet — 116
　3.3 Internet (IP) — 125
　3.4 Routing — 151
　3.5 Implementation — 169

**4 Advanced Internetworking**（高级网络互联）— 179
　4.1 Global Internet — 180
　4.2 IP Version 6 — 190
　4.3 Multicast — 200
　4.4 Multiprotocol Label Switching — 210
　4.5 Routing Among Mobile Devices — 221

**5 End-to-End Protocols**（端到端协议）— 229
　5.1 Simple Demultiplexor (UDP) — 230
　5.2 Reliable Byte Stream (TCP) — 233
　5.3 Remote Procedure Call — 260
　5.4 Transport for Real-Time (RTP) — 276

<span style="color:#1e8449;">**[译注]** 这半页目录是整个学期的时间表。**第 5 章 §5.2 Reliable Byte Stream (TCP) 占 233–259 共 27 页**，是全书单节篇幅最长的一块，也正好压在 CS144 的 Lab1–Lab4 上：Lab1 的 `ByteStream`（把乱序到达的子串拼成有序字节流）对应 §5.2 里 sequence number 与 reassembly 的处理；Lab2 的 TCP receiver 对应 acknowledgement 与 window advertisement；Lab3 的 TCP sender 对应 retransmission timer 与 sliding window；Lab4 的 connection management 对应三次握手与状态机。**第 6 章 Congestion Control（287 起）却在 CS144 的 lab 范围之外**——这是最容易误判的地方，见 p004 那条注。

**消失测试**：不知道 §5.2 = 233–259 这个区间，读 CS144 时会把讲义里的 `seqno` / `ackno` / `window size` 当成 lab 自定义的概念，而不知道教材已经在 27 页里把「为什么 window 必须由接收端公告」讲完了；后果是 Lab2 的 `window_size()` 会写成写死 64000 的常数，而 `Recv` 的容量检查失效。

**落点**：§5.2（书页 233）；CS144 Lab1 `byte_stream.cc` / Lab2 `tcp_receiver.cc` / Lab3 `tcp_sender.cc`；本机已有 CS144 讲义双语版可作为对照，术语以本卷的 application layer / transport layer / network layer / link layer 为准。</span>

---

<!-- ===== p004 ===== -->
> 来源注：PDF 第 4 页 = 目录第 2 页（书页 ii）；页脚 `ii`。

## TABLE OF CONTENTS（目录，第 2 页）

**6 Congestion Control**（拥塞控制）— 287
　6.1 Issues in Resource Allocation — 288
　6.2 Queuing Disciplines — 296
　6.3 TCP Congestion Control — 301
　6.4 Advanced Congestion Control — 311
　6.5 Quality of Service — 323

**7 End-to-End Data**（端到端数据）— 341
　7.1 Presentation Formatting — 342
　7.2 Multimedia Data — 355

**8 Network Security**（网络安全）— 373
　8.1 Trust and Threats — 374
　8.2 Cryptographic Building Blocks — 375
　8.3 Key Predistribution — 383
　8.4 Authentication Protocols — 389
　8.5 Example Systems — 395

**9 Applications**（应用）— 415
　9.1 Traditional Applications — 416
　9.2 Multimedia Applications — 437
　9.3 Infrastructure Applications — 449
　9.4 Overlay Networks — 461

<span style="color:#1e8449;">**[译注]** 目录下半页很容易被读成「后面还有四章，慢慢来」，但这里真正要标出的是**第 6 章与 CS144 Lab3 的边界**。CS144 的 TCP sender 只要求实现「基于 window 的发送与超时重传」，**不要求实现 congestion control**：lab 里没有一个地方要你维护 `cwnd`（拥塞窗口）。第 6 章 §6.3 讲的 AIMD 与慢启动，是端主机在**没有网络反馈**的前提下靠丢包/延迟推断拥塞的一整套启发式；把它误当成 Lab3 的必做项，会让你在 `tcp_sender.cc` 里塞一个没人调用的 `cwnd` 变量，而真正的 lab 判据（`bytes_in_flight()` 与 `window_size()` 的关系、`tick()` 的超时翻倍）反而没写对。

**判据的量化形式**：发送窗口实际受限量为

```math
\text{send\_window} = \min\big(\text{cwnd},\ \text{rwnd}\big),
```

其中 rwnd 由接收端公告（Lab2/Lab4）、cwnd 由拥塞控制算法维护（第 6 章）。Lab3 只测 rwnd 那一半；抽掉 cwnd 这一项，Lab3 全部测试仍应通过——如果通不过，说明你把 rwnd 的逻辑写进了本不属于它的位置。这个「抽掉仍成立」的检验，就是判断某个机制是否属于当前 lab 范围的最快办法。

**落点**：§6.3 TCP Congestion Control（书页 301）与 §5.2 的 window 机制（书页 233）；CS144 Lab3 的 `TCPSender::fill_window()` / `tick()` 两个函数；对照 CSAPP 第 11 章「socket 与阻塞 I/O」理解发送缓冲区为何不是无限大。</span>

---

<!-- ===== p005 ===== -->
> 来源注：PDF 第 5 页 = PREFACE 首页（书页 1）；页眉 `PREFACE`；页脚 `1`。

## PREFACE（前言）

自《Computer Networks: A Systems Approach》第 5 版出版以来已近十年。这些年变化很大，最显著的是 cloud（云）与 smartphone app（智能手机应用）的爆发式涌现。从许多方面看，这让人想起 1996 年我们出版本书第 1 版时，Web 对 Internet 正在产生的剧烈影响。†

> † 原文作 "the dramatic **affect** the Web was having"；此处 affect 应为 effect（名词）。这是全书反复出现的同一处拼写互换（另一例见 p014 附近 "the impact ... affect"），不影响语义。

第 6 版适应了时代，但把 **Systems Approach** 保持为它的北极星。概括来说，我们在四个方面更新与改进了这个新版本：

- 我们刷新了例子以反映世界的当前状态。这包括删除过时的东西（例如 dial-up modem，拨号调制解调器），改用流行应用（例如 Netflix、Spotify）来引出正在讨论的问题，并更新数字以代表当代技术的水平（例如 10-Gbps Ethernet）。
- 我们把「导致 multicast、real-time video streaming、quality-of-service 这些技术出现的原始研究」与「如今已为人熟知的 GoToMeeting、Netflix、Spotify 这类 cloud 应用」之间的点连了起来。这与我们强调**设计过程而非只看最终结果**一脉相承——在今天这一点尤其重要，因为 Internet 的很大一部分主要以专有商业服务的形式提供。
- 我们把 Internet 放进 Cloud 的更大背景中，同样重要的是，放进塑造 Cloud 的商业力量背景中。这对全书呈现的技术细节影响很小，但它在每章末尾新增的 **Perspective** 一节里得到讨论。我们希望这一讨论的一个副产品，是培养对 Internet 持续演化及其所代表的创新机会的理解。
- 我们把网络设计的重要原则提炼成贯穿全书的 **Key Takeaways**（要点）。每条 takeaway 都是一句简明的陈述，或是一条通用的系统设计规则，或是一个基本的网络概念，取材于其上下文中的例子。在教学法上，这些 takeaway 对应本书的高层学习目标。

<span style="color:#1f6feb;">**[Key Takeaway]**</span> 上面四条里，真正具有机制意义的是第三条与第四条：**Perspective** 把「技术为什么这样演化」写进书里，**Key Takeaways** 把每节的结论压缩成一句可被检验的设计规则。这两者都不是装饰——它们是这本 489 页的书能被当作一学期课程主干的原因。

<span style="color:#1e8449;">**[译注]** 前言的这四条更新里，只有第 4 条会影响你**怎么读**这本书，其余三条只影响**读什么例子**。Key Takeaways 的作用可以用一句话判据表达：**每条 takeaway 必须能独立成一条可被反驳的设计规则**，否则它就是章节摘要而不是 takeaway。例如本卷 p018（书页 14）那条「a network can be constructed from a nesting of networks」，可以被反驳的版本是「网络必须递归可组合：任取两个网络，用一个节点把它们连起来仍是网络」——这条一旦不成立，internetwork（网络之网络）的整套递归构造就崩掉，第 3 章 Internetworking 的前提也就没了。

**消失测试**：如果没有 Key Takeaways 这一层抽象，读者面对的是 9 章 ≈ 300 个具体协议决策；要回答「QUIC 为什么把可靠传输从内核搬到用户态」这类问题，只能靠记忆比对，而不能沿「哪条设计规则被放松了」这条线索回溯。反过来，有了这层规则，第 5 章新增的 QUIC 小节就落在一个可判定的位置上：它改变了「可靠字节流必须由 OS 提供」这条假设。

**落点**：全书 Key Takeaways 的分布可用 `grep -c "Key Takeaway"` 统计（本书每节末一条）；方法论上对应第 1 章 §1.2 的六条 requirements（本卷 p009–p020）。</span>

---

<!-- ===== p006 ===== -->
> 来源注：PDF 第 6 页 = PREFACE 续页（书页 2）；页脚 `2 TABLE OF CONTENTS`。

更具体地说，第 6 版包含以下主要变化：

- 第 1 章新增 Perspective 一节，引出反复出现的 **Cloudification**（云化）主题。
- 新增 §2.8 描述 Access Network（接入网），包括 Passive Optical Networks（PON，无源光网络）与 5G 的 Radio Access Networks（RAN，无线接入网）。
- 重构了 §3.1（Switching Basics）与 §3.2（Switched Ethernet）之间的主题分布，扩充了 VLAN 的覆盖。
- §3.5 更新，纳入 White-Box Switches（白盒交换机）与 Software-Defined Networks（SDN，软件定义网络）的描述。
- 第 3 章新增 Perspective 一节，描述 VXLAN 以及 overlay（覆盖网）在 Cloud 中的作用。
- 重构了 §4.1（Global Internet）与 §4.2（IP Version 6）之间的主题分布。
- 第 4 章新增 Perspective 一节，描述 Cloud 如何影响 Internet 的结构。
- §5.2 扩充，加入对 QUIC 的讨论。
- §5.3 扩充，加入对 gRPC 的描述。
- §6.3 与 §6.4 更新，加入对 TCP CUBIC、DCTCP 与 BBR 的描述。
- §6.4 扩充，加入对 Active Queue Management（AQM，主动队列管理）的描述。
- §7.1 扩充，加入对 Protocol Buffers 的描述。
- §7.2 扩充，加入对 HTTP Adaptive Streaming（HTTP 自适应流）的描述。
- 新增 §8.1，引出 Threats 与 Trust 的对偶关系。
- 重构了 §8.3（Key Predistribution）与 §8.2（Authentication Protocols）之间的主题分布。†

> † 原文写作 "Sections 8.3 (Key Predistribution) and 8.2 (Authentication Protocols)"，节号倒序排列；按正文实际顺序应为 8.2 在前、8.3 在后。这只是列举顺序问题，但读者若按此处描述去找「8.2 里的 Key Predistribution」会找不到。

- 第 8 章新增 Perspective 一节，描述 Decentralized Identity Management（去中心化身份管理）与 Blockchain（区块链）的作用。
- §9.1 更新，加入对 HTTP/2 的描述，并讨论 REST、gRPC 与 Cloud Services。
- §9.3 扩充，加入对现代 Network Management Systems 的描述，包括 OpenConfig 与 gNMI 的使用。

<span style="color:#1e8449;">**[译注]** 这 17 条变更清单里，**只有两条会改变你已有的 CS144 心智模型**，其余都是内容增补。第一条是 **§5.2 加入 QUIC**：CS144 教你的是「可靠字节流由内核的 TCP 提供，应用通过 socket 使用」；QUIC 把重传、流控、甚至部分连接管理搬到用户态库（如 Chromium 的 QUICHE），于是 transport layer 的实现位置从 kernel 变成了 application-linked library。第二条是 **§3.5 加入 White-Box Switches 与 SDN**：CS144 Lab6 里你写的是一个固定转发表的 IP router，而 SDN 把转发表的**生成**从路由器自身的分布式算法（第 3.4 节）搬到一个中心控制器——这两件事的改变量正好是「控制平面与数据平面分离」，对应本卷 p018 起讲的 routing 与 forwarding 的区分。

**可判定的判据**：判断一条变更是否触及模型，看它是否改变了「哪个组件持有哪份状态」。QUIC 把 congestion control 的 `cwnd` 状态从内核 socket 搬到用户进程，SDN 把路由表从每台交换机搬到控制器——两者都改变了状态归属；而「加入 Protocol Buffers 的描述」只是序列化格式的例子替换，状态归属未变。

**落点**：§3.5（书页 169）、§5.2 QUIC 小节（书页 233 起，具体位置以正文标题为准）；CS144 Lab5/Lab6 的网络接口与路由器；对照第 3 章 §3.4 Routing 理解「分布式 vs 集中式」的对偶。</span>

---

<!-- ===== p007 ===== -->
> 来源注：PDF 第 7 页 = Acknowledgements（书页 3）；页脚 `TABLE OF CONTENTS 3`。

## Acknowledgements（致谢）

我们要感谢以下人士对新增内容的帮助：

- Larry Brakmo：TCP Congestion Control
- Carmelo Cascone：White-Box Switches
- Charles Chan：White-Box Switches
- Jude Nelson：Decentralized Identity
- Oguz Sunay：Cellular Networks
- Thomas Vachuska：Network Management

以及以下各位（github 用户）的各类贡献与缺陷修复：

- Mohammed Al-Ameen
- Andy Bavier
- Manuel Berfelde
- Chris Goldsworthy
- John Hartman
- Diego López León
- Matteo Scandolo
- Mike Wawrzoniak
- (spacewander)
- Arnaud (arvdrpoo)
- Desmond (kingdido999)
- Guo (ZJUGuoShuai)
- Hellman (eshellman)
- Xtao (vertextao)
- Mike Appelman
- Seth (springbov)

最后，我们要感谢以下审阅者提出的许多有益评论与建议。他们的影响是显著的。

- Mark J. Indelicate，Rochester Institute of Technology
- Michael Yonshik Choi，Illinois Institute of Technology
- Sarvesh Kulkarni，Villanova University
- Alexander L. Wijesinha，Towson University

Larry & Bruce
2019 年 11 月

<span style="color:#1e8449;">**[译注]** 这一页在阅读上通常被跳过，但对这本以 GitHub 托管的开源教材而言，致谢名单是一条**可用的技术线索**：它按主题点名了新增内容的作者，等价于一份「哪一节是后来加进来的」索引。具体地，"Larry Brakmo: TCP Congestion Control" 这条指向 §6.3 的 CUBIC/BBR 增补（Brakmo 本人是 TCP Vegas 的作者之一，1994 年那篇论文是 delay-based congestion control 的源头）；"Oguz Sunay: Cellular Networks" 指向 §2.8 的 5G RAN。判据是：当你在 §6.3 读到 BBR 的瓶颈带宽估计时，知道它不是教科书作者的第一手工作，而是一位 TCP 研究者的投稿，因此该处的表述会更贴近实现而非教学顺序。

**消失测试**：没有这页，你仍然能读完全书；但你会少一条「哪一节与全书其他部分的年代不同」的线索——尤其在第 1 章的 1996 年话语与第 6 章的 BBR（2016）之间，读者容易误以为全书同代。

**落点**：§6.3（书页 301）的 CUBIC/BBR 段落；§2.8（书页 95）的 5G RAN。开源发行版的 GitHub issue 与 commit 历史是比致谢更细的一手索引（A 级来源）。</span>

---

<!-- ===== p008 ===== -->
> 来源注：PDF 第 8 页 = 空白页（书页 4），仅页眉与页脚；文字层只有两行重复的页眉/页脚。

本页为空白页，无正文内容（原书版式留白，用于让第 1 章从右手页起排）。

---

<!-- ===== p009 ===== -->
> 来源注：PDF 第 9 页 = 第 1 章首页（书页 5）；页眉 `CHAPTER ONE / FOUNDATION`；页脚 `5`。

# 第 1 章 FOUNDATION（基础）

> *I must create a System, or be enslav'd by another Man's; I will not Reason and Compare: my business is to Create.*
> ——William Blake（我必须造出一个系统，否则就要被别人的系统奴役；我不去推理与比较：我的事业是创造。）

## Problem: Building a Network（问题：造一个网络）

假设你想造一个计算机网络，它要有可能扩展到全球规模，并支撑从 teleconferencing（远程会议）、video on demand（视频点播）、electronic commerce（电子商务）、distributed computing（分布式计算）到 digital libraries（数字图书馆）这样多样的应用。哪些现成技术可以作为底层的 building block（积木），你又该设计什么样的软件架构，把这些积木整合成一个有效的通信服务？回答这个问题就是本书的首要目标——描述可用的建筑材料，然后展示如何从地面起步（from the ground up）把它们搭成一个网络。

在我们理解如何设计一个计算机网络之前，首先应当就「计算机网络究竟是什么」达成一致。曾经，"network" 一词指的是把哑终端（dumb terminal）接到大型机（mainframe）上的那组串行线路。其他重要的网络还包括语音电话网（voice telephone network）与用于分发视频信号的有线电视网（cable TV network）。这些网络的共同点是：它们专门处理某一种数据（击键、语音或视频），并且通常连接专用设备（终端、听筒、电视机）。

那么，把计算机网络与这些其他类型的网络区分开的是什么？**计算机网络最重要的特征大概是它的 generality（通用性）。** 计算机网络主要由通用可编程硬件（general-purpose programmable hardware）构成，而且它们不为某个特定应用（例如打电话或传送电视信号）做优化。相反，它们能够承载许多不同类型的数据，并支持宽广且不断增长的应用范围。今天，计算机网络的职能已在很大程度上接管了以往由单一用途网络承担的功能。本章考察计算机网络的一些典型应用，并讨论「想支持这些应用的网络设计者」必须意识到的 requirements（需求）。

一旦我们理解了需求，接下来怎么做？幸运的是，我们并不是在造第一个网络。前人已经走过——其中最著名的是缔造 Internet 的那个研究者群体。我们将用 Internet 积累的丰富经验来指导自己的设计。这些经验凝结在一个 network architecture（网络架构）之中：它指明了可用的硬件与软件组件，并展示了如何把它们编排成一个完整的网络系统。

<span style="color:#1e8449;">**[译注]** 这一页给出的唯一**可判真假**的命题是「generality 是计算机网络区别于其他网络的特征」，而它支撑的是全书后续所有复用决策。把这条命题写成量：设网络需承载 $m$ 种应用、$n$ 种物理介质，则**专用网络**的方案数是「一种应用一张网」即 $m$ 张网、每张只认一种介质；**通用网络**的方案数是一条公共的 packet 接口，介质差异被压到最底层的 link layer。前者的维护成本随 $m$ 线性增长，后者近似常数——这就是 packet switching 经济性的来源，而不是「效率」这个空词。

**消失测试**：抽掉 generality，回到电话网那种按应用特化的设计，则新增任何一种数据类型（例如把视频从模拟信号换成 IP 分组的流）都要新建一张网并新建一套交换设备；对应到今天的实践，就是每次换应用都要重做物理层，视频会议不可能复用网页浏览的链路。

**落点**：§1.2.2 Scalable Connectivity（本卷 p014）与 §1.2.3 Cost-Effective Resource Sharing（本卷 p018）；CS144 的对应点是 Lecture 1「the hourglass / narrow waist」与 Lab0 里「所有应用都通过 socket 访问同一套网络」这一事实；跨学科坐标是 6.042J 的 counting（乘积 vs 线性增长）。</span>

<span style="color:#1e8449;">**[译注]** 本页末段那句「用 Internet 的经验指导设计」不是修辞，它决定了这本书的**来源等级**：全书几乎所有机制都以真实协议为样本（ARPANET 的 TCP、Ethernet 的 CSMA/CD、ATM 的 cell），因此读者可以随时回到 RFC 与源码核对。这与「先讲抽象模型再举例子」的教材相反——本书是先有实物、再抽原则。判断一处内容是否属于这类「实物抽样」有个简单测试：看它能不能被一个 `文件:行` 或一份 RFC 号定位。§5.2 的 TCP 能（RFC 9293 的 `tcp_input.c` 逻辑），而 §1.3 的 OSI 七层不能（它是 ISO 7498 的参考模型，没有任何 OSI 网络在运行——原书在 p028 自己承认了这一点）。

**消失测试**：如果把「以真实协议为样本」换成「以理想模型为样本」，第 2 章的 stop-and-wait 与 sliding window 就不会带上「sender 在重传时要不要翻倍超时」这类实现细节，读者写 Lab3 时会漏掉 `tick()` 里的 RTO doubling，导致 `tick()` 测试全线失败。

**落点**：§2.5 Reliable Transmission（书页 65）；CS144 Lab3 的 `tick()` 与非对称重传；一手来源是 RFC 9293（TCP）与 RFC 5681（congestion control），属 A 级。</span>

---

<!-- ===== p010 ===== -->
> 来源注：PDF 第 10 页 = 第 1 章（书页 6）；页脚 `6 Chapter 1. Foundation`。

除了理解网络如何建造，理解网络如何被**运营或管理**、以及网络应用如何被**开发**，也正变得越来越重要。我们几乎所有人现在都在家里、办公室、有时在车里拥有计算机网络，所以运营网络不再是少数专家的事。而随着智能手机的普及，这一代人中从事网络应用开发的人数比过去多得多。因此我们需要从多个视角来考虑网络：建造者（builders）、运营者（operators）、应用开发者（application developers）。

为了让读者走上「理解如何建造、运营与编程一个网络」的道路，本章做四件事。第一，它探究不同应用与不同人群对网络提出的 requirements。第二，它引入 network architecture 这一观念，为全书其余部分奠定基础。第三，它介绍实现计算机网络时的一些关键要素。最后，它指出用于评价计算机网络性能的关键度量（metrics）。

### 1.1 Applications（应用）

大多数人通过 Internet 的应用来认识它：World Wide Web、email、social media、streaming music or movies、videoconferencing、instant messaging、file-sharing，仅举数例。也就是说，我们是以网络**用户**的身份与 Internet 交互的。Internet 用户是「以某种方式与 Internet 交互」的最大一类人群，但还有若干其他重要的群体（constituencies）。

有一群人是**创造应用**的——近年来这群人大幅扩张，因为强大的编程平台与新设备（如智能手机）创造了快速开发应用并把它们带向大市场的新机会。

然后是**运营或管理网络**的人——这多半是幕后工作，但它至关重要，而且往往非常复杂。随着家庭网络的普及，越来越多的人也在（哪怕只是很小程度地）成为网络运营者。

最后是**设计与建造构成 Internet 的设备与协议**的人。这最后一个群体是诸如本书这类网络教材的传统目标读者，也将继续是我们的主要着眼点。不过全书我们也会考虑应用开发者与网络运营者的视角。

考虑这些视角将使我们更好地理解网络必须满足的多样需求。应用开发者如果理解底层技术如何工作、如何与应用交互，也能做出更好的应用。因此，在我们着手弄清如何建造网络之前，先更贴近地看看今天的网络所支持的应用类型。

<span style="color:#1e8449;">**[译注]** 这段把读者分成三类，其实际用途是**决定你读每一节时要问哪个问题**：建造型读「这个机制怎么实现」（→ 源码与 Lab），运营型读「这个机制失效时怎么定位」（→ 状态、日志、计数器），开发型读「这个机制向我承诺什么语义」（→ API 与契约）。三者对同一段文字问的问题不同，混着读就会两头落空。以 §1.2.4 的 reliable message delivery 为例：开发者只需要知道「要么全收到要么报错」，建造者要关心重传定时器怎么设，运营者要关心丢包率突增时如何区分「链路坏」与「拥塞」。

**可判定的判据**：一个机制是否属于你当前视角，看它是否出现在你的**可观测面**上。开发者能观测到的只有 socket 调用返回值与 errno；运营者能观测到接口计数器（`netstat -s` 里的 retransmit、`/proc/net/snmp` 里的 InErrs）；建造者能观测到的是寄存器和缓冲区。三者不重叠。

**消失测试**：如果不区分视角，CS144 的 Lab5 会变成灾难——网络接口（`NetworkInterface`）既面向「上层给 datagram」的开发者契约，又面向「下层收发 frame」的建造细节，混在一起写就会出现「把 frame 当 datagram 直接投递给上层」的经典错误，导致 ARP 应答被当成 IP 分组解析。

**落点**：§1.2.4 Support for Common Services（下一卷 p017 起）；CS144 Lab5/Lab6；CSAPP 第 11 章对「应用可见的 socket 语义 vs 内核实现」的划分是同一区分的另一处实例。</span>

<span style="color:#1e8449;">**[译注]** 「大多数人以用户身份与 Internet 交互」这句话之后紧跟的三段其实是**需求来源的分类学**，而不是社会观察。它对本书结构有一处硬约束：既然三类人提出的 requirement 会冲突，网络架构就必须提供一个**分派机制**，让不同需求落到不同的层。这正是 §1.3 要用 layering 的原因——layering 不是美学，而是冲突消解的手段：应用开发者要的是丰富的语义（可靠、有序、加密），建造者要的是每跳最快的转发（无尽头的重传、无状态），两者的需求被强制分到 transport layer 与 network layer 两层里各自满足。

**消失测试**：抽掉这层分派，把可靠性做进路由器（即让 network layer 保证可靠传输），则每一跳都要维护重传缓冲与序号，路由器状态从 $O(1)$ 变成 $O(\text{连接数})$，核心网的可扩展性直接崩掉——这正是 end-to-end argument 的反面，也是第 5 章把可靠性放在端主机的原因。

**落点**：§1.3.1 Layering and Protocols（本卷 p021 起）与 §1.3.5 Internet Architecture（下一卷 p029）；CS144 Lab6 的路由器只做 forwarding，不维护任何连接状态，可作为这条判据的实物证据。</span>

---

<!-- ===== p011 ===== -->
> 来源注：PDF 第 11 页 = 第 1 章（书页 7）；页脚 `1.1. Applications 7`。

#### 1.1.1 Classes of Applications（应用的类别）

World Wide Web 是那个把 Internet 从「主要由科学家与工程师使用的、有点冷门的工具」推成今日主流现象的应用。Web 本身已成为如此强大的平台，以至于许多人把它与 Internet 混为一谈；而把 Web 说成**单个**应用也多少有些牵强。

在其基本形态中，Web 呈现一个直觉上简单的界面。用户浏览满是文字与图形对象的页面，点击他们想进一步了解的对象，相应的新页面就出现了。大多数人也知道，在表象之下，页面上每个可选中对象都绑定着「下一个要查看的页面或对象」的一个标识符。这个标识符称为 **Uniform Resource Locator（URL，统一资源定位符）**，它提供了一种标识「你从浏览器里可能查看的所有对象」的办法。例如，

`http://www.cs.princeton.edu/llp/index.html`

是介绍本书作者之一的一个页面的 URL：字符串 `http` 表示应当使用 **Hypertext Transfer Protocol（HTTP，超文本传输协议）** 来下载该页面，`www.cs.princeton.edu` 是提供该页面的机器名，`/llp/index.html` 唯一标识 Larry 在该站点上的主页。

然而大多数 Web 用户并不知道的是：仅仅点击这样一个 URL，Internet 上可能就要交换**十几条 message**；如果网页很复杂、嵌入了大量对象，交换的条数还要多得多。这些 message 交换包括：

- 最多 **6 条 message** 用于把服务器名（`www.cs.princeton.edu`）翻译成它的 **Internet Protocol（IP）** 地址（`128.112.136.35`）；
- **3 条 message** 用于在你的浏览器与该服务器之间建立一条 **Transmission Control Protocol（TCP）** 连接；
- **4 条 message** 用于你的浏览器发出 HTTP "GET" 请求、服务器用被请求的页面响应（并且双方各自确认收到对方的那条 message）；
- **4 条 message** 用于拆除这条 TCP 连接。

当然，这还不包括 Internet 节点整天为彼此交换的成千上万条 message——那些 message 只是为了让彼此知道「自己存在、且准备好提供网页、把名字翻译成地址、把 message 向最终目的地转发」。

Internet 另一个广泛流行的应用类别是投递「流式（streaming）」音频与视频。video on demand 与 Internet radio 这类服务使用这项技术。虽然我们常常从一个网站开始发起一次 streaming session，但音频与视频的投递与「取一个纯文字加图片的简单网页」有一些重要差别。例如，你往往并不想先把整个视频文件下载完（这个过程可能要几分钟）再去看第一幕。streaming audio and video 意味着 message 从发送者到接收者的传递要更及时，接收者在到达时基本上就一边显示视频或播放音频。

<span style="color:#1e8449;">**[译注]** 这是全章信息密度最高的一段：它把「点一个链接」拆成 $6+3+4+4 = 17$ 条 message。**最值得挂上去的是 CS144 Lab0**：Lab0 要你写 `webget`，做的事正是「解析 URL → 建 TCP 连接 → 发 GET → 读响应 → 关闭」。教材这段给了这五步的报文账本，于是 lab 里那些看似莫名其妙的等待就有了归因——连接建立占 3 条（SYN / SYN-ACK / ACK），也就是 **1 个 RTT**；拆除占 4 条（FIN、ACK、FIN、ACK，可合并成 3 条），又是 1 个 RTT。所以一次不带缓存的 `webget`，首字节时间至少是

```math
T_{\text{first byte}} \;\ge\; T_{\text{DNS}} + 1\,\text{RTT} + T_{\text{server}},
```

DNS 那 6 条若缓存未命中，按迭代解析（root → TLD → 权威）三层各一次往返计算，就接近 $3$ 个 RTT。这条不等式解释了为什么 HTTP keep-alive 与 DNS 缓存是第一代 Web 性能优化里最划算的两项：前者省掉 1 个 RTT，后者直接删掉 $T_{\text{DNS}}$ 这一项。

**消失测试**：如果不做这份报文账本，Lab0 的 `webget` 超时后就无从判断是 DNS、握手还是服务器慢；更实际地，你在 Lab4 写 `TCPConnection` 时会以为「关闭」只是一次 FIN，而实际要覆盖 FIN_WAIT/TIME_WAIT 的多报文交换。

**落点**：§9.1 Traditional Applications（书页 416，HTTP 细节）与 §9.3.1 DNS（书页 449）；CS144 Lab0（`webget`）、Lab4（连接关闭）；一手来源是 RFC 9110（HTTP 语义）与 RFC 1034/1035（DNS）。</span>

<span style="color:#1e8449;">**[译注]** 这段的「6 条 DNS message」需要一个限定语，否则读者会把它当成典型值去核对。用 UDP 的 DNS 单次查询-应答只有 **2 条**；要到 6 条，得是缓存未命中且解析器逐层向上问（root、TLD、authoritative 各一次往返）。原文用 "up to six messages" 表达的是上界，不是均值——这个区别在测量上很要命：如果你在 Lab0 里观测 `webget` 的耗时并按 6 条报文的固定成本建模，缓存命中时（2 条）你会把 2 个 RTT 的差额错误归给「服务器慢」。

**可算的方向性判断**：局域网内 RTT ≈ 0.2 ms 时，6 条 vs 2 条 DNS 的差额是 $2\,\text{RTT} \approx 0.4$ ms，可忽略；跨洋 RTT ≈ 150 ms 时同一差额是 $300$ ms，比一次 TCP 握手还贵一倍——这正是 DNS 缓存（TTL 通常 300–86400 s）在广域上比在局域上重要得多的原因，也是原文脚注 1 说「300 ms 是电话通话可容忍的往返延迟上界」的同一量级。

**落点**：§9.3 Infrastructure Applications（书页 449）；RFC 1035 的 recursion 描述；本页脚注 1（human factors：300 ms 上界、100 ms 很好）。</span>

> <span style="color:#943126;">**[note]**</span> 脚注 1：并非真的「尽可能快」……人因（human factors）研究表明，在人们开始抱怨之前，电话通话中可容忍的往返延迟上限约为 300 ms 是合理的；而 100 ms 的延迟听起来已经很好。

---

<!-- ===== p012 ===== -->
> 来源注：PDF 第 12 页 = 第 1 章（书页 8），本页整页为插图；页脚 `8 Chapter 1. Foundation`。

[Figure 1.1：一张**多媒体应用**的界面截图，演示「视频会议（videoconferencing）作为一类网络应用长什么样」——图中有参与者的视频画面与旁侧的信息面板。]

> **无法复原说明**：本页在源 PDF 中只有图题一行文字，图像本身未进入文字层。截图内的具体控件、参与者姓名、分辨率等细节无法从文字层复原，故不描述。正文可核的部分只有一句：这是「一个此类系统的例子」（"An example of one such system is shown in Figure 1.1"），指 1990 年代初以来在 Internet 上运行的视频会议工具中的一款商业产品。

<span style="color:#1e8449;">**[译注]** 图 1.1 被单独排成整页，这个版式选择本身是本页唯一可讨论的判断。插图独立成页的代价是正文被打断一页，收益是图能以足够尺寸呈现「多媒体界面」这种细节不可缩小的内容。对读者的实际影响是：**别把这张图当作技术图去读**——它没有拓扑、没有箭头、没有时间轴，唯一的信息是「视频会议这类应用在界面上是多个视频窗口 + 元信息」。真正承载机制的图是 p016 的 Figure 1.3（switched network）与 p017 的 Figure 1.4（interconnection of networks），那两张才有节点与链路。

**消失测试**：如果删掉图 1.1，正文论证不受影响（它只是举例）；但如果删掉 p016/p017 的两张拓扑图，§1.2.2 关于「直接连通 vs 间接连通」的论证就失去载体，读者会误以为 switch 与 router 的区别只是名字不同。

**落点**：§1.1.1（本页所属节）；§1.2.2 Scalable Connectivity（本卷 p014–p017）；CS144 里与之对应的「应用层长什么样」的位置是 Lab0 的 `webget`——同样是应用视角，同样不涉及协议内部。</span>

---

<!-- ===== p013 ===== -->
> 来源注：PDF 第 13 页 = 第 1 章（书页 9）；页脚 `1.2. Requirements 9`。

运行在 Internet 上的视频会议工具从 1990 年代初就已存在，但在最近几年获得了广泛使用，市场上已有若干商业产品。这类系统的一个例子如图 1.1 所示。正如下载一个网页所涉及的比表面看到的多一点，视频应用也是如此。例如，把视频内容装进一个相对低带宽的网络，或者确保视频与音频保持同步并按时到达以获得良好的用户体验，都是网络与协议设计者必须操心的问题。我们将在本书后面考察这些问题以及许多其他与多媒体应用相关的问题。

虽然只是两个例子，从 Web 下载页面与参加一次视频会议已经展示了可以构建在 Internet 之上的应用的多样性，并暗示了 Internet 设计的复杂性。在本书后面，我们将给出一套更完整的应用类型分类学（taxonomy），以帮助我们讨论关键设计决策，因为我们要建造、运营并使用能支撑如此广泛应用范围的网络。本书结尾会重新回到这两个具体应用，以及其他几个能说明今日 Internet 上可能性之广的应用。

眼下，对几个典型应用的这番快速浏览已经足以让我们开始考察那些必须解决的问题——如果我们要建造一个能支撑如此应用多样性的网络。

### 1.2 Requirements（需求）

我们为自己设定了一个雄心勃勃的目标：理解如何从地面起步（from the ground up）建造一个计算机网络。实现这个目标的方法将是：从第一性原理出发，然后提出我们在真正建造一个网络时会自然提出的那类问题。每一步我们都会用今天的协议来说明可供选择的各种设计取舍，但我们不会把既有的产物当作教条（gospel）。相反，我们会追问（并回答）「网络为什么被设计成现在这个样子」。虽然「只理解今天是怎么做的」很诱人，但重要的是认识到底层的概念，因为随着技术演进与新应用的发明，网络在不断变化。我们的经验是：一旦你理解了基本思想，之后遇到的任何新协议都会相对容易消化。

<span style="color:#1e8449;">**[译注]** 「不会把既有产物当作教条」这句是本书的方法论承诺，它有一个可操作的推论：**每个协议细节都应能回答「如果反过来会怎样」**。以本段所在的 §1.2 为例，后面要讲的 requirements（可扩展连通、成本有效的资源共享、公共服务的支持、可管理性）每一条都对应一个反例：抽掉 scalability 就得到「只为几台机器服务的局域网」；抽掉 cost-effective sharing 就得到 circuit switching 的电话网；抽掉 manageability 就得到「一旦配好就谁也不敢动」的电信机房文化。

**可判定的判据**：判断你是否真的理解了某个机制，看你能不能说出它的**失效版本**长什么样、在什么条件下更优。例如 stop-and-wait 的失效版本是 sliding window；而 sliding window 在「带宽时延积远小于一个 packet」的链路上并不优于 stop-and-wait——此时窗口大于 1 只增加缓冲区而不增加吞吐，因为 $B\cdot\text{RTT} \ll \text{MSS}$。

**消失测试**：如果只记住「今天是这么做的」，遇到 QUIC 这种把可靠传输搬进用户态的协议就无法定位它改变了哪条假设——而本书 p006 的变更清单恰好把它列为 §5.2 的新增内容。

**落点**：§1.2.1–§1.2.5（本卷 p013–下一卷 p021）；CS144 的对应读法是「每讲一个机制，就在 lab 里找到它对应的那几行测试」；对照 6.042J 的「反例优先」训练。</span>

<span style="color:#1e8449;">**[译注]** 这里出现了本书的第一个「尺度声明」：视频会议工具「从 1990 年代初就已存在」，而广泛使用发生在最近几年——这句话在 2019 年的语境里指的是 cloud 化之后的 WebRTC 路线。它提示一个读法：**本书对每项技术的年代标注是刻意的**，因为第 6 版的自述目标之一就是把「原始研究」与「当代 cloud 应用」连起来（见 p005 第 2 条）。所以读者遇到年代信息时，应当把它当作「这条机制在哪个约束下被发明」的坐标，而不是历史花絮。

**可算的例子**：视频会议对延迟的要求比 streaming 严得多（本段之后 §1.1.1 续页给出量化：300 ms 往返是可容忍上界，100 ms 很好）。设单跳排队延迟为 $q$、编码延迟为 $e$、网络往返为 $R$，则端到端可感知延迟近似 $e + q + R/2$；要让总延迟低于 150 ms，在 $R = 60$ ms 的城际链路上留给编解码与排队的预算只有约 120 ms——这解释了为什么实时视频必须用能在 100 ms 内编完一帧的编码器，而不能用为压缩率优化的离线编码。

**落点**：§1.1.1（本页所属节）与 §9.2 Multimedia Applications（书页 437）；§5.4 Transport for Real-Time (RTP)（书页 276）；一手坐标是 RFC 3550（RTP）。</span>

---

<!-- ===== p014 ===== -->
> 来源注：PDF 第 14 页 = 第 1 章（书页 10）；页脚 `10 Chapter 1. Foundation`。

#### 1.2.1 Stakeholders（利益相关者）

如上所述，网络的学习者可以采取若干视角。当我们写本书第 1 版时，大多数人口根本没有 Internet 接入，而有接入的人在上班时、在大学里、或在家通过 dial-up modem（拨号调制解调器）获得它。流行应用的数量一只手就能数完。因此，像当时大多数书一样，我们把重点放在「设计网络设备与协议的人」的视角上。我们继续以此为焦点，并希望读完本书后你知道如何设计未来的网络设备与协议。

不过，我们也想覆盖另外两类利益相关者的视角：开发网络应用的人，以及管理或运营网络的人。让我们看看这三类利益相关者会怎样列出他们对网络的要求：

- 一位**应用程序员**会列出他或她的应用所需的服务：例如，保证应用发出的每条 message 都在一定时间内无错地送达，或者当用户在移动时能够平滑地在不同网络连接之间切换。
- 一位**网络运营者**会列出一个易于管理与运营的系统的特征：例如，其中的故障可以被轻易隔离，新设备可以被加入网络并正确配置，用量容易被计量核算。
- 一位**网络设计者**会列出一个成本有效的设计所具有的性质：例如，网络资源被高效利用，并被公平地分配给不同用户。性能问题也很可能是重要的。

本节试图把不同利益相关者的要求提炼成一份对「驱动网络设计的主要考虑」的高层导引，并在此过程中指出本书其余部分所应对的挑战。

<span style="color:#1e8449;">**[译注]** 这三条要求的排列顺序值得注意：**应用程序员要「保证」，运营者要「可隔离」，设计者要「高效 + 公平」**——三者两两冲突。最典型的冲突是「高效 vs 公平」与「保证 vs 高效」：如果给每个应用预留带宽以保证其时延（这是保证），链路在空闲时就被浪费（不高效）；如果完全按需抢占（高效），就没有任何应用能拿到保证。解决这对冲突的工程手段是**统计复用 + 概率性保证**：不是保证「$100$ Mbps 一定给你」，而是保证「$99.9\%$ 的包延迟低于 $t$」。这条从「确定性保证」到「统计保证」的降级，是本页三条要求能被同时满足的唯一途径。

**可判定的判据**：看到一个 QoS 承诺时，先问它是确定性的还是统计性的。DiffServ 的 per-hop behavior 是统计性的（无绝对上界），IntServ/RSVP 试图做确定性的（有每流预留状态，代价是路由器状态 $O(\text{流数})$）。前者可扩展、后者不可——这就是为什么 Internet 最终选了 DiffServ 路线。

**消失测试**：抽掉「统计保证」这个中间态，网络只能在「完全无保证（best effort）」与「按流预留（不可扩展）」两端选；前者无法支撑实时语音，后者无法支撑十亿级用户。

**落点**：§6.5 Quality of Service（书页 323）；§1.2.3（本卷 p018 起的 statistical multiplexing）；CS144 的对应观察点是 Lab3 里没有任何带宽预留，发送速率完全由接收窗口与丢包反馈决定。</span>

<span style="color:#1e8449;">**[译注]** 「用户移动时平滑切换连接」这一条在 1996 年（第 1 版）是幻想，在 2019 年（第 6 版）是 §4.5 Routing Among Mobile Devices 的主题，而在 CS144 里完全没有对应 lab——这不是教材的问题，而是 lab 的边界：**移动性属于路由层，而 CS144 的 lab 只做到「端主机 + 一台固定路由器」。** 读者应该明确这条边界，否则会试图在 `TCPConnection` 里解决 IP 变化的问题。

**消失测试**：如果不把「移动性」明确划归 §4.5，你会误以为 TCP 连接能扛住 IP 地址变化；实际上 TCP 的四元组（源 IP、源端口、目的 IP、目的端口）一旦有一项变了，内核就认为这是另一条连接，旧连接只能靠重传超时后报错。QUIC 用 connection ID 解耦了「连接标识」与「四元组」，才可以做到换网不断连——这正是 §5.2 新增 QUIC 小节的实质。

**落点**：§4.5（书页 221）；§5.2 QUIC 小节；一手坐标是 RFC 9000（QUIC transport）里的 Connection ID 机制。</span>

---

<!-- ===== p015 ===== -->
> 来源注：PDF 第 15 页 = 第 1 章（书页 11）；页脚 `1.2. Requirements 11`。本页上半为 Figure 1.2，下半为 §1.2.2 正文起首。

[Figure 1.2：**直接链路（direct link）的两种形态**——(a) point-to-point（点对点），一条物理介质只连接两个节点；(b) multiple-access（多址），一条物理介质被两个以上节点共享。图只呈现介质与节点的连接关系，不涉及时序。]

#### 1.2.2 Scalable Connectivity（可扩展的连通性）

从最明显的说起：一个网络必须在一组计算机之间提供 connectivity（连通性）。有时建造一个只连接少数几台选定机器的有限网络就够了。事实上，出于隐私与安全的考虑，许多私有（公司）网络的明确目标就是**限制**被连接的机器集合。相反，另一些网络（Internet 是首要例子）被设计成能够增长，从而有可能连接世界上所有计算机。一个被设计为可支撑任意大规模增长的系统，被称为 **scale（可扩展）**。以 Internet 为模型，本书处理 scalability（可扩展性）这一挑战。

要更完整地理解连通性的要求，我们需要更贴近地看看网络中的计算机是如何连接的。连通性发生在许多不同层次上。在最底层，一个网络可以由两台或多台计算机通过某种物理介质（例如同轴电缆 coaxial cable 或光纤 optical fiber）**直接**连接而成。我们把这样的物理介质称为 **link（链路）**，并常常把它连接的计算机称为 **node（节点）**。（有时一个节点是某种更专门的硬件而不是计算机，但在本讨论中我们忽略这个区分。）如图 1.2 所示，物理链路有时被限制为一对节点（这样的链路称为 **point-to-point**，点对点），而在其他情况下，两个以上的节点可以共享一条物理链路（这样的链路称为 **multiple-access**，多址）。无线链路——例如蜂窝网与 Wi-Fi 网提供的那些——是一类重要的 multiple-access 链路。multiple-access 链路在规模上总是受限的，既包括它能覆盖的地理距离，也包括它能连接的节点数。正因如此，它们常常实现所谓的 **last mile（最后一英里）**，把终端用户连到网络的其余部分。

<span style="color:#1e8449;">**[译注]** point-to-point 与 multiple-access 的区别不是描述性的，它决定了**谁来负责「不要同时说话」**。point-to-point 链路上只有一个发送者，冲突不存在，因此链路层可以极简（例如 SONET 的 framing）；multiple-access 链路上有 $n$ 个潜在发送者，必须引入介质访问控制（medium access control, MAC）算法，而 MAC 算法的吞吐上限有一个硬约束：在共享介质上，无论算法多聪明，**有效吞吐不超过链路速率**，且随冲突概率上升而下降。这一点正好接上第 2.6 节的多址协议（ALOHA、CSMA/CD）。

**可算的判据**：纯 ALOHA 的最大吞吐是 $S = G e^{-2G}$，在 $G = 0.5$ 处取极大 $S_{\max} = 1/(2e) \approx 0.184$；slotted ALOHA 是 $S_{\max} = 1/e \approx 0.368$。也就是说明明有 10 Mbps 的介质，纯 ALOHA 只能跑出 1.84 Mbps——这个数字是「multiple-access 必须付的税」，point-to-point 不付。

**消失测试**：如果不区分这两类链路，CS144 Lab5 里 `NetworkInterface` 的「发送」会被理解成无竞争的写操作；实际上以太网在共享介质时代要靠 CSMA/CD 退避重发，现代交换式以太网虽然点对点化了，但仍然保留了帧格式与最小帧长（64 B）这些当年为冲突检测留的痕迹。

**落点**：§2.6 Multi-Access Networks（书页 77）；§2.8 Access Networks（书页 95，PON 与 5G RAN 就是 last mile 的当代形态）；CS144 Lab5 的 Ethernet frame 与 MTU；一手坐标是 IEEE 802.3 与 ALOHA 的原始论文（Abramson, 1970）。</span>

<span style="color:#1e8449;">**[译注]** 本页最后一句「multiple-access 总是受限」给出的两个限制维度（地理距离与节点数）值得定量化，因为它是 last mile 技术选择的全部理由。设共享介质的总速率为 $B$、节点数为 $n$、每节点平均需求为 $r$，则必须满足 $n r \le \eta B$，其中 $\eta \le 1$ 是 MAC 效率（ALOHA 的 $\eta \approx 0.18$，CSMA/CD 在中等负载下 $\eta \approx 0.6\text{–}0.9$）。这条不等式解释了三件事：为什么同轴电缆时代的以太网必须限制单段长度（冲突检测窗口 = $2 \times$ 传播时延，长度越长窗口越大、效率越低）；为什么 PON 用分光器把一条光纤分给 32 户而不是 3200 户；为什么 5G 的 RAN 要把小区做小（节点数少、每节点可用速率高）。

**消失测试**：抽掉这个约束，你会认为「把一条链路分给越多用户越经济」；实际上超出 $\eta B$ 之后每增加一个用户，所有人的平均吞吐都下降，而不是简单摊薄。

**落点**：§2.8（书页 95）的 PON 与 5G RAN 描述；§2.6（书页 77）的 MAC 效率讨论；跨学科坐标是 6.042J 的 pigeonhole / 平均数论证。</span>

---

<!-- ===== p016 ===== -->
> 来源注：PDF 第 16 页 = 第 1 章（书页 12），本页整页为插图；页脚 `12 Chapter 1. Foundation`。

[Figure 1.3：**switched network（交换式网络）**——一组节点，每个节点连到一条或多条 point-to-point link；连接到至少两条链路的节点运行转发软件，把从一条链路收到的数据从另一条链路发出。图中的云（cloud）区分了两类节点：云**内部**实现网络的节点（switch，交换机，主要功能是存储并转发 packet）与云**外部**使用网络的节点（host，主机，支撑用户并运行应用程序）。]

> **无法复原说明**：本页文字层只有图题 `Figure 1.3.: Switched network.`。节点的具体个数、云的轮廓、链路的方向箭头均未进入文字层，故不描述。**可核的部分**来自正文（书页 11，本卷 p015）：云内的节点是 switch、云外的节点是 host，且云本身是「任意一种网络」的占位符。若需要精确的拓扑，应以源 PDF 的图像层为准。

<span style="color:#1e8449;">**[译注]** 这张图与它的图题之间有一个容易被忽略的问题：图题只写 "Switched network"，但正文给它的载荷远多于「一个交换网络」——它同时定义了 **switch 与 host 的界线**，以及 **cloud 作为占位符**的用法。这个界线是全书后面所有讨论的坐标系：**host 是通信的端点，switch 是通信的中转**，而「谁必须实现哪一层」由这条界线决定（见下一卷 p029 的「下三层在所有节点上实现，transport 及以上只在端主机上实现」）。

**可判定的判据**：拿到任何一个网络设备，问它是否终结了某个连接的语义。switch 不终结（它不维护端到端状态），host 终结（它维护 socket 与序号空间）。这条判据比「看是否转发」更可靠，因为 NAT 设备也转发但确实终结了连接语义。

**消失测试**：抽掉这条界线，CS144 Lab6 的路由器就会被迫维护每条 TCP 连接的重传状态，路由器内存从 $O(1)$（转发表 + 队列）变成 $O(\text{并发连接数})$，核心网无法承载十亿级连接。

**落点**：§3.1 Switching Basics（书页 104）与 §3.3 Internet (IP)（书页 125）；CS144 Lab5（`NetworkInterface` 是 host 侧的边界）、Lab6（`Router` 是 switch 侧的边界）；cloud 作为占位符的用法见图 1.4（本卷 p017）。</span>

---

<!-- ===== p017 ===== -->
> 来源注：PDF 第 17 页 = 第 1 章（书页 13），本页整页为插图；页脚 `1.2. Requirements 13`。

[Figure 1.4：**interconnection of networks（网络的互连）**——一组相互独立的网络（各自画成一朵云）被互连成一个 internetwork（互联网，简称 internet）。连接两朵或多朵云的节点是 router 或 gateway，它从一朵云接收 message 并向另一朵云转发，作用与 switch 类似。]

> **无法复原说明**：本页文字层只有图题 `Figure 1.4.: Interconnection of networks.`。云的个数、router 的位置与链路箭头无法从文字层复原，故只给正文可核的部分（书页 11–12）：连接两朵以上云的节点称为 router 或 gateway；一个 internet 本身可被看作另一种网络，因此 internet 可以由一组 internet 递归构成。

<span style="color:#1e8449;">**[译注]** 图 1.3 与图 1.4 是**同一个构造的两次应用**，而两者的差别正是全书最关键的抽象：图 1.3 把「节点 + 链路」折叠成一朵云，图 1.4 把「云」当成新的「节点」再连起来。这个折叠之所以合法，是因为云对外只承诺一件事——**把交给它的 message 从入口送到指定的出口**；内部有多少跳、用什么技术，对外不可见。这就是分层里的「接口与实现分离」在图形上的第一次出现。

**可算的判据**：递归折叠允许规模指数增长。若每朵云最多连 $k$ 个子网、每个子网最多 $m$ 台 host，则两层嵌套可寻址 $k \cdot m$ 台；若允许 $d$ 层嵌套，则可达 $k^{d}$ 量级——这正是 IP 地址分层（前缀聚合）的算术基础。反过来说，如果地址是扁平的（每台 host 一个随机 32 位数），转发就退化成线性查表 $O(n)$，因为前缀之间没有可聚合的结构。

**消失测试**：抽掉递归可组合性，internetwork 只能在「一张扁平的大网」与「互不相通的孤岛」之间选；前者地址空间不可扩，后者就是 1980 年代各厂商私有网络协议的处境（SNA、DECnet 各自为政），也正是「Internet 的根本创新是互连异构网络」这句话的所指。

**落点**：§3.3 Internet (IP)（书页 125）与 §4.1 Global Internet（书页 180，前缀聚合与 CIDR）；CS144 Lab6 的转发表用的就是**最长前缀匹配**；跨学科坐标是 6.042J 的树与 trie（前缀结构的查找复杂度 $O(\log n)$ 到 $O(w)$，$w$ 为前缀位宽）。</span>

---

<!-- ===== p018 ===== -->
> 来源注：PDF 第 18 页 = 第 1 章（书页 14）；页脚 `14 Chapter 1. Foundation`。

如果计算机网络被限制在「所有节点都通过一条公共物理介质直接互连」的情形，那么要么网络能连接的计算机数量非常有限，要么从每个节点背后引出的线缆数量很快就会变得既难以管理又非常昂贵。幸运的是，两个节点之间的连通性并不必然意味着它们之间有一条**直接的**物理连接——间接连通（indirect connectivity）可以在一组协作的节点之间实现。考虑下面两个「一组计算机如何被间接连接」的例子。

图 1.3 展示了一组节点，每个节点连到一条或多条 point-to-point link 上。那些连接到至少两条链路的节点运行软件，把从一条链路收到的数据从另一条链路发出。若以系统化的方式组织起来，这些转发节点就构成一个 **switched network（交换式网络）**。交换式网络有许多类型，其中最常见的两种是 **circuit switched（电路交换）** 与 **packet switched（分组交换）**。前者最著名地用于电话系统，后者被绝大多数计算机网络采用，也是本书的焦点。（不过，circuit switching 在光网络领域正有某种程度的回归，随着对网络容量需求的持续增长，这一点变得重要。）

packet-switched 网络的重要特征是：这类网络中的节点彼此发送**离散的数据块**。把这些数据块想象成对应某份应用数据——一个文件、一封 email、一张图像。我们把每个数据块称为 **packet（分组）** 或 **message（消息）**，眼下我们互换使用这两个术语。†

> † **术语不一致**：此处明说 packet 与 message 互换使用，但同一节稍后（书页 16，本卷 p020）为解释 statistical multiplexing 又必须把两者区分开——"This limited-size block of data is typically referred to as a packet, to distinguish it from the arbitrarily large message that an application program might want to transmit."。这是作者自己意识到并修正的用法：message 指应用层任意长度的整体，packet 指网络上传输的有界块。阅读时应以第二处定义为准，本处属宽泛用法。

packet-switched 网络通常采用一种称为 **store-and-forward（存储转发）** 的策略。顾名思义，store-and-forward 网络中的每个节点先从某条链路接收一个完整的 packet，把该 packet 存进内部存储器，然后把完整的 packet 转发给下一个节点。相反，circuit-switched 网络先沿一系列链路建立一条专用电路（dedicated circuit），然后允许源节点在这条电路上向目的节点发送一串比特。在计算机网络中使用 packet switching 而非 circuit switching 的主要理由是 efficiency（效率），下一个小节讨论。

图 1.3 中的云把**内部**实现网络的节点（它们通常称为 **switch**，主要功能是存储并转发 packet）与云**外部**使用网络的节点（它们传统上称为 **host**，支撑用户并运行应用程序）区分开来。还要注意，云是计算机网络最重要的图标之一。一般来说，我们用云来表示任意类型的网络，无论它是一条 point-to-point link、一条 multiple-access link，还是一个 switched network。因此，每当你在图中看到一朵云，都可以把它看作本书所涵盖的任何一种网络技术的占位符。†

> † 脚注 1（原文）："The use of clouds to represent networks predates the term cloud computing by at least a couple of decades, but there **an** increasingly rich connection between these two usages, which we explore in the Perspective discussion at the end of each chapter." 此处 "there an" 缺谓语，应为 "there **is** an"。

<span style="color:#1e8449;">**[译注]** store-and-forward 与 circuit switching 的对比是本章第一个**可以算出来**的取舍，而算它的关键量是「链路空闲时间占比」。设一条链路上有 $n$ 条流、每条流活跃概率为 $p$（活跃时以速率 $R$ 发送），circuit switching 为每条流预留固定时隙，链路容量必须按 $nR$ 配置，实际利用率期望为 $p$；statistical multiplexing（packet switching）按需分配，只需按 $npR$ 加余量配置，同样负载下所需容量约为前者的 $p$ 倍。$p = 0.1$ 时，packet switching 只需约 1/10 的容量就能承载同样的流量——这就是原文所说的 "efficiency"。

**消失测试**：如果不做 store-and-forward（即让节点边收边转发，cut-through），端到端延迟能省掉每个节点的一个 packet 时间，但节点必须能处理「头部到了、尾部还没到」的中间状态，且无法在转发前做完整性检查；在误码率 $10^{-6}$ 的铜缆时代，这会让坏包在网络里跑完全程才被丢弃，浪费的带宽大于省下的延迟。现代数据中心（误码率极低）重新启用 cut-through 交换，正是因为那个前提变了。

**落点**：§1.2.3 Cost-Effective Resource Sharing（本卷 p015 起）；§3.1 Switching Basics（书页 104）对 cut-through 与 store-and-forward 的正式对比；CS144 Lab6 的路由器实现的是 store-and-forward（收完整 datagram、查表、入队）。</span>

<span style="color:#1e8449;">**[译注]** 本页脚注那句「云的用法比 cloud computing 这个词早至少二十年」看似闲笔，其实预告了第 6 版的一条主线：**同一个词在两层含义上同时被使用**。网络拓扑里的云是「不可见的内部」，cloud computing 里的云是「不可见的实现」——两者的共同点是「把实现细节藏在一个边界之后」，即封装。这个巧合被作者刻意保留，是因为它正好可以解释为什么 §1.3 的 layering 与今天的数据中心抽象是同一种思维。

**可判定的判据**：判断一处「云」是哪种用法，看**边界内是否存在可观测的状态**。拓扑里的云：内部节点有转发表与队列，但对端点不可见。cloud computing 的云：内部有虚拟机和存储，但对应用只暴露 API。两者的判据一致——边界之外只承诺语义，不承诺实现。

**消失测试**：抽掉「云 = 占位符」这个约定的后果很具体：读者会在第 2、3、4 章的图里把每朵云都当成一个具体设备去追问内部结构，从而错失「图在不同抽象层上可复用」这一读图技巧。图 1.4（p017）就是把图 1.3 的云当成节点再用一次的直接证据。

**落点**：§3.3 与 §4.1 的图；每章末的 Perspective（本卷 p005 提到其设立理由）；跨学科坐标是 CSAPP 里「抽象层」的同一手法（ISA 对编译器隐藏微架构）。</span>

<span style="color:#1e8449;">**[译注]** 本段还有一句容易被读过去的话：「circuit switching 在光网络领域正有某种程度的回归」。它指的是全光交换（optical circuit switching）在数据中心骨干里的使用——因为当链路速率到 100 Gbps 以上时，电的 packet switching 的功耗与缓存成本变得不可接受，而波长级的电路交换只需配置光开关。这条「回归」对本课程的实际意义是：**packet switching 不是无条件更优**，它的优势建立在「流量突发性强、平均速率低」这个统计前提上；当流量变成大块、持续的（如数据中心间的批量同步），电路交换重新划算。

**可算的判据**：判据是带宽时延积 $B \cdot \text{RTT}$ 与「平均突发长度」之比。若单条流持续以接近线速发送（突发长度 $\gg B\cdot\text{RTT}$），预留电路的浪费率趋近 0，packet switching 的统计增益消失，反而多付了每包的头部与排队开销。

**消失测试**：如果坚持「packet 一定优于 circuit」，就无法解释为什么 §2.8 的 PON 在下行用广播、上行用 TDMA（一种电路式分配），也无法解释为什么 5G 的 RAN 要做调度式分配而非纯竞争。

**落点**：§2.8 Access Networks（书页 95）；§3.1 的 switching 分类；一手坐标是数据中心光交换的公开文献（如 Google 的 Jupiter 网络架构，A 级）。</span>

---

<!-- ===== p019 ===== -->
> 来源注：PDF 第 19 页 = 第 1 章（书页 15）；页脚 `1.2. Requirements 15`。

一组计算机可以被间接连接的第二种方式如图 1.4 所示。在这种情况下，一组相互独立的网络（云）被互连起来，形成一个 **internetwork**，简称 **internet**。我们采用 Internet 的约定：把一般性的「网络的网络」写作小写 i 的 internet，而我们每天使用的 TCP/IP Internet 写作大写 I 的 Internet。连接到两个或更多网络的节点通常称为 **router（路由器）** 或 **gateway（网关）**，它扮演的角色与 switch 大致相同——它把 message 从一个网络转发到另一个网络。注意，一个 internet 本身可以被看作另一种网络，这意味着 internet 可以由一组 internet 构成。因此，我们可以通过把云互连成更大的云，递归地建造任意大的网络。可以合理地论证，「互连彼此差异极大的网络」这一想法正是 Internet 的根本创新，而 Internet 成功扩展到全球规模、数十亿节点，是早期 Internet 架构师若干非常好的设计决策的结果，这些我们稍后讨论。

仅仅因为一组 host 彼此直接或间接连通，并不意味着我们已经成功提供了 host-to-host connectivity。最终的要求是：每个节点必须能够说明它想与网络上的**哪一个**其他节点通信。这是通过给每个节点分配一个 **address（地址）** 来实现的。地址是一个标识节点的字节串；也就是说，网络可以用节点的地址把它与连到该网络的其他节点区分开。当源节点想让网络把一条 message 投递给某个目的节点时，它就指明目的节点的地址。如果发送与接收节点不是直接相连的，网络的 switch 与 router 就用这个地址来决定如何把 message 向目的地方向转发。基于目的地址、系统地决定如何把 message 向目的节点转发的这一过程，称为 **routing（路由）**。

这段关于 addressing 与 routing 的简要介绍，预设了源节点想把一条 message 发给**单个**目的节点（**unicast**，单播）。虽然这是最常见的场景，源节点也有可能想向网络上**所有**节点广播（**broadcast**）一条 message。或者，源节点可能想发给其他节点中的某个子集而非全部，这种情形称为 **multicast（多播）**。因此，除了节点专用的地址，网络的另一项要求是它支持 multicast 与 broadcast 地址。

<span style="color:#1f6feb;">**[Key Takeaway]**</span> 从这段讨论中要带走的主要想法是：我们可以**递归地**定义一个网络——它由两个或多个被物理链路连接的节点构成，或由两个或多个被一个节点连接的网络构成。换言之，网络可以由网络的嵌套来构造，而在最底层，网络由某种物理介质实现。提供网络连通性的关键挑战之一，是为网络上可达的每个节点（无论逻辑的或物理的）定义地址，并使用这些地址把 message 向适当的目的节点转发。[Next]

<span style="color:#1e8449;">**[译注]** 本页把 routing 定义为「基于目的地址、系统地决定如何转发」——注意动词是 **decide**，不是 **do**。这个区分是第 3 章 §3.4 与 §3.5 的分水岭，也是 CS144 Lab6 最容易踩空的坑：**routing 是生成转发表的过程（分布式算法、可能很慢），forwarding 是每个 packet 到达时按表查一次的动作（必须在纳秒级完成）**。Lab6 只测 forwarding——路由表是给定的静态文件。如果你在 `Router::route` 里做任何与「学习拓扑」相关的事，就已经越界了。

**可算的判据**：forwarding 的时间预算由线速决定。10 Gbps 链路上一个最小帧（64 B）到达间隔是

```math
t_{\min} = \frac{64 \times 8\ \text{bit}}{10^{10}\ \text{bit/s}} = 51.2\ \text{ns},
```

所以查表必须在几十纳秒内完成——这解释了为什么转发表要用硬件 trie 或 TCAM 实现，而不是软件里的哈希表遍历。相对地，routing 的一次收敛可以在秒级（BGP 的收敛时间常以十秒计）。

**落点**：§3.4 Routing（书页 151）与 §3.5 Implementation（书页 169）；CS144 Lab6 `Router` 与 `route_table` 静态文件；跨学科坐标是 6.042J 里的图与最短路（Dijkstra 的复杂度 $O(E + V\log V)$），以及 CSAPP 第 6 章存储层次对「查表要在 cache 里」的支撑。</span>

<span style="color:#1e8449;">**[译注]** Key Takeaway 那句「递归定义」有一个直接的算术后果，值得在这里算一次，因为它是全章唯一能自证的可扩展性论证：设第 $0$ 层是「节点 + 链路」，每往上叠一层，把 $k$ 个下层实体用一个节点连起来。则第 $d$ 层的实体数是 $k^{d}$，跨越任意两个端点最多经过 $2d$ 个中间节点（最坏路径长度线性于层数，而非规模）。也就是说，从 $10^{3}$ 台主机扩到 $10^{9}$ 台主机，路径长度只从 $2 \times 3$ 增加到 $2 \times 5$，而扁平拓扑（全互连或单层交换）需要 $O(N)$ 条链路。

**消失测试**：抽掉递归嵌套，只留「所有节点直接互连」，$N$ 台主机需要 $N(N-1)/2$ 条链路——$N = 1000$ 时约 50 万条，机架与布线成本直接让方案不可行。这正是本页开头那句话（「要么数量有限，要么线缆多得难以管理」）的定量版本。

**落点**：§1.2.2（本卷 p014–p015）与 §4.1 Global Internet（书页 180）；跨学科坐标是 6.042J 的树与深度（$k$ 叉树的深度是 $\log_{k} N$）。</span>

---

<!-- ===== p020 ===== -->
> 来源注：PDF 第 20 页 = 第 1 章（书页 16）；页脚 `16 Chapter 1. Foundation`。

#### 1.2.3 Cost-Effective Resource Sharing（成本有效的资源共享）

如上所述，本书聚焦 packet-switched 网络。本节解释计算机网络的关键要求——**efficiency（效率）**——它如何促使我们把 packet switching 作为策略选择。

给定一组由网络嵌套间接连接的节点，任意一对 host 都有可能跨越一串链路与节点互相发送 message。当然，我们想做的远不止支持一对通信中的 host——我们想为**所有** host 对提供交换 message 的能力。那么问题就是：所有想通信的 host 如何共享这个网络，特别是当它们想**同时**使用它的时候？而且，好像这个问题还不够难似的：当若干 host 都想同时使用同一条链路时，它们如何共享这条链路？

要理解 host 如何共享一个网络，我们需要引入一个基本概念：**multiplexing（复用）**，即使一项系统资源在多个用户之间共享。在直觉层面，multiplexing 可以用与分时计算机系统的类比来解释：单个物理 processor 在多个 job 之间被共享（复用），每个 job 都相信自己拥有一个私有的 processor。类似地，多个用户发送的数据可以在构成网络的物理链路上被复用。

要看清这可能如何工作，考虑图 1.5 所示的简单网络：网络左侧的三台 host（发送方 S1–S3）通过共享一个只包含一条物理链路的交换式网络，向右侧的三台 host（接收方 R1–R3）发送数据。（为简单起见，假设 S1 发给 R1，依此类推。）在这种情况下，三条数据流（对应三对 host）被 switch 1 复用到一条物理链路上，然后被 switch 2 解复用（demultiplexed）回分离的流。注意，我们对「一条数据流」究竟对应什么有意保持含混。就本讨论而言，假设左侧每台 host 都有大量数据想发给右侧的对应方。

[Figure 1.5：**把多条逻辑流复用到一条物理链路上**——三对 host（S1–R1、S2–R2、S3–R3）经两条 switch 之间的**单条**物理链路相连；switch 1 把三条流合并进这条链路，switch 2 再把它们分开。图上并列画出了三条流各自的数据块在共享链路上的排列（即第 1.6 图将展开的 packet 交错）。]

> **部分可复原**：图题进入文字层（`Figure 1.5.: Multiplexing multiple logical flows over a single physical link.`），正文又给出了节点名与角色，因此拓扑（三对 host、两台 switch、一条共享链路）可核。但图中数据块的**具体交错顺序与时间刻度**未进入文字层，无法复原，故不描述。

<span style="color:#1e8449;">**[译注]** multiplexing 这段的类比有一个**必须修正**的地方：分时计算机系统的「每个 job 都相信自己独占一个 processor」是由 **context switch 的时间片**保证的，而网络里的「每条流都觉得自己独占带宽」并不是靠时间片，而是靠**统计**——只在有数据时才占用，且每次占用有上界（即 packet 的长度）。这个差别决定了后面 §1.2.3 的整个论证走向：正因为不是时间片，才需要额外的机制保证「不漏掉任何一条流」（否则一条大流可以永远占住链路），于是引入了 packet 的大小上界与交换机的调度。

**可算的方向性判断**：设链路速率 $B$、平均 packet 长度 $L$、活跃流数 $n_{\text{active}}$，则每条流在「公平轮转」下得到的速率约为 $B/n_{\text{active}}$。注意分母是**活跃**流数而不是总流数——这是 statistical multiplexing 与 STDM 的本质区别：STDM 的分母是**固定**流数 $n$，因此单流上限被永久锁死在 $B/n$，即使其他 $n-1$ 条流一个比特都不发。

**消失测试**：抽掉「按需占用」这条，回到时间片轮转，则一条只想发 1 MB 的流也必须等自己那一轮时隙，且其他时隙空转；在 $n = 1000$、$p = 0.01$ 的突发流量下，链路利用率上限是 $1\%$。

**落点**：§1.2.3 全节（本卷 p019–下一卷 p021）；CS144 Lab3 里「发送窗口内的数据交错发出」正是同一机制的端主机侧；跨学科坐标是 CSAPP 第 8 章 signal/进程调度里的时分复用（相同的思想，不同的资源）。</span>

<span style="color:#1e8449;">**[译注]** 「我们对『一条数据流』究竟对应什么有意保持含混」这句是作者的免责声明，但它有实质后果：**flow（流）不是网络协议里的一个原语**。在 IP 层，路由器看到的是一个个独立 datagram，没有任何「流」的状态；flow 是端主机或测量工具事后通过五元组（源 IP、源端口、目的 IP、目的端口、协议号）**归类**出来的概念。这个区别在 CS144 里非常具体：Lab5/Lab6 的网络接口与路由器**都不维护 flow 表**，你唯一能看到的 per-flow 状态在 `TCPConnection` 与 `TCPSender` 里。

**可判定的判据**：如果某个机制需要「记住上一条包属于哪条流」，它就引入了 $O(\text{流数})$ 的状态，因此不可能是无状态网络层的义务。NAT 与 stateful firewall 正是靠打破这条判据工作的（它们维护连接表），代价是连接表耗尽时新连接被拒——这是可观测的失败模式（表满、丢新连接）。

**消失测试**：抽掉「flow 只是归类」这个认识，你可能在 Lab6 的路由器里试图做 per-flow 排队以保证公平；但 lab 的测试框架只按 datagram 转发计数，per-flow 状态会让转发变慢且在测试中不可见。

**落点**：§1.2.3（本节）；§6.2 Queuing Disciplines（书页 296，per-flow 排队才在这里正式出现）；§3.3 IP 的 datagram 模型（书页 125）；CS144 Lab5/Lab6 无流状态这一事实。</span>

---

> **本卷末说明**：本卷止于书页 16 的 §1.2.3 中段（"There are several different methods for multiplexing..." 之前）。STDM / FDM / statistical multiplexing 的完整论证、Figure 1.6、Key Takeaway、§1.2.4–§1.5 与 §1.3 全文见下一份 `Peterson网络_p021-040.bilingual.md`（PDF 第 21–40 页 = 书页 17–36）。

# Computer Networks: A Systems Approach（计算机网络：系统方法）· 行内双语

> **原书**：Larry L. Peterson, Bruce S. Davie, *Computer Networks: A Systems Approach*, Release Version 6.1（第 6 版开源发行版），2019-11-26。
> **本卷覆盖**：源文件 `Peterson网络_p021-040.pdf`，PDF 第 21–40 页 = 书页 **17–36**。内容依次为：§1.2.3 尾（调度与拥塞）、§1.2.4 Support for Common Services、§1.2.5 Manageability、§1.3 Architecture 全节（layering / encapsulation / multiplexing / 7-layer OSI / Internet architecture）、§1.4 Software 前半（socket API 与 simplex-talk 示例代码起首）。
> **切块约定**：每块以 `<!-- ===== pNNN ===== -->` 开头，NNN 为 **PDF 页码**；块内 `> 来源注` 声明对应的印刷页码。**本卷 PDF 页码与印刷页码恒定相差 16**（PDF p021 = 书页 17，PDF p040 = 书页 36），下文「书页 N」一律指印刷页码。

**阅读约定**

1. 译文以中文为主体，术语首次出现写 `English（中文短注）`，此后只留 English 原词。
2. 网络四层术语与 CS144 讲义双语版一致，一律不意译：**application layer（应用层）/ transport layer（传输层）/ network layer（网络层）/ link layer（链路层）**。其余保留原词：packet / frame / datagram / message / segment / socket / ACK / window / congestion control / flow control / routing / forwarding / MTU / ARP / DNS / TCP / UDP / IP / switch / router / host / channel / multiplexing / demultiplexing / encapsulation / header / payload / protocol graph / service interface / peer interface / demux key / protocol stack。
3. 行内公式 `$...$`，块级公式 ```math 围栏；不用行间美元定界符，不用反斜杠方括号定界符。
4. 标签配色：<span style="color:#1f6feb;">**[Key Takeaway]**</span> 原书小结框；<span style="color:#943126;">**[note]**</span> 原书脚注与插框；<span style="color:#6c3483;">**[example]**</span> 例子。**自写内容一律绿色 `[译注]`，不占用上述任何颜色。**
5. 代码照抄原书，不翻译、不改逻辑；代码自身的缺陷以 `†` 标出并说明。源 PDF 的文字层把长行折断，凡属提取伪影的折行，按语句边界复原并注明。
6. 插图以 `[Figure N：它演示的是什么]` 保留；只有图题、无法从文字层复原的图像明写「无法复原」。
7. 原文自身错误以 `†` 标记。

---

<!-- ===== p021 ===== -->
> 来源注：PDF 第 21 页 = 书页 17（页脚 `1.2. Requirements 17`）。本页为 §1.2.2 末与 §1.2.3 的前段。

决定「共享链路上接下来发哪个 packet」的方式可以有若干种。例如，在一个由链路互连的 switch 构成的网络（如图 1.5）中，这项决定由把 packet 发上共享链路的那台 switch 做出。（我们后面会看到，并非所有 packet-switched 网络都真的有 switch，它们可能用其他机制来决定接下来谁的 packet 上链路。）packet-switched 网络中的每台 switch 都**独立地、逐个 packet 地**做这项决定。网络设计者面临的问题之一，是如何**公平地**做这项决定。例如，一台 switch 可以设计成一先进先出（**first-in, first-out，FIFO**）的方式服务 packet。另一种做法是，把当前正通过该 switch 发送数据的各条流的 packet 以**轮转（round-robin）** 方式发送。这样做可能是为了保证某些流获得链路的特定份额，或者保证它们的 packet 在 switch 中被延迟的时间不超过某个长度。一个试图把带宽分配给特定流的网络，有时被说成支持 **quality of service（QoS，服务质量）**。

另外注意图 1.6 中，由于 switch 必须把三条入向 packet 流复用到一条出向链路上，switch 收到 packet 的速度有可能超过共享链路能够容纳的速度。在这种情况下，switch 被迫把这些 packet **缓冲（buffer）** 在它的内存中。如果一台 switch 在相当长的一段时间内收到 packet 的速度都快于它发出的速度，那么这台 switch 最终会耗尽缓冲空间，一些 packet 将不得不被**丢弃（drop）**。当一台 switch 处于这种状态运行时，就说它发生了 **congestion（拥塞）**。

<span style="color:#1f6feb;">**[Key Takeaway]**</span> 底线是：statistical multiplexing 为多个用户（例如 host 到 host 的数据流）定义了一种**成本有效**的方式，以细粒度（fine-grained manner）共享网络资源（链路与节点）。它把 **packet** 定义为「网络链路被分配给不同流」的粒度，每台 switch 都能逐个 packet 地调度它所连物理链路的使用。把链路容量公平地分配给不同流、以及在拥塞发生时处理拥塞，是 statistical multiplexing 的关键挑战。[Next]

<span style="color:#1e8449;">**[译注]** FIFO 与 round-robin 的差别不是价值判断，而是一个**可测的排队现象**：FIFO 下所有流共用一个队列，于是某类包的排队延迟完全由「队列中其它流的包量」决定，这就是 head-of-line blocking（队头阻塞）。设共享队列的平均占用为 $\bar{Q}$ 个 packet、出向速率为 $B$、每个 packet 长 $L$，则平均排队延迟是 $\bar{Q}L/B$。在 $B = 100$ Mbps、$L = 1500$ B 下，一个包的发送时间是 $120\ \mu s$；若队列平均排了 50 个包，延迟就是 $6$ ms——对一次 TCP 握手（1 RTT）尚可，但对实时语音（端到端预算 150 ms，单跳只应分到几毫秒）已经不可接受。

**可判定的判据**：若你需要「某类流的最大排队延迟有界」，FIFO **给不出**这个界（上界由队列总长度决定，与流的类别无关）；必须换成本篇 §6.2 的 weighted fair queuing 一类调度器，代价是每个包要额外维护与比较若干 virtual time 值。这条判据可实测：大文件传输与语音同队列时观测语音 packet 间隔，FIFO 下抖动随文件突发放大到十毫秒以上，WFQ 下基本不变。

**消失测试**：抽掉「调度策略」这个自由度，只剩 FIFO，则 QoS 的一切承诺都无法表达；可观测现象是：文件下载一开始，语音就断续（jitter 从 2 ms 涨到 30 ms 以上）。

**落点**：§6.2 Queuing Disciplines（书页 296）与 §6.5 Quality of Service（书页 323）；CS144 Lab6 的 `Router` 队列就是 FIFO，因此 lab 里观测不到任何优先级行为；跨学科坐标是排队论 M/M/1 的平均队长 $\bar{Q} = \rho/(1-\rho)$，$\rho \to 1$ 时延迟发散——这是「拥塞一旦发生就崩」的数学原因。</span>

<span style="color:#1e8449;">**[译注]** 「收得比发得快 → 缓冲 → 耗尽 → drop」这一串定义了**丢包的两个来源**，而这两个来源在诊断上必须分开：一是**链路误码**（bit error 导致帧校验失败，书页 20 给出 $10^{-6}$–$10^{-7}$ 的铜缆量级），二是**缓冲溢出**（congestion 导致 drop）。同一次丢包，归因错了修法完全相反：前者要换介质或加重传，后者要降速率。TCP 恰恰把两者混在一起——它只看「丢包」这个事件，因此在低误码的有线链路上拥塞是主因，而在高误码的无线链路上 TCP 会把误码误判为拥塞，白白减半窗口。

**可算的判据**：一条 100 Mbps 链路、RTT 50 ms，其带宽时延积 $B\cdot\text{RTT} = 5\times 10^{6}$ bit，约 417 个 1500 B 的 packet；要让链路不空转，发送端必须有约 417 个包在途，缓冲也应在这个量级。缓冲再大只会增加延迟而不增加吞吐——这正是 §6.4 的 AQM（Active Queue Management）要主动丢包的动机。

```math
\text{in\text{-}flight} \;\le\; \min\big(\text{cwnd},\ \text{rwnd}\big),
\qquad
\text{throughput} \;\le\; \frac{\text{in\text{-}flight}}{\text{RTT}}
```

**消失测试**：抽掉「缓冲」这一级，switch 只能在「丢」与「反压（backpressure）」之间选；后者要求上游暂停发送，而在 packet-switched 网络里上游是无数个互不协调的端主机，无法被统一暂停——这就是为什么丢包成了唯一的拥塞信号。

**落点**：§6.2 与 §6.4（书页 296、311）；§2.4 Error Detection（书页 59）给出误码那一半；CS144 Lab3 的 `fill_window()` 直接体现「在途字节数不得超过 window」。</span>

---

<!-- ===== p022 ===== -->
> 来源注：PDF 第 22 页 = 书页 18（页脚 `18 Chapter 1. Foundation`）。本页含 Figure 1.7 与 §1.2.4 前半。

#### 1.2.4 Support for Common Services（对公共服务的支持）

上面的讨论聚焦于「在一组 host 之间提供成本有效的连通性」所涉及的挑战，但把计算机网络简单看作「在一群计算机之间投递 packet」就过于简化了。更准确的想法是：网络为「分布在这些计算机上的一组应用进程（application process）」提供相互通信的手段。换句话说，计算机网络的下一项要求是：运行在连网 host 上的应用程序必须能够以有意义的方式通信。从应用开发者的视角看，网络需要让他或她的日子更好过。

当两个应用程序需要相互通信时，除了「把一条 message 从一台 host 送到另一台 host」，还必须有大量复杂的事情发生。一种选择是让应用设计者把那套复杂功能都做进每一个应用程序里。然而，由于许多应用需要**共同的**服务，更合乎逻辑的做法是把这些公共服务实现一次，然后让应用设计者用这些服务来构建应用。网络设计者的挑战在于识别出正确的公共服务集合。目标是：把网络的复杂性对应用隐藏起来，同时又不过度约束应用设计者。

直觉上，我们把网络看作提供**逻辑通道（logical channel）**，应用层进程可以在这些通道上相互通信；每条通道提供该应用所需的那组服务。换句话说，正如我们用一朵云来抽象地表示一组计算机之间的连通性，现在我们把一条 channel 看作把**一个进程**连到**另一个进程**。图 1.7 展示了一对应用层进程在一条逻辑通道上通信，而这条通道又实现在一朵连接一组 host 的云之上。我们可以把 channel 想象成一根连接两个应用的管道（pipe），于是发送方应用可以把数据放进一端，并期望这些数据被网络投递给管道另一端的应用。

[Figure 1.7：**进程在抽象通道上通信**——两台 host 各有一个应用层进程，两者之间画一条跨过中间那朵云的抽象 channel；云表示实际实现这条通道的 host 间网络。图示要点是「进程到进程」与「host 到 host」是两个不同层级的连接。]

> **部分可复原**：图题进入文字层（`Figure 1.7.: Processes communicating over an abstract channel.`），正文给出了两侧为应用层进程、中间为云、通道是抽象的。图中是否画出协议栈的层次框、箭头走向未进入文字层，无法复原，故不描述。

像任何抽象一样，逻辑的进程到进程通道实现在一组物理的 host 到 host 通道之上。这就是分层的本质（原文此处误拼作 "essense"，应为 essence †），也是下一节讨论的网络架构的基石。

> † 拼写错误：`essense` → `essence`。这类拼写错误在本书 PDF 发行版中反复出现（另见书页 23 的 `ulikely`、书页 31 的 `impies` / `Internt`），可作为「Release 6.1 未经最终校对」的证据。

**Identify Common Communication Patterns（识别共同的通信模式）**

设计抽象通道，首先要理解一组有代表性应用的通信需求，然后提取它们共同的通信要求，最后把满足这些要求的功能纳入网络。

在任何网络上被支持的最早的应用之一，是像 **File Transfer Protocol（FTP，文件传输协议）** 或 **Network File System（NFS，网络文件系统）** 那样的文件访问程序。虽然许多细节各有不同——例如是整个文件在网络上传送，还是在某一时刻只读写文件中的单个块——但远程文件访问的通信成分由一对进程刻画：一个进程请求读或写某个文件，另一个进程满足这个请求。请求访问文件的进程称为 **client（客户）**，支持文件访问的进程称为 **server（服务器）**。

<span style="color:#1e8449;">**[译注]** 「channel 像一根管道」这个类比必须标出**边界**：管道是有序的字节流（FIFO 字节序列），而 channel 的语义是**可参数化的四元组**，顺序只是其中一维。把 channel 一律想成管道，会漏掉「无序」这一类，从而无法理解为什么 §5.1 要先讲一个**什么都不保证**的 UDP。判据很简单：管道必然有序，channel 不必。

```math
\text{channel 语义} = \big(\text{reliability},\ \text{ordering},\ \text{timing},\ \text{group}\big)
```

**可判定的判据**：给出任何一条通道，逐维填这四个量就能唯一确定它：TCP = (可靠, 有序, 无时限, 单播)；UDP = (不可靠, 无序, 无时限, 支持多播/广播)；RTP = (不可靠, 有序（带序号可检测乱序）, 有时限, 支持多播)。CS144 的 `ByteStream` 只实现了第一维与第二维（可靠、有序），第三、四维在 lab 里根本不出现——这是理解 lab 边界的最快方法。

**消失测试**：抽掉「channel 可参数化」这层含义，只保留「有序字节流」，则 UDP、RTP（§5.4）、multicast（§4.3）都无法表达；具体失效现象是：你会试图用 TCP 承载实时视频，于是丢一个包就等重传，播放端出现数百毫秒卡顿，而正确做法是丢弃该帧继续播。

**落点**：§5.1（书页 230）、§5.2（书页 233）、§5.4（书页 276）；CS144 Lab0 的 `ByteStream` 与 Lab5 的 `NetworkInterface`（后者交付的是无序 datagram 语义）。</span>

<span style="color:#1e8449;">**[译注]** 「这就是分层的本质」这半句给出了分层的**形式化判据**：如果「逻辑进程到进程通道」实现在「物理 host 到 host 通道」之上，那么两层之间必须存在一个**只依赖接口、不依赖实现**的组合关系。可检验的形式是：把下层整体替换掉（把以太网换成 Wi-Fi），上层代码一行不改仍应工作。

**可判定的判据**：一处设计是否真的分层，看它是否允许「下层替换而无需重编上层」。若某机制需要上层感知下层的具体参数（例如应用把 MTU 硬编码成 1500），这一处就是分层的**漏点**——IPv6 引入 path MTU discovery 正是为了堵这个漏点。

**消失测试**：如果分层只在名词上成立、换下层必须改上层，那么第 2 章讲 Ethernet、第 3 章讲 IP 时读者会以为是「同一件事的两种说法」，从而无法解释 IP 为何能同时跑在 Ethernet、Wi-Fi、PPP、PON 之上。

**落点**：§1.3.1 Layering and Protocols（书页 22，本卷 p026）；§2.8 中 PON 与 5G 对同一 IP 层的复用（书页 95）；CS144 Lab5 的 `NetworkInterface` 与 Lab6 的 `Router` 之间的接口边界。</span>

---

<!-- ===== p023 ===== -->
> 来源注：PDF 第 23 页 = 书页 19（页脚 `1.2. Requirements 19`）。

读一个文件涉及：client 向 server 发送一条**小的请求 message**，server 用一条**包含文件数据的大 message** 响应。写则相反——client 发送一条包含待写数据的大 message 给 server，server 用一条确认「写入磁盘已完成」的小 message 响应。

数字图书馆（digital library）是比文件传输更精巧的应用，但它需要类似的通信服务。例如，计算机协会（Association for Computing Machinery，ACM）在

`http://portal.acm.org/dl.cfm`

运营着一个大型的计算机科学文献数字图书馆。这个图书馆有大量检索与浏览功能来帮用户找到想要的论文，但归根到底它做的大部分事情是响应用户对文件的请求，例如期刊论文的电子副本。

以文件访问、数字图书馆，以及引言中描述的两个视频应用（videoconferencing 与 video on demand）作为有代表性的样本，我们可能决定提供以下两类通道：**request/reply channel（请求/应答通道）** 与 **message stream channel（消息流通道）**。request/reply channel 供文件传输与数字图书馆应用使用。它会保证一方发出的每条 message 都被另一方收到，并且每条 message 只被投递一份。request/reply channel 还可能保护在它上面流动的数据的隐私与完整性，使得未授权方无法读取或修改 client 与 server 进程之间交换的数据。

message stream channel 供 video on demand 与 videoconferencing 使用，前提是它被参数化为支持单向与双向流量，并支持不同的延迟特性。message stream channel 可能不需要保证所有 message 都被投递，因为即使某些视频帧没收到，视频应用也能勉强运作。不过，它需要确保**被投递的那些 message 按发送顺序到达**，以避免乱序显示帧。像 request/reply channel 一样，message stream channel 可能也希望保证视频数据的隐私与完整性。最后，message stream channel 可能需要支持 multicast，以便多方参与电话会议或观看视频。

虽然网络设计者常常力求用**最少**的抽象通道类型服务**最多**的应用，但试图用太少种通道抽象来应付是有危险的。简单地说：如果你手里只有一把锤子，那么一切看起来都像钉子。例如，如果你只有 message stream 与 request/reply 两种通道，那么下一个出现的新应用就会诱使你用它们，即使这两种都不提供该应用恰好需要的语义。因此，只要应用程序员还在发明新应用，网络设计者大概就会不断发明新的通道类型——并给既有通道添加选项。

还要注意，与某条通道究竟提供什么功能无关，还有一个问题：这些功能**在哪里**实现。在许多情况下，最简单的看法是：把底层网络的 host 到 host 连通性看作只提供一根 **bit pipe（比特管道）**，任何高层通信语义都在**端主机（end host）** 提供。这种做法的优点是它让网络中间的 switch 尽可能简单——它们只转发 packet——但它要求端主机承担支撑语义丰富的进程到进程通道的许多负担。另一种选择是把额外功能推到 switch 上，从而允许端主机成为「哑」设备（例如电话听筒）。我们将会看到，「各种网络服务如何在 packet switch 与端主机（设备）之间划分」这个问题，是网络设计中反复出现的问题。

<span style="color:#1e8449;">**[译注]** 「读 = 小请求 + 大响应；写 = 大请求 + 小响应」这组不对称是本页的隐藏重点，它解释了**为什么可靠传输必须有窗口**。设请求 $r$ 字节、响应 $R$ 字节（$r \ll R$），MSS 为 1460 B、RTT 为 50 ms。在 stop-and-wait 下每发一个段等一个 ACK，吞吐被锁死在

```math
\text{throughput}_{\text{stop-and-wait}} = \frac{\text{MSS}}{\text{RTT}} = \frac{1460 \times 8}{0.05} \approx 233\ \text{kbps},
```

即一条 1 Gbps 链路只被用到万分之二。要让链路跑满，在途字节数必须达到 $B\cdot\text{RTT} = 6.25$ MB，约 4280 个段——这就是 §2.5 必须引入 sliding window、CS144 Lab1 必须先写带 `capacity` 的 `ByteStream` 的量化理由。

**可判定的判据**：如果一条可靠传输协议没有「多个段在途」的机制，它的吞吐上限就是 $\text{MSS}/\text{RTT}$，与链路带宽无关。判据可以用测量落地：`iperf` 在 RTT 50 ms 的链路上如果只跑到约 200 kbps，先查窗口而不是查带宽。

**消失测试**：抽掉「窗口」，Lab2 的 `window_size()` 会永远公告 64000（一个与接收缓冲容量无关的常数），`Recv` 的容量检查永不触发；可观测现象是 lab 的 `recv_window` 测试失败，而其它测试全过。

**落点**：§2.5 Reliable Transmission（书页 65）与 §5.2（书页 233）；CS144 Lab1 `byte_stream.hh` 的 `capacity`、Lab2 `TCPReceiver::window_size()`、Lab3 的 `fill_window()`。</span>

<span style="color:#1e8449;">**[译注]** 最后这段是全书**最重要的一次立场宣告**：它其实就是 end-to-end argument（端到端论证）的雏形，但原文只说这是「反复出现的问题」，没有点名。判据可以写得很硬：把功能放在 switch 上的代价是 switch 必须维护**每条连接的状态**，核心网内存随时延与连接数增长；放在端主机上的代价是端主机各实现一遍，但状态只与**本机连接数**成正比。设核心网 $N$ 台 switch、$C$ 条并发连接、每连接状态 $s$ 字节，前者需 $O(NCs)$，后者需 $O(Cs)$（分散在 $2C$ 个端点）。

**可算的失败现象**：$N = 1000$、$C = 10^{6}$、$s = 1$ KB 时，$NCs = 10^{12}$ B = 1 TB 的核心网状态；任何一台 switch 重启都会带走所有经过它的连接（而不是让端主机各自重传恢复）。

**消失测试**：抽掉这条判据，把可靠传输做进路由器，就变成「每一跳都重传」：一个包在 5 跳路径上被独立重传，端到端成功率为 $(1-p_{\text{hop}})^{5}$，且端主机无法判断收到的包是否被某一跳改写过。

**落点**：§5.2（书页 233）把可靠性放在 end host；CS144 **Lab6 的 `Router` 不实现任何重传与序号**，是这条判据的实物；一手来源是 Saltzer, Reed & Clark, *End-to-End Arguments in System Design*（1984），A 级；跨学科坐标是「状态放在共享处还是本地」的一般原则，与 CSAPP 里 cache 一致性同构。</span>

---

<!-- ===== p024 ===== -->
> 来源注：PDF 第 24 页 = 书页 20（页脚 `20 Chapter 1. Foundation`）。本页含「Reliable Message Delivery」全节前半与 Key Takeaway 起首。

**Reliable Message Delivery（可靠的消息投递）**

正如刚才考虑的例子所提示的，可靠的消息投递是网络能提供的最重要功能之一。然而，在首先理解网络**如何失败**之前，很难确定如何提供这种可靠性。要认识的第一件事是：计算机网络并不存在于一个完美的世界里。机器会崩溃并在之后被重启，光纤会被挖断，电干扰会破坏正在传输的数据中的比特，switch 会用尽缓冲空间；而且，好像这些物理问题还不够让人担心似的，管理硬件的软件可能有缺陷，有时会把 packet 转发到虚无之中。因此，网络的一项主要要求是**从某些类型的故障中恢复**，使得应用程序不必处理它们，甚至不必意识到它们。

网络设计者必须操心的故障有三大类。第一类：当一个 packet 在物理链路上传输时，数据中可能被引入 **bit error（比特错误）**；也就是说，一个 1 变成了 0，或反之。有时是单个比特被破坏，但更常见的是发生 **burst error（突发错误）**——若干个连续比特被破坏。bit error 通常发生的原因是外部力量，例如雷击、电源浪涌与微波炉，干扰了数据传输。好消息是这类 bit error 相当罕见：在典型的铜基电缆上平均每 $10^{6}$ 到 $10^{7}$ 个比特才有一个受影响，在典型光纤上每 $10^{12}$ 到 $10^{14}$ 个比特才有一个。† 正如我们将看到的，有一些技术能以很高的概率**检测**这些 bit error。一旦检测到，有时就有可能**纠正**这类错误——如果我们知道哪个或哪些比特被破坏了，直接把它们翻转即可——而在另一些情况下破坏太严重，必须丢弃整个 packet。在这种情况下，可能期望发送方重传该 packet。

> † **上标丢失**：源 PDF 文字层此处显示为 "one out of every 106 to 107 bits" 与 "one out of every 1012 to 1014 bits"，上标未进入文本层。按上下文与数量级还原为 $10^{6}$–$10^{7}$ 与 $10^{12}$–$10^{14}$。这一处若不还原，读者会以为误码率是 $1/106$ 这种荒谬量级。

第二类故障发生在 **packet** 层面而非比特层面；也就是说，网络丢失了一个完整的 packet。发生这种情况的一个原因是该 packet 含有不可纠正的 bit error，因此不得不被丢弃。然而更可能的原因是：必须处理该 packet 的某个节点（例如把它从一条链路转发到另一条链路的 switch）超载到没有地方存放该 packet，因而被迫丢弃它。这就是刚讨论过的 congestion 问题。较少见的情况是：处理该 packet 的某个节点上运行的软件犯了错，例如它可能错误地把一个 packet 从错误的链路转发了出去，使该 packet 永远无法到达最终目的地。正如我们将看到的，处理丢失 packet 的主要困难之一，是**区分「确实丢失的 packet」与「只是迟到到达目的地的 packet」**。

<span style="color:#1e8449;">**[译注]** 这段给出全书第一个可用的**误码算术**，它后面所有检错机制的规格都由它决定。一个 1500 B 的以太网帧 = $12000$ bit；在 $p = 10^{-6}$ 的铜缆上，至少一个比特出错的概率是

```math
P_{\text{frame error}} = 1 - (1-p)^{12000} \approx 12000p = 1.2 \times 10^{-2},
```

即约**每 83 个帧坏一个**；同样的式子在 $p = 10^{-12}$ 的光纤上给出 $1.2\times10^{-8}$，约每 8300 万个帧坏一个。这个 $10^{6}$ 倍的差距解释了两件事：为什么铜缆时代的链路层必须有重传；为什么 **CRC-32 在 $10^{-6}$ 下仍不够兜底**——CRC-32 漏检率约 $2^{-32}$，以 1 Gbps 计（约 8 万帧/秒、其中约 960 个坏帧），未被发现的坏帧期望约 $960\times2^{-32} \approx 2.2\times10^{-7}$ 个/秒，即大约两个月一次。

**消失测试**：没有这套算术，你在 Lab2 实现 `Internet checksum`（16 位反码求和）时会以为它「保证正确」；实际上它连突发错误都可能漏检，且对两个 16 位字的顺序互换完全无感（反码求和可交换）。lab 的测试只验证「校验和算得对」，不验证检出能力——这个区别必须自己知道。

**落点**：§2.4 Error Detection（书页 59，CRC 的正式处理）；CS144 Lab2 的 `Internet checksum`（RFC 1071）；一手坐标是 Peterson & Brown, *Cyclic Codes for Error Detection*（1961）与 RFC 1071。</span>

<span style="color:#1e8449;">**[译注]** 「区分丢失与迟到」这半句是本节技术含量最高的一句，它的正式名字是**重复包问题**：重传太早，接收方收到两份相同数据；重传太晚，吞吐下降。判别的唯一手段是**序号 + 超时**，而超时值必须大于 RTT 分布的上界——这个上界在原理上不可知（路径变化、排队），所以任何固定超时都在两类错误之间取舍。TCP 的做法是自适应估计

```math
\text{RTO} = \text{SRTT} + 4\cdot\text{RTTVAR},
```

并按 Karn 算法只对**未重传**的样本更新估计（否则重传包的往返时间会被重复计入，导致估计偏大）。

**可判定的判据**：RTT 抖动 $\text{RTTVAR}$ 越大，RTO 越松，丢包恢复越慢。这条式子直接解释了无线上的 TCP 为什么差：不是因为「信号不好」，而是因为抖动使 RTO 保守，一次丢包要等更久才重传。

**消失测试**：抽掉序号，接收方无法区分「重传的重复包」与「新包」，同一笔数据被上层处理两次；在 Lab2 里表现为 `TCPReceiver` 把重复字节写进 `ByteStream`，`bytes_written` 超过发送方的值——这正是 lab 的 `recv_repeat` 测试要抓的东西。

**落点**：§2.5 Reliable Transmission（书页 65）与 §5.2 的 RTT 估计（书页 233）；CS144 Lab3 的 `tick()` 与 RTO 翻倍；一手坐标是 RFC 6298。</span>

---

<!-- ===== p025 ===== -->
> 来源注：PDF 第 25 页 = 书页 21（页脚 `1.3. Architecture 21`）。本页含 Key Takeaway 尾、§1.2.5 Manageability、§1.3 Architecture 起首。

<span style="color:#1f6feb;">**[Key Takeaway]**</span> 从这段讨论中要带走的关键想法是：定义有用的通道，既涉及理解应用的需求，也涉及认识底层技术的局限。挑战在于填补「应用期待什么」与「底层技术能提供什么」之间的差距。这个差距有时被称为 **semantic gap（语义鸿沟）**。[Next]

#### 1.2.5 Manageability（可管理性）

最后一项要求（它似乎常常被忽视或被留到最后，正如我们在此所做的那样），是网络需要被管理。管理一个网络包括：随着网络增长以承载更多流量或触及更多用户而**升级设备**；在出问题或性能不如预期时**排查故障**；以及为支持新应用而**添加新功能**。网络管理历来是网络工作中人力密集的一面，虽然我们不太可能把人完全移出这个回路（原文误拼作 "ulikely"，应为 unlikely †），但它正越来越被自动化与自愈（self-healing）设计所应对。

> † 拼写错误：`ulikely` → `unlikely`。

这项要求部分地与上面讨论的 scalability 问题相关——随着 Internet 扩展到支撑数十亿用户与至少数亿台 host，让整个系统正确运行、并在新设备加入时正确配置它们的挑战，已变得越来越成问题。在一个网络中配置一台 router，常常是受过训练的专家的工作；配置成千上万台 router，并弄清一个如此规模的网络为何不按预期行为，则可能成为超出任何单个人能力的任务。这就是自动化变得如此重要的原因。

让网络更容易管理的一种办法是**避免变更**。一旦网络工作起来，就干脆别去动它！这种心态暴露了 **stability（稳定性）** 与 **feature velocity（功能速度）** 之间的根本张力：后者指新能力被引入网络的速度。偏好稳定性是电信行业（更不用说大学系统管理员与公司 IT 部门）多年来采取的做法，这使它成为你能找到的最慢、最厌恶风险的行业之一。但最近 cloud 的爆发改变了这个动态，使得把稳定性与功能速度更多地拉向平衡成为必要。cloud 对网络的影响是一个贯穿全书的主题，我们在每章末的 Perspectives 一节给予特别关注。眼下只需说：管理一个快速演化的网络，可以说是今天网络领域的核心挑战。

### 1.3 Architecture（架构）

上一节为网络设计确立了一组相当可观的要求——一个计算机网络必须在大量计算机之间提供通用的、成本有效的、公平的、健壮的连通性。好像这还不够似的，网络并不是在某个时间点上固定不变，而必须演化，以适应它所依赖的底层技术的变化，以及应用程序对它提出的需求的变化。此外，网络必须能被技能水平各不相同的人管理。设计一个满足这些要求的网络绝非小事。

为了帮助应对这种复杂性，网络设计者发展出通用的蓝图（blueprint）——通常称为 **network architecture（网络架构）**——来指导网络的设计与实现。本节通过引入所有网络架构共有的中心观念，更仔细地界定我们所说的网络架构是什么意思。它还引入两个被最广泛引用的架构——**OSI（或 7 层）架构**与 **Internet 架构**。

<span style="color:#1e8449;">**[译注]** 「避免变更」这段的最佳落地方式是**变更频率与故障率的统计关系**，而原文只给了定性。公开的业界数据（Google SRE，A 级）是：**约 70% 的故障由变更引起**。把它写成可算的取舍：设每次变更带来故障概率 $\epsilon$、变更频率 $f$、单次故障代价 $D$（含 MTTR 与业务损失），则单位时间故障成本为 $f\epsilon D$；若新功能带来的价值为 $V(f)$，最优解在

```math
\frac{dV}{df} = \epsilon D
```

处。这条式子给出了一个反直觉但正确的推论：**降低 $\epsilon$（灰度、可回滚、自动化测试）在数学上等价于「允许更高的 $f$」**——CI/CD 与 canary deployment 不是「敏捷文化」，而是把 $\epsilon$ 压下来以便上式允许更快的变更。

**消失测试**：抽掉 $\epsilon$ 这个可控量，只讨论「该不该变」，就只能在「冻结」与「裸奔发布」之间选；可观测的失败现象是：一次配置变更导致大范围中断，且没有可回滚的上一版本——这正是过去电信机房与今天缺 IaC 的企业网络的共同处境。

**落点**：§9.3 Infrastructure Applications（书页 449，NMS 与 OpenConfig/gNMI）；§3.5 的 SDN（书页 169，把变更做成中央可回滚的配置源）；跨学科坐标是 CSAPP 第 9 章虚拟内存的 copy-on-write（同一种「先保住旧版本再切」的思路）。</span>

<span style="color:#1e8449;">**[译注]** 「配置一千台 router 超出任何单个人的能力」可以算：设每台设备配置出错概率 $q$、设备数 $n$，则至少一处出错的概率是 $1-(1-q)^{n}$；$q = 10^{-2}$、$n = 1000$ 时

```math
1 - 0.99^{1000} \approx 1 - 4.3\times10^{-5},
```

也就是**几乎必然出错**。把「正确配置 $n$ 台」当作一个任务，其失败概率随 $n$ 单调逼近 1。这是声明式配置与自动化校验的**充分**理由（不是「省人力」）：只有当人工步骤数不随 $n$ 增长时，任务的成功率才不随 $n$ 衰减。

**可判定的判据**：一个运维流程是否可扩展，看它的人工步骤数是否随设备数增长。若增长，其上限在 $10^{2}$ 台量级；若不增长（「改一个源文件、推给所有设备」），上限由工具而非人决定。

**消失测试**：抽掉自动化，网络规模被锁在人能核对的设备数上；历史上企业网络停留在数千台，而 cloud 运营商在同一套架构哲学下运营百万级设备，差别不在协议而在配置方式。

**落点**：§9.3（书页 449）；§3.5 的白盒交换机与控制平面集中化（书页 169）；跨学科坐标是 6.042J 的独立事件概率（$1-(1-q)^n$ 的推导只用到独立性）。</span>

---

<!-- ===== p026 ===== -->
> 来源注：PDF 第 26 页 = 书页 22（页脚 `22 Chapter 1. Foundation`）。本页为 §1.3.1 Layering and Protocols 全节，含 Figure 1.8。

#### 1.3.1 Layering and Protocols（分层与协议）

**Abstraction（抽象）**——把实现细节藏在一个定义良好的接口之后——是系统设计者用来管理复杂性的基本工具。抽象的想法是：定义一个能抓住系统某个重要方面的模型，把这个模型封装进一个提供接口的对象里（该接口可被系统的其他组件操纵），并把「该对象如何实现」的细节对它的使用者隐藏起来。挑战在于识别出这样的抽象：它同时（1）在大量情形下都证明有用，（2）能在底层系统中被高效实现。这正是我们在上一节引入 channel 这一观念时所做的：我们在为应用提供一个抽象，把网络的复杂性对应用编写者隐藏起来。

[Figure 1.8：**分层网络系统的一个例子**——应用程序之下、硬件之上夹着两层抽象：紧贴硬件的那一层提供 host-to-host connectivity（主机到主机连通性），把「任意复杂的网络拓扑」抽象掉；再往上一层建立在 host-to-host 服务之上，提供 process-to-process channel（进程到进程通道），把「网络偶尔丢 message」等事实抽象掉。图呈现的是两层与上下边界的包夹关系。]

> **部分可复原**：图题进入文字层，正文明确写出两层各自的职责与各自抽象掉的东西，故「两层 + 上下边界」的结构可核；图中是否标注具体协议名、是否画双向箭头未进入文字层，无法复原。

抽象自然导致**分层（layering）**，在网络系统中尤其如此。总体想法是：你从底层硬件提供的服务出发，然后加上一连串层，每一层提供更高（更抽象）级别的服务。高层提供的服务是以低层提供的服务为基础实现的。借用上一节给出的需求讨论，例如我们可以想象一个简单网络，在应用程序与底层硬件之间夹着两层抽象，如图 1.8 所示。在这种情况下，紧贴硬件之上的那一层可能提供 host-to-host connectivity，把「任意两台 host 之间可能存在任意复杂的网络拓扑」这一事实抽象掉。再往上一层建立在可用的 host-to-host 通信服务之上，提供对 process-to-process channel 的支持，把「网络偶尔丢失 message」这类事实抽象掉。

分层提供两个有用的特性。第一，它把「建造一个网络」的问题分解成更易管理的组件。你不必实现一个把所有你想要的东西都做进去的**整块（monolithic）** 软件，而可以实现若干层，每一层解决整体问题的一部分。第二，它提供更模块化的设计。如果你决定要添加某项新服务，可能只需要修改**一层**的功能，复用其他所有层提供的功能。

不过，把一个系统想成一串线性的层是一种过度简化。很多时候，在系统的任一给定层级上都提供着多种抽象，每一种向高层提供不同的服务，但都建立在相同的低层抽象之上。为看清这一点，考虑上一节讨论的两种通道。一种提供 request/reply 服务，一种支持 message stream 服务。这两种通道可能是某个多级网络系统某一层上的**备选**提供物（alternative offerings），如图 1.9 所示。

以这套关于分层的讨论为基础，我们现在准备更精确地讨论网络的架构。

<span style="color:#1e8449;">**[译注]** 「抽象要同时（1）有用、（2）能高效实现」这两条是本页唯一可被反驳的判据，而**第二条在教材里最常被忽略**。能算的反例：把「可靠有序字节流」做成抽象，但实现方式是「每发一个字节等一个 ACK」，它在 (1) 上完美、在 (2) 上崩溃（吞吐 $\le \text{MSS}/\text{RTT}$，RTT 50 ms 时约 $233$ kbps）。这说明**抽象的价值不能脱离实现成本评估**，而成本在本课程里的物理载体就是缓冲区容量与窗口。

**可判定的判据**：一个抽象是否「能高效实现」，看它是否需要在下层维护 $O(\text{上层实体数})$ 的状态。TCP 的可靠流只需每连接 $O(1)$ 的序号区间，所以能高效实现；而「保证每条 message 在 100 ms 内到达」需要每流的队列与调度状态（$O(\text{流数})$），所以在核心网里无法高效实现——这就是 QoS 只能落在边缘、拥塞控制只能落在端主机的同一个理由。

**消失测试**：抽掉条件 (2)，第 6 章整章失去动机（拥塞控制正是为了让「可靠流」这个抽象在真实网络里仍能高效实现）；可观测失效是：所有流以最大速率发送，队列持续溢出，吞吐跌到 $\text{MSS}/\text{RTT}$ 以下。

**落点**：§6.3 TCP Congestion Control（书页 301）；CS144 Lab1 `byte_stream.hh` 的 `capacity` 与 `bytes_read`/`bytes_written`；§2.5 的 sliding window（书页 65）。</span>

<span style="color:#1e8449;">**[译注]** 图 1.8 的画法有一处容易被低估的技术选择：它把**硬件**而不是「物理层协议」放在最底下。后果是——分层的起点是「硬件提供什么」，而不是「标准规定了什么」。判据可以这样立：如果分层的底层是标准，则任何未标准化的技术都无法参与（1970 年代各厂商私有链路就是这种处境）；如果底层是硬件能力，则「一条裸线也能跑 IP」成立。后者正是 IP over anything 的根据——包括那个著名的玩笑标准 RFC 1149（IP over Avian Carriers），它严肃的地方在于：IP 只要求「能送 datagram」。

**可判定的判据**：判断某个「层」是能力还是标准，看它是否允许下层的未标准化实现。CS144 Lab5 的 `NetworkInterface` 面对的正是「一张网卡能做什么」（能力），而不是「以太网标准说了什么」（标准）——所以 lab 里可以用 `TunTapFD` 这种非以太网设备冒充 link layer。

**消失测试**：抽掉「底层是硬件能力」这个视角，会把 §1.3.5 的窄腰误读成「IP 之上只能是 TCP/UDP」；实际上图 1.15（书页 30）明说应用可以绕过 transport 直接用 IP。

**落点**：§1.3.5 Internet Architecture（书页 29–30）；CS144 Lab5 的 `NetworkInterface` 与 `TAPDevice`；§2.1 Technology Landscape（书页 48）。</span>

---

<!-- ===== p027 ===== -->
> 来源注：PDF 第 27 页 = 书页 23（页脚 `1.3. Architecture 23`）。本页含 Figure 1.9、protocol 的定义与两种接口、Figure 1.10 引用。

[Figure 1.9：**在某一层上提供多种备选抽象的分层系统**——同一层级上并排放着 request/reply 与 message stream 两种通道，两者都建立在同一组低层抽象之上。图演示「层不是线性序列：同一层可以并列多种服务」。]

> **部分可复原**：图题进入文字层；正文给出「两种通道并列在同一层、共用同一低层」这一结构。图中具体标注与连线方向未进入文字层，无法复原。

首先，构成网络系统各层的抽象对象称为 **protocol（协议）**。也就是说，一个 protocol 提供一种通信服务，供更高层的对象（例如应用进程，或者更高层的协议）用它来交换 message。例如，我们可以想象一个网络支持 request/reply protocol 与 message stream protocol，分别对应上面讨论的 request/reply channel 与 message stream channel。

每个 protocol 定义两种不同的接口。第一，它定义一个 **service interface（服务接口）**，面向同一台计算机上想使用其通信服务的其他对象。这个 service interface 定义本地对象可以在这个 protocol 上执行的操作。例如，一个 request/reply protocol 会支持「应用可以发送与接收 message」的那些操作。HTTP 协议的一个实现可能支持「从远端服务器取一页超文本」这一操作。像 Web 浏览器这样的应用会在需要取得新页面时（例如用户在当前显示的页面上点击一个链接时）调用这样的操作。

第二，一个 protocol 定义一个 **peer interface（对等接口）**，面向另一台机器上的对应者（peer）。这第二个接口定义协议对等者之间为实现该通信服务而交换的 message 的形式与含义。这将决定一台机器上的 request/reply protocol 如何与另一台机器上的对等者通信。以 HTTP 为例，协议规范详细定义了 GET 命令如何格式化、该命令可以使用哪些参数，以及 Web 服务器收到这样一条命令时应如何响应。

总结起来：一个 protocol 定义它**本地导出**的通信服务（service interface），以及一组管辖「该协议为提供此服务而与对等者交换的 message」的规则（peer interface）。这种情形如图 1.10 所示。

除了硬件层面（对等者在此层通过物理介质直接通信）之外，对等者之间的通信是**间接的**——每个协议通过把 message 交给某个更低层的协议来与它的对等者通信，而后者又把 message 投递给自己的对等者。此外，在任一给定层级上都有可能存在不止一个协议，各自提供不同的通信服务。因此我们用一个 **protocol graph（协议图）** 来表示构成一个网络系统的那套协议。图的**节点**对应协议，**边**表示 **depends on（依赖于）** 关系。例如，图 1.11 画出了我们一直在讨论的那个假想分层系统的协议图——协议 RRP（Request/Reply Protocol）与 MSP（Message Stream Protocol）实现两种不同类型的进程到进程通道，两者都依赖于提供 host-to-host 连通性服务的 Host-to-Host Protocol（HHP）。

在这个例子中，假设 host 1 上的文件访问程序想用 RRP 提供的通信服务，向 host 2 上的对等者发送一条 message。在这种情况下，文件应用请求 RRP 代它发送这条 message。

<span style="color:#1e8449;">**[译注]** service interface 与 peer interface 的区分，正是 **CS144 Lab0 与 Lab1 的分界线**，这是本卷最值得挂的一处。Lab0 里写 `webget`，用的是 `TCPSocket` 的 `connect` / `write` / `read` / `close`——这些是 **service interface**：本地调用、同步返回、失败抛异常。Lab1 之后你实现 `TCPSender`，产出的是要放到线上去的 `TCPSegment`（含 `seqno`、`SYN`、`FIN`、`window size`）——这些是 **peer interface**：必须与远端**另一份独立实现**逐字节一致才能互通。

**可判定的判据**：判断你正在改的代码属于哪一侧，看它能否在**不改变线上字节**的前提下被替换。改 `webget` 的调用方式不影响任何远端；改 `TCPSegment` 的字段编码就立刻与远端不兼容。这条判据解释了两件事：为什么 peer interface 那一侧的测试断言的是「解析出的字节序列」，而 service interface 那一侧只用返回值断言；为什么 Lab4 的 `TCPConnection` 必须同时消费两侧（对上暴露 `read`/`write`，对下产出 segment）。

**消失测试**：抽掉这个区分，最常见的错误是把 service interface 的「同步阻塞」语义带进 peer interface——例如让 `TCPSender::fill_window()` 阻塞等 ACK。可观测后果是 Lab3 的超时测试挂死（`tick()` 永远等不到推进），因为没有任何线程在推进时间。

**落点**：§1.3.1（本节）；§5.2 的 TCP 报文格式（书页 233）；CS144 Lab0（`TCPSocket`/`webget`）、Lab1/Lab3（`TCPSegment` 的序列化）；跨学科坐标是 CSAPP 第 11 章「socket API 是内核提供给用户态的服务接口」。</span>

<span style="color:#1e8449;">**[译注]** 「多种抽象并列在同一层」纠正的是「层 = 线性序列」这一误读，技术后果很具体：**同一层可以有多条并列协议，它们是「选择」关系而不是「上下」关系**。原文用 "alternative offerings" 表达并列，但没给判据；判据是：**并列的两个实体之间不存在数据依赖**——一个不需要另一个的输出来工作。

**可判定的判据**：若 A 的实现需要 B 的输出，A 在 B 之上；若两者各自独立地使用同一组下层服务，则并列。用这条判据检查图 1.9：RRP 与 MSP 都用 HHP，互不调用 → 并列。同样的判据在 Internet 里给出 TCP 与 UDP 并列（都直接用 IP，互不调用），这就是为什么 §5.1 与 §5.2 是两节平行内容，而不是递进关系。

**消失测试**：把并列误读成线性，会得出「TCP 建立在 UDP 之上」或「UDP 是 TCP 的简化版」这类错误结论；可观测后果是：在 Lab4 里试图用 UDP 的接口去实现 TCP 的连接管理，而 lab 提供的 `TCPSocket` 根本没有 datagram 语义。

**落点**：§5.1 与 §5.2（书页 230、233）；图 1.14（书页 29）把 TCP 与 UDP 画成同一层的两个并列框；CS144 Lab4 的 `TCPConnection` 与 Lab5 的 `NetworkInterface` 之间的层界。</span>

---

<!-- ===== p028 ===== -->
> 来源注：PDF 第 28 页 = 书页 24（页脚 `24 Chapter 1. Foundation`）。本页为整页插图，含两张图。

[Figure 1.10：**服务接口与对等接口**——同一台机器上的上层对象通过 service interface 调用某个 protocol；该 protocol 的另一份实例在远端机器上，两者之间通过 peer interface 交换 message；每个 protocol 还向下使用更低层 protocol 的 service interface。图中两条方向不同的关系（向上=服务、横向跨机器=对等）是分层的两种接口。]

[Figure 1.11：**一个协议图的例子**——三个节点：RRP（Request/Reply Protocol）与 MSP（Message Stream Protocol）并列在上，两者各有一条 depends-on 边指向下方的 HHP（Host-to-Host Protocol）。图演示两件事：(a) 同一层可以并列两个协议；(b) 一个下层协议可被多个上层协议依赖，因此结构是有向无环图而不是一条链。]

> **无法复原说明**：本页文字层只有两个图题（`Figure 1.10.: Service interfaces and peer interfaces.` 与 `Figure 1.11.: Example of a protocol graph.`）。节点位置、边的箭头方向、是否画出 host 边界框均未进入文字层，故不描述。**可核的部分**来自正文（书页 23）：service interface 向上、peer interface 横向；RRP 与 MSP 都依赖 HHP，文件应用使用 RRP/HHP 这个 stack。

<span style="color:#1e8449;">**[译注]** 图 1.11 的三节点结构可以当成**最小反例**来记：它用最少的元素否证「分层 = 线性栈」。否证过程可以写得很干净——若分层真是链，则每个协议的上层必须唯一；图中 HHP 有两个上层（RRP、MSP），矛盾。因此正确结构只能是：**协议图 = 有向无环图，边是 depends-on，分层是它的特例（入度与出度都 $\le 1$ 时）**。

**可算的判据**：DAG 上「从应用到线路」的路径数决定封装的确定性。图中从 RRP 到 HHP 只有 1 条路径，封装层数确定；若某协议图里从应用到链路有 $k$ 条路径（例如 IPv4 与 IPv6 双栈，$k = 2$），同一份应用数据就有 $k$ 种封装方式，规范必须说明如何选择——这正是双栈实现里「按目的地址决定用哪条路径」的由来。

**消失测试**：抽掉「DAG 而非链」这个认识，双栈主机无法被描述，工程师会以为「IP 层有两个并行实例」，进而在 Lab5 里写不出按 EtherType 分派（0x0800 IPv4 / 0x86DD IPv6）的代码。

**落点**：§4.2 IP Version 6（书页 190）的双栈与隧道；CS144 Lab5 的 `NetworkInterface` 需要按 EtherType 分派；跨学科坐标是 6.042J 的 DAG 与路径计数（拓扑序保证封装顺序无歧义）。</span>

<span style="color:#1e8449;">**[译注]** 两张图排在同一页并非偶然：图 1.10 是**单个协议的接口分解**（一个 protocol 的两侧），图 1.11 是**多个协议的关系分解**（协议之间的边）。读图顺序应当是「先 1.10 再看 1.11」——因为 1.11 里每条红线（depends-on）在 1.10 里都是「下层 protocol 的 service interface」，如果不先建立 1.10 的接口概念，1.11 的边就只是线而不是语义。

**可判定的判据**：判断一张协议图是否完整，看它是否隐含了**未画出的接口**。1.11 只画了 RRP→HHP 与 MSP→HHP，没有画 peer interface（那是跨主机的水平关系，不属于单机的协议图）——所以 1.11 的边都是「垂直的服务依赖」，没有任何一条是「水平的对等契约」。这个区分对 lab 有直接用处：`TCPConnection` 内部的 `TCPSender`/`TCPReceiver` 是垂直依赖（服务调用），而 `TCPSender` 与远端 `TCPReceiver` 之间是水平契约（线上格式）。

**消失测试**：若不区分「垂直线」与「水平契约」，你会把 lab 里 `TCPSender` 与 `TCPReceiver` 画进同一张协议图并按「依赖」连线；实际上它们分处两台主机，互相之间没有任何函数调用。

**落点**：§1.3.1（本节）与 §1.3.2（书页 25）；CS144 Lab2/Lab3/Lab4 的模块划分（`TCPReceiver` 与 `TCPSender` 是同机两个对象，peer 在远端）。</span>

---

<!-- ===== p029 ===== -->
> 来源注：PDF 第 29 页 = 书页 25（页脚 `1.3. Architecture 25`）。本页含 RRP/HHP 示例续、protocol specification 的讨论、标准组织、以及 §1.3.2 Encapsulation 全节。

为了与自己的对等者通信，RRP 调用 HHP 的服务，HHP 又把 message 传送给另一台机器上的对等者。一旦 message 到达 host 2 上 HHP 的实例，HHP 就把 message 向上交给 RRP，RRP 再把它投递给文件应用。在这个特定情形中，我们说该应用使用了 **RRP/HHP protocol stack（协议栈）** 的服务。

注意 **protocol** 这个术语有两种用法。有时它指抽象的接口——即由 service interface 定义的操作，以及在对等者之间交换的 message 的形式与含义；有时它指真正实现这两个接口的**模块**。为了区分接口与实现接口的模块，我们一般把前者称为 **protocol specification（协议规范）**。规范通常用散文、伪代码（pseudocode）、状态转移图（state transition diagram）、packet 格式图以及其他抽象记法的组合来表达。应当做到：一个给定的 protocol 可以被不同的程序员以不同方式实现，只要各自遵守规范。挑战在于确保同一规范的两个不同实现能够成功交换 message。两个或多个**确实**正确实现了某个 protocol specification 的协议模块，被说成彼此 **interoperate（互操作）**。

我们可以想象许多不同的协议与协议图，满足一组应用的通信要求。幸运的是，存在标准组织，例如 **Internet Engineering Task Force（IETF）** 与 **International Standards Organization（ISO）**，为某个特定的协议图确立政策。我们把管辖一个协议图的形式与内容的那套规则称为 **network architecture（网络架构）**。虽然超出本书范围，标准组织已经为「在其各自的架构中引入、验证并最终批准协议」建立了定义明确的程序。我们稍后简要描述 IETF 与 ISO 定义的架构，但首先还有两件关于协议分层机制的事需要解释。

#### 1.3.2 Encapsulation（封装）

考虑当某个应用程序把一条 message 交给 RRP、请它发送给对等者时发生了什么。从 RRP 的视角看，应用交给它的 message 是一串**未经解释的字节**。RRP 并不关心这些字节表示一个整数数组、一封 email、一张数字图像还是别的什么；它只是被委以「把这些字节发给对等者」的任务。然而，RRP 必须向对等者传达控制信息，指示它在收到 message 时如何处理。RRP 通过在 message 上附加一个 **header（首部）** 来做这件事。一般说来，header 是一个小的数据结构——从几字节到几十字节——用于对等者之间相互通信。顾名思义，header 通常附在 message 的**前面**。不过在某些情况下，这种对等者之间的控制信息被放在 message 的**末尾**，此时它被称为 **trailer（尾部）**。RRP 所附 header 的确切格式由它的 protocol specification 定义。message 的其余部分——即为应用而传输的数据——称为该 message 的 **body（主体）** 或 **payload（载荷）**。我们说应用的数据被**封装（encapsulated）** 在 RRP 创建的新 message 中。

这个封装过程随后在协议图的每一层重复；例如，HHP 通过附上自己的 header 来封装 RRP 的 message。若我们现在假设 HHP 通过某个网络把 message 发给它的对等者，那么当 message 到达目的 host 时，处理顺序相反：HHP 先解释 message 前面的 HHP header（即依据 header 内容采取适当行动），并把 message 的主体（**不含** HHP header）向上交给 RRP；RRP 依据其对等者所附的 RRP header 采取相应行动，并把 message 的主体（**不含** RRP header）向上交给应用程序。从 RRP 向上交给 host 2 上应用的这条 message，与 host 1 上应用交给 RRP 的那条 message **完全相同**；应用看不到任何为实现低层通信服务而附加的 header。

<span style="color:#1e8449;">**[译注]** 封装必须算一次**字节预算**，否则它永远只是动作名。以以太网承载 TCP 为例：应用数据加 TCP header 20 B、加 IP header 20 B、加 Ethernet header 14 B 与尾部 FCS 4 B。若链路 MTU（payload 上限）是 1500 B，则应用层单次可送

```math
\text{MSS} = 1500 - 20\ (\text{IP}) - 20\ (\text{TCP}) = 1460\ \text{B},
```

头部开销比例 $(20+20+14+4)/1518 \approx 3.8\%$。这个数字决定 TCP 的核心参数：窗口要能装下 $B\cdot\text{RTT}$ 才不浪费带宽。$B = 1$ Gbps、RTT $= 100$ ms 时 $B\cdot\text{RTT} = 12.5$ MB，需要约 8560 个未确认段；而 TCP 的 16 位窗口字段最大表示 65535 B，**差 190 倍**——这就是 window scaling 选项（RFC 7323）存在的全部理由，也是 CS144 Lab2 `window_size()` 返回 `uint16_t` 时你必须知道的天花板。

**消失测试**：不算这笔预算，Lab5 的 `NetworkInterface::send_datagram()` 会忘记按 MTU 切分（或让帧超过 1514 B），可观测现象是回环测试通过、对端解析出错（帧被截断）。

**可判定的判据**：一条链路上「应用数据占线路字节的比例」就是这套预算的体检指标。它在报文越小时越差：$P = 40$ B 时 $(20+20+14+4)/(40+58) \approx 59\%$，即一半以上的线路带宽在传 header——这是 Nagle 算法与「小包合并」存在的经济理由，也是 VoIP 必须做头部压缩的原因。

**落点**：§1.3.2（本节）；§5.2 的 window scaling；CS144 Lab5 的 `MTU = 1500` 与 `send_datagram`；一手坐标是 RFC 9293 与 RFC 7323。</span>

<span style="color:#1e8449;">**[译注]** 「specification vs 实现它的模块」这条区分是本页唯一能被**实验检验**的命题，检验方式就是互操作测试。判据很硬：两份实现是否 interop，取决于它们对**规范未规定之处**的选择是否偶然一致。规范里总有留白——TCP 的初始序号选择、重传计时器的起算点、window scaling 的协商时机、`TIME_WAIT` 的时长——两份实现对留白的不同选择会表现为「各自与自己对测通过、互相不通」。

**可判定的判据**：写一份新实现时，先列出「规范未规定」的清单，再决定每一项的默认值；这份清单就是 interop 的风险表。CS144 的 peer 由 `libsponge/tcp_helpers` 提供，它的留白选择已被测试固定，因此你的实现必须迎合它，而不是「按课本最自然的写法」。

**消失测试**：没有 specification/implementation 之分，你会直接照抄课本伪代码，然后在 lab 的 `listen`/`connect` 时序上与测试期望差一条 segment；可观测现象是所有自测通过、集成测试第一项就失败。

**落点**：§5.2（书页 233）与 §2.5（书页 65）；CS144 `libsponge/util/tcp_minnow_socket` 与测试脚本；一手来源是 RFC 9293 的 "implementation notes" 小节——那里专列规范未规定、各实现自由选择之处。</span>

---

<!-- ===== p030 ===== -->
> 来源注：PDF 第 30 页 = 书页 26（页脚 `26 Chapter 1. Foundation`）。本页为整页插图。

[Figure 1.12：**高层 message 被封装在低层 message 之内**——自上而下三个矩形套嵌：应用数据被包进 RRP message（加 RRP header），再被包进 HHP message（加 HHP header）；到接收端自下而上逐层剥掉 header，最终交回一份与原应用数据相同的 message。图演示「同一条 message 在每一层都绑定一个不同的 header」。]

> **无法复原说明**：本页文字层只有图题 `Figure 1.12.: High-level messages are encapsulated inside of low-level messages.`。套嵌矩形的高度比例、header 的宽度示意、箭头方向均未进入文字层，无法复原。**可核的部分**来自正文（书页 25–26）：封装自下而上剥除，最终交付给应用的内容与原始 message 完全相同；中间节点（switch、router）可能检查 HHP header。

<span style="color:#1e8449;">**[译注]** 这张图最该追问的是它**没有画**的东西：中间节点只拆到某一层就停。图只画了两端，读者极易以为「每个节点都拆到顶」。实际规则是：switch 拆到 link layer（读目的 MAC），router 拆到 network layer（读目的 IP），host 才拆到 application layer。这个「拆到哪一层」的差别是分层设计的核心，也是 CS144 Lab5 与 Lab6 的分工：Lab5 的网络接口生成/解析 Ethernet frame，Lab6 的路由器解析 IP header 但**不碰** TCP header。

**可判定的判据**：一个节点拆到哪一层，等价于「它需要读哪个 header 才能完成职责」。switch 只需 MAC → link layer；router 需要目的 IP → network layer；NAT 需要目的端口 → transport layer（这正是 NAT 破坏「端到端不变性」的技术含义）。

**消失测试**：抽掉这条规则，你会以为「中间节点越多、header 被剥得越多」，进而无法解释为何 IP header 每跳都重新解析而 TCP header 全程不被触碰；可观测后果是 Lab6 里你去检查 TCP 的 `seqno`，而这在 lab 的数据结构里根本不可见。

**落点**：§3.1 Switching Basics（书页 104）与 §3.3 Internet (IP)（书页 125）；CS144 Lab5（Ethernet frame）与 Lab6（IP datagram）；§1.3.4 的「下三层在所有节点上、transport 及以上只在端主机上」（书页 29）。</span>

<span style="color:#1e8449;">**[译注]** 「header 通常在前，控制信息也可能在末尾（trailer）」这半句背后有硬件理由：**发送方在开始发送时还不知道校验值**。CRC 必须对整帧计算，流式发送时只能边发边算并把它放在最后，这就是以太网 FCS 在尾部的原因。反过来，长度、类型、地址这类「接收方一看开头就要用」的字段必须在前。判据可以立为：**接收方在收到第 1 个字节后就必须用到的字段放前面；必须等全部字节到齐才能算出的字段放后面。**

**可算的判据**：这条判据直接决定硬件实现的可能性。若把 FCS 放前面，网卡必须缓存整帧再发（store-and-forward 到硬件缓冲），延迟增加一个帧时间（1500 B @ 1 Gbps = 12 $\mu s$）；放后面则该网卡可以 cut-through（收到 6 字节目的地址即开始转发），延迟降到约 $1\ \mu s$。

**消失测试**：抽掉这条判据，你无法解释为什么 IPv4 的 header checksum 在前（只覆盖头部、可增量算）而以太网 FCS 在后（覆盖整帧）；也无法解释 IPv6 为何取消 header checksum（把完整性交给 transport，减少每跳重算）。

**落点**：§2.3 Framing（书页 54）与 §2.4 Error Detection（书页 59）；§3.3 的 IPv4 header 格式（书页 125）；CS144 Lab5 的 `HEADER_SIZE = 14`（FCS 不计入 payload）。</span>

---

<!-- ===== p031 ===== -->
> 来源注：PDF 第 31 页 = 书页 27（页脚 `1.3. Architecture 27`）。本页含封装说明收尾、§1.3.3 Multiplexing and Demultiplexing、§1.3.4 起首。

注意，当我们说一个低层协议**不解释**高层协议交给它的 message 时，我们的意思是它不知道如何从 message 所含数据中提取任何含义。然而，有时低层协议会对交给它的数据施加某种简单的变换，例如压缩或加密它。在这种情况下，这个协议变换的是 message 的**整个主体**，既包括原始应用数据，也包括高层协议附加到该数据上的所有 header。

#### 1.3.3 Multiplexing and Demultiplexing（复用与解复用）

回忆 packet switching 的一个基本想法：把多条数据流复用到一条物理链路上。同一个想法在协议图中上下都适用，而不只是用于交换节点。例如在图 1.11 中，我们可以把 RRP 看作实现了一条逻辑通信通道，来自两个不同应用的 message 在源 host 上被复用到这条通道上，然后在目的 host 上被解复用（demultiplexed）回适当的应用。

实际地说，这仅仅意味着 RRP 附在它 message 上的 header 中含有一个标识符，记录该 message 属于哪个应用。我们把这个标识符称为 RRP 的 **demultiplexing key（解复用键）**，简称 **demux key**。在源 host 上，RRP 在它的 header 中包含适当的 demux key。当 message 在目的 host 上被交给 RRP 时，RRP 剥掉自己的 header，检查 demux key，并把 message 解复用给正确的应用。

RRP 在支持复用方面并不特殊；几乎每个协议都实现这一机制。例如 HHP 有自己的 demux key，用来决定哪些 message 向上交给 RRP、哪些交给 MSP。不过，各协议之间——甚至在同一网络架构之内的协议之间——对于「demux key 究竟由什么构成」没有统一约定。有些协议使用 **8 位字段**（意味着它们只能支持 256 个高层协议），另一些使用 **16 位或 32 位**字段。此外，有些协议的 header 中只有一个解复用字段，另一些则有两个解复用字段。在前一种情况下，通信双方使用**相同**的 demux key；在后一种情况下，每一方用**不同**的 key 来标识 message 应被投递给哪个高层协议（或应用程序）。

#### 1.3.4 7-Layer OSI Model（七层 OSI 模型）

ISO 是最早正式定义「连接计算机的通用方式」的组织之一。他们的架构称为 **Open Systems Interconnection（OSI，开放系统互连）** 架构，如图 1.13 所示，它把网络功能划分为**七层**，其中一层或多层协议实现分配给某一层的功能。在这个意义上，图 1.13 给出的示意图本身不是一个协议图，而是一个协议图的**参考模型（reference model）**。它常被称为 **7-layer model（七层模型）**。虽然今天没有任何基于 OSI 的网络在运行，但它所定义的术语仍被广泛使用，因此仍值得粗略一看。

从最底层开始向上：**physical layer（物理层）** 处理在一条通信链路上传输原始比特。**data link layer（数据链路层）** 随后把比特流汇集成一个更大的聚合体，称为 **frame（帧）**。网络适配器（network adaptor）以及运行在节点操作系统中的设备驱动程序（device driver）通常实现 data link 层。

<span style="color:#1e8449;">**[译注]** 「8 位 → 只能 256 个高层协议」这句话在今天的 Internet 里有一个精确对应物：**TCP/UDP 的端口号是 16 位，理论上限 65535**。这个上界不是抽象的：Linux 的临时端口默认范围是 32768–60999（约 2.8 万个），于是单台机器到同一目的地址+端口的并发出向连接数上限就在这个量级——压测时会真实撞上，报 `Cannot assign requested address`（EADDRNOTAVAIL）。想突破只能加源 IP 或缩短 `TIME_WAIT`。

**可算的判据**：并发连接上限 $\approx$ 临时端口数 $\times$ 源地址数。要压到 10 万连接，单源 IP 做不到，至少需要 4 个源地址。这条算术是「demux key 宽度 = 系统容量上限」的直接实例，也解释了原文为何强调 8 位与 16 位的差别——那是容量，不是风格。

**消失测试**：抽掉这层理解，Lab4 里会把 `TCPSocket` 的本地端口当成随机数；可观测现象是同一台机器开两个测试实例时端口冲突（`bind: Address already in use`），或压测时连接数上不去。

**落点**：§5.1（书页 230，UDP 端口）、§5.2（书页 233）；CS144 Lab4 的 `TCPSocket` 与 `bind`/`connect`；§3.3 的 IP header `protocol` 字段（8 位，与本节「8 位」同构）。</span>

<span style="color:#1e8449;">**[译注]** 「有些协议一个 demux 字段、有些两个」这个观察在 TCP 里有非常具体的后果：**TCP 的 demux key 是一对（源端口, 目的端口）**，UDP 相同，而 IP 只有单字段（protocol 号）。为什么传输层必须用一对？因为**服务端只有一个知名端口（如 80），客户端有成千上万条连接**；若只用目的端口，所有到 80 的连接会被解复用到同一个应用实例，无法区分。用一对就能把「四个客户端同时连 80」区分开，代价是每条连接多 16 位。

**可算的判据**：连接的唯一标识是五元组（源 IP、源端口、目的 IP、目的端口、protocol）。它的地址空间是 $2^{32}\times2^{16}\times2^{32}\times2^{16}\times2^{8}$，但实际约束不是地址空间，而是**端口耗尽**与**连接表大小**。

**消失测试**：抽掉「一对端口」的设计，§5.2 的 TCP 无法支持多并发连接；可观测现象是第二个客户端连入时数据被投递给第一个连接（串话），这正是 lab 的 `connect`/`accept` 测试要抓的错。

**落点**：§5.2 的 TCP header 与四元组（书页 233）；CS144 Lab4 的 `TCPConnection` 按四元组区分连接；§9.1 的 HTTP keep-alive 在同一四元组内复用（书页 416）。</span>

---

<!-- ===== p032 ===== -->
> 来源注：PDF 第 32 页 = 书页 28（页脚 `28 Chapter 1. Foundation`）。本页为整页插图。

[Figure 1.13：**OSI 七层模型**——自下而上七层：physical、data link、network、transport、session、presentation、application；每层标出该层处理的数据单元（bit / frame / packet / message）。图演示七层的相对位置与各层的数据单元名称。]

> **无法复原说明**：本页文字层只有图题 `Figure 1.13.: The OSI 7-layer model.`。七层的具体画法（是否画成两台主机的镜像、是否标注协议实例）未进入文字层，无法复原。**可核的部分**来自正文（书页 27–29）：physical 处理 raw bit；data link 把比特流汇成 frame，由网卡与驱动实现；network 处理 packet-switched 网络中的 routing；transport 实现 process-to-process channel；application 的例子是 HTTP；presentation 关心数据格式；session 提供名字空间以串接同一应用的多条 transport 流。

<span style="color:#1e8449;">**[译注]** 原文那句「今天没有任何基于 OSI 的网络在运行」是本节最硬的事实陈述，也是**唯一**能省时间的判据：OSI 七层不是待实现的设计，而是一套**术语来源**。可检验的形式是——OSI 自家的协议栈（CLNP、TP4、X.400）在今天的公开 Internet 上已无部署，而 ISO 7498 定义的层名仍出现在每一份网络文档里。因此正确读法是「记住词、不记结构」，不要试图把 TCP/IP 硬塞进七层。

**可算的方向性判断**：Internet 到 OSI 的映射是 application→7、transport→4、IP→3、链路→2，即七层里有两层（presentation、session）在 Internet 里**没有对应实体**。这不是遗漏而是选择：格式转换交给应用（§7.1），会话管理交给应用（cookie、token）。代价是每个应用重写序列化与认证，收益是核心协议更少。

**消失测试**：若强行要求 Internet 有完整的 session layer，每个中间设备都可能需要维护会话状态，端到端原则被破坏；可观测后果是一个会话中间的一台设备重启，所有连接都要重建——这正是「有状态防火墙」的运维痛点。

**落点**：§7.1 Presentation Formatting（书页 342，presentation 功能在 Internet 的实际去处）；§1.3.5（书页 29–30）；CS144 的实践里只有三层出现在 lab（link / network / transport）。</span>

<span style="color:#1e8449;">**[译注]** 「参考模型 vs 协议图」这个区分有一处具体证据：图里各层的数据单元名（bit、frame、packet、message）是**层间约定的叫法，不是格式**。判据是：数据单元名描述的是**该层的视角**，同一串字节在四层里各有名字——应用看 message、transport 看 segment、network 看 datagram、link 看 frame。CS144 里这套命名可核：`TCPSegment`（transport）、`IPv4Datagram`（network）、`EthernetFrame`（link）三个类型分别在 `libsponge/` 的对应头文件里，且逐层套嵌。

**消失测试**：不区分「名字是视角、不是格式」，会以为 frame 与 packet 是两种字节布局，进而在 Lab5 里对 datagram 多复制一次；实际关系是同一个字节序列在不同层的视图（frame 的 payload 就是 datagram 的完整字节）。可观测的代价是每包多一次 `memcpy`：1500 B 在 1 Gbps 下是 $12\ \mu s$ 的额外占用，而该链路的包间隔恰好也是 $12\ \mu s$——一次多余拷贝就把转发能力砍半。

**落点**：§1.3.2（书页 25）与 §1.3.4（本节）；CS144 Lab5 中 `EthernetFrame::payload()` 返回的正是 `IPv4Datagram` 的字节；§2.3 Framing（书页 54）；跨学科坐标是 6.042J 的拓扑序（封装顺序必须与依赖顺序一致，否则视图之间没有唯一的套嵌关系）。</span>

---

<!-- ===== p033 ===== -->
> 来源注：PDF 第 33 页 = 书页 29（页脚 `1.3. Architecture 29`）。本页含七层描述的其余部分与 §1.3.5 Internet Architecture 起首、Figure 1.14。

这意味着实际交付给 host 的是 frame，而不是原始比特。**network layer（网络层）** 处理 packet-switched 网络中节点之间的 routing。在这一层，节点之间交换的数据单元通常称为 **packet** 而不是 frame，尽管它们根本上是同一个东西。**下面三层在所有网络节点上实现，包括网络内部的 switch 与连在网络外部的 host。** 接着，**transport layer（传输层）** 实现我们到此为止一直称为 process-to-process channel 的东西。在这里，所交换的数据单元通常称为 **message**，而不是 packet 或 frame。**transport layer 及更高层通常只在端主机上运行，而不在中间的 switch 或 router 上运行。**

跳到最上面（第七）层并向下走，我们看到 **application layer（应用层）**。应用层协议包括像 **Hypertext Transfer Protocol（HTTP）** 这样的东西——它是 World Wide Web 的基础，也是让 Web 浏览器能够向 Web 服务器请求页面的东西。在它下面是 **presentation layer（表示层）**，它关心对等者之间交换的数据的**格式**——例如一个整数是 16、32 还是 64 位长，最高有效字节（most significant byte）是先传还是后传，或者一条视频流如何被格式化。最后，**session layer（会话层）** 提供一个名字空间（name space），用于把属于同一应用的那些**可能不同的** transport 流串接在一起。例如，它可能管理一个电话会议应用中被合并在一起的一路音频流与一路视频流。

#### 1.3.5 Internet Architecture（Internet 架构）

Internet 架构有时也以其两个主要协议命名为 **TCP/IP architecture**，如图 1.14 所示。另一种表示见图 1.15。Internet 架构是从一个更早的 packet-switched 网络 **ARPANET** 的经验中演化出来的。Internet 与 ARPANET 都由 **Advanced Research Projects Agency（ARPA）** 资助，它是美国国防部的研究与开发资助机构之一。Internet 与 ARPANET 早于 OSI 架构出现，而从建造它们获得的经验对 OSI 参考模型产生了重大影响。

[Figure 1.14：**Internet 协议图**——自上而下：application protocols（HTTP、FTP、Telnet、SMTP 等）→ transport（TCP 与 UDP 并列）→ IP（单一）→ 多种 network protocols（NET1、NET2、……）。图的形状是宽—窄—宽的沙漏，窄腰处只有 IP。]

> **部分可复原**：图题进入文字层，正文给出每层的构成，因此结构可核；各框的具体位置与连线方式未进入文字层，无法复原。

虽然七层 OSI 模型外加一些想象可以套用到 Internet 上，但通常改用一套更简单的栈。最低层是各种各样的 network protocols，记作 NET1、NET2 等等。在实践中，这些协议由硬件（例如网络适配器）与软件（例如网络设备驱动程序）的组合来实现。例如，你可能在这一层看到 Ethernet 或无线协议（例如 802.11 Wi-Fi 标准）。（这些协议本身可能又涉及若干子层，但 Internet 架构对它们不做任何预设。）下一层由**单个**协议构成——**Internet Protocol（IP）**。正是这个协议支持把多种网络技术互连成一个单一的、逻辑的 internetwork。IP 之上的层包含两个主要协议——**Transmission Control Protocol（TCP）** 与 **User Datagram Protocol（UDP）**。TCP 与 UDP 向应用程序提供可选的逻辑通道：**TCP 提供可靠的字节流通道（reliable byte-stream channel），UDP 提供不可靠的数据报投递通道（unreliable datagram delivery channel）**（datagram 可以看作 message 的同义词）。用 Internet 的语言来说，TCP 与 UDP 有时被称为 **end-to-end protocols（端到端协议）**，尽管把它们称为 **transport protocols** 同样正确。

<span style="color:#1e8449;">**[译注]** 「下三层在所有节点上实现，transport 及以上只在端主机上实现」这一句是整章的**结构性断言**，直接推出一条 CS144 必须遵守的边界：**路由器里没有 TCP**。这不是难度问题而是状态问题——把 transport 放进路由器，就要为每条穿过的连接维护序号与重传缓冲，内存是 $O(\text{并发连接数})$；骨干路由器每秒新建连接可达百万级，任何固定内存都撑不住。

**可算的判据**：设单连接 transport 状态 $s$ 字节（Linux 的 TCP 控制块约 1–2 KB）、并发连接 $C$、核心网 $N$ 台设备，则把 transport 放进路由器的额外内存是 $N\cdot C\cdot s$；$N = 1000$、$C = 10^{6}$、$s = 1$ KB 时是 $10^{12}$ B = 1 TB。这是「端到端」在工程上的量化版本。

**消失测试**：抽掉这条边界，可扩展性论证全部失效；具体失效现象是路由器成为状态爆炸点，任何一台失效都会带走所有经过它的连接（而不是让它们各自重传恢复）。

**落点**：§5.2（书页 233）把可靠传输放在端主机；CS144 **Lab6 的 `Router` 只有转发表与队列，没有连接概念**；一手来源是 Saltzer/Reed/Clark 1984（A 级）；跨学科坐标是 CSAPP 第 11 章「内核维护 socket 状态、网卡不维护」。</span>

<span style="color:#1e8449;">**[译注]** 沙漏的窄腰值得用一个算术事实来记：**它把 $M\times N$ 的适配问题变成 $M+N$**。设上层有 $M$ 种 transport、下层有 $N$ 种链路技术；若无公共层，每种 transport 都要为每种链路实现一遍，共 $M N$ 份适配代码；有 IP 之后只需 $M$ 份（上行）+ $N$ 份（下行）。以 $M = 4$（TCP、UDP、SCTP、QUIC）、$N = 10$（Ethernet、Wi-Fi、PPP、5G、PON……）计，从 40 份降到 14 份；$M, N$ 越大，收益越接近 $MN \to M+N$ 的量级差。反过来说，$M$ 或 $N$ 很小时窄腰的收益也小——这正是嵌入式专网（$M=1$、$N=1$）不必上 IP 的原因。

**消失测试**：抽掉窄腰，QUIC 想跑 5G 就要单独适配 5G、跑 Wi-Fi 又要单独适配 Wi-Fi；可观测后果是代码量按 $M\times N$ 爆炸，且新链路技术上线时所有上层都要改。

**落点**：§1.3.5（本节）；§3.3 Internet (IP)（书页 125）；§4.2 IPv6（书页 190，扩窄腰的尝试）；CS144 Lab5 的 `NetworkInterface` 正是「$N$ 种链路 → 一个 IP datagram 接口」那一侧；跨学科坐标是 6.042J 的笛卡尔积基数 $|A\times B| = |A|\cdot|B|$ 与并集基数 $|A\cup B| \le |A|+|B|$ 之差（乘法与加法之别）。</span>

---

<!-- ===== p034 ===== -->
> 来源注：PDF 第 34 页 = 书页 30（页脚 `30 Chapter 1. Foundation`）。本页含 Figure 1.15、应用协议讨论、以及 IETF 标准化插框。

[Figure 1.15：**Internet 架构的另一种视图**——用「四层 + 一个 subnetwork 层」表示；图注原文："The 'subnetwork' layer was historically referred to as the 'network' layer and is now often referred to as 'Layer 2'"（受 OSI 模型影响）。图演示同一架构的两种画法：图 1.14 强调窄腰与协议并列，图 1.15 强调层号与历史命名。]

> **无法复原说明**：本页文字层只有图题与其图注。层次框的排布与层号标注未进入文字层，无法复原。**可核的部分**来自图注本身：subnetwork 层 = 历史上的 network 层 = 今天的 Layer 2。

运行在 transport layer 之上的一系列应用协议，例如 HTTP、FTP、Telnet（远程登录）与 **Simple Mail Transfer Protocol（SMTP）**，使流行应用能够互操作。要理解应用层协议与应用之间的区别，想一想曾经或现在可用的所有不同 World Wide Web 浏览器（例如 Firefox、Chrome、Safari、Netscape、Mosaic、Internet Explorer）。Web 服务器的不同实现也同样数量庞大。你能够用这些应用程序中的**任何一个**访问 Web 上某个特定站点的原因，是它们都遵守同一个应用层协议：HTTP。令人困惑的是，同一个术语有时既指一个应用，也指它所使用的应用层协议（例如 FTP 常被用作「实现 FTP 协议的那个应用」的名字）。†

> † **术语复用（非笔误）**：本段自己承认了 FTP 的歧义（既是协议也是应用名）。这是历史遗留：SMTP 与 HTTP 对应的应用分别叫「邮件客户端」与「浏览器」，唯独 FTP 的应用没有专名，于是同名。

大多数积极从事网络领域工作的人都同时熟悉 Internet 架构与七层 OSI 架构，并且对「层如何在两个架构之间映射」有普遍共识。Internet 的 application layer 被认为在第 7 层，它的 transport layer 是第 4 层，IP（internetworking 或简称 network）层是第 3 层，而 IP 之下的 link 或 subnet 层是第 2 层。

<span style="color:#943126;">**[note]**</span> **IETF 与标准化**

> 虽然我们把它称为「Internet 架构」而不是「IETF 架构」，但可以公允地说，**IETF 是负责定义它的主要标准组织**，也是其许多协议（如 TCP、UDP、IP、DNS 与 BGP）的规范制定者。但 Internet 架构也包容许多由其他组织定义的协议，包括 IEEE 的 **802.11 ethernet** 与 Wi-Fi 标准、W3C 的 HTTP/HTML Web 规范、3GPP 的 4G 与 5G 蜂窝网络标准，以及 ITU-T 的 **H.232** 视频编码标准，仅举几例。†

> † **两处错误**：(1) **IEEE 802.11 是 Wi-Fi 标准，不是 ethernet 标准**——以太网是 IEEE 802.3；把 802.11 与 "ethernet" 并列为 "802.11 ethernet and Wi-Fi standards" 混淆了两个不同工作组。(2) **H.232 应为 H.323**（ITU-T 的 IP 语音/视频协议族），H.232 不是既有的 ITU-T 建议号。

除了定义架构与规范协议之外，还有其他组织支持互操作性这个更大的目标。一个例子是 **IANA（Internet Assigned Numbers Authority，互联网号码分配局）**……

<span style="color:#1e8449;">**[译注]** 图 1.15 的图注那句「subnetwork 层历史上被称为 network 层，今天常被称为 Layer 2」是全书**最有诊断价值**的一句话，因为「Layer 2」今天被用得像口头禅，而所指一直在漂。判据：**听到「Layer 2」先问是哪一个**——OSI 的 Layer 2 是 data link layer（frame 的层）；而工程口语里「L2 网络」常指「一个广播域内的交换式以太网」，「L3 网络」指跨子网路由。同一台设备可以同时被称为 L2 与 L3 设备，因为它在两个抽象上都成立。

**可算的后果**：术语漂移会导致 MTU/帧长约定出错。CS144 Lab5 的 `EthernetFrame` 最大长度 1514 B（14 B header + 1500 B payload，FCS 不计入 payload）；若按「L2 = 以太网 = 1500」去推 L3 的 MTU，会把 IP datagram 上限算成 1500 而漏掉 20 B header，结果发出 1520 B 的帧超限。

**消失测试**：抽掉「同一词在不同语境指不同层」这个认识，读第 3 章时会以为 switched Ethernet 与 Internet 是同一层的两种技术，从而无法理解 §3.2 与 §3.3 为何分成两节。

**落点**：§3.2 Switched Ethernet（书页 116）与 §3.3 Internet (IP)（书页 125）；CS144 Lab5 的 `EthernetFrame` 与 MTU 常量；跨学科坐标是 CSAPP 里「同一名词在不同抽象层指不同对象」（如 cache）的一般现象。</span>

<span style="color:#1e8449;">**[译注]** IETF 插框里的「其他组织」名单本身就是一张**证据分级表**：IEEE 802.3/802.11（链路层）、W3C（HTTP/HTML）、3GPP（4G/5G）、ITU-T（H.323），全部是各自领域的 A 级一手标准。它想说明的是 Internet 架构**不拥有**自己的底层与上层，只拥有中间的 IP 与 transport。这对阅读有一个直接操作：凡涉及链路层细节，去 IEEE 文档；凡涉及 HTTP 语义，去 RFC/W3C；本书在这些地方只是 B 级综述。

**可判定的判据**：判断一句话该信谁，看它是否落在该组织的**管辖权**内。IETF 说「TCP 的 RTO 怎么估」是权威（RFC 6298），IETF 说「以太网帧间隔是 96 bit-time」只是转述（那是 IEEE 802.3 的地盘）。

**消失测试**：不做这个区分，遇到 802.11 与以太网的 MTU 差异（1500 vs 2304）时会以为其中一个是错的；实际上它们由不同组织规定，互不约束。

**落点**：§2.1 Technology Landscape（书页 48）与 §2.7 Wireless（书页 83）；一手坐标：IEEE 802.3、IEEE 802.11、ITU-T H.323、3GPP TS 23/38 系列。</span>

---

<!-- ===== p035 ===== -->
> 来源注：PDF 第 35 页 = 书页 31（页脚 `1.4. Software 31`）。本页含 IANA/ICANN 收尾、Internet 架构三特征、David Clark 引语、Key Takeaway，以及 §1.4 Software 起首。

……正如它的名字所暗示的（原文误拼作 "impies"，应为 implies †），它负责发放使协议得以工作的那些唯一标识符。IANA 反过来又是 **ICANN（Internet Corporation for Assigned Names and Numbers）** 下属的一个部门——后者是一个非营利组织，负责 Internet 的整体管理（原文误拼作 "Internt Corporation" †）。

> † 两处拼写错误：`impies` → `implies`；`Internt` → `Internet`。语义不受影响，但可作为「Release 6.1 未经最终校对」的又一证据（同页另有 `ulikely`、`essense`，见书页 18 与 21）。

Internet 架构有三个值得强调的特征。第一，正如图 1.15 最好地展示的那样，**Internet 架构并不隐含严格的分层**。应用可以自由地绕过已定义的 transport 层，直接使用 IP 或某个底层网络。事实上，程序员可以自由地定义新的通道抽象或应用，运行在任何既有协议之上。

第二，如果你仔细看图 1.14 的协议图，会注意到一个**沙漏形（hourglass shape）**——上面宽、中间窄、下面宽。这个形状实际上反映了该架构的中心哲学。也就是说，**IP 充当该架构的焦点**——它定义了一种在大量网络之间交换 packet 的通用方法。IP 之上可以有任意多的 transport 协议，每一个向应用程序提供不同的通道抽象。于是，「把 message 从 host 送到 host」这个问题，与「提供一个有用的 process-to-process 通信服务」这个问题被**完全分开**。IP 之下，架构允许任意多种不同的网络技术，从 Ethernet 到无线，到单条 point-to-point 链路。

第三个属性（更准确地说，是 **IETF 文化**的属性）是：为了让一个新协议被正式纳入架构，必须既有 protocol specification，又有该规范的**至少一份**（最好是两份）代表性实现。工作实现的存在是标准被 IETF 采纳的必要条件。设计共同体的这个文化假设有助于确保架构中的协议能被高效实现。或许 Internet 文化对「能跑的软件」的重视，最好地体现在 IETF 会议上常穿的 T 恤上的一句话：

> *We reject kings, presidents, and voting. We believe in rough consensus and running code.*
> ——David Clark（我们拒绝国王、总统与投票。我们相信粗略共识与能跑的代码。）

<span style="color:#1f6feb;">**[Key Takeaway]**</span> 在 Internet 架构的这三个属性中，沙漏设计哲学重要到值得重复。沙漏的窄腰代表了**最小且精心选择的一组全局能力**，它让高层应用与低层通信技术得以共存、共享能力并快速演化。窄腰模型对 Internet 适应新用户需求与变化技术的能力至关重要。[Next]

### 1.4 Software（软件）

网络架构与协议规范是根本性的东西，但一份好的蓝图不足以解释 Internet 惊人的成功：连接到 Internet 的计算机数量已连续三十多年指数增长（尽管精确数字难以获得）。到 2018 年底，Internet 用户数估计约为 **41 亿**——大致是世界人口的一半。

什么解释了 Internet 的成功？当然有许多促成因素（包括一个好的架构），但使 Internet 如此势不可挡地成功的一件事是：它的许多功能是由运行在**通用计算机**上的软件提供的。

<span style="color:#1e8449;">**[译注]** 「不隐含严格分层」这条比它看起来更强：它给了「跨层（cross-layer）」设计以合法性，而跨层最典型的成功案例正是 CS144 学生最熟悉的方向——**应用直接读底层**。判据很具体：应用可以直接用 IP（`IPPROTO_RAW` / `SOCK_RAW`），也可以直接用以太网帧（`AF_PACKET`，即 p037 中 `socket()` 的 `PF_PACKET`）。这类操作能成立，正是因为图 1.15 的层之间没有「必须经过」的强制。

**可算的收益与代价**：跨层收益是省掉一层封装的时延与字节开销（QUIC 在用户态直接对接 UDP，省掉内核 TCP 的拷贝与调度，首字节可省 1 RTT）；代价是**破坏「下层可替换」**——直接操作以太网帧的应用无法在 Wi-Fi 上运行，因为帧格式不同。

**消失测试**：抽掉这个自由度，就不会有 `AF_PACKET`、不会有用 raw socket 实现的 ping 与抓包，也不会有 QUIC 的快速部署（它必须绕过内核 TCP 才能在新版本里快速迭代）。

**落点**：§5.2 的 QUIC 小节（书页 233 起）；§1.4.1 的 `socket()` 三参数（书页 33）；CS144 Lab5（`NetworkInterface` 就是被绕过的那个「下层」）；一手坐标是 Linux `packet(7)` 手册页与 RFC 9000。</span>

<span style="color:#1e8449;">**[译注]** 「至少一份、最好两份实现」这条 IETF 规矩是**工程上最可检验的一条文化约束**，作用是把「规范是否可实现」的风险前移到标准化之前。判据：一份规范若只有一份实现，就无法证明自己无歧义——唯一实现可以自由选择规范留白处而不暴露冲突（见 p029 关于 specification 与 implementation 的注）。两份独立实现的 interop test 才能暴露留白。

**可算的历史证据**：TCP 的规范（RFC 793，1981）发布时已有多个实现（BSD、BBN），这是它能在 ARPANET 上迅速替换 NCP 的原因；OSI 的 TP4 在标准化完成时缺少可用实现，等实现出现时 TCP/IP 已占满市场。两者相差约 10 年，是「running code」胜过「完整规范」的最强证据。

**消失测试**：抽掉这条约束，标准会倾向「理论上完备、实现上不可行」；具体失效现象是新协议标准化后三五年仍无生产级部署。

**落点**：§1.3.5（本节）；§1.4 开头对「软件提供功能」的强调（本节末）；一手坐标是 RFC 2026（Internet 标准流程）。</span>

---

<!-- ===== p036 ===== -->
> 来源注：PDF 第 36 页 = 书页 32（页脚 `32 Chapter 1. Foundation`）。本页含 §1.4 续与「Sockets Enabled Application Explosion」插框。

这个事实的意义在于：新功能可以靠「**只不过是一点编程工作**（just a small matter of programming）」就轻易加上。结果是，新应用与服务以令人难以置信的速度涌现。

一个相关的因素是商用机器（commodity machine）可用的算力大幅增长。虽然计算机网络在原则上一直能够传输任何种类的信息，例如数字语音采样、数字化图像等等，但如果收发这些数据的计算机太慢、无法用这些信息做任何有用的事，这份潜力就不那么有意思了。今天几乎所有的计算机都能以相当可用的速度与分辨率回放数字化音频与视频。

在本书第 1 版问世以来的这些年里，编写网络应用已成为一种主流活动，而不再只是少数专家的活儿。许多因素起了作用，包括让这件工作更轻松的工具，以及新市场的开辟（例如智能手机应用）。

要指出的重点是：知道如何实现网络软件是理解计算机网络的一个本质部分；虽然你多半不会被要求去实现像 IP 这样的低层协议，但你很可能会有理由去实现一个**应用层协议**——那个难以捉摸的「杀手级应用」，它将带来难以想象的声名与财富。为了让你起步，本节介绍在 Internet 之上实现网络应用所涉及的一些问题。典型地，这类程序同时既是一个**应用**（即设计来与用户交互），又是一个**协议**（即跨网络与对等者通信）。

#### 1.4.1 Application Programming Interface (Sockets)（应用编程接口：套接字）

实现网络应用时要起步的地方是网络导出的接口。由于大多数网络协议是软件（尤其是协议栈高处的那些），而几乎所有计算机系统都把网络协议作为操作系统的一部分来实现，因此当我们提到「网络导出的接口」时，一般指的是操作系统提供给其网络子系统的接口。这个接口常被称为 **network application programming interface（网络 API）**。

虽然每个操作系统都可以自由定义自己的网络 API（而且大多数确实这样做了），但随着时间推移，其中某些 API 得到了广泛支持；也就是说，它们被移植到了原生系统之外的操作系统上。**最初由 Unix 的 Berkeley 发行版提供的 socket 接口**就是这样：它现在几乎被所有流行操作系统支持，也是语言专属接口（例如 Java 或 Python 的 socket 库）的基础。本书所有代码示例使用 Linux 与 C——用 Linux 因为它开源，用 C 因为它仍是网络内部实现的首选语言。（C 还有一个好处：它把所有低层细节暴露出来，这对理解底层想法有帮助。）

<span style="color:#943126;">**[note]**</span> **Sockets 引爆了应用**

> Socket API 的重要性怎么强调都不为过。它定义了「运行在 Internet 之上的应用」与「Internet 如何实现的细节」之间的**分界点**。由于 Sockets 提供了一个定义良好且稳定的接口，编写 Internet 应用爆炸式地成长为一个数十亿美元的产业。从 client/server 范式的微末起点，以及 email、文件传输、远程登录这几个简单程序开始，如今每个人都能从智能手机上获得源源不断的 cloud 应用。

这一节通过回顾「一个 client 程序打开 socket 以便与 server 程序交换 message」的简洁性来打基础，但今天在 Socket API 之上已经堆叠了丰富的软件生态。这一层包括大量云端工具，它们降低了实现可扩展应用的门槛。我们在每一章都会回到 cloud 与网络的相互作用，从第 1 章末的 Perspective 一节开始。

在描述 socket 接口之前，重要的是在你的头脑中把两件事分开：**每个 protocol 提供一组特定的服务，而 API 提供一种语法**，借此在某个特定计算机系统上调用那些服务。然后，实现负责把 API 定义的那组可触可感的操作与对象，映射到 protocol 定义的那组抽象服务上。如果你把接口定义得很好，就有可能用该接口的语法去调用**许多不同**协议的服务。这种通用性当然也是 socket 接口的目标之一，尽管它远非完美。

<span style="color:#1e8449;">**[译注]** Socket API 的稳定性有一个可量化的生态后果，而这段只给了形容词。判据是**接口稳定的年数 vs 上层生态的规模**：BSD socket 的接口形态自 1983 年（4.2BSD）基本未变，四十年里其上的应用从 `telnet` 长到整个 Web 生态。这个稳定性也带来一个代价——它把一些**时代错误**固化进了接口：`sockaddr` 里 `sa_family` 与 `sa_len` 的字段顺序在不同系统上不同（BSD 有 `sa_len`，Linux 没有），`struct sockaddr_in` 的 `sin_zero` 8 字节是历史填充，`accept()` 的地址长度参数既是入参又是出参。

**可判定的判据**：一个接口是否「稳定到值得学」，看它是否被 POSIX 固化而不仅是某家实现。socket 是 POSIX.1-2001 的一部分，所以学一次可以跨 Linux/macOS/BSD 使用；而 `epoll` 不是（Linux 专属），学它只覆盖一部分系统。

**消失测试**：抽掉 socket 这层稳定接口，每个应用都要直接操作网卡或协议栈内部结构；可观测后果是：一个应用从 Linux 移植到另一系统要重写全部网络代码，这正是 socket 出现之前（各厂商私有 API：SNA、DECnet）的真实处境。

**落点**：§1.4.1（本节）与 §9.1（书页 416）；CS144 Lab0 的 `TCPSocket` 直接封装 socket API；跨学科坐标是 CSAPP 第 11 章（socket 的完整系统调用序列与错误处理）。</span>

<span style="color:#1e8449;">**[译注]** 「protocol 提供服务、API 提供语法、implementation 负责映射」这三层分离是本页最精炼的一句，它解释了 CS144 里一个常见困惑：为什么 `TCPSocket`（API 层）与你的 `TCPConnection`（implementation 层）能互换使用。`TCPSocket` 把 socket API 的语法映射到**内核**的 TCP 实现上；你的 `TCPConnection` 把同一套语法映射到**你自己写的** TCP 实现上——只要语义对齐，上层代码一行不改就能切换。这正是 lab 用 `TCPSocket` 做对照测试的原因。

**可判定的判据**：三层是否分离，看 API 层能否指向不同的 implementation。判据可以实测：lab 里把 `TCPSocket` 换成 `TCPMinnowSocket`（指向你的实现），`webget` 代码不变仍能跑；如果不能，说明你把语义细节泄漏到了 API 层。

**消失测试**：抽掉这三层分离，测试就无法做「同一测试脚本跑两套实现」的对照；可观测后果是：你只能拿自己的实现与自己比，无法判断是「符合规范」还是「两边一致地错」。

**落点**：§1.4.1（本节）；CS144 的 `TCPSocket` 与 `TCPMinnowSocket` 双实现对照；§5.2 的 TCP 规范（书页 233）；一手坐标是 POSIX 的 socket 规范与 RFC 9293。</span>

---

<!-- ===== p037 ===== -->
> 来源注：PDF 第 37 页 = 书页 33（页脚 `1.4. Software 33`）。本页含 socket 抽象、`socket()`、`bind`/`listen`/`accept` 三个操作与端口说明。

socket 接口的主要抽象，毫不意外地，就是 **socket（套接字）**。理解 socket 的一个好办法是把它想成「本地应用进程接入网络的**那个点**」。该接口定义了这些操作：创建 socket、把 socket 附着到网络上、通过 socket 发送/接收 message、关闭 socket。为简化讨论，我们把自己限制在「socket 如何与 TCP 一起使用」。

第一步是创建一个 socket，用下面这个操作完成：

```c
int socket(int domain, int type, int protocol);
```

这个操作带三个参数，原因是 socket 接口被设计得足够通用，可以支持任何底层协议族。具体地，`domain` 参数指明将要使用的协议族：`PF_INET` 表示 Internet 族，`PF_UNIX` 表示 Unix 管道设施，`PF_PACKET` 表示**直接访问网络接口**（即绕过 TCP/IP 协议栈）。`type` 参数指明通信的语义：`SOCK_STREAM` 用来表示字节流；`SOCK_DGRAM` 是另一种选择，表示面向 message 的服务，例如 UDP 提供的那种。`protocol` 参数标识将要使用的具体协议；在我们的情形里这个参数是 `UNSPEC`，因为 `PF_INET` 与 `SOCK_STREAM` 的组合已隐含 TCP。最后，`socket` 的返回值是新创建 socket 的一个**句柄（handle）**——即一个标识符，以后我们用它来指代这个 socket。它被作为参数传给后续对该 socket 的操作。

下一步取决于你是 client 还是 server。在 server 机器上，应用进程执行一次**被动打开（passive open）**——server 说自己准备好接受连接，但并不真的建立连接。server 通过调用以下三个操作来做这件事：

```c
int bind(int socket, struct sockaddr *address, int addr_len);
int listen(int socket, int backlog);
int accept(int socket, struct sockaddr *address, int *addr_len);
```

`bind` 操作，正如其名，把新建的 socket 绑定到指定的地址上。这是本地参与者——server——的网络地址。注意，当与 Internet 协议一起使用时，`address` 是一个数据结构，既包含 server 的 **IP 地址**，也包含一个 **TCP 端口号（port number）**。端口用来间接标识进程。它们是 demux key 的一种形式。端口号通常是某个特定于所提供服务的知名数字；例如，Web 服务器通常**在端口 80 上**接受连接。

<span style="color:#1e8449;">**[译注]** `socket()` 三个参数的正交性是本节最值得记的一处，因为它把「协议族 / 语义 / 具体协议」三个独立维度分开填，而这三者在很多语境里被混为一谈。判据：`PF_INET + SOCK_STREAM` 隐含 TCP，而 `PF_INET + SOCK_DGRAM` 隐含 UDP——**同一个协议族内换语义就换了协议**，这就是第三个参数常常填 `UNSPEC`（意为「由前两者推断」）的原因。

**可算的方向性判断**：`PF_PACKET` 这一维的价值是绕过整条 TCP/IP 栈。代价是你要自己构造以太网帧（14 B header + 46–1500 B payload），收益是省掉内核协议栈的分层解析——`tcpdump` 与 lab 的 `TunTapFD` 都走这条路。

**消失测试**：抽掉 `PF_PACKET`（即强制所有 socket 都过协议栈），你无法抓包、无法实现 ARP 应答器、无法在 Lab5 之下做任何「构造原始帧」的实验；本课程 Lab5 的调试几乎完全依赖 `tcpdump`，而它正是 `PF_PACKET` 的用户。

**落点**：§1.4.1（本节）；§1.3.5 的「不隐含严格分层」（书页 31）；CS144 Lab5 的 `NetworkInterface` 与 `TunTapFD`；一手坐标是 Linux `packet(7)` 与 `socket(2)` 手册页。</span>

<span style="color:#1e8449;">**[译注]** 「端口是 demux key 的一种形式」这句话把本节与书页 31 的 §1.3.3 缝在一起，也给出一个可算的容量上限：16 位端口 = 65535 个 demux 值。这条算术在 server 侧表现为「一个 IP 最多 65k 个监听端口」，在 client 侧表现为临时端口耗尽（Linux 默认 32768–60999）。

**可判定的判据**：一个服务能否用知名端口，看它是否被 IANA 分配。80 = HTTP、443 = HTTPS、22 = SSH、53 = DNS 都在 IANA 的 Service Name and Port Number Registry 里；**5432 是 PostgreSQL**（这一点与本书书页 34 的说法冲突，见 p038 的 † 标注）。

**消失测试**：抽掉「端口 = demux key」这层认识，Lab4 里会以为端口只是「给用户看的编号」，从而在 `TCPConnection` 里不按四元组分派，导致多个连接串话。

**可算的容量上限**：单机可同时保持的「到同一目的地的出向连接」$\le$ 临时端口数；Linux 的 `net.ipv4.ip_local_port_range` 默认 `32768 60999`，即 28232 个，再减去 `TIME_WAIT` 中不可复用的部分（默认 60 s）。峰值 $R$ 连接/秒、每次连接平均存活 $T$ 秒时，稳态占用 $\approx RT$，故 $R = 1000$、$T = 60$ 时必然耗尽——这是压测里最常见的假性瓶颈。

**落点**：§5.1 与 §5.2（书页 230、233）；§1.3.3 的 demux key（书页 27）；CS144 Lab4 的 `TCPSocket::bind`/`connect`；一手坐标是 IANA 的 Service Name and Port Number Registry（A 级）。</span>

---

<!-- ===== p038 ===== -->
> 来源注：PDF 第 38 页 = 书页 34（页脚 `34 Chapter 1. Foundation`）。本页含 `listen`/`accept`/`connect`/`send`/`recv` 与 §1.4.2 的 client 说明。

`listen` 操作随后定义在指定 socket 上可以有多少个连接处于**待处理（pending）** 状态。最后，`accept` 操作执行被动打开。它是一个**阻塞操作**，直到某个远端参与者已建立连接才返回；当它完成时，返回一个**新的 socket**，对应该刚刚建立的连接，而 `address` 参数中含有远端参与者的地址。注意当 `accept` 返回时，作为参数传入的那个原 socket 仍然存在、仍对应那个被动打开；它用于未来的 `accept` 调用。

在 client 机器上，应用进程执行一次**主动打开（active open）**；也就是说，它通过调用下面这**一个**操作来说明想与谁通信：

```c
int connect(int socket, struct sockaddr *address, int addr_len);
```

这个操作直到 TCP 成功建立连接才返回，此时应用便可以开始发送数据。在这种情况下，`address` 含有远端参与者的地址。在实践中，client 通常只指定远端参与者的地址，让系统填写本地信息。**server 通常在知名端口上监听 message，而 client 通常不在意自己用哪个端口**；操作系统只是选一个未使用的。

一旦连接建立，应用进程调用以下两个操作来发送与接收数据：

```c
int send(int socket, char *message, int msg_len, int flags);
int recv(int socket, char *buffer, int buf_len, int flags);
```

第一个操作在指定 socket 上发送给定的 message，第二个操作从指定 socket 接收一条 message 到给定缓冲区。两个操作都接受一组 `flags`，用来控制该操作的某些细节。

#### 1.4.2 Example Application（示例应用）

我们现在展示一个简单的 client/server 程序实现，它使用 socket 接口在一条 TCP 连接上发送 message。该程序还使用了其他 Linux 网络工具，我们随讲随介绍。我们的应用允许一台机器上的用户向另一台机器上的用户输入并发送文本。它是 Linux `talk` 程序的简化版，后者类似于即时消息应用核心的那个程序。

**Client（客户端）**

我们从 client 一侧开始，它把远端机器名作为参数。它调用 Linux 工具把这个名字翻译成远端 host 的 IP 地址。下一步是构造 socket 接口所期望的地址数据结构（`sin`）。注意这个数据结构指明我们将用该 socket 连接到 Internet（`AF_INET`）。在我们的例子里，我们用 TCP 端口 **5432** 作为知名服务器端口；这碰巧是一个**尚未被分配给任何其他 Internet 服务**的端口。† 建立连接的最后一步是调用 `socket` 与 `connect`。一旦该操作返回，连接就建立了，client 程序进入它的主循环：从标准输入读取文本并通过 socket 发送出去。

> † **事实错误**：端口 **5432/tcp 是 PostgreSQL 的注册端口**，在 IANA 的 Service Name and Port Number Registry 中长期登记为 `postgresql`。原文说它「尚未被分配给任何其他 Internet 服务」在 2019 年（本版发布日期）已不成立，在 1996 年（第 1 版）也可能不成立（PostgreSQL 自 1996 年起就用 5432）。正确做法是使用 IANA 的 **动态/私有端口段 49152–65535**，或查注册表确认未占用。这个错误的实际风险是：在装了 PostgreSQL 的机器上跑本书示例，`bind` 会直接报 `Address already in use`。

<span style="color:#1e8449;">**[译注]** `accept` 返回**新 socket** 这条语义是本节最容易被实现细节掩盖的一句话，而它正是 **CS144 Lab4 状态机的分水岭**。被动打开的原 socket 只负责「迎来」，不负责「送往」：它保持在 LISTEN 状态，每来一个新连接就 `fork` 出一个新 socket（或在新 socket 上建一个新 `TCPConnection`）。这就是为什么 `listen` 需要 `backlog` 参数——在 `accept` 被调用之前，已完成的握手只能排队，队列满则新 SYN 被丢弃（可观测现象：客户端 `connect` 超时，而服务端毫无日志）。

**可算的判据**：`backlog = 5`（本书示例值）意味着最多 5 个完成握手的连接等待被 `accept`。若服务端处理一个连接耗时 $t$、到达率 $\lambda$，队列满的概率随 $\lambda t$ 上升；$\lambda t \gt 5$ 时必然丢弃。这就是为什么生产环境要把 `backlog` 设到数百——它其实是「半连接 + 全连接队列」的容量参数。

**消失测试**：抽掉「新 socket」这条语义，把 `accept` 实现成「在原 socket 上继续对话」，则一个服务端同时只能服务一个客户端；可观测后果是第二个客户端连入后与第一个串话，且第一个连接的数据被写入第二个。

**落点**：§5.2 的三次握手与 LISTEN 状态（书页 233）；CS144 Lab4 的 `TCPConnection` 状态机（`listen`/`accept`）；CSAPP 第 11 章 11.4（`listen`/`accept` 的完整语义与 backlog）。</span>

<span style="color:#1e8449;">**[译注]** `connect` 是阻塞的，且它阻塞的时间恰好是**一个 RTT**（三次握手中 client 只需看到 SYN-ACK 即可返回，第三个 ACK 可随首个数据段捎带）。这条事实给了「首字节时间」一个可算的分解：一次不带缓存的 HTTP 请求 ≈ DNS + 1 RTT（握手）+ 1 RTT（请求/响应）+ 拆连，与书页 7 的 17 条报文账本一致。

**可判定的判据**：如果 `connect` 立刻返回而 `send` 阻塞很久，说明连接建立成功但服务端未读取（接收窗口为 0）；如果 `connect` 超时（`ETIMEDOUT`），说明 SYN 无人应答——两者完全不同，诊断必须分开。这个区分是 CS144 Lab4 里 `TCPConnection::connect()` 与 `TCPSender::fill_window()` 的分工依据。

**消失测试**：抽掉「`connect` = 1 RTT」这个量化认识，你就无法解释为什么长肥管道（如跨洋链路）上的短连接效率极低——每次请求都要先付 1 RTT 建连，TLS 再付 1–2 RTT。

**落点**：§5.2（书页 233）与 §9.1 的 HTTP 连接复用（书页 416）；CS144 Lab4；一手坐标是 RFC 9293 的三次握手时序图。</span>

---

<!-- ===== p039 ===== -->
> 来源注：PDF 第 39 页 = 书页 35（页脚 `1.4. Software 35`）。本页为 client 代码全文。

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <netdb.h>

#define SERVER_PORT 5432
#define MAX_LINE 256

int
main(int argc, char *argv[])
{
    FILE *fp;
    struct hostent *hp;
    struct sockaddr_in sin;
    char *host;
    char buf[MAX_LINE];
    int s;
    int len;

    if (argc==2) {
        host = argv[1];
    }
    else {
        fprintf(stderr, "usage: simplex-talk host\n");
        exit(1);
    }

    /* translate host name into peer's IP address */
    hp = gethostbyname(host);
    if (!hp) {
        fprintf(stderr, "simplex-talk: unknown host: %s\n", host);
        exit(1);
    }

    /* build address data structure */
    bzero((char *)&sin, sizeof(sin));
    sin.sin_family = AF_INET;
    bcopy(hp->h_addr, (char *)&sin.sin_addr, hp->h_length);
    sin.sin_port = htons(SERVER_PORT);

    /* active open */
    if ((s = socket(PF_INET, SOCK_STREAM, 0)) < 0) {
        perror("simplex-talk: socket");
        exit(1);
    }
    if (connect(s, (struct sockaddr *)&sin, sizeof(sin)) < 0)
    {
        perror("simplex-talk: connect");
        close(s);
        exit(1);
    }
```

> **提取说明**：源 PDF 的文字层把 `main(int argc, char *argv[])` 中的 `*` 折到单独一行，并把 `fprintf(stderr, "usage: ...")` 拆成两截（先出现字符串、后出现 `fprintf(stderr,`）。这是 PDF 文本层的折行伪影，不是原书错误；此处已按语句边界复原。下一行起（主循环）见下一块（PDF p040）。†

> † **代码缺陷**：本段使用了 `bzero`、`bcopy`、`strlen`（见 p040 的主循环），但**没有包含 `<string.h>` 与 `<strings.h>`**。在 C99 之后的严格模式下，`strlen` 的隐式声明是错误；在 glibc 下 `bzero`/`bcopy` 声明在 `<strings.h>`。原书未包含这两个头文件，实际编译会得到 `implicit declaration of function 'strlen'` 之类的警告（或错误）。此外 `FILE *fp;` 声明后从未使用，`int len;` 在 p040 才被赋值——这两处是无害的冗余。

<span style="color:#1e8449;">**[译注]** 这段代码有三个点直接对应 **CSAPP 第 11 章**与 **CS144 Lab0**，而教材没有解释其中任何一个。第一，`htons(SERVER_PORT)` 是主机字节序到网络字节序（大端）的转换；漏掉它，5432（0x1538）在小端机上会变成 0x3815 = 14357 号端口。第二，`bcopy(hp->h_addr, ...)` 复制的是 `gethostbyname` 返回的**第一个**地址（`hp->h_addr` 等价于 `hp->h_addr_list[0]`），在双栈时代这只取到 A 记录或 AAAA 记录之一——现代代码应改用 `getaddrinfo`。第三，`gethostbyname` 是**阻塞且不可重入**的（返回静态缓冲区），在多线程程序里会串数据。

**可判定的判据**：判断一份 socket 代码是否「现代」，看它用 `getaddrinfo` 还是 `gethostbyname`。后者已被 POSIX.1-2008 标记为 obsolescent。

**消失测试**：抽掉字节序转换，跨架构通信（x86 与网络设备）时端口字段静默错位，错误不报语法、只表现为「连不上」；这是本课程最典型的「无 traceback 的 bug」。

**落点**：CSAPP 第 11 章（`getaddrinfo`、`htons`、`sockaddr_in`）；CS144 Lab0 的 `webget`（同一个 active open 流程）；§1.3.4 关于 presentation layer 的「最高有效字节先传后传」（书页 29）——Internet 把这件事降级为每个应用的义务。</span>

<span style="color:#1e8449;">**[译注]** 这段代码在书里被当成「socket 接口很简洁」的例证，但它自身暴露了 API 设计上的一个历史包袱，值得单独标出：**`connect()` 的失败处理与 `close()` 的顺序**。代码在 `connect` 失败时先 `perror` 再 `close(s)`，这是正确的（失败时 s 仍是有效描述符）；但很多教材示例漏掉 `close`，导致每失败一次泄漏一个文件描述符。这是一个可测量的资源泄漏：每连接失败泄漏 1 个 fd，Linux 默认上限 1024（`ulimit -n`），1000 次失败后程序报 `EMFILE`（`Too many open files`）。

**可判定的判据**：判断一段 socket 代码是否会泄漏 fd，看每一条失败路径上是否有 `close`。这条判据可以直接用在 lab 上：CS144 的 `TCPSocket` 用 RAII 管理 fd，所以你自己写的 `TCPConnection` 不需要（也不应该）手动 close。

**消失测试**：抽掉「失败也要 close」这条，长时间运行的服务端会在数天后突然拒绝新连接，且 CPU 与内存都正常——这是最难查的一类故障。

**落点**：CSAPP 第 11 章（fd 生命周期与 `EMFILE`）；CS144 的 `FileDescriptor` RAII 封装；§5.2 的 TCP 连接状态与资源释放（`TIME_WAIT`，书页 233）。</span>

---

<!-- ===== p040 ===== -->
> 来源注：PDF 第 40 页 = 书页 36（页脚 `36 Chapter 1. Foundation`）。本页含 client 主循环结尾、Server 说明与 server 代码起首。

```c
    /* main loop: get and send lines of text */
    while (fgets(buf, sizeof(buf), stdin)) {
        buf[MAX_LINE-1] = '\0';
        len = strlen(buf) + 1;
        send(s, buf, len, 0);
    }
}
```

**Server（服务器）**

server 同样简单。它先通过填入自己的端口号（`SERVER_PORT`）来构造地址数据结构。**由于不指定 IP 地址，该应用程序愿意在本地 host 的任意一个 IP 地址上接受连接。** 接下来，server 执行被动打开涉及的预备步骤：创建 socket、把它绑定到本地地址、并设置允许的最大待处理连接数。最后，主循环等待某个远端 host 尝试连接；当有连接进来时，它接收并打印在该连接上到达的字符。

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <netdb.h>

#define SERVER_PORT  5432
#define MAX_PENDING  5
#define MAX_LINE     256

int
main()
{
    struct sockaddr_in sin;
    char buf[MAX_LINE];
    int buf_len, addr_len;
    int s, new_s;

    /* build address data structure */
    bzero((char *)&sin, sizeof(sin));
    sin.sin_family = AF_INET;
    sin.sin_addr.s_addr = INADDR_ANY;
    sin.sin_port = htons(SERVER_PORT);

    /* setup passive open */
    if ((s = socket(PF_INET, SOCK_STREAM, 0)) < 0) {
        perror("simplex-talk: socket");
        exit(1);
    }
    if ((bind(s, (struct sockaddr *)&sin, sizeof(sin))) < 0) {
        perror("simplex-talk: bind");
        exit(1);
    }
```

<span style="color:#1e8449;">**[译注]** `sin_addr.s_addr = INADDR_ANY`（值为 0）这句的语义常被误解成「监听所有网卡」，更精确的说法是：**它匹配本机任意一个本地地址**，因此 `bind` 后该 socket 会同时接受来自回环、局域网、公网接口的连接。判据在于——若你只想让服务只在回环上可达，必须显式 bind `127.0.0.1`；只 bind 端口而不指定地址，等价于把服务暴露在所有接口上。这是「本机调试服务意外对公网开放」这类事故的直接原因。

**可算的判据**：`INADDR_ANY` 的可达集合 = 本机所有活动接口地址的并集。在一台有 3 个网卡 + 回环的机器上是 4 个地址；这也是 `netstat -tlnp` 里显示 `0.0.0.0:5432` 而非 `127.0.0.1:5432` 的含义。

**消失测试**：抽掉「`INADDR_ANY` = 全部本地地址」这层理解，你会以为它表示「任意远端地址」（那是 `accept` 之后对端地址的事）；可观测后果是安全配置错误：以为服务只在内网，实际公网可达。

**落点**：§9.1 的服务器部署讨论（书页 416）；CS144 Lab4 的 `TCPSocket::bind` 语义；CSAPP 第 11 章（`INADDR_ANY` 与「最广义地址」）；一手坐标是 `bind(2)` 手册页。</span>

<span style="color:#1e8449;">**[译注]** 这段 server 代码的结构本身就是一条重要事实：**它没有 `fork`、没有多线程、没有事件循环**。主循环拿到一个 `new_s` 后就进入「收-打印」循环，直到该连接结束——也就是说，这是一个**单连接、串行**的服务端，第二个客户端必须等第一个断开（或数据流向被阻塞）。这不是教材偷懒，而是要展示「被动打开」的最小形态；但它是 CS144 Lab4 之后必须明确跨过的坎：真实的 TCP 服务端要能同时持有 $n$ 个 `TCPConnection`。

**可算的判据**：串行服务端的吞吐上限是 $1/\text{每连接处理时间}$；若每个连接的处理占 $t$ ms，则并发 $n$ 个客户端时每个客户端的有效吞吐下降为 $1/(nt)$ 量级（排队论下更差）。这也是为什么 `backlog = 5` 的示例服务端在第二个客户端连入时就可能触发 SYN 丢弃。

**消失测试**：抽掉并发能力，服务端无法支撑任何真实负载；可观测现象是：第二个客户端 `connect` 成功但数据始终收不到（三次握手完成，数据排在 backlog 与接收缓冲里等 `accept`）。

**落点**：§5.2 的连接管理（书页 233）；CS144 Lab4 需要同时持有多个 `TCPConnection`；CSAPP 第 12 章（并发服务器三种范式：进程、线程、I/O 多路复用）——这三者是同一条判据的三种实现。</span>

---

> **本卷末说明**：本卷止于书页 36 的 server 代码 `bind` 之后（`(continues on next page)`）。`listen`/`accept`/`recv` 的主循环、§1.4.3 及其后内容、§1.5 Performance 全部在下一卷 `Peterson网络_p041-060.bilingual.md`。
>
> **与上一卷的衔接**：书页 1–16（PDF p001–p020）已由 `Peterson网络_p001-020.bilingual.md` 覆盖，止于 §1.2.3 的 "There are several different methods for multiplexing..." 之前；本卷首块（书页 17）正是那句之后的内容，两卷之间无缺口。
