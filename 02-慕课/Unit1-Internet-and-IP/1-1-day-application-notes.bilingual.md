> <span style="color:#7f8c8d;">Source: Stanford CS144 Fall 2019 慕课讲义 · Unit1-Internet-and-IP/1-1-day-application-notes.pdf · 行内双语翻译版</span>

# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> <span style="color:#c0392b;">1-1 A Day in the Life of an Application（一个应用的一天）· 网络应用</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Networked Applications（网络应用）</span>

最终让网络变得有趣的，是使用网络的应用。Dave Clark——互联网设计的关键贡献者之一——曾写道（出自 1988 年的论文《The Design Philosophy of the DARPA Internet Protocols》，即 DARPA 互联网协议的设计哲学）：网络当前的指数级增长似乎表明，**connectivity（连接性）本身就是回报**，它比任何单个应用——比如 email（电子邮件）或 World-Wide Web（万维网）——都更有价值。**<span style="color:#00838f;">connectivity（连接性）指的是：世界上不同角落的两台计算机可以彼此连接并交换数据。</span>** 当你把自己的计算机接入互联网，你就突然可以与互联网上所有其他计算机对话——好吧，至少是那些也愿意与你对话的计算机。下面我们来看这究竟意味着什么，以及一些现代应用——万维网（World Wide Web）、Skype 和 BitTorrent——是如何使用它的。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">网络应用的基本模型</span>

网络应用的能力图示：两台计算机分处"Internet（互联网）"云的两端，各自运行程序并通过网络交换数据。

- 在网络上读、写数据
- <span style="color:#00838f;">**主流模型：bidirectional（双向的）、reliable（可靠的）byte stream connection（字节流连接）**</span>
- 一方读取另一方写入的数据
- 双向皆可读写
- 可靠（除非连接本身断掉）

网络应用的巨大威力在于：你可以让多台计算机——每台拥有自己的私有数据、可能由不同的人拥有和控制——彼此交换信息。本地应用只能访问驻留在本地系统上的数据，而网络应用可以跨世界交换数据。举例来说，设想用浏览器阅读一本杂志：出版商运行的服务器拥有全部文章，可能还存有过去每一期的存档；文章被修订或增补时，你能立刻看到新版本和新内容。全部历史存档对你来说可能太大了没法一次下载，于是你可以按需加载。如果没有网络，就得有人给你寄一张装着最新一期的 DVD 或 U 盘。

所以基本模型是：两台计算机各自运行一个程序，这两个程序通过网络通信。最常用的通信模型是**双向的可靠字节流**：计算机 A 上的程序 A 写入数据，数据穿过网络，计算机 B 上的程序 B 便能读到它；反过来程序 B 写数据、程序 A 读，亦然。当然还有其他通信模式（课程后面会讲），但可靠的、双向的字节流是目前最常见的。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Byte Stream Model（字节流模型）</span>

图示：左右两台计算机（A 与 B）隔着 Internet 云建立一条连接，双向箭头表示数据可以沿两个方向流动。

走一遍这个模型：右边的计算机 B 等待其他计算机来连接，它可能是一台 web server（Web 服务器）。左边的计算机 A 想与 B 通信，在这个例子里它是一部跑着浏览器的手机。A 与 B 建立一条 connection（连接）。此后，A 往连接里写数据，数据穿过网络，B 就能读到；B 写数据、A 读，同理。任何一方都可以 close（关闭）连接。比如浏览器请求完数据后可以关闭连接；服务器愿意的话也可以主动关闭。如果你在浏览器里见过 "connection reset by peer"（连接被对端重置）的报错，那就是这个含义：服务器在浏览器没有预料到的时候关闭了连接。服务器当然也可以拒绝连接：你大概见过 connection refused（连接被拒绝）的提示，或者浏览器长时间等待——因为服务器连拒绝都不回。

本课程后面你会学到这一切在底层是如何实现的；现在只从应用视角考虑它就够了：**<span style="color:#00838f;">即"两个程序通过网络可靠地读写数据"这一能力。</span>**

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">World Wide Web（万维网，HTTP）</span>

图示：Client（客户端）与 Server（服务器）隔着 Internet 云，构成典型的 client-server（客户端-服务器）交互。

