# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> The Internet Control Message Protocol (ICMP) Service Model（互联网控制消息协议（ICMP）服务模型）

> <span style="color:#7f8c8d;">Source: 慕课/Unit2-Transport/2-3-icmp-service-model.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1：标题页</span>

本页为 CS144《计算机网络导论》（Introduction to Computer Networking）课程的标题页。本讲主题：互联网控制消息协议（ICMP）服务模型。主讲：Nick McKeown，斯坦福大学电气工程与计算机科学教授。

<span style="color:#7f8c8d;">（Slide 1/10）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 2：让网络层工作起来</span>

本讲属于"让网络层工作"（Making the Network Layer Work）主题，分三部分：

1. **互联网协议（Internet Protocol，IP）**——创建 IP 数据报（IP datagram）；逐跳（hop-by-hop）从一端交付到另一端。
2. **路由表（routing tables）**——填充路由器转发表（forwarding table）的算法。
3. **互联网控制消息协议（ICMP）**——在终端主机（end host）与路由器之间传递网络层信息；报告差错情况；帮助我们诊断问题。

<span style="color:#7f8c8d;">（Slide 2/10）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 3：ICMP 运行在网络层之上</span>

本页为分层封装图：协议栈（protocol stack）自上而下为应用层（application layer）、传输层（transport layer）、网络层（network layer）、链路层（link layer）。图示表明：ICMP 消息（ICMP Message）作为数据被封装在 IP 数据报中——"数据 + 首部（header）"构成 IP 数据报（IP datagram），再往下"数据 + 链路首部"构成链路帧（Link Frame）。也就是说，ICMP 报文携带的是**关于网络层的信息**，而它本身却搭载在 IP 之上运行。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 把 Slide 3 的封装图拆成字节，会看到一件不太体面的事：ICMP 报文自己的首部只有 4 个字节，其余内容几乎全是「别人分组的残骸」。RFC 792（1981 年 9 月，J. Postel 起草）规定的差错报文格式是：
>
> ```
>    +--------+--------+-----------+----------+
>    |  Type  |  Code  | Checksum  |  unused  |
>    +--------+--------+-----------+----------+
>    |   原始数据报的 IP 首部 + 原始数据报的前 64 比特   |
>    +--------------------------------------------+
>               ↓ 整条塞进一个 IP 数据报的数据区
>    +---------+--------------------------------+
>    | IP 首部 |          ICMP 报文              |   IP 首部的 protocol 字段 = 1
>    +---------+--------------------------------+
> ```
>
> 为什么要回贴原始首部，而不是只报一句「出错了」？因为差错消息的目的地是**源主机**，而源主机上可能同时有一百条连接在跑 —— 它必须知道这条消息说的是哪一条。答案就藏在被回贴的那段字节里：原始 IP 首部的源地址与目的地址，加上传输层前 8 个字节里的端口号，正好构成一个四元组，让内核按这个四元组把「这条差错」分派回具体的 socket（套接字），这正是 demultiplexing（解复用）在做的事。这与你在 CSAPP 第 11 章学过的 socket（套接字）解复用，是同一个思想在控制平面上的复刻：**所有需要归属的信息，必须随消息本身携带**。
>
> 代价也跟着来。任何改写地址的中继设备（NAT，网络地址转换）都必须同时改写这段内嵌首部，否则内网主机收到的差错消息对不上号 —— ICMP 差错穿过 NAT 比普通分组麻烦得多，RFC 5508 专门为 NAT 设备规定了处理 ICMP 的行为要求。而更常见的代价是：安全设备一眼看不出这段裸字节的用途，索性整条丢掉，下一段要说的 PMTUD 黑洞就是这笔账。
>
> 连接：这种「封套里装封套」的结构你其实很熟 —— CSAPP 第 3 章讲过程调用时，栈帧里要保存「调用者的现场」才能正确返回；这里是「报错时要保存出错者的现场」才能正确归属。两者的共同点在于：跨层传递的信息必须自带上下文，接收方才有资格无状态地处理它。
>
> 不妨把这条链路走一遍，看它如何落回一个具体的 socket。假设本机一个进程正用 TCP（Transmission Control Protocol，传输控制协议）向服务器的 443 端口发大包，本机端口 50000。某个中间路由器发现下一跳的 MTU 只有 1400 字节，于是丢掉这个 1500 字节的包，回一条 type 3 code 4 的 ICMP 报文，里面嵌着被丢包的 IP 首部与紧随其后的 8 个字节（源端口 50000、目的端口 443）。本机内核收到这条 ICMP 后，从内嵌首部里取出这个四元组，在 socket 表里查到对应的连接控制块，把「路径 MTU 变成 1400」记在这条连接上，之后这条连接就按更小的段长发包。整个过程应用一行代码都不用改 —— 前提是内核确实收到了那条 ICMP，而这条前提在真实网络里经常不成立。
>
> 一句话收束：ICMP 之所以能「运行在网络层之上却描述网络层」，靠的不是特权地位，而是这条把原始分组残骸一起打包带走的约定。

