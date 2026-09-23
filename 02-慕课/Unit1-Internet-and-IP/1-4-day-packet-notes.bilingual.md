> <span style="color:#7f8c8d;">Source: Stanford CS144 Fall 2019 慕课讲义 · Unit1-Internet-and-IP/1-4-day-packet-notes.pdf · 行内双语翻译版</span>

# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> <span style="color:#c0392b;">1-4 A Day in the Life of a Packet（一个分组的一天）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">总览：数据流的四层旅程</span>

互联网的四层模型是这样工作的：application layer（应用层）交出一段数据流（stream），transport layer（传输层）把这段流切成 segment（报文段），并可靠地投递给运行在另一台计算机上的应用；传输层把这些 segment 作为 network layer（网络层）的 packet（分组）发出，由网络层投递到对方计算机。我们来看看这在实际中是什么样子——一个 web 浏览器和 web 服务器真正收发的那些 packet。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">TCP 字节流与 three-way handshake（三次握手）</span>

图示：Client（客户端）与 Server（服务器）隔着 Internet 云建立一条 TCP 连接。

先看传输层。几乎所有 Web 流量都走 TCP（Transmission Control Protocol，传输控制协议）。在典型运作中有一端是 client、一端是 server：服务器 listen（监听）连接请求；要打开连接，客户端发出一个连接请求，服务器对此回应。具体细节这里不展开，但事实证明这个交互需要三条消息，这就是所谓的 "three way handshake"（三次握手）：

- **第一步：** 客户端向服务器发送一个 "synchronize"（同步）消息，通常称为 SYN。
- **第二步：** 服务器回应一个同样也是同步、但还确认（acknowledge）了客户端同步的消息——"synchronize and acknowledge"，通常称为 SYN-ACK。
- **第三步（最后一步）：** 客户端回应确认服务器的同步，通常称为 ACK（确认）。

所以三次握手常被描述为 "synchronize, synchronize and acknowledge, acknowledge"，或 "SYN, SYN-ACK, ACK"。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 🎥 *Levis 在视频中解释： "我们将看到如何我的Web浏览器与布朗Web服务器建立TCP连接"*（字幕来源：005 - p4 1-4 A Day in the Life of a Packet.zh.srt）。三次握手你在 CSAPP ch11 的 `open_clientfd`/`open_listenfd` 里已经"用过但没看见过"：你调用 `connect()` 时，内核替你发出 SYN；对端内核收到后回 SYN-ACK 并把连接放进 accept 队列；你的 `connect()` 返回意味着第三次握手的 ACK 已经发出。为什么是三次而不是两次？用第一性原理推一遍：握手的目标是双方都确认"我能发、你也能发"这条双向信道可用。两次握手只能让客户端确认双向可用，服务器却无法确认自己发的 SYN-ACK 被收到；更要命的是，网络里迟到的旧 SYN（这是 IP 尽力而为服务的自然产物）会让服务器误建一个无人使用的连接。三次握手让服务器也获得对信道的确认，并顺带让双方交换初始序列号（sequence number）——这正是后续"按序交付"的坐标原点。SYN/SYN-ACK/ACK 三个名字将贯穿整个传输层章节，现在把它们与"双向信道验证"绑定记忆最牢。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">IP 地址与 TCP 端口号：两个地址的分工</span>

图示：Client 标注 "IP address: 171.67.76.157, TCP port: 23946"；Server 标注 "IP address: 128.148.252.129, TCP port: 80"。

回忆一下：**网络层负责把 packet 送到计算机，而传输层负责把数据送到应用。** 从网络层的视角看，送往同一台计算机上不同应用的 packet 长得一模一样。这意味着，要向另一个程序打开一条 TCP 流，我们需要两个地址：第一个是 Internet Protocol address（IP 地址），网络层用它把 packet 送到计算机；第二个是 TCP port（TCP 端口），它告诉计算机的软件该把数据递交给哪个应用。Web 服务器通常运行在 TCP 端口 80 上。所以当我们连向一个 web 服务器时，我们向运行该服务器的计算机发送 IP packet，目的地址是那台计算机的 IP 地址，而这些 IP packet 里的 TCP segment 的目的端口是 80。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;"> hop（跳）与 router（路由器）</span>

图示：Client 与 Server 之间隔着若干 Routers（路由器）串联。

可这些 IP packet 是怎么到达目的地的呢？我们并没有一根从客户端直连服务器的导线。相反，我的客户端连接着一台中间计算机——router（路由器）；这台路由器又连着其他路由器。客户端与服务器之间的 IP packet 要经过许多 "hop"（跳），一跳就是连接两台路由器的一条链路。例如，我的客户端在 WiFi 网络上，所以第一跳是到 WiFi 接入点（access point）的无线链路；接入点与更广的互联网有有线连接，于是它把我的 packet 沿这条有线的一跳转发出去。