看过网络应用的基本通信方式后，来看第一个例子：万维网。万维网使用叫做 HTTP（HyperText Transfer Protocol，超文本传输协议）的协议工作。当你在浏览器里看到 `http://` 时，就意味着它正在用 HTTP 通信。课程后面讲应用时我们会深入 HTTP 的细节，这里只给一个高层概览。

在 HTTP 中，client（客户端）打开一条到 server（服务器）的连接并向它发送命令。最常见的命令是 GET，用来请求一个页面。HTTP 被设计成一种 document-centric（以文档为中心）的程序间通信方式。举例来说，如果我在浏览器地址栏输入 `http://www.stanford.edu/`，浏览器就打开一条到服务器 `www.stanford.edu` 的连接，并对站点的根页面发送一个 GET 请求。服务器收到请求，检查它是否有效、用户是否有权访问该页面，然后发送 response（响应）。响应带有一个数字状态码。比如服务器对 GET 回以 `200 OK` 响应，就意味着请求被接受、响应其余部分是文档数据。对 Stanford 主页这个例子来说，`200 OK` 响应里装的就是描述 Stanford 主页的超文本。还有其他种类的请求，比如 PUT、DELETE 和 INFO，也有其他响应，比如 `400 Bad Request`。

因为 HTTP 以文档为中心，客户端的请求会指名一个文件。HTTP 全部用 ASCII 文本写成：人类可直接阅读。例如，对 Stanford 的 GET 请求开头长这样：`GET / HTTP/1.1`；一个成功响应的开头则是 `HTTP/1.1 200 OK`。

但基本模型很简单：客户端往连接里写一个请求，服务器读请求、处理它，再把响应写进连接，客户端随后读取响应。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">BitTorrent（比特流）</span>

图示：多台 Client（客户端）彼此互联并围绕一个 Tracker（追踪服务器）组成网络。

第二个应用：BitTorrent，一个让人们共享和交换大文件的程序。与"客户端向服务器请求文档"的 Web 不同，BitTorrent 中客户端向**其他客户端**请求文档。为了让一个客户端能同时向多个对象并行请求，BitTorrent 把文件切分成叫做 piece（块）的数据单元。当一个客户端从另一个客户端完整下载到一块 piece 后，它就告诉其他客户端"我有了这块"，别人便可以再来下载。这些协作的客户端集合被称为 swarm（集群/蜂群），所以我们说一个客户端加入或离开某个 swarm。

BitTorrent 使用的机制与万维网完全相同：可靠的、双向的数据流——只是用得更复杂。客户端想下载文件时，先要找到一个 torrent file（种子文件）；通常你用万维网找到它，并且（你猜对了）用 HTTP 下载它。种子文件描述了你想下载的数据文件的一些信息，还告诉 BitTorrent 这个种子对应的 tracker（追踪器）是谁。tracker 是一个记录（这正是它名字的由来）哪些客户端是 swarm 成员的节点。要加入一个 torrent，你的客户端（同样通过 HTTP）联系 tracker，请求一份其他客户端的列表；然后你的客户端与其中一些客户端建立连接，开始请求文件的各个 piece；对方反过来也可以向你请求 piece。此外，当新客户端加入 swarm 时，它可能被指引来连接你的客户端。于是，不同于"一个客户端对一个服务器"的单条连接，你得到的是一张在客户端之间动态交换数据的稠密连接图（dense graph of connections）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "swarm 是一张稠密图"这个说法值得用 6.042J 的图论语言精确化：把每个在线客户端看作 vertex（顶点）、每条打开的 TCP 连接看作 edge（边），BitTorrent 的运行过程就是这张图的动态生成与边权（带宽分配）调度问题。种子文件 + tracker 解决的是"初始连通性"——tracker 告诉你图里有哪些顶点可连；而"下载完一块就广播"则是典型的贪心信息传播，与图论里广度优先搜索的层级扩散同构。更有意思的是健壮性：Tracker 只是"介绍人"，即使它宕机，已建立的客户端子图仍可继续交换数据——这正好演示了"连接性（connectivity）本身是回报"这句话：应用的价值分布在整张对等图上，而不集中于一个服务器节点。Clark 在 1988 年前后写下那句话时，指的正是这种"接入即受益"的网络外部性（network externality）：连接的效用随节点数增长，这与你在概率论中学过的期望的叠加性不同，它是一种结构性的、超线性的价值增长。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Skype</span>

