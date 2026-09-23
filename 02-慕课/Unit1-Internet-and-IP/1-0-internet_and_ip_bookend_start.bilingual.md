> <span style="color:#7f8c8d;">Source: Stanford CS144 Fall 2019 慕课讲义 · Unit1-Internet-and-IP/1-0-internet_and_ip_bookend_start.pdf · 行内双语翻译版</span>

# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> <span style="color:#c0392b;">Unit 1《The Internet and IP（互联网与 IP）》开场白</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">单元开场：先看全局，再补细节</span>

**Phil：** 欢迎来到第一个单元！在这个单元里，你将首先获得"big picture（全局图景）"，以及最初的一些细节。

**Nick：** 你将学习互联网工作原理的基础知识。你甚至可能会弄清楚我们俩谁才是 Phil、谁才是 Nick。我们希望帮助您理解**为什么**互联网会被设计成现在这个样子——它的优势（strengths）和劣势（weaknesses）分别是什么？我们还会教你一些公认的网络设计原则（design principles），比如 layering（分层）、encapsulation（封装）和 packet switching（分组交换）。学完这个单元之后，你应该能够回答这样的问题：什么是互联网（What is the Internet）？什么是互联网地址（What is an Internet Address）？诸如 Web、Skype 和 BitTorrent 这样的应用是如何工作的？这些原则将帮助你在未来设计出更好的网络。

**Phil：** 在第一个单元结束时，你应该熟悉一个叫做互联网"4-layer model（四层模型）"的东西。它描述了互联网如何被分解为四个各司其职的层次（layers）。你会学到 layer（层）到底是什么，以及为什么它是良好网络设计的一条基本原则。你会学到互联网的四层分别是什么、它们如何协同工作。你还会学到：大多数应用使用一种叫做 Transmission Control Protocol（传输控制协议，TCP）的 transport layer（传输层），以及一些应用是如何使用它的。

你还会学到：互联网是通过把数据切分成叫做 packet（分组）的小单位来工作的。举例来说，当你请求一个网页时，你的计算机会向 web server（Web 服务器）发送若干 packets，而互联网负责决定这些数据分组如何正确到达目的地。

**Nick：** 本单元还会稍微深入地考察其中一层，叫做 network layer（网络层）。你可能听说过 IP（the Internet Protocol，互联网协议）。它之所以以"互联网"命名，是因为它是让整个互联网粘合在一起的"胶水"。你可以更换其余所有的层，但要想使用互联网，你就必须在网络层使用 Internet Protocol。你会学到 Internet Protocol 做什么、以及它如何做到；你会学到 Internet Protocol addresses（IP 地址）以及它们是如何分配的；你还会初步了解互联网是如何根据 IP 地址来决定一个 packet 应该走的路径的。

**Phil：** 最后，我们会向你展示几款软件工具，你可以用它们来观察你的计算机是如何使用互联网的。这样，下次你浏览网页时，就能把本单元学到的东西用上了！

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">本单元的四个主题（Nick）</span>

**Nick：** 在本单元中你将学习四个主题：

1. <span style="color:#00838f;">**应用如何使用互联网。**</span> Phil 会以 Skype、BitTorrent 和 Web 为具体例子，讲解各种不同的应用使用互联网的共同方式。你会学到：大多数应用都希望在两个或多个端点（end points）之间，通过一条 reliable（可靠的）、bidirectional（双向的）byte-stream（字节流）进行通信。

2. <span style="color:#00838f;">**互联网的结构。**</span> 正如 Phil 刚才所说，互联网由四层组成，每一层在让互联网运转这件事上都有明确定义的角色。我会描述四层模型是什么，以及每一层的职责。你将学到为什么我们每次通过互联网发送 packets 时都要使用 Internet Protocol（简称 "IP"），以及为什么我们称 IP 为互联网的"thin waist（细腰）"。

