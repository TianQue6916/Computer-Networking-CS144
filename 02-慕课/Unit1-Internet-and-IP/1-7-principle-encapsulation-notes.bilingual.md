> <span style="color:#7f8c8d;">Source: 1-7-principle-encapsulation-notes.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#c0392b;">Principle: Encapsulation（原理：封装）</span>

### <span style="color:#c0392b;">Slide 1 · Principle: Encapsulation（原理：封装）</span>

<span style="color:#2471a3;">**[section]**</span>

McKeown：本视频讲解一个我们称之为 **encapsulation（封装）** 的架构原则。<span style="color:#00838f;">所谓 encapsulation（封装），正是 layering（分层）与 packet switching（分组交换，即把数据切成一个个独立的小分组、各自独立地穿越网络）相结合的产物。</span>我们想把数据拆成离散的单元——packet（分组）——但每个 packet（分组）里却装着来自多个层的数据。举例来说，当你发送一个 TCP（Transmission Control Protocol，传输控制协议）segment（报文段）时，它在 IP（Internet Protocol，网际协议）packet（分组）里面，而 IP packet（分组）又在 Ethernet frame（以太网帧）里面。这个 encapsulation（封装）描述的正是这套机制如何运作：<span style="color:#00838f;">所谓 encapsulation（封装），就是这样一种原则——按它来组织 packet（分组）中的信息，就能既维持各层之间的分离，又让它们共享同一个 packet（分组）的 payload（载荷）内容。</span>

### <span style="color:#c0392b;">Slide 2 · Layering（分层）</span>

<span style="color:#2471a3;">**[section]**</span>

图示：七层 OSI（Open Systems Interconnection，开放系统互连）模型的层叠图——自上而下依次为 Application（应用层）、Presentation（表示层）、Session（会话层）、Transport（传输层）、Network（网络层）、Link（链路层）、Physical（物理层）。

- separation of concerns（关注点与职责的分离）
- 让每一层的 service（服务）可以独立演化
- 例子：
  - Transport（传输层）：应用之间的通信
  - Link（链路层）：同一条共享 link（链路）上 host（主机）之间的通信

Levis：回顾一下，layering（分层）让你能把一个复杂系统拆成较小的部分。每一层向上层提供一个 **service（服务）**——即对网络的一种 abstraction（抽象），而它自己则借助下一层来实现这个 abstraction（抽象）。每一层都是 self-contained（自包含）的，所以只要它提供了上层所期望的 service（服务），上层就不必关心它是怎么做到的。这种 separation of concerns（关注点分离）意味着每一层可以独立演化。正如 Network（网络层）的 IP 不需要关心 Transport（传输层）的 TCP 有什么改动，HTTP 这样的 Application（应用层）协议也不必关心 TCP 的变化。比如近几年，大多数操作系统都调整了自己使用的 TCP 算法，以更好地应对不断增长的网速——但同一个网页浏览器，无论新旧算法都能正常工作。

注意：这张分层图用的是 7 层 OSI 模型。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "关注点分离"这个词你在 CSAPP（《深入理解计算机系统》）里其实已经体验过一次了：virtual memory（虚拟内存，ch9）把"程序要连续的地址空间"和"物理内存碎片化的现实"这两件麻烦事拆开，各自独立演化。而 layering（分层）网络做的是同一件事——Transport（传输层）向上只承诺"应用间 reliable byte stream（可靠字节流）"这个 abstraction（抽象）。你在 CSAPP ch11 写 socket（套接字）客户端/服务器时，只用过 `connect()`、`read()`、`write()` 这几个调用，从未操心过 TCP 内部 window size（窗口大小）怎么变、retransmission（重传）何时触发——这正是"只要 service（服务）兑现，上层无需关心实现"的直接证据。讲义里的例子换个角度说就是：内核把 TCP 算法升级了，你的浏览器二进制一个字节都不用改。学网络时不妨把每层都当成一个"接口与实现"的分离体来看，这与 CSAPP 中"抽象掩蔽实现"的思路一脉相承。

### <span style="color:#c0392b;">Slide 3 · 4 Layer Model（四层模型）</span>

<span style="color:#2471a3;">**[section]**</span>

