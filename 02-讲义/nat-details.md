# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Network Address Translation（网络地址转换，NAT）详解

> <span style="color:#7f8c8d;">Source: 慕课/Unit5-Applications-and-NATs/nat-details.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1：Network Address Translation（NAT）</span>

本讲深入网络地址转换（Network Address Translation，NAT）：它如何工作、有哪些类型、对应用意味着什么。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 2：强端到端原则</span>

（本 Slide 为图示：主机 171.64.15.55 与 157.166.226.26 通过 Internet（互联网）直接通信。）

> "The network's job is to transmit datagrams as efficiently and flexibly as possible. Everything else should be done at the fringes."
> （"网络的任务是以尽可能高效、尽可能灵活的方式传输数据报。其余一切都应该在边缘（fringes）完成。"）

这就是强端到端（strong end-to-end）原则：网络只做一件事——传数据报；可靠传输、加密、缓存、应用逻辑统统放在主机上。图中两台主机（171.64.15.55 与 157.166.226.26）中间只有一个"笨"互联网。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 3：网络地址转换器（RFC 1631）</span>

（本 Slide 为图示：在上述两端主机之间加入一台 NAT（网络地址转换器，Network Address Translator，RFC 1631），内部主机使用私有地址，NAT 对外呈现为单一 IP。）

RFC 1631 定义的网络地址转换器（NAT）是对强端到端原则的一次公开背离：它在路径中安插了一个**理解并改写**分组（而不仅仅是转发分组）的设备。私有地址空间（RFC 1918，如 10.0.0.0/8）的主机要上网，就得靠 NAT 把内部地址转换成共享的公网地址。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 4：NAT 示例拓扑</span>

（本 Slide 为图示：服务器 S（sshd 监听 22 端口，IP 18.181.0.31）位于公网；客户端 A（10.0.0.101）位于 NAT（128.34.22.8）之后；客户端 B（10.1.1.9）位于另一台 NAT（76.18.117.20）之后。）

后面的讲解将反复使用这个拓扑：服务器 S 运行 sshd（监听 22 端口），公网地址 18.181.0.31；客户端 A 的内部地址是 10.0.0.101，在 NAT（对外地址 128.34.22.8）之后；客户端 B 的内部地址是 10.1.1.9，在另一台 NAT（对外地址 76.18.117.20）之后。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 5：NAT 如何工作</span>

（本 Slide 为图示：A（10.0.0.101）发出分组，穿过 NAT（128.34.22.8）到达 S（18.181.0.31）；NAT 改写分组的源地址与源端口，并在内部表中记录映射。）

NAT 的工作方式：内部主机 A 发出的分组到达 NAT 时，NAT 把分组的源 IP 从 10.0.0.101 改写成自己的公网地址 128.34.22.8，把源端口改写成一个当前未占用的公网端口，并在内部映射表中记下这条对应关系。服务器 S 回复时把分组发回 128.34.22.8 的那个端口，NAT 查表，把目的地址/端口改写回 10.0.0.101 的原端口，转发给 A。**映射表的建立与查询，就是 NAT 的全部魔法。**

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 6：NAT 的类型（Types of NATs）</span>

（本 Slide 为分节标题页：Types of NATs——NAT 的类型。接下来的几页展示不同 NAT 在"放行谁"这个问题上的不同策略。）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 7：NAT 的内部映射</span>

（本 Slide 为图示：A（10.0.0.101）向 S（18.181.0.31）的 80 端口发出分组，五元组为 18.181.0.31 : 80 ← 10.0.0.101 : 4512；NAT（128.34.22.8）的映射表项为"10.0.0.101 : 4512 ↔ 128.34.22.8 : 6641"；改写后的分组五元组为 18.181.0.31 : 80 ← 128.34.22.8 : 6641。）

