> <span style="color:#7f8c8d;">Source: 1-7-principle-encapsulation-notes.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#c0392b;">Principle: Encapsulation（原理：封装）</span>

### <span style="color:#c0392b;">Slide 1 · Principle: Encapsulation（原理：封装）</span>

<span style="color:#2471a3;">**[section]**</span>

McKeown：本视频讲解一个我们称之为**封装（encapsulation）**的架构原则。<span style="color:#00838f;">封装是"分层（layering）"与"分组交换（packet switching）"相结合的产物。</span>我们想把数据拆成离散的单元——分组（packet）——但每个分组里却装着来自多个层的数据。举例来说，当你发送一个 TCP 报文段（segment）时，它在 IP 分组里面，而 IP 分组又在以太网帧（Ethernet frame）里面。封装描述的正是这套机制如何运作：<span style="color:#00838f;">封装是这样一种原则——按它来组织分组中的信息，就能既维持各层之间的分离，又让它们共享同一个分组的载荷内容。</span>

### <span style="color:#c0392b;">Slide 2 · Layering（分层）</span>

<span style="color:#2471a3;">**[section]**</span>

图示：七层 OSI 模型的层叠图——自上而下依次为应用层（Application）、表示层（Presentation）、会话层（Session）、传输层（Transport）、网络层（Network）、链路层（Link）、物理层（Physical）。

- 关注点与职责的分离（separation of concerns）
- 让每一层的服务（service）可以独立演化
- 例子：
  - 传输层（Transport）：应用之间的通信
  - 链路层（Link）：同一条共享链路（link）上主机（host）之间的通信

Levis：回顾一下，分层让你能把一个复杂系统拆成较小的部分。每一层向上层提供一个**服务（service）**——即对网络的一种抽象（abstraction），而它自己则借助下一层来实现这个抽象。每一层都是自包含的（self-contained），所以只要它提供了上层所期望的服务，上层就不必关心它是怎么做到的。这种关注点分离（separation of concerns）意味着每一层可以独立演化。正如网络层的 IP 不需要关心传输层的 TCP 有什么改动，HTTP 这样的应用层协议也不必关心 TCP 的变化。比如近几年，大多数操作系统都调整了自己使用的 TCP 算法，以更好地应对不断增长的网速——但同一个网页浏览器，无论新旧算法都能正常工作。

注意：这张分层图用的是 7 层 OSI 模型。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "关注点分离"这个词你在 CSAPP 里其实已经体验过一次了：虚拟内存（ch9）把"程序要连续的地址空间"和"物理内存碎片化的现实"这两件麻烦事拆开，各自独立演化。分层网络做的是同一件事——传输层向上只承诺"应用间可靠字节流（reliable byte stream）"这个抽象。你在 CSAPP ch11 写 socket（套接字）客户端/服务器时，只用过 `connect()`、`read()`、`write()` 这几个调用，从未操心过 TCP 内部窗口大小（window size）怎么变、重传（retransmission）何时触发——这正是"只要服务兑现，上层无需关心实现"的直接证据。讲义里的例子换个角度说就是：内核把 TCP 算法升级了，你的浏览器二进制一个字节都不用改。学网络时不妨把每层都当成一个"接口与实现"的分离体来看，这与 CSAPP 中"抽象掩蔽实现"的思路一脉相承。

### <span style="color:#c0392b;">Slide 3 · 4 Layer Model（四层模型）</span>

<span style="color:#2471a3;">**[section]**</span>

图示：把七层模型压缩后的四层互联网模型——应用层（Application）、传输层（Transport）、网络层（Network）、链路层（Link）、物理层（Physical）。

- 关注点与职责的分离
- 让每一层的服务可以独立演化
- 例子：
  - 传输层：应用之间的通信
  - 链路层：共享链路上主机之间的通信

McKeown：所以，让我们把上面那 7 层"压缩"回四层模型。

### <span style="color:#c0392b;">Slide 4 · Encapsulation（封装）</span>

<span style="color:#2471a3;">**[section]**</span>

图示：四层模型旁画着一个分组，自外向内依次是各层的首部（headers）与数据（data）。

- 分层如何体现在数据的表示形式上
- 第 N 层的数据就是第 N−1 层的**有效载荷（payload）**
- 例子：HTTP（网页）应用数据 → 装在 TCP 传输层报文段里 → 装在 IP 网络层分组里 → 装在 WiFi 链路层帧（frame）里