3. <span style="color:#00838f;">**互联网协议（IP）。**</span> 因为 IP 太重要了，我们会用几段视频来描述 IP 为我们做了什么、以及它如何工作。现在我们先聚焦于 IP version 4（第 4 版 IP），因为它是当今使用最广泛的版本。你会学到 IP 地址、router（路由器）如何查找 IP 地址等等。在本课程后面你还会学到更新的 IPv6。

4. <span style="color:#00838f;">**基本的体系结构思想与原则。**</span> 你还将学习四条网络的基本原则，它们都与我们理解互联网密切相关。第一条是 packet switching（分组交换）：数据被切分成一个个自包含的、自足的信息分组，并依据 packet header（分组首部）中的信息 hop-by-hop（逐跳）转发——这是一种简单而巧妙的工作方式。第二条是 layering（分层），前面已经提到过。第三条是 encapsulation（封装），即把某一层处理好的 packet 放进下一层数据部分的过程。这有助于在各层之间实现清晰的 separation of concerns（关注点分离）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">结语：变化中的互联网，不变的核心（Phil）</span>

**Phil：** 在本单元结束时，你应该对互联网的基本结构和三个基本体系结构思想有很好的理解。你会明白像浏览器这样的应用是如何工作的，以及互联网如何在两台计算机之间投递 packets。你以前可能听说过 TCP/IP 这个词：你将学到 TCP 是什么、IP 是什么，以及它们为什么相关。乍一看，这些可能显得像是些"脏活累活"式的底层细节。但事实上，它们正是互联网赖以存在的 bedrock（基石）。每年都有新的应用和互联网的新用途涌现，但它们全都使用我们即将讨论的这些基本原则，而且几乎全都使用 TCP/IP。从这些惊人稳定的基本原理出发，你学到的知识将始终重要——即便未来我们进入 5G 无线网络、Web 3.0 和物联网（Internet of Things）时代也依然如此。

这正是令人兴奋的地方之一。互联网及其能力在不断扩张和变化，但在整个演化过程中，有一些核心思想和原则是恒定不变的。学会了它们，你不仅知道互联网和网络今天如何工作，很可能还知道它们 20 年后将如何工作。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 把"学 TCP/IP 就是学底层脏活"这个直觉纠正是本单元最重要的铺垫。你已经用过 CSAPP ch11 的 socket（套接字）接口：当时 `connect()`、`read()`、`write()` 背后的 TCP 细节被系统调用完全遮住了，你只拿到一个"可靠的字节流文件描述符"。这个单元反过来做同一件事——把 socket 之下的四层模型一层层打开给你看：为什么 `read()` 写进去的字节能按序到达（transport layer 的功劳）、为什么断网时会 `ECONNRESET`（network layer 只是尽力而为）。换句话说，CSAPP 教你"怎么用"这个抽象，CS144 教你"为什么这个抽象长这样、代价是什么"。Phil 说这些原则"20 年后依然成立"并非鸡汤：从 1981 年的 RFC 793 到今天的 QUIC，底层传输协议换了好几代，但"分层 + 封装 + 分组交换 + 尽力而为的网络内核"这套骨架从未动摇。带着"我已在 ch11 用过这个 API"的先验来学，每个概念都会落到你已经写过的代码上。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 🎥 *McKeown 在视频中解释： "我们希望能帮助你理解互联网为什么被设计成这个样子"*（字幕来源：001 - p0 1-0 The Internet and IP Introduction.zh.srt）。注意本单元的教学顺序暗藏一条方法论：先从应用视角看"用户想要什么"（可靠字节流），再自底向上看"各层如何拼出这个服务"，最后才谈设计原则。这和你在 6.042J 里学"先承认公理、再推定理"的顺序相反，却更符合工程直觉：设计原则是从"需求"与"约束"的张力中逼出来的。例如"IP 为什么只做尽力而为"这个看似偷懒的选择，其实是把复杂性推到端系统的深思熟虑决策——这正是后面 [note] 会反复回扣的 end-to-end principle（端到端原则）。学这一单元时，建议每看到一条设计选择就问一句"如果反过来设计会怎样"，这是理解"为什么"最有效的提问方式。