一台路由器可以连接许多条链路。每来一个 packet，路由器决定从哪条链路把它发出去。路由器自己也有 IP 地址，所以也存在"不转发、而是递交给自己软件"的情况：例如你用 TCP 登录一台路由器时，那些 IP packet 的目的地就是路由器自己的 IP 地址。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Forwarding Table（转发表）与默认路由</span>

（原图示例：一台路由器，address: 216.239.47.186，其转发表条目形如 "dest: default → link 1"、"pattern A → 5"、"pattern B → 2"、"pattern C → 4"、"pattern D → 2"、"pattern E → 6"、"pattern F → 1"。）

路由器如何做这个决定？靠的是 forwarding table（转发表），如右图所示。一张转发表由一组 IP 地址模式（pattern）和每个模式对应的转发链路组成。packet 到达时，路由器检查哪条表项的模式与这个 packet 匹配得最好，然后沿该表项的链路转发。一般来说，"最好"意味着**最具体的匹配**。这个匹配如何工作的细节，会在讲 longest prefix match（最长前缀匹配，LPM）的视频里展开；但在这个简单例子里，我们只看 default route（默认路由），即上表第一条。默认路由是最不具体的路由——它匹配**每一个** IP 地址。当 packet 到达时若找不到比默认路由更具体的表项，路由器就直接用默认路由。

默认路由在 edge network（边缘网络）里特别有用。比如你是 Stanford 大学，有一台路由器把你连到更大的互联网：你的路由器会为 Stanford 网络内的 IP 地址保留许多具体路由——"去工学院的包走这条链路""去图书馆的包走那条链路"；但如果目的 IP 地址不属于 Stanford，那它就该被发往更广的互联网。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "IP 地址 + 端口号"这组两层编址是网络世界里最优雅的分工设计之一，你其实在 CSAPP ch11 已经用过它的另一面：`getaddrinfo` 返回的 `sockaddr_in` 结构里正是 `sin_addr`（32 位 IP 地址）与 `sin_port`（16 位端口号）两个字段。用职责分离的语言重述：网络层只需回答"哪台主机"（host-to-host），于是全网的路由器只需根据 IP 地址转发，完全不必理解上面跑着什么应用；传输层再回答"哪个进程"（process-to-process），靠端口号在主机内做 demultiplexing（分用）。这个设计有个漂亮的推广：两层编址让应用空间与网络空间解耦——同一台服务器换程序（换端口）、或同一程序迁移主机（换 IP），另一维都不受影响。而转发表与默认路由的关系，你可以类比 CSAPP 里的虚拟内存机制：TLB 未命中后走页表，页表缺失最终落到"所有人都一样"的默认处理；边缘路由器的具体路由是"本地知识"，默认路由则把"我管不着的一切"托付给上层的互联网核心——逐级外包，直到 packet 汇入骨干网。后面的 longest prefix match 会告诉你"最具体优先"在硬件上如何高效实现。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Hood 之下：Wireshark 实测</span>

- 从 www.cs.brown.edu 请求一个网页
- 用 wireshark 观察 TCP 字节流的建立与数据交换
- 用 traceroute 观察 packet 穿过互联网的路径

现在我们来看真实网络中的 IP packet。我将请求 www.brown.edu 的一个网页，并用一个叫 Wireshark 的工具展示所有 packet。我们会看到我的浏览器如何用 SYN、SYN-ACK、ACK 的三次握手与 Brown 的 web 服务器建立 TCP 连接，然后开始发出 HTTP GET 请求、服务器返回数据。看完客户端与 Brown 大学 web 服务器之间的 packet 交换后，我会用另一个工具 traceroute 来观察这些 packet 穿过互联网的路径。

先启动 wireshark。由于我的计算机同时跑着许多网络应用、发送大量不同的 packet，我会告诉 wireshark 只显示"发往 Brown 服务器 80 端口的 TCP segment"，这样就只看到我产生的 Web 流量。我还要让 wireshark 监听 en1——我的 Mac 给 WiFi 链路层起的名字。你可以看到我有好几个链路层可用，但只看 en1，因为我就是通过它连网的。

接着我打开浏览器，请求 Brown 大学计算机系（这是我的本科母校，所以我喜欢关注系里的新闻）的网页。你能在 wireshark 里看到：加载这个页面涉及收发大量 packet！wireshark 显示每个 packet 的时间戳、源 IP 地址、目的 IP 地址、所用协议、长度和更多信息。看第一个 packet：它从我的计算机（地址 192.168.0.106）发往 Brown CS 的 web 服务器（地址 128.148.32.12），目标是 TCP 端口 80——服务器上的 HyperText Transport Protocol 端口，这从 Info 列的 `> http` 字段可以看出来。这个 packet 是 SYN packet——三次握手的第一步。

