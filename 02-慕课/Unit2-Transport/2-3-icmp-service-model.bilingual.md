# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> The Internet Control Message Protocol (ICMP) Service Model（互联网控制消息协议（ICMP）服务模型）

> <span style="color:#7f8c8d;">Source: 慕课/Unit2-Transport/2-3-icmp-service-model.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1：标题页</span>

CS144: An Introduction to Computer Networks（CS144：计算机网络导论）。本讲主题：互联网控制消息协议（ICMP）服务模型。主讲：Nick McKeown，斯坦福大学电气工程与计算机科学教授。

<span style="color:#7f8c8d;">（Slide 1/10）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 2：让网络层工作起来</span>

本讲属于"让网络层工作"（Making the Network Layer Work）主题，分三部分：

1. **互联网协议（IP）**——创建 IP 数据报；逐跳（hop-by-hop）从一端交付到另一端。
2. **路由表（routing tables）**——填充路由器转发表（forwarding table）的算法。
3. **互联网控制消息协议（ICMP）**——在终端主机与路由器之间传递网络层信息；报告差错情况；帮助我们诊断问题。

<span style="color:#7f8c8d;">（Slide 2/10）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 3：ICMP 运行在网络层之上</span>

本页为分层封装图：协议栈自上而下为应用层、传输层、网络层、链路层。图示表明：ICMP 消息（ICMP Message）作为数据被封装在 IP 数据报中——"数据 + 首部（Hdr）"构成 IP 数据报（IP Datagram），再往下"数据 + 链路首部"构成链路帧（Link Frame）。ICMP 报文携带的是**关于网络层的信息**，而它本身却搭载在 IP 之上运行。

<span style="color:#7f8c8d;">（Slide 3/10）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 4：一个例子——"目的网络不可达"</span>

本页为例子图：源终端主机 A（运行 http 应用）经过两台路由器前往目的终端主机 B（运行 http 应用）；途中某台路由器发现目的网络不可达，于是向 A 返回一条 "Destination Network Unreachable"（目的网络不可达）消息。图示说明了 ICMP 的典型使用场景：**路由器**在转发失败时，把网络层的差错信息报告给源主机——A 上的应用由此能得知"这次请求为什么没有到达"。

<span style="color:#7f8c8d;">（Slide 4/10）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 5：ICMP 服务模型</span>

ICMP 的服务模型只有两行：

| 性质（Property） | 行为（Behavior） |
|---|---|
| 报告（Reporting） | 自含（self-contained）的消息，报告某项差错 |
| 不可靠（Unreliable） | 简单数据报服务——不做重试（no retries） |

<span style="color:#7f8c8d;">（Slide 5/10）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 6：（部分）ICMP 消息类型</span>

ICMP 报文由"类型（Type）+ 代码（Code）"二级编号区分含义（规范见 RFC 792）：

| ICMP 类型 | ICMP 代码 | 含义 |
|---|---|---|
| 0 | 0 | Echo 回复（Echo Reply，被 ping 使用） |
| 3 | 0 | 目的网络不可达（Destination Network Unreachable） |
| 3 | 1 | 目的主机不可达（Destination Host Unreachable） |
| 3 | 3 | 目的端口不可达（Destination Port Unreachable） |
| 8 | 0 | Echo 请求（Echo Request，被 ping 使用） |
| 11 | 0 | TTL 过期（TTL Expired，被 traceroute 使用） |

<span style="color:#7f8c8d;">（Slide 6/10）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 7："ping" 如何使用 ICMP</span>

本页为纯时序图（主机 A 与主机 B 之间的往返示意）：ping 的工作方式是——A 向 B 发送类型 8 的 Echo 请求，B 以类型 0 的 Echo 回复应答（类型对应见 Slide 6 的表格）。测量请求-回复的往返时间即可估计连通性与时延。

<span style="color:#7f8c8d;">（Slide 7/10）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 8："traceroute" 如何使用 ICMP</span>

本页为纯时序图（主机 A 沿途经多台路由器到主机 B 的示意）：traceroute 的工作方式是——A 依次发出 TTL（生存时间）逐次递增的探测包，每个包在 TTL 耗尽时被途经的路由器丢弃，路由器并返回类型 11 的 "TTL Expired"（TTL 过期）消息；A 收集这些消息，就能画出从 A 到 B 的逐跳路径（类型对应见 Slide 6 的表格）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> traceroute 是"把协议缺陷变成工具"的经典案例：TTL 每过一台路由器减一、减到零就丢弃并回送差错——这个机制本来是防止分组在路由环路里永久打转的安全网，traceroute 却故意把 TTL 设成 1、2、3……让每一跳路由器都"报错"，从而把沿途节点一个个钓出来。你可以在 CSAPP 的实现视角体会一下它的门槛：普通应用无法直接构造或接收 ICMP 报文，收发 ICMP 需要 `SOCK_RAW` 原始套接字并通常要求 root 权限——这就是为什么 Windows 上 `tracert` 与 Linux 的 traceroute 实现细节不同（有的改用 UDP 高端口探测、靠"端口不可达"类型的 ICMP 回报终点）。把 Slide 6 的表格与这两个工具对齐记忆：ping 用类型 8/0（Echo 请求/回复）测连通性与时延；traceroute 用类型 11（TTL 过期）测路径；"目的不可达"的三个代码（0/1/3）则是路由器向你报错的三种粒度。学完本讲，动手 `ping www.stanford.edu` 并观察输出，你会对"网络层信息如何回到终端主机"有直观感受。

<span style="color:#7f8c8d;">（Slide 8/10）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 9：总结</span>

ICMP 向终端主机和路由器提供关于网络层的信息。它位于 IP 之上，因此严格说是一种传输层机制。常用工具 "ping" 和 "traceroute" 都依赖 ICMP。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "ICMP 位于 IP 之上、所以算传输层机制，但它报告的却是网络层的事"——这个层间错位是本讲最值得咀嚼的细节。对比 2-2 的 UDP 服务模型表你会发现两者几乎是同构的：都是"自含消息 + 不可靠、无重试"的简单数据报服务。为什么差错报告自己偏偏设计成不可靠？可以做个思想实验：如果 ICMP 要可靠交付，那么当一条 ICMP 差错消息本身丢失时，就得再发一条差错消息去报告"差错消息丢了"，而后者也可能丢失……这是一个无穷递归，还会在网络故障时雪上加霜。把 ICMP 定为"尽力而为、不重试"，就切断了这条递归链。另外注意类型/代码的两级编码设计：类型 3 是"目的不可达"这个大类，代码 0/1/3 细分网络/主机/端口不可达——先分类再细化的编号方式，让你可以在只看类型字段时就做粗粒度处理，和中断向量表的设计思想一致。RFC 792 发布于 1981 年，与 IP 规范（RFC 791）同期，是最古老的互联网协议之一。

<span style="color:#7f8c8d;">（Slide 9/10）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 10：空白尾页</span>

<span style="color:#7f8c8d;">（Slide 10/10：本页无内容，为 PDF 尾页。）</span>

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 2-3《ICMP service model》的配套 slides（10 页）；IP 数据报与逐跳交付的机制详见"Making the Network Layer Work"系列后续讲义。</span>
