# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Unit 2: Transport — Recap（第 2 单元：传输层 · 总结）

> <span style="color:#7f8c8d;">Source: 慕课/Unit2-Transport/2-12-transport-recap.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">第 1 页 · Nick McKeown：本单元回顾——三种传输层与端到端原则</span>

在本单元中，你学习了**传输层（transport layer）**，具体来说，学习了当今使用中最重要的三种传输层：

1. **TCP——传输控制协议（Transmission Control Protocol）**：超过 95% 的互联网应用在使用。TCP 几乎被普遍采用，因为它提供了几乎所有应用都需要的**可靠的、端到端的、双向的字节流服务（reliable, end-to-end, bidirectional byte-stream service）**。本单元的大部分视频都围绕 TCP 展开：你学习了如何检测分组在途中未被送达或被损坏，学习了 TCP 为成功重传（retransmission）数据直到正确交付而使用的各种机制。我们用了 3 段视频探讨在不可靠的互联网上可靠交付数据的不同方法。

2. **UDP——用户数据报协议（User Datagram Protocol）**：供不需要 TCP 那种"保证交付"服务的应用使用——要么应用以自己的私有方式处理重传，要么它根本不需要可靠交付。UDP 所做的全部事情是：取来应用数据，创建一个 UDP 数据报（datagram），数据报中标识出数据在另一端应送达的应用。仅此而已。虽然用 UDP 的应用很少，但我们见过 DNS 与 DHCP 这两个例子，它们都是简单的请求—响应式查询协议。

3. **ICMP——互联网控制消息协议（Internet Control Message Protocol）**：它的主要职责是在出现问题时发送反馈。例如，路由器收到一个 IP 数据报却不知道下一步该发往何处，就向源头发送一条 ICMP 消息告知情况。ICMP 对诊断端到端通信为何不正常非常有用。

最后，你学习了指导互联网设计、至今仍在指导我们思考的最重要的总体架构原则之一——"**端到端原则（End to End Principle）**"。我们学习了它的两个版本：

- **温和版本（milder version）**：某些功能只有在网络边缘（edges or fringe）才能正确实现，这些功能显然就该在那里实现——端到端可靠文件传输与安全性就是两个例子。可以通过给网络添加功能来辅助它们，但只能辅助、不能替代端到端的功能。
- **强版本（stronger version）**：如果某个功能能在终端主机上实现，那就应该在终端主机上实现。基本思想是让网络保持简单精炼，尽量少出现会出错、会拖慢速度或需要升级的功能；同时假设终端主机相当聪明（如笔记本电脑或智能手机），能够实现应用所需的许多功能。

<span style="color:#7f8c8d;">（第 1 页 / 共 3 页）</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> Nick 概括的端到端原则有两个强弱版本，这个区分比口号本身重要得多。弱版本是 1984 年 Saltzer、Reed 与 Clark 的论文《End-to-End Arguments in System Design》的原意：某些功能只有在端系统上实现才能完全正确，网络内部可以加辅助功能但只能"帮忙"不能"替代"——因为中间节点看不见全部故障模式（主机内存里的位翻转、路由器缓存中的损坏，网络无从知晓）。强版本更进一步：能在主机上做就应在主机上做，把网络保持简单。你已经知道 TCP 就是弱版本的落点：IP 层尽力而为（best-effort），可靠字节流（reliable byte stream）由两端主机的 TCP 实现补齐；这还带来命运共享（fate sharing）的好处——连接断了，两端状态一起消失，不会在网络里留下半死的中间状态。CSAPP 第 11 章里你写的服务器只调 socket API，看不到这层论证；但"为什么不在路由器上为可靠传输打补丁"这个问题，答案就藏在这一段里。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">第 2 页 · Phil Levis：五大主题</span>

在本单元你学习了五个主要主题：