再看前三个 packet：第一个是从我的计算机到 web 服务器的 SYN；第二个是 web 服务器回到我计算机的 SYN-ACK；第三个是我计算机回给 web 服务器的 ACK。这就是三次握手！现在两台计算机可以交换数据了，你能看到第一个数据 packet 就是 HTTP 请求——我的计算机向 web 服务器发 GET 请求。对这个 GET 的响应是三个 packet——wireshark 在收到第三个时才显示响应，即 Info 列为 HTTP/1.1 200 OK 的那一行。于是我们亲眼看到：向 Brown CS 服务器请求一个网页，如何通过三个 IP packet 的三次握手建立 TCP 连接，再用更多 packet 完成 HTTP 请求与响应。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Traceroute：看 packet 在网络层内部走过的路</span>

以上是终端主机（计算机）在网络层收发 packet 时看到的网络。那网络层内部是什么样子？这些 packet 走了哪些 hop？为此我用第二个工具：traceroute。traceroute 显示发往某目的地的 packet 经过了哪些 hop。我们输入 `traceroute www.cs.brown.edu` 就能看到穿过互联网的路径；加上 `-w` 标志可以指定 timeout（超时），这里设为 1 秒。

packet 的第一跳是我的无线路由器（IP 地址 192.168.0.1）。从下一跳能看出我在家——我有 cable modem（电缆调制解调器），ISP 是 Astound。之后 packet 再跳到地址为 74.14.0.3 的路由器；再下一跳是加利福尼亚旧金山的一台路由器，然后是 San Jose 的几台路由器——sjc 属于 above.net、sanjose1 属于 level3.net。过了 sanjose1.level3.net，packet 就已经横穿美国到纽约了！它们穿过纽约的一系列路由器——ebr、csw、ebr——然后在第 13 跳到达波士顿。波士顿离 Brown 所在的 Providence 很近。在 oshean.org 之后我们看到三颗星 `* * *`——意思是有一台路由器不肯向 traceroute 报告自己。星星是 traceroute 表达"等待回复但超时了"的方式。第 20 跳我们看到 Brown CS 系的一台路由器；再往后一切都隐藏了——Brown CS 不想让你看清它网络内部的样子。

因为没能看到路径延伸到 Brown 的 web 服务器，我们换一个试试：MIT 的计算机科学与人工智能实验室（CSAIL）。可以看到到第 15 跳为止 packet 走的是同一条到波士顿的路：去 Brown 的路径在第 15 跳转去 oshean，而去 MIT 的路径继续留在 level3 的网络里。到 www.csail.mit.edu 的路径上只有两台路由器隐藏着（第 13 和第 19 跳）。我们还能看到 www.csail.mit.edu 也叫 akron.csail.mit.edu；经过 22 跳，从我计算机发出的 packet 抵达了 MIT 的 web 服务器。看那些时间值——我的 packet 到达 MIT web 服务器、其响应返回给我的时间，一来一回，即 RTT（round-trip time，往返时延）——不到 90 毫秒，比一次眨眼还快。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">结语：原则就在你每天的流量里</span>

我们现在已经看完了一个 packet 的一生：它作为应用层的客户端 Web 请求出发，穿过近 20 跳的互联网抵达目的地。对我而言，这是教这门课最快乐的事之一：我们讲的每样东西都是你和我每天都在打交道的东西——哪怕只是看这段视频的这会儿功夫！这些原理和思想很容易在实践中看到，用几个简单的工具，你就能实时观察互联网的运转！

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> traceroute 的原理值得一提，因为它巧妙地"反向利用"了 TTL 字段：traceroute 发送 TTL=1 的探测包，第一台路由器把 TTL 减到 0 后丢弃并回送一个 ICMP（互联网控制报文协议）超时报文——于是你就知道了第一跳的地址；随后发 TTL=2 的包获得第二跳，以此类推逐跳画出路径。这正好回扣上一节"IP 刻意简单、TTL 只为兜底"的设计：一个防循环的保险丝，被开发者挪用成了网络测绘工具。视频里那些 `* * *` 也有明确的语义：某些路由器对探测包既不响应也不转发 ICMP，或策略性地静默丢弃——这是真实互联网中"路由不可见性"的常态，也解释了为什么本例到 Brown 的路径结尾全被隐藏。测得的 RTT（往返时延）小于 90 毫秒、跨越大半个美国，你可以做个量级验算：光在光纤中约 200,000 km/s，Stanford 到 MIT 直线约 4300 km，单程光速下限约 21.5 毫秒，往返约 43 毫秒——实测 90 毫秒说明还叠加了每台路由器的处理与排队时延。这种"先算物理下限再分析开销"的估算习惯，是判断网络性能数据的行家手段，建议养成。
