> <span style="color:#7f8c8d;">Source: Stanford CS144 Fall 2019 慕课讲义 · Unit1-Internet-and-IP/1-2-four_layers.pdf · 行内双语翻译版</span>

# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> <span style="color:#c0392b;">1-2 The Four Layer Internet Model（互联网四层模型）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">从应用到模型：为什么需要分层</span>

在上一段视频中，你学到了不同的应用——比如 BitTorrent、Skype 和万维网——如何通过双向可靠字节流（bi-directional reliable byte stream）在互联网上通信。

要为我们的应用构建这个可靠的通信模型，需要许多不同的部件协同工作。但尽管我们使用的互联网应用五花八门、发送的数据种类和速率千差万别，应用收发数据的方式却有着惊人的一致性。例如，应用希望收发数据时**不必操心**数据在互联网上走的路径（path）或路由（route）；几乎所有应用也都希望确信数据被正确投递——丢失或损坏的数据会被自动重传（retransmit），直到被正确接收为止。

早期的互联网先驱们创建了"4 Layer Internet Model（互联网四层模型）"来描述构成互联网的操作层级（hierarchy），让应用可以一遍又一遍地复用同样的 building blocks（构件），而不必为每个应用从头造轮子。layering（分层）是网络中重要且高频使用的概念，本课程会反复见到它——甚至有一段专门讲分层的视频。

我们来看四层模型中每一层做什么。一个有助于记忆的要点是：**<span style="color:#00838f;">所有四层存在的目的，都是让终端主机（end-host）中的应用能够可靠地互相通信。</span>** 为了解释它如何工作，我将从最底层开始讲起。我们会看到每一层职责不同，每一层都在下一层之上构建服务（service），层层向上，直到顶层——应用之间的双向可靠字节流通信。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Link Layer（链路层）：一次搬运一跳</span>

先看一张示意图：互联网由 end hosts（终端主机）、links（链路）和 routers（路由器）组成；数据沿着一节节链路 hop-by-hop（逐跳）投递；数据以 packet（分组）为单位传输，一个 packet 由想投递的数据加上一个 header（首部）组成，首部告诉网络这个包要送到哪里、从哪里来等等。（原图演示了一个拓扑上的逐跳投递过程。）

**<span style="color:#00838f;">链路层（Link Layer）的职责：把数据一次一个链路地搬运过去。</span>** 你大概听说过 Ethernet（以太网）和 WiFi——它们就是两种不同的链路层。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Network Layer（网络层）：端到端投递 packet</span>

往上一层是（对我们而言）最重要的层：network layer（网络层）。**<span style="color:#00838f;">网络层的职责：把 packet 跨越整个互联网从源（source）端到端（end-to-end）地送到目的地（destination）。</span>**

packet 是网络中一个重要的基本构件。packet 是我们给"一个自包含的数据集合加上一个首部"起的名字，首部描述数据是什么、要去哪里、从哪来。你经常会看到 packet 被画成"header + data"两段的样子。

网络层的 packet 叫做 datagram（数据报）。它由一些数据和一个首部组成，首部里包含 "To"（收件）与 "From"（发件）地址——就像我们在信封上写"To:"和"From:"地址一样。（原图画了一个带收发件地址的数据报。）

网络层把 datagram 交给下面的链路层，告诉它"把这个 datagram 从第一条链路发出去"。换句话说，**链路层在向网络层提供一种 service（服务）**。本质上，链路层在说："你给我一个要发的 datagram，我就替你把它从一条链路上传过去。"

链路另一端是一台 router（路由器）。路由器的链路层从链路上接收 datagram，把它向上交给路由器中的网络层。路由器上的网络层检查 datagram 的目的地址（destination address），负责把 datagram 向最终目的地一跳一跳地路由（routing）——做法是再次交给链路层，让它走下一条链路。如此往复，直到抵达目的地的网络层。（原视频用一连串点击逐步演示这些步骤。）

请注意：**网络层不需要关心链路层*如何*把 datagram 送上链路。** 事实上，不同链路层的工作方式差异极大；Ethernet 和 WiFi 显然非常不同。网络层与链路层之间的这种 separation of concerns（关注点分离）让各自能专注于自己的职责，不必操心对方如何工作。这也意味着单一的网络层只需要一种通用方式和许多不同的链路层对话——把 datagram 递给它们去发送就行了。这种关注点分离之所以可能，靠的是每层的 modularity（模块化）以及一个共同且定义良好的、指向下层的 API。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 🎥 *McKeown 在视频中解释： "这种关注点分离是由每个层的模块化实现的"*（字幕来源：003 - p2 1-2 The four layer Internet model.zh.srt）。"通过统一的 API 对抗多样性"这个思想你其实非常熟悉：CSAPP ch11 里 socket 接口把 Ethernet、WiFi 等一切差异都藏在了内核协议栈之下，你写的 `socket(AF_INET, ...)` 代码完全不知道物理层用的是双绞线还是无线电。网络层对链路层说"给我一个 datagram 我就传一跳"，本质上和操作系统对硬件说"给我一个 syscall 我就干活"是同一个设计模式：接口（interface）固定、实现（implementation）自由替换。这也正是 Linux 内核里网络设备抽象成 `net_device` 结构的原因——IP 协议代码一行都不用改就能支持新网卡。从抽象层面看，这与你学过的 ADT（抽象数据类型）一模一样：链路层就是"传输一跳"这个操作的多个实现，网络层只依赖操作签名，不依赖实现细节。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">IP 与 best-effort：没有保证的底层</span>