这是贯穿全讲的映射表示例：A 用内部端口 4512 连接 S 的 80 端口。NAT 在表中写入 `10.0.0.101 : 4512 ↔ 128.34.22.8 : 6641`，对外的分组看起来就像是从 128.34.22.8 的 6641 端口发出的。S 完全不知道 A 的存在——它以为自己在跟 128.34.22.8:6641 通信。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这张映射表正是 CSAPP ch11 五元组知识的"反面应用"。CSAPP 里内核用（协议、源 IP、源端口、目的 IP、目的端口）五元组把到达的 TCP 段分用（demultiplexing）到正确的套接字（socket，套接字）；NAT 干的事一模一样，只是把这张表从内核套接字表挪到了一个中间盒（middlebox）里：内部方向的键是（10.0.0.101, 4512, 18.181.0.31, 80），外部方向的键是（128.34.22.8, 6641, 18.181.0.31, 80）。端口号 16 位、单个公网地址可提供约 $2^{16}$ 个映射，这就是"一块钱的家宽路由器让几十台设备共享一个公网 IP"的数学基础。注意 NAT 改写的不只是 IP 还有端口——所以 NAT 严格说是 NAPT（网络地址端口转换）；而改写带来的代价也在这里埋下：TCP 校验和覆盖伪首部（含 IP 地址），NAT 每改一次地址就必须**重算校验和**，若还要求支持 FTP 这类在应用层 payload 里携带 IP 地址的古老协议，NAT 甚至得改写应用数据——理解这一点，就理解了为什么"端到端原则信徒"视 NAT 为眼中钉。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 8：两个问题</span>

- NAT 允许哪些分组穿过映射？
- NAT 如何、何时分配映射？
- NAT 的术语/分类见 RFC 3489

关于 NAT 有两个关键问题：什么样的分组允许穿过一条已建立的映射（从外到内的方向）？NAT 如何以及何时分配映射？这两个问题的不同回答，构成了 NAT 的类型学；RFC 3489 给出了标准术语与分类。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 9：全锥型 NAT（Full Cone NAT）</span>

（本 Slide 为图示：与 Slide 7 相同的映射表项"10.0.0.101 : 4512 ↔ 128.34.22.8 : 6641"；本图强调任何外部主机发往 128.34.22.8 : 6641 的分组都会被转发给 A。）

全锥型（full cone）NAT：一旦内部端口（10.0.0.101:4512）被映射到外部端口（128.34.22.8:6641），**任何**外部主机发往 128.34.22.8:6641 的分组都会被转发到内部。映射对"谁可以进来"没有任何限制——最宽松的 NAT。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 10：受限锥型 NAT（Restricted Cone NAT）</span>

（本 Slide 为图示：映射表项为"10.0.0.101 : 4512 ↔ 128.34.22.8 : 6641"，但表项中额外记录了外部对端 18.181.0.31；只有来自该对端的分组才被转发。）

受限锥型（restricted cone）NAT：映射建立后，NAT 额外记录首次出站分组的**目的 IP**（这里是 18.181.0.31）。此后，只有从 18.181.0.31 发往 128.34.22.8:6641 的分组会被转发进来；其他外部主机想进，门都没有——对"进来的人"做了来源 IP 级别的过滤。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 11：端口受限型 NAT（Port Restricted NAT）</span>

（本 Slide 为图示：映射表项记录了完整对端"18.181.0.31 : 80"；只有来自该 IP 且该端口的分组才被放行。）

端口受限型（port restricted）NAT：过滤再收紧一档——记录的是完整对端（IP + 端口），即 18.181.0.31:80。只有 S 从 80 端口发回的分组才能穿过映射。绝大多数家用路由器都工作在这个类型。过滤越严，安全性越高，但对等连接（P2P）也越难穿透。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 12：对称型 NAT（Symmetric NAT）</span>

（本 Slide 为图示：A（10.0.0.101:4512）与两个外部对端通信——对 S（18.181.0.31）用映射"10.0.0.101 : 4512 ↔ 128.34.22.8 : 6641"；对另一个服务器 S'（18.181.0.32，端口 3311）用**另一条**映射"10.0.0.101 : 4512 ↔ 128.34.22.8 : 9821"。）

对称型（symmetric）NAT：映射是**按（内部端口，外部对端）二元组**分配的——A 从内部端口 4512 连 S（18.181.0.31）时 NAT 分配外部端口 6641；同一个内部端口 4512 连 S'（18.181.0.32）时，NAT 分配一个**全新的**外部端口 9821。同一个内部端点，从外面看是两个不同的身份。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 13：NAT 行为建议</span>