图示：把七层模型压缩后的四层互联网模型——自上而下依次为 Application（应用层）、Transport（传输层）、Network（网络层）、Link（链路层），这四层下面还有一层 Physical（物理层），表示实际的传输介质（线缆或无线电波），通常不计入这四层协议。

- separation of concerns（关注点与职责的分离）
- 让每一层的 service（服务）可以独立演化
- 例子：
  - Transport（传输层）：应用之间的通信
  - Link（链路层）：共享 link（链路）上 host（主机）之间的通信

McKeown：所以，让我们把上面那 7 层"压缩"回四层模型。

### <span style="color:#c0392b;">Slide 4 · Encapsulation（封装）</span>

<span style="color:#2471a3;">**[section]**</span>

图示：四层模型旁画着一个 packet（分组），自外向内依次是各层的 header（首部）与 data（数据）。

- layering（分层）如何体现在数据的表示形式上
- 第 $N$ 层的数据就是第 $N-1$ 层的 **payload（有效载荷）**
- 例子：HTTP（网页）应用数据 → 装在 TCP Transport（传输层）segment（报文段）里 → 装在 IP Network（网络层）packet（分组）里 → 装在 WiFi Link（链路层）frame（帧）里

McKeown：这个 encapsulation（封装）就是那条让我们能把协议各层"叠放"进同一个 packet（分组）的存储空间、又互不干扰的原则。它是 layering（分层）在真实数据表示上的体现。工作方式是：<span style="color:#00838f;">每个协议层都有自己的 header（首部），其后是它的 payload（载荷），再往后可能还有一个 footer（尾部）。</span>例如，IP packet（分组）的 header（首部）里有 source address（源地址）和 destination address（目的地址）。要用 IP 发送一个 TCP segment（报文段），我们就把 TCP segment（报文段）整体作为 IP packet（分组）的 payload（载荷）。这样，IP packet（分组）就"封装"了 TCP segment（报文段）。IP 根本不知道也不在乎自己的 payload（载荷）是什么——它只负责把 packet（分组）送到一台 end host（终端主机）。当 packet（分组）到达后，host（主机）查看 payload（载荷）内容，发现里面是个 TCP segment（报文段），于是按 TCP 的规则处理它。

McKeown：下面是个更完整的例子。假设你用一台通过 WiFi（无线以太网）上网的电脑浏览网页。你的浏览器生成一个 HTTP GET 请求。这个 HTTP GET 请求是某个 TCP segment（报文段）的 payload（载荷）；这个封装着 HTTP GET 的 TCP segment（报文段）又是某个 IP packet（分组）的 payload（载荷）；而这个封装着 TCP segment（报文段）和 HTTP GET 的 IP packet（分组）又是某个 WiFi frame（帧）的 payload（载荷）。如果你去看电脑实际发出的字节，它们就长这样：最外层的 encapsulation（封装）格式是 WiFi frame（帧），里面是 IP packet（分组），IP packet（分组）里是 TCP segment（报文段），TCP segment（报文段）里是 HTTP GET 的文本。

Levis：McKeown 画 packet（分组）的方式引出了一个可能让你困惑的问题！画 packet（分组）其实有两种画法，差别来自画图者的背景和他所研究的系统部位。McKeown 在这里把 header（首部）画在右边：这个 packet（分组）的 first bit（第一位）在最右，last bit（最后一位）在最左。

McKeown：这完全说得通！当 router（路由器）或 switch（交换机）发送 packet（分组）时，我们习惯把 packet（分组）画成从左向右移动——那么最先离开设备的那一位自然画在最右边。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 用第一性原理看，encapsulation（封装）就是一个递归嵌套的 `(header, payload)` 结构：每一层只读自己那段固定格式的 header（首部）前缀，然后把"剩下的字节"整体交给上一层协议去解释——IP 不解释 payload（载荷），正如 C 语言里你拿到一个 `void*` 指针后，必须靠约定去解读它指向的内容。从 CSAPP ch3 的内存视角看，这非常像结构体的嵌套布局：外层 header（首部）是低地址处的固定字段，payload（载荷）紧随其后，解析时按偏移量逐层下潜。Wireshark（一款开源网络抓包分析工具）之所以能一层层展开协议，就是因为它按这个嵌套约定逐级切割字节流。理解了"encapsulation（封装）= 字节流上的递归分区"，你就明白为什么任何一台新设备只要遵守这套格式就能接入互联网——设备不需要理解每一层的语义，只需要知道"我这一层的边界在哪、该把剩余部分交给谁"。这也是后续 Lab 里你亲手解析报文时会反复用到的思维模型。