在互联网中，网络层很特殊：**<span style="color:#00838f;">当我们把 packet 送进互联网时，必须使用 Internet Protocol（互联网协议，IP）。正是 IP 把整个互联网粘在一起。</span>** IP 的细节我们会在后面的视频里学，但现在值得先记住几条基本事实：

- IP 尽力而为（best-effort）地把我们的 packet 送到对端，但**它不做任何承诺（no promises）**。
- IP packet 可能丢失（lost）、可能乱序（out of order）到达、也可能损坏（corrupted）。

没有任何保证。这可能出乎你的意料：packet 都不保证送达，互联网怎么可能正常工作？答案是：如果应用想要"必要时重传、按序且无损坏地送达"这类保证，那它需要一个运行在 IP 之上的协议。这正是 transport layer（传输层）的工作……

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Transport Layer（传输层）：TCP 与 UDP</span>

最常见的传输层是 TCP（Transmission Control Protocol，传输控制协议）。（你大概听说过 TCP/IP 这个词——那是应用同时使用 TCP 和 IP 的情况。）

**<span style="color:#00838f;">TCP 确保互联网一端应用发送的数据被正确送达——按正确顺序交给互联网另一端的应用。</span>** 如果网络层丢了某些 datagram，TCP 会重传它们，必要时重传多次；如果网络层乱序投递——也许因为两个 packet 走了不同的路径——TCP 会把数据重新排好序。后面的视频会深入 TCP 的工作原理。现在要记住的主旨是：TCP 在"网络层提供的不可靠 datagram 投递服务"之上，向应用提供"数据正确按序送达"的保证。

可以想见，web client（网页客户端）、email client（邮件客户端）这类应用觉得 TCP 好用极了：借助 TCP 保证数据正确送达，它们就不必在应用内部实现所有这些机制，而是复用开发者们在正确实现 TCP 上付出的巨大努力。**复用（reuse）是分层的又一大优点。**

但并非所有应用都需要"数据被正确送达"。比如视频会议应用在一个 packet 里发送一小段视频时，等待这个包被重传多次可能毫无意义——不如直接跳过。有些应用就是不需要 TCP 服务。若不需要可靠投递，应用可以用简单得多的 UDP（User Datagram Protocol，用户数据报协议）：UDP 只是把应用数据打成一个包，交给网络层送出去，不提供任何投递保证。

换句话说，应用在传输层至少有两种服务可选：TCP 与 UDP。实际上还有许多其他选择，但这两个是最常用的传输层服务。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Application Layer（应用层）与对等通信</span>

最后是四层模型顶部的 application layer（应用层）。使用互联网的应用成千上万，虽然每个应用各不相同，但它可以通过"应用层到下层 TCP 或 UDP 服务的定义良好的 API"来复用传输层。

正如上一段视频所见，应用通常想要的是两个端点之间的双向可靠字节流。它们想发什么字节流就发什么字节流，而应用有自己的协议（protocol）来定义两端点之间数据的 syntax（语法）和 semantics（语义）。举例来说，上一段视频里，当 web client 向 web server 请求页面时，客户端发送一个 GET 请求——这是 hypertext transfer protocol（超文本传输协议，http）的命令之一。http 规定 GET 命令以 ASCII 字符串形式发送，并附上所请求页面的 URL。**<span style="color:#00838f;">在应用层看来，GET 请求是直接发给对端的——对端的 web server 应用。</span>** 应用不需要知道请求是如何到达那里的、中途被重传了多少次。在 web client 这边，应用层把 GET 请求交给 TCP 层，由 TCP 提供可靠送达的服务；TCP 又使用网络层的服务，网络层再使用链路层的服务。

**<span style="color:#00838f;">我们说每一层都在与它的对等层（peer layer）通信。</span>** 就好像每一层只与链路或互联网另一端的同一层对话，全然不顾下层是如何把数据送到那里的。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">把一切拼起来：数据的下行之路</span>

网络工程师发现，把构成互联网的所有功能组织成 *layers（层）* 非常方便。顶层是 application（应用），比如 BitTorrent、Skype 或万维网，它与目的地的对等层对话。应用有数据要发送时，把数据交给 transport layer（传输层）；传输层的职责是把数据（可靠地，或不可靠地）送到另一端，做法是把数据交给 network layer（网络层）；网络层的职责是把数据切成 packet、给每个 packet 写上正确的目的地址；最后，packet 被交给 link layer（链路层），链路层负责把 packet 沿路径从一跳送到下一跳。