<span style="color:#7f8c8d;">（Slide 3/10）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 4：一个例子——"目的网络不可达"</span>

本页为例子图：源终端主机 A（运行 HTTP（HyperText Transfer Protocol，超文本传输协议）应用）经过两台路由器前往目的终端主机 B（运行 HTTP 应用）；途中某台路由器发现目的网络不可达，于是向 A 返回一条 "Destination Network Unreachable"（目的网络不可达）消息。图示说明了 ICMP 的典型使用场景：**路由器**在转发失败时，把网络层的差错信息报告给源主机——A 上的应用由此能得知"这次请求为什么没有到达"。

<span style="color:#7f8c8d;">（Slide 4/10）</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> Slide 4 的场景有一个隐含前提：路由器报的这条错，真的送到了 A。ICMP 最著名的工程伤口恰恰是它经常送不到 —— 这条伤口叫 PMTUD black hole（PMTUD 黑洞）。
>
> 先把机制铺开。PMTUD（Path MTU Discovery，路径 MTU 发现）是 RFC 1191（1990 年 11 月，Mogul 与 Deering）定义的技术：发送方在所有数据报上置 DF（Don't Fragment，禁止分片）位，按自己第一跳的 MTU 发尽量大的包；如果途中有链路的 MTU 更小，那台路由器既不能分片（DF 已置位）又不能转发，于是丢弃该包并回一条 ICMP type 3 code 4（fragmentation needed and DF set，「需要分片但 DF 已置位」）—— 这正是 Slide 6 表格里没有列出的那个 code，却是本讲「路由器向源主机报告差错」最要紧的实例。RFC 1191 还改进了这条消息：把报文里原本必须置零的 32 位 unused（未使用）字段征用为 Next-Hop MTU（下一跳 MTU），直接告诉发送方下游链路的 MTU 是多少。
>
> 收敛怎么走？若路由器只报「太大」而不给数字，发送方只能一档一档往下试。RFC 1191 的做法是维护一张常见 MTU 的 plateau（台阶表），把搜索压到这张小表上 —— 原文说最坏情况下收敛速度不差于 binary search（二分搜索）。这正好是你在 6.006 / CLRS 第 2 章学过的二分查找：搜索空间是离散的 MTU 候选，比较器是「丢包与否」这个 1 比特反馈，每一步把候选区间砍半。它之所以需要，是因为 ICMP 这条通道的带宽极低（每次丢包才换来一条消息），必须用对数轮数换取线性试探的浪费。
>
> 黑洞是这样形成的：如果路径上任何一台防火墙或路由器把 ICMP 错误消息丢掉 —— 很多管理员出于「ICMP 会被用来扫描」的顾虑整段封掉 —— 发送方就永远收不到这条 3/4，于是它继续按大 MTU 发包，每个包都在同一台路由器上被静默丢弃，丢包却没有任何反馈信号。RFC 2923（2000 年 9 月，Lahey，《TCP Problems with Path MTU Discovery》）把这个状态记了下来，原文的诊断特征极其锋利：「ping 和部分交互式 TCP 连接工作正常，大块传输在第一个大包上失败，连接最终超时」—— 也就是说，小包通、大包不通就是它的指纹。RFC 2923 给出的兜底叫 black hole detection（黑洞探测）：连接反复超时后关掉 PMTUD、清掉 DF 位、退回 536 字节的老默认值；工程上更常见的做法是在防火墙上放行 ICMP 差错消息，或在 TCP 握手时做 MSS 改写（MSS clamping，把通告的最大段长压到路径 MTU 以内）。
>
> 规范怎么站队？RFC 4890（2007 年 5 月，Davies 与 Mohacsi，ICMPv6 防火墙过滤建议）第 4.3.1 节把「必须不丢弃」的类别点名列出：Destination Unreachable（type 1）的全部 code、Packet Too Big（type 2）、Time Exceeded（type 3）的 code 0、Parameter Problem（type 4）的 code 1 与 2，以及 Echo Request/Response（type 128/129）。注意它是 ICMPv6 的文档 —— IPv6 里 PMTUD 那条消息叫 Packet Too Big、编号是 type 2，而 IPv4 里的对应物是 type 3 code 4，编号不同、机制同构。同一节还点出一句耐人寻味的话：在 IPv4 网络里丢弃 Echo Request 曾是常见做法 —— 连 ping 都被大规模封过。
>
> 连接：这是「控制平面被当成可选组件」的典型代价。数据平面（转发）照样工作，控制平面（反馈）被切断之后，系统不是变慢，而是从「慢」退化成「卡死」—— 因为发送方失去了唯一的反馈通道，只剩超时这一种极其迟钝的信号。同构的例子在 CSAPP 第 8 章：忽略 signal（信号）的处理，一个本该被打断的长循环会一直跑到结束；反馈被吞掉，系统就从事件驱动退化成超时驱动。
>
> 所以 Slide 5 那句「不可靠、不重试」不只是优雅的设计取舍，它同时也是整套机制最脆的 seam（接缝）—— 一旦有人替它「加固」，代价是整条路径上所有依赖 MTU 反馈的传输一起失效。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 5：ICMP 服务模型</span>