- 更多复杂情形：静态映射（static mappings）、触发器（triggers）、更复杂的行为
- TCP 行为建议：RFC 5382
- UDP 行为建议：RFC 4787
- 回环（hairpinning）：从内部地址发往外部映射地址的分组被正确转换（内部映射到外部）

现实中的 NAT 还有更多复杂情形：静态映射（管理员手工配置的固定映射）、触发器、更复杂的行为。RFC 5382 给出 TCP 的 NAT 行为建议，RFC 4787 给出 UDP 的行为建议。回环（hairpinning）指的是：内部主机发往"自己 NAT 的外部地址 + 映射端口"的分组，也能被正确转换送回内部——听起来天经地义，但许多 NAT 实际上不支持。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 14：回环示例（Hairpinning Example）</span>

（本 Slide 为图示：NAT（128.34.22.8）下挂一个交换机（switch）与两台主机 A（10.0.0.101）、B（10.0.0.99）；映射表项"10.0.0.101 : 4512 ↔ 128.34.22.8 : 6641"。A 发往 128.34.22.8:6641 的分组需要被 NAT 正确"反射"回 A 自己。）

回环示例：A 想跟同处一个 NAT 之后的 B 通信，却只知道 B 的**外部**身份 128.34.22.8:6641。A 把分组发往这个地址；支持 hairpinning 的 NAT 会把分组绕一圈：转换成外部形式又立刻转换回内部形式，送达 10.0.0.101。不支持的话，这两个内部主机之间经外部地址的通信直接失败——这在语音/游戏等要交换候选地址的场景里是常见故障源。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 15：NAT 的影响（NAT Implications）</span>

（本 Slide 为分节标题页：NAT Implications——NAT 对应用的影响。）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 16：应用场景：入站连接（Incoming Connections）</span>

（本 Slide 为图示：A（10.0.0.101）运行 sshd，NAT 映射"10.0.0.101 : 4512 ↔ 128.34.22.8 : 6641"已存在；B（18.181.0.32）与 S（18.181.0.31）尝试从外部主动连入 128.34.22.8:6641。）

入站连接是 NAT 世界的头号痛点：映射是 A **主动出站**时建立的，NAT 天然只放行"这条出站流量的回程"。如果 A 本机运行着 sshd 想让别人来连，外部主机 B 连 128.34.22.8:6641 时——在受限型 NAT 上——B 的地址与记录的对端不符，分组被丢弃。**没有入站连接，主机就只能当客户端，不能当服务器**；这是后面"连接反转"与"中继"存在的根本原因。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 17：连接反转（Connection Reversal）</span>

（本 Slide 为图示：R（18.181.0.31）想连接位于 NAT 之后的 A（10.0.0.101）；解决方案是 R 把自己的意图告知 A，由 A 主动向 R 发起连接。）

连接反转（connection reversal）：外部对等方 R（18.181.0.31）想主动连接 NAT 后的 A。直接连是进不来的，于是角色对调：通过某个带外渠道（例如一台双方都可达的服务器）把 R 的存在告知 A，然后由 **A 主动**向 R 发起连接。A 的出站 SYN 顺利穿出 NAT 并建立映射，连接就此反转成功——前提是 R 有公网地址。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 18：中继（Relays）</span>

（本 Slide 为图示：R（18.181.0.31）与 A（10.0.0.101，NAT 之后）、B（192.168.2.5，另一台 NAT 之后）三方通信；引入一台公网中继服务器，A 与 B 各自向中继发起出站连接，流量经中继转接。）

如果通信双方**都**在 NAT 之后（A 是 10.0.0.101，B 是 192.168.2.5），连接反转也不够用了——谁也无法主动连进对方。最后的手段是中继（relay）：双方各自向一台公网中继服务器发起出站连接（出站总是可行的），之后所有流量都经中继转发。代价是双倍带宽、额外时延，以及中继服务器的运营成本——但对任何 NAT 组合它都有效。实际系统（如 TURN 协议）正是这么做的。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 19：应用场景：NAT 打洞（Hole-Punching）</span>