### <span style="color:#c0392b;">Slide 5 · Encapsulation（封装：硬件视角）</span>

<span style="color:#2471a3;">**[section]**</span>

图示：与上一页相同的封装图，但标注为"Hardware: first bit on right（硬件习惯：第一位在右边）"，各层 header（首部）画在右侧。

- layering（分层）如何体现在数据的表示形式上
- 第 $N$ 层的数据就是第 $N-1$ 层的 payload（有效载荷）
- 例子：HTTP（网页）应用数据 → TCP Transport（传输层）segment（报文段） → IP Network（网络层）packet（分组） → WiFi Link（链路层）frame（帧）

Levis：但我画 packet（分组）是另一种方向：header（首部）在左，footer（尾部）在右。像这样。

### <span style="color:#c0392b;">Slide 6 · Encapsulation（封装：软件视角）</span>

<span style="color:#2471a3;">**[section]**</span>

图示：与上页相同的封装图，但标注为"Software: first bit on left（软件习惯：第一位在左边）"，各层 header（首部）画在左侧、footer（尾部）画在右侧。

- layering（分层）如何体现在数据的表示形式上
- 第 $N$ 层的数据就是第 $N-1$ 层的 payload（有效载荷）
- 例子：HTTP（网页）应用数据 → TCP Transport（传输层）segment（报文段） → IP Network（网络层）packet（分组） → WiFi Link（链路层）frame（帧）

Levis：这种画法来自软件界。你阅读 IETF（Internet Engineering Task Force，互联网工程任务组）文档和许多其他协议规范时看到的就是它。想法是：这个 packet（分组）的起始地址是 address zero（地址零），所以一个 header（首部）的第一个字节位于 address zero（地址零）；由于地址从左向右递增，packet（分组）的开头就在左边，结尾就在右边。

McKeown：这里没有对错之分。两种画法都有价值，取决于你用图来做什么。两种你都得习惯。我一般把 header（首部）画在右边。

Levis：而我一般画在左边。McKeown 的背景是电气工程与 switch（交换机）设计，我的背景是计算机科学与协议软件。

### <span style="color:#c0392b;">Slide 7 · Encapsulation（封装：两种画法等价，Wireshark 演示）</span>

<span style="color:#2471a3;">**[section]**</span>

（本页标题栏仍为"Encapsulation"，正文承接上页，含一个"="号，表示两种画法等价。）

Levis：现在回到 McKeown 的例子——WiFi frame（帧）里的 IP packet（分组）里的 TCP segment（报文段）里的 HTTP GET。我们看看它在真实网络中是什么样子。开始录制之前，我打开了 Wireshark，抓取了一次网页请求的 packet trace（分组轨迹）。只看其中一个 packet（分组）：可以看到 Wireshark 告诉我们这是一个 Ethernet frame（以太网帧），里面是 IP packet（分组），里面是 TCP segment（报文段），里面是 HTTP GET。如果我点击每一个协议 header（首部），Wireshark 会在下方的"天书"（原始字节）中高亮出对应位置：WiFi 在最前，WiFi 里是 IP，IP 里是 TCP，而在 TCP 里面，我们能看到 HTTP GET 的文本本身！

### <span style="color:#c0392b;">Slide 8 · Encapsulation Flexibility（封装的灵活性）</span>

<span style="color:#2471a3;">**[section]**</span>

- 这个 encapsulation（封装）允许你**递归地**分层
- 例子：Virtual Private Network（虚拟专用网络，VPN）：
  HTTP（网页）应用数据 → TCP Transport（传输层）segment（报文段） → IP Network（网络层）packet（分组） → 受保护的 TLS Presentation（表示层）消息 → TCP Transport（传输层）segment（报文段） → IP Network（网络层）packet（分组） → Ethernet Link（链路层）frame（帧）