图示：Client A 与 Client B 隔着 Internet 云直接相连。

第三个也是最后一个应用：Skype，广受欢迎的语音、聊天和视频服务。Skype 是专有系统，没有任何关于其内部工作方式的官方文档。2008 年，Columbia 的一些研究者通过观察 Skype 客户端在何时何地发送消息，大致摸清了它的工作原理——不过消息是加密的，他们无法看到内部。2011 年，Efim Bushmanov 对该协议做了 reverse engineering（逆向工程）并发布了开源代码，于是我们对它的机制有了更好的了解。

在其最简单的模式下，你呼叫某人时就是一次简单的 client-server（客户端-服务器）式交互，有点像 HTTP：你作为呼叫方，打开一条到接收方的连接；如果对方接受呼叫，你们就开始交换语音、视频或聊天数据。

某些方面这看起来像万维网的例子：一方打开到另一方的连接并交换数据。但与"一方是客户端、一方是服务器"的 Web 不同，Skype 的场景里**双方都是客户端**。于是不再是"个人计算机向专用服务器请求东西"，而是"两台个人计算机互相请求数据"。这个差异最终对 Skype 的工作方式产生了非常大的影响。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Skype with Complications（有麻烦的 Skype）：NAT</span>

图示：Client B 位于一个 NAT（Network Address Translator，网络地址转换器）之后，从红色一侧发起的连接进不来，绿色一侧发出的连接畅通。

麻烦来自叫做 NAT（Network Address Translator，网络地址转换器）的东西。今天 NAT 无处不在：一台家用小型无线路由器就是 NAT；手机接入互联网时也处于 NAT 之后。课程后面会详细讲 NAT，现在你只需要知道：**<span style="color:#00838f;">如果你位于 NAT 之后，你可以向外主动打开连接，但互联网上的其他节点无法轻易地向你打开连接。</span>** 在这个例子里，这意味着 Client B 可以自由地向外建连，而别人却很难连进来。图中的红-绿渐变表达的就是这个：从绿色一侧来的连接畅通无阻，从红色一侧来的连接则不行。

所以麻烦在于：如果 Client A 想呼叫 Client B，它无法打开连接。Skype 必须绕过这个问题。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Skype 的解法一：Rendezvous（会合服务器）</span>

图示：Client B 先与一个 Rendezvous server（会合服务器）建立连接，Client A 通过服务器向 B 转达呼叫请求。

Skype 用叫做 rendezvous server（会合服务器）的东西解决它。当你登录 Skype 时，你的客户端会打开到一组控制服务器网络的连接。在这个例子里，Client B 打开一条到 rendezvous server 的连接——这没问题，因为服务器不在 NAT 之后，而 Client B 向外建连也毫无障碍。当 Client A 呼叫 Client B 时，它向 rendezvous server 发一条消息；由于服务器与 Client B 之间有一条已打开的连接，服务器就告诉 B "A 有一个呼叫请求"，呼叫对话框随即在 Client B 上弹出。如果 Client B 接受了呼叫，就由**它**打开一条到 Client A 的连接。Client A 本来想连 B，但 B 在 NAT 后面连不上；于是改为向一台 B 已经连着的计算机发消息，由它请 B 回头连向 A。由于 Client A 不在 NAT 之后，这条连接可以正常建立。这叫做 reverse connection（反向连接），因为它逆转了预期的建连方向：A 想连 B，结果却是 B 连 A。

Skype 之所以需要这一手，是因为 Skype 客户端通常是个人机器，而可公开访问的 web server 很少位于 NAT 之后——你希望全互联网都能访问服务器，把它放在 NAT 后面是坏主意。所以连向 Web 服务器很容易；而个人计算机出于安全等原因常常在 NAT 之后，因此 Skype 必须引入一些新的通信模式来绕开它们。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Skype 的解法二：Relay（中继）——双方都在 NAT 后怎么办</span>

图示：Client A 与 Client B 各在一个 NAT 之后，双方都无法直接连到对方。