（本 Slide 为图示：Server（服务器 18.181.0.31）居中协调；NAT（128.34.22.8）后的 Client A（10.0.0.101）与 NAT（76.18.117.20）后的 Client B（10.1.1.9）各自向服务器注册，随后互相向对方的公网映射地址发包，在各自 NAT 上"打洞"。）

NAT 打洞（hole-punching）是比中继便宜的 P2P 方案：A、B 先都向公网服务器注册，服务器把 A 的外部映射（128.34.22.8:portA）告诉 B、把 B 的外部映射（76.18.117.20:portB）告诉 A；随后 A、B **同时**向对方的外部映射地址发包。A 的第一个包也许到不了 B（被 B 的 NAT 丢弃），但它已在 A 的 NAT 上开出一条"允许 B 的地址进来"的洞；B 的包反向同理。两边的洞一旦对上，直接 P2P 通道就建立了——服务器只参与撮合，不承担流量。UDP 上这套机制很成熟（STUN）；TCP 打洞更难，因为 NAT 的 TCP 映射行为严格得多。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 打洞/反转/中继这三板斧，本质是在"出站连接总是可行"这个唯一可靠前提下，重建"任意两台主机可互连"的图连通性。用 6.042J 的图论语言说：NAT 把互联网的连通图从"完全图"退化为"星形图"（每台 NAT 内的主机只能主动够到公网节点）；连接反转是把有向边翻转；打洞是双方各送出一条边、在中间"焊接"成双向边；中继则引入枢纽节点把星形重新拼成通路。时机上，打洞要求双方的第 1 个包几乎同时发出（各自 NAT 的洞存活期内互相抵达），这是分布式算法里典型的"没有全局时钟下的同步问题"——所以实战中要用服务器协调 + 重试序列（按指数退避间隔发 N 个包）来提高对上的概率。顺带一提，你天天在用的视频通话软件（微信/Zoom/WebRTC）背后就是这套机制：能打洞就 P2P（省服务器带宽），打不了就 TURN 中继（保连通）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 20：传输层：别发明新传输层！</span>

（本 Slide 为图示：与 Slide 7 相同的映射场景，强调 NAT 只工作在 IP/端口层面，不动传输层语义。）

面对 NAT 带来的种种麻烦，一个诱惑是"发明一个能理解应用的传输层协议来兼容 NAT"。本页的立场很明确：别。NAT 的问题应该在 NAT 的层面解决（RFC 4787/5382 规范其行为）、在应用层面绕过（打洞/中继），而不是往传输层塞新东西——传输层（TCP/UDP）的简洁与通用性是分层模型最值钱的部分。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 21：NAT 大辩论</span>

- 极其有用
  - 复用地址（reuse addresses）
  - 安全（不开放入站连接也可以是好事！）
- 极其痛苦
  - 让应用开发复杂化了很多
  - Speak Freely（Skype 之前的 VoIP！）
- 这场辩论有趣但没有意义：NAT 已经扎根，不会消失

NAT 大辩论的两面。极其有用：它让 IPv4 地址濒临耗尽的今天仍能运转（成百上千台设备复用一个公网地址）；它还意外地充当了防火墙——"不开入站连接"本身就是安全收益。极其痛苦：它让应用开发复杂化了一大截——Speak Freely（一款早于 Skype 的 VoIP 软件）就死在 NAT 穿透上，而 Skype 赢在打洞做得好。结论：这场辩论有趣但没有实际意义——NAT 已经无处不在，回不去了。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> NAT 是"端到端原则的实用主义胜利"的最佳案例。Slide 2 那句"其余一切应在边缘完成"在 1981 年的 RFC 796 语境下是金科玉律，但 IPv4 的 32 位地址空间（约 43 亿）在 1990 年代就显露出不够用的迹象——NAT 是 1994 年（RFC 1631）给出的**应急创可贴**，原计划用几年等 IPv6 交接；结果 IPv6 部署太慢，创可贴贴了三十年。这段历史的启示在于：工程世界里"临时方案"的寿命取决于它被依赖的速度，而不是它的设计寿命——一旦整个生态（防火墙、运营商、应用）围绕它生长，撤销的政治与技术成本就超过维持成本。这跟 CSAPP 里"为性能而引入的缓存往往变成正确性的负担"是同一种债。另外注意"安全收益"这条的反讽：NAT 的"安全"只是**默认拒绝入站**（没映射就进不来），并非真正的访问控制——UPnP 协议让应用能请求 NAT 开映射，恰恰证明这层"安全"挡不住会自己开洞的恶意软件。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 22：新的沙漏（The New Hourglass）</span>