数据就这样一跳一跳地、从一个 router 到下一个 router 前进。网络层把它逐台转发，直到抵达目的地；在那里，数据沿层向上传递，最终到达应用。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">四层的职责清单（slide 总结）</span>

（原文 slide 上依次出现的四句话，对应自顶向下的四层：）

- **Application layer（应用层）：** 支持网络应用（如 HTTP、Skype、BitTorrent）——程序之间语义层面的约定。
- **Transport layer（传输层）：** 在两个进程（process）之间可靠地、按序地搬运数据（TCP；UDP 则不保证）。
- **Network layer（网络层）：** 把 datagram（数据报）从源主机路由（routing）到目的主机——互联网的"细腰"。
- **Link layer（链路层）：** 在一跳链路上搬运数据（如 Ethernet、WiFi、DSL、3G）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">延伸一：IP 是互联网的"thin waist（细腰）"</span>

现在四层都看过了，最后补充两件你需要知道的事。

第一件：IP 常被称为互联网的 "thin waist"（细腰/沙漏的收腰处）。**<span style="color:#00838f;">因为只要想使用互联网，就必须使用 Internet Protocol——没有选择。</span>**

但链路层我们却有很多选择：IP 可以运行在许多不同的链路层之上，比如 Ethernet、WiFi、DSL、3G 蜂窝网络等等。而在不可靠的 IP 层之上，我们同样可以在许多传输层里挑选：我们已经见过 TCP 和 UDP，还有面向实时数据的 RTP 以及许多其他协议；再往上，当然还有几万种不同的应用。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">延伸二：OSI 七层模型与层号的遗产</span>

第二件：20 世纪 80 年代，International Standards Organization（国际标准化组织，ISO）创建了一个 7-layer model（七层模型）来表示任意类型的网络，叫做 7 层 Open Systems Interconnection（开放系统互联，OSI）模型。本课程不在它上面花时间，因为它已被四层互联网模型取代。如果你感兴趣，随便一本网络教科书和 Wikipedia 都对七个层有详细描述。七层模型定义的层次在四层互联网模型中被合并了：例如今天我们所说的链路层，当年被拆成了定义成帧格式（framing format）的 Link Layer 和定义电压电平、接插件物理尺寸之类东西的 Physical Layer（物理层）；网络层两者基本相同；传输层与应用层在 OSI 中则各占两层。（原图把这些常用互联网协议标注到了 OSI 的编号体系上。）

今天，七层 OSI 模型唯一的真实遗产是它的编号系统。你会听到网络工程师把 network layer 叫做 "Layer 3（三层）"——尽管在互联网模型里它是自底向上的第二层；类似地，人们把 Ethernet 叫做 Layer 2 协议，把应用叫做 Layer 7。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 🎥 *McKeown 在视频中解释： "它提供了一种尽力而为的交付服务，没有保证"*（字幕来源：003 - p2 1-2 The four layer Internet model.zh.srt）。"尽力而为但毫无保证"的网络竟然能承载整个数字经济，初听是悖论，用第一性原理拆开看却很自然：**保证是有代价的**。如果 IP 内置重传、排序、去重，那么实时语音（丢一帧音频重传到了也早已过期）和心跳包这类"宁新勿旧"的流量就得为用不上的保证买单；而把保证做成可选项（TCP/UDP/RTP），代价就只由需要的人支付。这与你 CSAPP 中学过的分层思想同源：操作系统内核只提供进程与虚拟内存这两个"不多不少"的抽象，其余功能由库和用户程序按需组合。另外提醒一个高频考点级误解：四层模型里"传输层负责可靠"只对 TCP 成立，UDP 同样在传输层却不保证任何东西——判断某功能属于哪层，永远看"这一跳的边界在哪里"，而不是看"它听起来像什么"。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "对等层通信"（peer-to-peer communication，此处指协议对等而非 P2P 应用）是本节最容易被忽视却最核心的概念。发端的 GET 请求从未"直接"飞到对端：它在发送方逐层向下被封装（encapsulation），在接收方逐层向上被解封装，虚线画出的对等虚对话只是逻辑上的。你可以用 6.042J 的归纳结构来理解：纵向是严格的组合关系（下层服务叠加成上层服务），横向是逻辑等价关系（两端同层看到的是同一份协议数据单元）。这个"横向对等、纵向组合"的二维结构解释了很多工程现象：为什么抓包时看到的 header 是嵌套的（链路帧套 IP 头套 TCP 头套 HTTP 文本）；为什么改某一层的实现不影响别的层（TCP 从拥塞控制角度换了算法，浏览器代码毫发无损）；也为什么"Layer 3"这类 OSI 编号至今仍被沿用——语言会过时，但分类框架的寿命远长于它描述的对象。