McKeown：封装就是那条让我们能把协议各层"叠放"进同一个分组的存储空间、又互不干扰的原则。它是分层在真实数据表示上的体现。工作方式是：<span style="color:#00838f;">每个协议层都有自己的首部（header），其后是它的有效载荷（payload），再往后可能还有一个尾部（footer）。</span>例如，IP 分组首部里有源地址（source address）和目的地址（destination address）。要用 IP 发送一个 TCP 报文段，我们就把 TCP 段整体作为 IP 分组的载荷。这样，IP 分组就"封装"了 TCP 段。IP 根本不知道也不在乎自己的载荷是什么——它只负责把分组送到一台终端主机（end host）。分组到达后，主机查看载荷内容，发现里面是个 TCP 段，于是按 TCP 的规则处理它。

McKeown：下面是个更完整的例子。假设你用一台通过 WiFi（无线以太网）上网的电脑浏览网页。你的浏览器生成一个 HTTP GET 请求。这个 HTTP GET 请求是某个 TCP 段的载荷；这个封装着 HTTP GET 的 TCP 段又是某个 IP 分组的载荷；而这个封装着 TCP 段和 HTTP GET 的 IP 分组又是某个 WiFi 帧的载荷。如果你去看电脑实际发出的字节，它们就长这样：最外层的封装格式是 WiFi 帧，里面是 IP 分组，IP 分组里是 TCP 段，TCP 段里是 HTTP GET 的文本。

Levis：McKeown 画分组的方式引出了一个可能让你困惑的问题！画分组其实有两种画法，差别来自画图者的背景和他所研究的系统部位。McKeown 在这里把首部画在右边：分组的第一位（first bit）在最右，最后一位在最左。

McKeown：这完全说得通！当路由器（router）或交换机（switch）发送分组时，我们习惯把分组画成从左向右移动——那么最先离开设备的那一位自然画在最右边。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 用第一性原理看，封装就是一个递归嵌套的 `(header, payload)` 结构：每一层只读自己那段固定格式的前缀，然后把"剩下的字节"整体交给上一层协议去解释——IP 不解释载荷，正如 C 语言里你拿到一个 `void*` 指针后，必须靠约定去解读它指向的内容。从 CSAPP ch3 的内存视角看，这非常像结构体的嵌套布局：外层首部是低地址处的固定字段，载荷紧随其后，解析时按偏移量逐层下潜。Wireshark 之所以能一层层展开协议，就是因为它按这个嵌套约定逐级切割字节流。理解了"封装 = 字节流上的递归分区"，你就明白为什么任何一台新设备只要遵守这套格式就能接入互联网——设备不需要理解每一层的语义，只需要知道"我这一层的边界在哪、该把剩余部分交给谁"。这也是后续 Lab 里你亲手解析报文时会反复用到的思维模型。

### <span style="color:#c0392b;">Slide 5 · Encapsulation（封装：硬件视角）</span>

<span style="color:#2471a3;">**[section]**</span>

图示：与上一页相同的封装图，但标注为"Hardware: first bit on right（硬件习惯：第一位在右边）"，各层首部（headers）画在右侧。

- 分层如何体现在数据的表示形式上
- 第 N 层的数据就是第 N−1 层的有效载荷
- 例子：HTTP（网页）应用数据 → TCP 传输层报文段 → IP 网络层分组 → WiFi 链路层帧

Levis：但我画分组是另一种方向：首部在左，尾部（footers）在右。像这样。

### <span style="color:#c0392b;">Slide 6 · Encapsulation（封装：软件视角）</span>

<span style="color:#2471a3;">**[section]**</span>

图示：与上页相同的封装图，但标注为"Software: first bit on left（软件习惯：第一位在左边）"，各层首部画在左侧、尾部画在右侧。

- 分层如何体现在数据的表示形式上
- 第 N 层的数据就是第 N−1 层的有效载荷
- 例子：HTTP（网页）应用数据 → TCP 传输层报文段 → IP 网络层分组 → WiFi 链路层帧

Levis：这种画法来自软件界。你阅读 IETF 文档和许多其他协议规范时看到的就是它。想法是：分组的起始地址是地址零（address zero），所以一个首部的第一个字节位于地址零；由于地址从左向右递增，分组的开头就在左边，结尾就在右边。

McKeown：这里没有对错之分。两种画法都有价值，取决于你用图来做什么。两种你都得习惯。我一般把首部画在右边。

Levis：而我一般画在左边。McKeown 的背景是电气工程与交换机（switch）设计，我的背景是计算机科学与协议软件。

### <span style="color:#c0392b;">Slide 7 · Encapsulation（封装：两种画法等价，Wireshark 演示）</span>

<span style="color:#2471a3;">**[section]**</span>

（本页标题栏仍为"Encapsulation"，正文承接上页，含一个"="号，表示两种画法等价。）