互联网控制消息协议（ICMP）的服务模型只有两行：

| 性质（Property） | 行为（Behavior） |
|---|---|
| 报告（Reporting） | 自包含（self-contained）的消息，报告某项差错 |
| 不可靠（Unreliable） | 简单数据报服务——不做重试（no retries） |

<span style="color:#7f8c8d;">（Slide 5/10）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 6：（部分）ICMP 消息类型</span>

这些 ICMP 报文由"类型（Type）+ 代码（Code）"二级编号区分含义（完整定义见 RFC 792，即 Request for Comments，互联网标准的编号文档）：

| ICMP 类型 | ICMP 代码 | 含义 |
|---|---|---|
| 0 | 0 | Echo 回复（Echo Reply，被 ping 使用） |
| 3 | 0 | 目的网络不可达（Destination Network Unreachable） |
| 3 | 1 | 目的主机不可达（Destination Host Unreachable） |
| 3 | 3 | 目的端口不可达（Destination Port Unreachable） |
| 8 | 0 | Echo 请求（Echo Request，被 ping 使用） |
| 11 | 0 | TTL（Time To Live，生存时间）过期（TTL Expired，被 traceroute 使用） |

<span style="color:#7f8c8d;">（Slide 6/10）</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> Slide 6 的表格只有六行，但「Type + Code」这个二级编号本身是个值得看清的设计决定。Type 占 8 位（256 个槽），Code 也占 8 位：这个结构把「粗分类」与「细原因」分成两跳。为什么不干脆用一个 16 位的平坦编号？因为不同读者需要的粒度不同 —— 防火墙的过滤规则、内核的分派 switch（分支分派）、监控的计数器，绝大多数只关心「这是不是差错消息、是不是那条必须放行的」；只有在真正诊断时，才需要知道「不可达」到底是网络不可达还是端口不可达。两级编码把「关心细节」的成本推给真正需要细节的那一方，这是一种 progressive decoding（渐进式解码）：读 1 个字节就能做正确决定，读 2 个字节就能做精细决定。
>
> 一手事实（RFC 792，1981 年 9 月）：Type 3 的 Code 有 0 = net unreachable、1 = host unreachable、2 = protocol unreachable、3 = port unreachable、4 = fragmentation needed and DF set、5 = source route failed；Type 11 有 0 = time to live exceeded in transit、1 = fragment reassembly time exceeded。本讲表格只挑了 0 / 1 / 3 三个，正因为它们是今天仍在大量出现的三个；而 3/4 那个（PMTUD 的关键）反而落在表外，这也是 TCP 排障时最常见的盲区。
>
> 把这几个编号还原成「谁在说、说什么」，记忆量会骤降：code 0 是中途的路由器说「这个网段我不知道怎么去」；code 1 是路由器说「网段我知道、但那台主机不在」；code 3 是目的主机自己说「包到了、但没有进程监听这个端口」—— IP 层把载荷交给 UDP，UDP 查端口表落空，于是回一条 code 3。最后这一条正是 traceroute 的收尾机制：Linux 的传统实现把探测包发往 33434 起的高端口，收到 code 3 就知道「到终点了」。Type 11 的 code 0 是「TTL 减到 0、我把它丢了」，traceroute 靠它逐跳钓出路径；code 1 则是「分片重组超时」，属于另一类故障（分片丢失）。
>
> 连接：这种「主操作码加子字段」的两级编码，你在 CSAPP 第 3 章见过它的机器码版本 —— x86 指令是 opcode（操作码）加 ModRM（寻址后缀字节）等字节，先决定「这是哪类指令」，再决定用哪种寻址形式操作哪个寄存器。同一思路也出现在 HTTP（HyperText Transfer Protocol，超文本传输协议）状态码里：三位数，首位分类、后两位细化。分类在前、细化在后，是让接收方可以只读第一层就安全行动的最小设计。
>
> 一个容易混淆的点值得点破：code 3（port unreachable）严格说不是「不可达」，而是「到了、但没人接」。它被归进 Type 3 只是因为历史上需要一个统一的差错容器；语义上，它由目的主机的传输层产生，与路由器毫无关系。这也解释了为什么它今天如此常见 —— 任何一次对关机主机的 UDP 探测、任何一次 traceroute 的收尾，都会打出这条消息。相反，code 2（protocol unreachable）几乎绝迹：它要的是「主机在跑，但内核里没有装载对应协议号的模块」，而在 TCP/UDP/ICMP 三者都是内核标配的今天，这种情况极少发生。读协议表时能分辨「哪个 code 还活着、哪个已经成了化石」，比背下十六个 code 有用得多。
>
> 收束：所以「Type + Code」不是历史包袱，而是把「谁都能处理」与「专家才能处理」分层的一种廉价实现 —— 它让一台只会读 Type 的老设备，也能正确地把 type 3 差错放行、把无关消息忽略。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 7："ping" 如何使用 ICMP</span>