1. **三种广泛使用的传输层**：TCP 用于在应用之间可靠地交付字节流（byte stream）；UDP 用于在应用之间不可靠地交付数据报（datagram）；ICMP 用于检测何时出了问题。
2. **TCP 的工作原理**，重点是如何在两个应用之间可靠地交付字节。你学习了如何检测数据错误与分组丢失，分组如何被重传，以及几种不同的重传策略（retransmission strategies），包括**选择性重传（selective repeat）**与**回退 N（go back N）**。你学习了基础 TCP 机制使用 Go-Back-N，并用**滑动窗口（sliding window）**跟踪未确认（outstanding）的字节；还学习了跟踪 TCP 连接当前状态的 **TCP 状态机（TCP state machine）**。
3. **UDP 的工作原理**，以及为什么只有少数应用使用它。
4. **ICMP 的工作原理**：它如何帮助我们发现通信何时出错，并监测两个终端主机之间路由（route）的性能。
5. **端到端原则**：指导互联网及许多其他通信系统设计的重要总体原则。在这门课的整个学习过程中——以及你日后带着新的网络专业技能走出校园之后——你会发现许多人都在谈论这一原则，以帮助指导他们的设计决策。

<span style="color:#7f8c8d;">（第 2 页 / 共 3 页）</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> Phil 对 TCP 的那句概括值得细读："基础 TCP 机制使用 Go-Back-N，并用滑动窗口跟踪未确认字节"——它把 2-9 重传策略与 2-8 滑动窗口两份讲义焊在了一句话里：滑动窗口提供吞吐量（多条报文在途、管道充满），Go-Back-N 提供丢包后的恢复策略（接收窗口为 1、整窗回退），两者的组合是 1980 年代主机内存昂贵约束下的最优工程解。用期望的语言复核其代价：丢包率 $p$ 下，一次丢包使窗口内 $N$ 个在途分组全部作废，平均要重传 $O(N)$ 个分组，有效吞吐量随 $p$ 增大而坍缩——这正是后来高带宽时延积网络需要快速重传与 SACK 这类选择性重传补丁的原因。对照 CSAPP 第 11 章：你在 socket 层看到的 `connect()`/`accept()` 只是状态机的入口，真正让一条 TCP 连接"可靠"的全部机器——序号、确认、重传、窗口——都发生在内核协议栈里，而状态机就是它们的调度中枢。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">第 3 页 · 学习目标自查</span>

学完本单元后，你应该做到：

- 对**三种不同的传输层**有透彻的理解；
- 理解**不同的重传策略**，以及 TCP 为什么使用"滑动窗口"；
- 知道 TCP 为什么使用**连接（connections）**、连接如何建立、以及如何由**状态机**维护；
- 能够解释**端到端原则**。

<span style="color:#7f8c8d;">（第 3 页 / 共 3 页）</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这份自查清单正好是本单元五份 slides 的目录，建议把它当作复述练习：合上讲义，逐条独立讲出"机制是什么"与"为什么这样设计"两个层次——比如停等（stop-and-wait）与滑动窗口的效率差距要用带宽时延积解释，序列号空间要给出 $RWS + SWS$ 的推导，TIME-WAIT 的 $2\,\mathrm{MSL}$ 要能还原"最后 ACK 丢失 + 端口复用"两个场景。Nick 提到的"95% 应用用 TCP"也值得追问反面：UDP 上的 DNS/DHCP 是一问一答的短查询，为它付三次握手加慢启动的代价不划算，超时后直接重发整个查询更简单——CSAPP 里 `SOCK_STREAM` 与 `SOCK_DGRAM` 的参数区别就是这个选择的编程接口体现。三种传输层的分工本质上是"谁为可靠性买单"的三种答案：TCP 由内核买单、UDP 由应用自付、ICMP 只报告故障。

---

> <span style="color:#7f8c8d;">脚注：本文件为视频 2-12《Transport (recap)》的讲稿记录（Nick McKeown 与 Phil Levis 两位讲者），是 Unit 2 的收尾总结。</span>