Levis：现在回到 McKeown 的例子——WiFi 帧里的 IP 分组里的 TCP 段里的 HTTP GET。我们看看它在真实网络中是什么样子。开始录制之前，我打开了 Wireshark，抓取了一次网页请求的分组轨迹（packet trace）。只看其中一个分组：可以看到 Wireshark 告诉我们这是一个以太网帧（Ethernet frame），里面是 IP 分组，里面是 TCP 段，里面是 HTTP GET。如果我点击每一个协议首部，Wireshark 会在下方的"天书"（原始字节）中高亮出对应位置：WiFi 在最前，WiFi 里是 IP，IP 里是 TCP，而在 TCP 里面，我们能看到 HTTP GET 的文本本身！

### <span style="color:#c0392b;">Slide 8 · Encapsulation Flexibility（封装的灵活性）</span>

<span style="color:#2471a3;">**[section]**</span>

- 封装允许你**递归地**分层
- 例子：虚拟专用网络（Virtual Private Network, VPN）：
  HTTP（网页）应用数据 → TCP 传输层报文段 → IP 网络层分组 → 受保护的 TLS 表示层消息 → TCP 传输层报文段 → IP 网络层分组 → 以太网链路层帧

Levis：这种"把协议封装进协议"的极简做法带来了极大的灵活性。到目前为止我们把四层模型讲得像一个完全静态、不可变的东西，但实践中并非如此。你真的可以用封装去**递归**地堆叠协议。例如，如今办公室和企业里非常常用的一个东西叫虚拟专用网络（VPN）。通过 VPN，你与一个你信任的网络（比如公司办公室）建立安全连接——例如用传输层安全协议（Transport Layer Security, TLS）。之后当你访问互联网、发送 IP 分组时，不是正常发送，而是把它们装进这条 VPN 连接里。于是这些 IP 分组会先到达公司网络，再由公司网络正常路由。这让你可以访问公司里私有的、受保护的网络资源。你不必把网络安全防护撒得到处都是，只需要守好一个服务——让人们通过虚拟专用网络登录网络的服务。具体做法是借助一台**网关（gateway）**：一台接受经许可的 VPN 客户端连接、并把它们的流量转发进私有网络的计算机。

那么这长什么样呢？假设我要访问公司内部网站。我的浏览器生成一个 HTTP GET，照常把它装进 TCP 段，再把 TCP 段装进一个目的地址为公司内部 Web 服务器的 IP 分组。但我不能直接和内网服务器通信，所以我没有把这个 IP 分组装进普通的链路层帧，而是把它装进了一个 TLS 段——TLS 保护消息并对其保密。这条 TLS 会话又位于一条终止在 VPN 网关的 TCP 流里，所以这个"外层" TCP 段装在一个目的地址为 VPN 网关的 IP 分组里。我们再把外层 IP 分组装进链路层帧，发给下一跳（next hop）。于是它看起来是这样的：

HTTP 装在 TCP 里，TCP 装在 IP 里，IP 装在 TLS 里，TLS 装在 TCP 里，TCP 装在 IP 里，IP 装在以太网帧里。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> VPN 这个例子值得多想一步，它揭示了封装最不寻常的性质：层数不是物理世界的固定属性，而是可以按需"套娃"的逻辑结构。用 CSAPP ch11 的语言说，一条 TCP 连接对两端来说就是一条双向字节流——至于是"裸"的应用数据在流里跑，还是"另一个 IP 分组"在流里跑，TCP 全然不知也全然不管。隧道（tunneling）由此诞生：内层 IP 分组在外层 TCP 流里穿越公网，从协议视角看，公网只是被当成了"一跳更长的链路"。这也解释了安全性设计上的一条朴素结论：与其保护路径上所有中间设备，不如把信任收敛到唯一的入口（VPN 网关），这本身就是一种"减少受攻击面"的第一性思考。你在后续课程会看到同样的手法反复出现——IPv6 过渡隧道、加密 DNS，本质都是"用封装把旧网络当作新网络的一条链路"。

### <span style="color:#c0392b;">Slide 9 · Encapsulation（小结）</span>

<span style="color:#2471a3;">**[section]**</span>

图示：七层（Layer 7 至 Layer 1）排列的横向分层图，展示各层封装后的有效载荷。

- 分层如何体现在数据的表示形式上
- 封装后的有效载荷（encapsulated payloads）：
  - 有助于关注点分离
  - 有助于强制边界/分层
  - 简化各层的实现

McKeown：现在你已经了解了封装——这个把分层与分组交换统一起来的原则。封装就是我们把协议各层"组装"进分组的手段，方式灵活，同时维持了各层关注点的分离。你看到了一个计算机如何封装网页请求的例子，也看到了如何用更复杂的方式使用封装——比如虚拟专用网络。