本页为纯时序图（主机 A 与主机 B 之间的往返示意）：ping 的工作方式是——A 向 B 发送类型 8 的 Echo 请求，B 以类型 0 的 Echo 回复应答（类型对应见 Slide 6 的表格）。测量请求-回复的往返时间（round-trip time，RTT）即可估计连通性与时延。

<span style="color:#7f8c8d;">（Slide 7/10）</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> ping 只做一件事：送出一条 Echo Request（type 8，code 0），等一条 Echo Reply（type 0，code 0），报出两者的墙钟差。但这个「时间」的构成比看上去复杂：它包含两侧用户态与内核态之间的往返、两端内核的处理、两程的传播与排队、以及沿途每台路由器的转发时延。ping 给出的是端到端的 round-trip time（往返时间，RTT），你无法从这一个数字里把传播、排队、处理三项分开 —— 除非用不同大小的包做两三次实验去外推（大包多出的那一截基本上是序列化时间）。
>
> 更值得练的是「读哪一个统计量」。RTT 是随机变量，排队时延随负载起伏，所以 ping 的输出是一串样本。CS144 在本课程 4-3a（见 4-3a-single-AIMD-flow-worked-example）里明确要求取 ping 测得的最小值来当链路的传播时延 —— 理由是：最小值近似对应「队列恰好为空」的那次采样，也就近似是 bandwidth-delay product（带宽-时延积，BDP）里的那个时延项。这是极值统计而非均值统计：最小值是有偏但可解释的估计量，它估的是分布的下界；均值则把排队混了进来。给个可算的例子：十次采样若为 5.0、5.1、5.0、5.3、8.9、5.2、12.0、5.1、5.4、7.7 毫秒，均值约 6.5 毫秒、最小值 5.0 毫秒；用均值去乘带宽会把 BDP 高估三成，缓冲区就会按一条不存在的链路去配。
>
> 第二个陷阱是「ping 通不等于应用可用」。ICMP Echo 在很多路由器上是可选处理，且被限速 —— RFC 1812（1995 年，IPv4 路由器要求）第 4.3.2.8 节「Rate Limiting」（限速）要求路由器必须能限制 Source Quench（源抑制）的发送速率，并且应当能限制其他 ICMP 差错消息的速率。于是 ping 的丢包率并不等于链路的丢包率；更麻烦的是反方向：ping 通也不代表能传大包 —— 那正是前文所说的 PMTUD 黑洞的指纹（小包通、大包不通）。所以 ping 的结果只能证伪、不能证实：不通时基本可以断定路径断了，通时只能说控制平面可达。
>
> 连接：你在 CSAPP 第 11 章亲手写过应用层的 echo 服务器（客户端发字节、服务器原样送回），那是用 TCP 自己回显，可以顺带测吞吐；这里的 echo 由网络层的路由器或目的主机内核代劳，只能测可达性与小包往返。同一个「回显」动词，落在不同层就有完全不同的信息量 —— 这正是分层设计里最容易踩的直觉陷阱：看到「echo 通了」就以为「链路好了」。
>
> 最后一个实操细节：怎么用 ping 去怀疑 MTU。把载荷从小到大试，例如从默认的小载荷一路加到接近 1472 字节（以太网 1500 字节的 MTU 减去 20 字节 IP 首部与 8 字节 ICMP 首部），如果小载荷一路通、某个尺寸之后全部超时，那几乎可以断定路径上有一处更小的 MTU，而反馈被吞掉了 —— 这正是那种「浏览器打不开、但 ping 得通」的故障的指纹。判定时要留意：不通的那一档不能只看一次，因为 ICMP 本身会被限速，单次超时可能只是丢了这条控制消息，与数据面的丢包无关。这也是为什么网络排障的顺序总是「先用 ping（回显探测）确认控制平面、再用 TCP 大包确认数据面、最后才看 MTU」。
>
> 收束：ping 的正确用法是三件事一起看 —— 用最小值估传播时延、用方差估拥塞、用大小包的差异去怀疑 MTU。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 8："traceroute" 如何使用 ICMP</span>