（本 Slide 为沙漏图：原四层模型的沙漏颈部是 IP；"新沙漏"图中，颈部 IP 之上多出了一层——Session and Above（会话及以上）区域里标注了 ICMP、TCP、UDP，颈部被 HTTP 等应用层协议占据的趋势。图中文字：Session and Above（会话及以上）、ICMP / TCP / UDP、IP、Link and Below（链路层及以下）。）

这张"新沙漏"图提出了一个观察：互联网的传统沙漏以 IP 为颈部（所有上层协议都跑在 IP 上、IP 跑在所有链路上）；而如今，NAT 与防火墙让"IP 之上还有一层隐形的颈部"——分组不仅要符合 IP，还必须看起来像 TCP/UDP 的合法流（ICMP 反而被 many NAT 默吞），应用被迫迁就 NAT 的行为。颈部从 IP 实际下移到了"NAT 可通过的 TCP/UDP"。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 沙漏模型（hourglass）是课程第一单元的核心图景：颈部协议赢在**最小化**——IP 只提供"尽力而为的数据报"，任何链路都能实现、任何应用都能用，所以它能通吃上下两层。NAT 把这个逻辑倒过来了：它在 IP 之上加了第二道颈，而且这道颈**不是设计出来的**，是成千上万个厂商各自实现、行为互有差异的中间盒凑出来的。上层的应用被迫做的适配（STUN/TURN/ICE 打洞栈、HTTPS 全走 443 端口躲避端口过滤）正是"窄颈"的代价。这个视角也能解释今天的新趋势：QUIC 故意长成"长得像 UDP 的传输层"，就是为了穿过这道只认 TCP/UDP 的隐形颈——新协议与其说服全球升级中间盒，不如伪装成旧协议走既有的洞。第一性原理结论：协议生态里，颈部之争从未结束，只是从链路层搬到了应用层。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 23：NAT 操作细节（NAT Operation Details）</span>

（本 Slide 为分节标题页：NAT Operation Details——NAT 操作细节，指向两份 RFC。）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 24：NAT 内部映射（图示重现）</span>

（本 Slide 为图示，与 Slide 7 相同：A（10.0.0.101:4512）→ NAT（128.34.22.8）映射表"10.0.0.101 : 4512 ↔ 128.34.22.8 : 6641"→ S（18.181.0.31:80）。）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 25：UDP 行为规范（RFC 4787）</span>

- RFC 4787

UDP 的 NAT 行为由 RFC 4787 规范：它定义了映射行为（endpoint-independent / address-dependent / address-and-port-dependent）、过滤行为、端口保留、hairpinning 等标准术语，并规定 UDP 映射必须有至少几分钟的保活超时——这正是 P2P 应用每隔二三十秒发 keepalive 包的原因。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 26：TCP 行为规范（RFC 5382）</span>

- RFC 5382

TCP 的 NAT 行为由 RFC 5382 规范：包括何时建立映射（SYN）、何时可以放宽过滤、以及著名的 RST/FIN 处理与 TCP 保活超时（建议至少 2 小时 4 分钟）。两份 RFC 合起来，把 Slide 8 提出的"两个问题"（放行谁、何时建映射）变成了可测试的标准——NAT 一致性测试套件至今仍在使用它们。

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 5-1/5-2/5-3《NATs - Introduction/Types/Implications》的配套 slides；NAT 打洞的完整流程（STUN/TURN/ICE）在课程讨论区与课后阅读中展开，IPv6 是地址短缺的长期解法（见 Unit 6 names-ipv6）。</span>