那如果两个客户端都在 NAT 后面呢？反向连接也没用了：A 连不上 B，B 也连不上 A。

为此 Skype 引入了第二种服务器，叫做 relay（中继）。Relay 不能在 NAT 之后。如果 Client A 和 Client B 都在 NAT 后面，它们就通过一个 relay 通信：两者都打开到 relay 的连接；当 Client A 发送数据时，relay 沿着 B 已打开的那条连接把数据转发给 B；同样，Client B 发数据时，relay 沿着 A 打开的连接转发给 A。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">总结：Application Communication（应用间通信）</span>

- Bidirectional, reliable byte stream（双向、可靠的字节流）
- 当今大多数应用的 building block（构件）
- 其他模型也存在且被使用，课程后面会讲到
- 它把整个网络抽象掉了——只剩两段程序之间的一根"管道"；应用层自行控制通信模式与 payload（有效载荷）
- 三个例子：World Wide Web（HTTP）、Skype、BitTorrent

总结一下：我们已经看到网络应用最常见的通信模型——可靠、双向的字节流。它让运行在不同计算机上的两个程序得以交换数据，把整个网络抽象成一种简单的读写关系。

虽然这是一个非常简单的通信模型，却能以极富创造性的复杂方式使用。我们看了三个例子：万维网是 client-server 模型——客户端向服务器开连接、请求文档，服务器返回文档；BitTorrent 是 peer-to-peer（对等）模型——成群的客户端互相开连接交换数据块，织成一张稠密的连接网络；Skype 则是两者的混合——能直连时以对等方式直接通信，连不上时退而求其次，借助 rendezvous 或 relay 服务器。

于是你可以看到：一个看似简单的抽象——双向可靠数据流——能以多少有趣的方式被使用。只需改变"程序如何开连接、各自做什么"，就能造出从文档检索、集群下载到 IP 电话的复杂应用。比如 BitTorrent 的 tracker 与客户端的数据和角色截然不同，正如 Skype 除客户端外还有 relay 和 rendezvous 服务器。

当然，互联网在持续演化。谁知道下一个惊艳的应用会是什么？它的通信模式也许与这三个都不同，但几乎可以肯定：它仍会在某种新配置下使用双向、可靠的字节流。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 🎥 *Levis 在视频中解释： "是这种想法，即世界上的两个计算机可以在不同的部分连接并交换数据"*（字幕来源：002 - p1 1-1 A day in the life of an application.zh.srt）。这节课与 CSAPP ch11 是天然的镜像：ch11 里你用 `socket()` + `connect()` + `accept()` 搭起的正是这里说的"双向可靠字节流"，而且 ch11 的小型 Web 服务器也实现了 HTTP 的 GET/`200 OK` 子集——所以本页的 HTTP 例子你其实已经用 C 写过一半。值得注意的是本节给出的三个应用是同一抽象的三种拓扑用法：client-server（星形，中心是服务器）、peer-to-peer（稠密图，人人皆节点）、混合型（星形控制面 + 对等数据面）。判断一个应用属于哪种拓扑，只需问"谁主动向谁发起连接"——这个视角在下一节讲 NAT 时会变得至关重要，因为 NAT 恰恰限制了"谁有资格发起连接"这个动作。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> NAT/rendezvous/relay 这三页是 Unit 1 里"端到端连通性被打破后应用如何自救"的第一手案例。用第一性原理拆解：TCP 连接的本质是"由哪一方发出 SYN（同步位）决定方向"，NAT 的默认行为是"只允许由内向外的第一个 SYN 打通洞"，于是"可达性（reachability）"变成一种不对称资源。三个解法分别对应三个层次：reverse connection 是利用"方向可逆"（B 向外连 A 顺理成章）；rendezvous 是引入一个双方都可达的第三方做信令（signaling）交换；relay 则是彻底放弃直连、把数据面也搬到第三方。你在 ch11 写服务器时从未思考过这些问题，是因为服务器拥有公网 IP、天然在"绿色一侧"。等到 Unit 5 正式讲 NAT 时你会发现：本节的三种打洞技巧正是现代 WebRTC、视频会议和 P2P 网络仍在使用的标准模式，只是名字换成了 STUN（会合）与 TURN（中继）。