本页为纯时序图（主机 A 沿途经多台路由器到主机 B 的示意）：traceroute 的工作方式是——A 依次发出 TTL（生存时间）逐次递增的探测包，每个包在 TTL 耗尽时被途经的路由器丢弃，该路由器随即返回类型 11 的 "TTL Expired"（TTL 过期）消息；A 收集这些消息，就能画出从 A 到 B 的逐跳路径（类型对应见 Slide 6 的表格）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 把协议缺陷变成工具的经典案例，正是 traceroute：TTL 每过一台路由器减一、减到零就丢弃并回送差错——这个机制本来是防止分组在路由环路里永久打转的安全网，traceroute 却故意把 TTL 设成 1、2、3……让每一跳路由器都"报错"，从而把沿途节点一个个钓出来。你可以在 CSAPP（Computer Systems: A Programmer's Perspective，《深入理解计算机系统》）的实现视角体会一下它的门槛：普通应用无法直接构造或接收 ICMP 报文，收发 ICMP 需要原始套接字（raw socket，类型常量为 SOCK_RAW）并通常要求 root（超级用户）权限——这就是为什么 Windows 上 `tracert` 与 Linux 的 traceroute 实现细节不同（有的改用 UDP（User Datagram Protocol，用户数据报协议）高端口探测、靠"端口不可达"类型的 ICMP 回报终点）。把 Slide 6 的表格与这两个工具对齐记忆：ping 用类型 8/0（Echo 请求/回复）测连通性与时延；traceroute 用类型 11（TTL 过期）测路径；"目的不可达"的三个代码（0/1/3）则是路由器向你报错的三种粒度。学完本讲，动手 `ping www.stanford.edu` 并观察输出，你会对"网络层信息如何回到终端主机"有直观感受。

<span style="color:#7f8c8d;">（Slide 8/10）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 9：总结</span>

互联网控制消息协议（ICMP）向终端主机和路由器提供关于网络层的信息。它位于 IP 之上，因此严格说是一种传输层机制。常用工具 "ping" 和 "traceroute" 都依赖 ICMP。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 本讲最值得咀嚼的细节，是"ICMP 位于 IP 之上、所以算传输层机制，但它报告的却是网络层的事"这个层间错位。对比第 2-2 讲中 UDP（用户数据报协议）的服务模型表你会发现两者几乎是同构的：都是"自包含消息 + 不可靠、无重试"的简单数据报服务。为什么差错报告自己偏偏设计成不可靠？可以做个思想实验：如果 ICMP 要可靠交付，那么当一条 ICMP 差错消息本身丢失时，就得再发一条差错消息去报告"差错消息丢了"，而后者也可能丢失……这是一个无穷递归，还会在网络故障时雪上加霜。把 ICMP 定为"尽力而为、不重试"，就切断了这条递归链。另外注意类型/代码的两级编码设计：类型 3 是"目的不可达"这个大类，代码 0/1/3 细分网络/主机/端口不可达——先分类再细化的编号方式，让你可以在只看类型字段时就做粗粒度处理，和中断向量表（interrupt vector table，把中断按类型编号、按编号查处理程序的设计）的设计思想一致。RFC 792 发布于 1981 年，与 IP 规范（RFC 791）同期，是最古老的互联网协议之一。

<span style="color:#7f8c8d;">（Slide 9/10）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 10：空白尾页</span>

<span style="color:#7f8c8d;">（Slide 10/10：本页无内容，为 PDF 尾页。）</span>

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 2-3《ICMP service model》的配套 slides（10 页）；IP 数据报与逐跳交付的机制详见"Making the Network Layer Work"系列后续讲义。</span>