Levis：这种"把协议封装进协议"的极简做法带来了极大的灵活性。到目前为止我们把四层模型讲得像一个完全静态、不可变的东西，但实践中并非如此。你真的可以用 encapsulation（封装）去**递归**地堆叠协议。例如，如今办公室和企业里非常常用的一个东西叫 Virtual Private Network（虚拟专用网络，VPN）。通过 VPN，你与一个你信任的网络（比如公司办公室）建立安全连接——例如用 Transport Layer Security（传输层安全协议，TLS）。之后当你访问互联网、发送 IP packet（分组）时，不是正常发送，而是把它们装进这条 VPN 连接里。于是这些 IP packet（分组）会先到达公司网络，再由公司网络正常路由。这让你可以访问公司里私有的、受保护的网络资源。你不必把网络安全防护撒得到处都是，只需要守好一个 service（服务）——让人们通过 Virtual Private Network（虚拟专用网络）登录网络的服务。具体做法是借助一台 **gateway（网关）**：一台接受经许可的 VPN 客户端连接、并把它们的流量转发进私有网络的计算机。

那么这长什么样呢？假设我要访问公司内部网站。我的浏览器生成一个 HTTP GET，照常把它装进 TCP segment（报文段），再把 TCP segment（报文段）装进一个目的地址为公司内部 Web 服务器的 IP packet（分组）。但我不能直接和内网服务器通信，所以我没有把这个 IP packet（分组）装进普通的 Link（链路层）frame（帧），而是把它装进了一个 TLS segment（报文段）——TLS 保护消息并对其保密。这条 TLS 会话又位于一条终止在 VPN gateway（网关）的 TCP 流里，所以这个"外层" TCP segment（报文段）装在一个目的地址为 VPN gateway（网关）的 IP packet（分组）里。我们再把外层 IP packet（分组）装进 Link（链路层）frame（帧），发给 next hop（下一跳）。于是它看起来是这样的：

HTTP 装在 TCP 里，TCP 装在 IP 里，IP 装在 TLS 里，TLS 装在 TCP 里，TCP 装在 IP 里，IP 装在 Ethernet frame（以太网帧）里。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> VPN 这个例子值得多想一步，它揭示了 encapsulation（封装）最不寻常的性质：层数不是物理世界的固定属性，而是可以按需"套娃"的逻辑结构。用 CSAPP ch11 的语言说，一条 TCP 连接对两端来说就是一条双向 byte stream（字节流）——至于是"裸"的应用数据在流里跑，还是"另一个 IP packet（分组）"在流里跑，TCP 全然不知也全然不管。tunneling（隧道）由此诞生：内层 IP packet（分组）在外层 TCP 流里穿越公网，从协议视角看，公网只是被当成了"一跳更长的 link（链路）"。这也解释了安全性设计上的一条朴素结论：与其保护路径上所有中间设备，不如把信任收敛到唯一的入口（VPN gateway（网关）），这本身就是一种"减少受攻击面"的第一性思考。你在后续课程会看到同样的手法反复出现——IPv6 过渡隧道、加密 DNS，本质都是"用 encapsulation（封装）把旧网络当作新网络的一条 link（链路）"。

### <span style="color:#c0392b;">Slide 9 · Encapsulation（小结）</span>

<span style="color:#2471a3;">**[section]**</span>

图示：七层（Layer 7 至 Layer 1）排列的横向分层图，展示各层封装后的 payload（载荷）。

- layering（分层）如何体现在数据的表示形式上
- encapsulated payloads（封装后的有效载荷）：
  - 有助于 separation of concerns（关注点分离）
  - 有助于强制边界/layering（分层）
  - 简化各层的实现

McKeown：现在你已经了解了 encapsulation（封装）——这个把 layering（分层）与 packet switching（分组交换）统一起来的原则。而 encapsulation（封装）就是我们把协议各层"组装"进 packet（分组）的手段，方式灵活，同时维持了各层关注点的分离。你看到了一个计算机如何封装网页请求的例子，也看到了如何用更复杂的方式使用 encapsulation（封装）——比如 Virtual Private Network（虚拟专用网络）。
