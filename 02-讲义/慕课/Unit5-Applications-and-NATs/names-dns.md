# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Domain Name System（域名系统，DNS）

> <span style="color:#7f8c8d;">Source: 慕课/Unit5-Applications-and-NATs/names-dns.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1：Domain Name System（DNS）</span>

本讲介绍域名系统（Domain Name System，DNS）：互联网把人类可读的名字映射到 IP 地址的机制。🎥 *CS144 教授在视频中解释: "这是关于域名系统或DNS的三个视频之一"*（字幕来源：079 - p78 5-8 DNS 1.zh.srt）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 2：解析一个 URL</span>

以 `http://cs144.scs.stanford.edu/labs/sc.html` 为例，URL 由三部分组成：

| 部分 | 内容 | 含义 |
|---|---|---|
| application protocol | 应用协议 | `http`——使用 HTTP 协议（默认 TCP 80 端口） |
| host | 主机 | `cs144.scs.stanford.edu`——要联系的服务器名字 |
| file and port | 文件与端口 | `/labs/sc.html`——请求的文件路径 |

🎥 *CS144 教授在视频中解释: "它由三个基本部分组成……它首先在前面，应用协议……名称的中间部分指定主机"*（字幕来源：079 - p78 5-8 DNS 1.zh.srt）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 3：解析一个 URL——纯 IP 版本</span>

同一个 URL 也可以写成 `http://171.66.3.9/labs/sc.html`：协议部分与文件部分不变，只是主机部分从"名字"换成了点分十进制的 IP 地址。这台机器不需要 DNS 就能定位服务器——这恰好反衬出 DNS 存在的意义：名字是给人用的，IP 是给机器用的，二者之间需要一套映射系统。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 4：HOSTS.TXT——DNS 之前的世界</span>

- 最早，所有主机都记录在一个文件 HOSTS.TXT 中，由网络信息中心（Network Information Center）维护
  - 维护在主机 SRI-NIC.ARPA（26.0.0.73）上（RFC 952）
- 各主机周期性地用文件传输协议下载新版本
  - 需要 $n^2$ 量级的网络容量，扩展性很差

在 DNS 之前，互联网（当时的 ARPANET）的所有主机名到地址的映射都放在一个文件里：HOSTS.TXT，由位于 SRI 的网络信息中心统一维护（主机 SRI-NIC.ARPA，地址 26.0.0.73，见 RFC 952）。每台主机定期通过文件传输协议拉取新版本。问题显而易见：全网都要从同一台机器拉同一个不断变大的文件——更新走单一机构是瓶颈，$n^2$ 量级的拉取开销（每台主机都下载整份 $n$ 主机的表）是流量瓶颈，NIC 一挂全网就没法解析新名字是可用性瓶颈，想给自己的机器起名还得排队审批是管理瓶颈。下一页 DNS 的四条需求，恰好一一对应这四个瓶颈。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 5：域名系统的需求</span>

- 把名字映射到地址（更一般地，映射到各种值）
- 必须能处理海量的记录
- 必须支持分布式控制：人们可以控制自己的名字
- 必须对个别节点的故障健壮（robust）

DNS 要满足的需求：把名字映射到地址（更一般地说，映射到各种值）；必须能处理数量巨大的记录；必须支持分布式控制——每个组织能管理自己的名字；必须对个别节点的失效具有健壮性（robustness）。对照上一页 HOSTS.TXT 的四个瓶颈：海量记录靠分级 delegation（委托），分布式控制靠区域自治，容错靠每个区域多副本服务器——DNS 的整个架构就是这四行需求的直接展开。而"从单一全局表到分层命名空间"的演化，与 CSAPP 里"用一个全局锁保护所有数据"演化到"分桶 + 细粒度锁"是同构的：当数据规模和写入者数量增长时，分而治之是唯一可持续的路线，代价是一致性变复杂——这正是下一页要讲的。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 6：DNS 的设计权衡</span>

- 两个性质使 DNS 的设计可行：
  - 只读（read-only）或读多写少（read-mostly）的数据库：主机查找名字的频率远高于更新名字
  - 松一致性（loose consistency）：变更可以花一点时间传播
- 这两个性质允许大规模缓存（caching）：
  - 查一个名字，把结果保留很长时间

DNS 设计之所以可行，靠两个关键性质：其一，这是一个只读或读多写少的数据库——名字的查询频率远高于更新频率；其二，松散一致性（loose consistency）——变更允许花一点时间在全系统传播。这两个性质合起来，允许系统做大规模的缓存（caching）：查一次名字，把结果保留很久。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "读多写少 + 松一致性 ⇒ 可缓存"是一个可以推广到几乎所有系统设计的一般性定理。为什么读多写少可缓存？因为缓存的失效成本与写频率成正比：若每秒写一次，缓存几乎永远陈旧；若一年写一次，缓存命中率为 $1 - O(\text{写频} \times \text{TTL})$，接近 1。为什么必须接受松一致性？CAP 视角下，跨广域网的强一致复制要么牺牲可用性要么牺牲分区容忍性，而 DNS 的作者选择了 AP——名字解析是浏览一切的前置步骤，"暂时拿到旧 IP"远比"完全解析不了"可接受。TTL（time to live）字段就是这套折中的调节旋钮：TTL 大，缓存收益高但变更生效慢；TTL 小则相反。这个权衡你在 CSAPP 里其实做过类似的决策：多核缓存一致性协议（MESI）是硬件层面"强一致缓存"，代价是总线通信；而 DNS 选了软状态（soft state）路线——数据自带过期时间，过期重取，无需失效通知。两种路线的取舍值得对照记忆。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 7：DNS 名字架构</span>

（本 Slide 为树状图：根之下依次是顶级域 `edu`、`com`、`org`、`us`、`fr`、`cn`；`edu` 下有 `stanford`、`berkeley`，`com` 下有 `cisco`、`google`、`ietf`、`acm`、`yahoo`，`fr` 下有 `lemonde`，`cn` 下有 `baidu`；再往下一层是 `cs`、`www`、`maps`、`cs`、`www` 等主机名。）

DNS 的名字空间（namespace）是一棵树：根之下是顶级域（top-level domains）如 `edu`、`com`、`org`、国家码如 `us`、`fr`、`cn`；其下是二级域（`stanford`、`berkeley`、`cisco`、`google`、`ietf`、`acm`、`yahoo`、`lemonde`、`baidu`）；再往下是子域与主机名（`cs`、`www`、`maps` 等）。一个完整名字如 `www.cs.stanford.edu` 就是从叶子到根的路径，用点分隔。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 8：DNS 服务器</span>

- 分级的区域（zone）："根"区域（root zone）、`edu`、`stanford`、`scs` 等
- 每个区域可以单独管理
- 每个区域由若干台复制（replicated）的服务器共同服务
- 根区域：13 台服务器，高度复制（命名为 a、b、c、…、m）
  - 引导（bootstrap）：根服务器 IP 存放在名字服务器上的一个文件里
  - 通过任播（anycast，课程后文讨论）复制

DNS 服务器按区域（zone）分级组织：根（root）区域、`edu` 区域、`stanford` 区域、`scs` 区域……每个区域可以由不同的机构独立管理；每个区域由多台复制的服务器共同服务。根区域有 13 台服务器（按字母命名 a 到 m），并做了高度复制。引导（bootstrap）问题——你怎么知道第一台根服务器的 IP？——靠的是名字服务器上预置的根缓存文件；根服务器还通过任播（anycast，课程后文讨论）在世界范围复制同一 IP。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 9：DNS 根服务器</span>

（本 Slide 为世界地图图示，标注全球 13 个根服务器系统（A–M）与其镜像节点的地理分布，图注出处为 http://en.wikipedia.org/wiki/File:Root-current.svg 。）

这张图展示了 13 个 DNS 根服务器系统（a.root-servers.net 到 m.root-servers.net）在全球的分布。借助任播（anycast），每个"字母"实际上可以在全球许多地点复制，共享同一个 IP 地址。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 10：一次 DNS 查询</span>

- 两类查询：
  - 递归（recursive）
  - 非递归（non-recursive）
  - 由查询报文中的一个比特指明
- UDP 53 端口
  - 报文上限 512 字节
- 也可用 TCP 53 端口
  - 报文前缀加 16 位长度字段

（本 Slide 亦含图示：client（客户端）→ resolver（解析器）→ root（根 `.`）→ top level domain (TLD，顶级域 `.com`、`.edu`、`.cn`、`.de`) → domain（域，如 `stanford.edu`、`baidu.cn`）的层级查询路径，客户端问 resolver "www.stanford.edu?"。）

DNS 查询分两类：递归（recursive）与非递归（non-recursive），由查询报文里的一个标志位区分。默认传输用 UDP 53 端口，报文大小上限 512 字节；也可以改用 TCP 53 端口，此时每个报文前要加一个 16 位的长度字段。查询层级为：客户端 → 本地解析器（resolver）→ 根（`.`）→ 顶级域（TLD，如 `.com`、`.edu`、`.cn`、`.de`）→ 二级域（如 `stanford.edu`、`baidu.cn`）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 递归与迭代的区分是本讲最容易被面试问到的知识点，结合 CSAPP ch11 会非常清晰：你调用 `getaddrinfo("www.stanford.edu", ...)` 时，你的主机对本地解析器（校园网分配的 DNS 服务器）发出的是**递归查询**——"你必须给我最终答案"；而解析器对根、.edu、stanford 的服务器发出的是**非递归（迭代）查询**——每次只得到"下一个该问谁"的推荐（referral），一步步逼近答案。为什么要这样分工？把递归查询限制在客户端与本地解析器一跳，根/TLD 服务器就只需回答"问谁"而不必替全网跑腿——13 个根集群才能扛住全世界的解析流量；而迭代路径上的每个中间答案都可被解析器缓存，这让上一页的"大规模缓存"设计真正落地。512 字节的 UDP 上限也是个有趣的历史化石：DNS 报文当年被设计成恰好装进一个 IP 包（避免 IP 分片），这个约束后来催生了 EDNS0 扩展与"响应过大则转 TCP"的规则——协议演化中的每一道补丁都对应一条物理约束。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 11：DNS：查询与资源记录</span>

（本 Slide 为分节标题页：DNS Queries and Resource Records——DNS 查询与资源记录，本节起深入查询的报文格式与记录结构。）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 12：一次 DNS 查询（图示重现）</span>

（本 Slide 为图示，与 Slide 10 的层级图相同：client（客户端）向 resolver（解析器）发出递归查询 "www.stanford.edu?"；解析器依次向 root（根 `.`）、TLD（`.com`、`.edu`、`.cn`、`.de`）、domain（域 `stanford.edu`、`baidu.cn`）发出非递归查询，经 Internet（互联网）逐级获得 referral（推荐）。）

🎥 *CS144 教授在视频中解释: "然后解析器可能要么从其缓存中回答……或者使用缓存条目。如果解析器没有查询任何阶段的缓存条目，它可以向外部服务器提问"*（字幕来源：080 - p79 5-9 DNS 2.zh.srt）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 13：资源记录（Resource Records）</span>

- 所有 DNS 信息都用资源记录（Resource Records，RRs）表示：

```text
name [TTL] [class] type rdata
```

  - name：域名（如 `www.stanford.edu`）
  - TTL：生存时间（秒）
  - class：为扩展性保留，通常为 IN（1，Internet）
  - type：记录类型
  - rdata：依赖于 type 的资源数据
- 两个最关键的 RR 类型：A（IPv4 地址）记录与 NS（name server，名字服务器）记录
- `dig` 工具可以查询

所有 DNS 信息都用资源记录（Resource Record，RR）表示，格式为：`name [TTL] [class] type rdata`。name 是域名；TTL 是该记录可被缓存的时间（秒）；class 是为扩展性保留的字段，几乎总是 IN（数值 1，代表 Internet）；type 是记录类型；rdata 是随类型变化的资源数据。最关键的两类记录是 A 记录（名字 → IPv4 地址）和 NS 记录（区域 → 名字服务器）。用 `dig` 工具可以直接查询它们。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 14：DNS 消息结构（RFC 1035）</span>

（本 Slide 为横向结构图：一条 DNS 消息由 Header（首部）、Question（问题区）、Answer（回答区）、Authority（权威区）、Additional（附加区）五段组成；其中 Question 之后的三个区在查询报文中为空——"Empty in queries"（查询报文中为空）。）

一条 DNS 消息（RFC 1035）由五段组成：首部（Header）、问题区（Question）、回答区（Answer）、权威区（Authority）、附加区（Additional）。查询报文中，Question 之后的 Answer、Authority、Additional 三段为空；它们出现在响应里。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 15：DNS 首部结构（RFC 1035）</span>

（本 Slide 为首部字段布局图，每行 16 bits：第 1 行为 ID（16 位）；第 2 行为 QR、OPCODE、AA、TC、RD、RA（中间夹 Z 与 RCODE）共 16 位；第 3–6 行依次为 QDCOUNT、ANCOUNT、NSCOUNT、ARCOUNT（各 16 位）。图中同时标注五段消息结构 Header / Question / Answer / Authority / Additional。）

- QR：0 = 查询，1 = 响应
- OPCODE：0 = 标准查询
- RCODE：错误码
- 标志位：
  - AA：权威回答（authoritative answer）
  - TC：被截断（truncated）
  - RD：期望递归（recursion desired）
  - RA：递归可用（recursion available）

首部各字段：ID 是报文编号，用于把响应和查询配对；QR 区分查询（0）与响应（1）；OPCODE 中 0 表示标准查询；RCODE 携带错误码。四个标志位：AA 表示这是权威服务器的回答；TC 表示响应因超过 512 字节被截断（此时应改用 TCP）；RD 表示客户端期望递归查询；RA 表示服务器可提供递归。四个 COUNT 字段分别给出 Question、Answer、Authority、Additional 四个区的条目数。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 16：DNS RR 结构（RFC 1035）</span>

（本 Slide 为资源记录在报文中的字段布局图：NAME、TYPE（16 bits）、CLASS（16 bits）、TTL、RDLENGTH（16 bits）、RDATA；图中同时标注五段消息结构。）

- NAME：DNS 名字
- TYPE：1 = A，2 = NS，……
- CLASS：1 = Internet
- TTL：生存时间（秒）
- RDLENGTH：RDATA 的长度
- RDATA：记录数据

每个资源记录在报文内编码为六个字段：NAME（DNS 名字）、TYPE（1 = A 记录，2 = NS 记录，等等）、CLASS（1 = Internet）、TTL（秒）、RDLENGTH（RDATA 的字节长度）与 RDATA（记录数据本身）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 17：DNS 名字压缩</span>

- 名字可能很长，且在同一个报文里重复出现多次
  - 查询/回答中；NS 记录/A 记录中
- 把名字切成标签（label）：`www.stanford.edu` 即 `www`、`stanford`、`edu`
- 每个标签编码为"长度 + 文本"：`3www`、`8stanford`、`3edu`
  - 长度是二进制数；文本是 ASCII：`3www` 即 0x03、0x77、0x77、0x77
- 若长度 ≥ 192，则接下来 14 位指明名字在报文中的偏移量
  - `0xc00c` 表示名字位于偏移 `0xc00c - 0xc000 = 0x0c = 12` 字节处

名字可能很长，而且在同一个报文里重复多次——查询里的名字会原样出现在回答里，NS 记录引用的名字也可能与 A 记录的名字重叠。DNS 的省字节技巧：把名字切成标签（label），`www.stanford.edu` 即三个标签 `www`、`stanford`、`edu`；每个标签编码为"1 字节长度 + ASCII 文本"，如 `3www`、`8stanford`、`3edu`（长度是二进制数、文本是 ASCII，`3www` 的字节序列就是 0x03、0x77、0x77、0x77）。压缩指针则更进一步：若标签长度字节的值 ≥ 192（即最高两位为 11），说明这不是长度，而是一个指针——接下来的 14 位给出名字在报文内的字节偏移。例如 `0xc00c` 表示名字在偏移 `0xc00c - 0xc000 = 12` 字节处。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这个压缩编码是"用高位做模式位"的经典工程手法，你能立刻联想到 CSAPP 里的数据表示练习：x86 指令的 ModRM 字节、ARM 条件码、UTF-8 的多字节序列，全都靠"最高几位当模式标记"来区分同一字段的不同解释——本质是用变长编码（variable-length encoding）节省空间，代价是解析器必须按字节位运算分派。0x03 `www` 这种"长度前缀 + 载荷"的格式你在 CSAPP 的 echo 协议 homework 里也见过（先读 4 字节长度再读 payload），DNS 只是把它压到 1 字节。指针 ≥ 0xc000 的设计还有个巧妙的约束：报文最长 512 字节，偏移永远落在 $0 \sim 511$，与 14 位指针的编码空间天然吻合；而"长度字节 < 64 才是真长度"（0xc0 = 192 起是指针）保证了两个语义空间不重叠。但要小心：指针可以指向指针，恶意报文可以构造压缩环让解析器死循环——2012 年前后多个 DNS 软件的漏洞正源于此，这是"格式即攻击面"的又一例证。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 18：DNS A 记录</span>

（本 Slide 为 A 记录在线缆上的实际编码图，消息分段标注为 Header / Question / Answer / Authority / Additional。）

一条 A 记录的文本形式与线上编码对照：

```text
market.scs.stanford.edu. 3600 IN A 171.66.3.10
```

| 字段 | 线上编码 | 含义 |
|---|---|---|
| NAME | `market.scs.stanford.edu`（可能被压缩） | 记录名 |
| TYPE | 1（A） | IPv4 地址记录 |
| CLASS | 1（IN） | Internet |
| TTL | 3600 | 可缓存 1 小时 |
| RDLENGTH | 4 | RDATA 为 4 字节 |
| RDATA | 0xab 0x42 0x03 0x0a | 即 171.66.3.10 |

RDATA 的 4 个字节 0xab、0x42、0x03、0x0a 换算成十进制正好是 $171.66.3.10$（$0xab = 171$、$0x42 = 66$、$0x03 = 3$、$0x0a = 10$）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 19：DNS NS 记录</span>

（本 Slide 为 NS 记录在线缆上的实际编码图，消息分段标注同上。）

一条 NS 记录的文本形式与线上编码对照：

```text
scs.stanford.edu. 3600 IN NS mission.scs.stanford.edu
```

| 字段 | 线上编码 | 含义 |
|---|---|---|
| NAME | `scs.stanford.edu`（已压缩） | 记录名（被查询的区域） |
| TYPE | 2（NS） | 名字服务器记录 |
| CLASS | 1（IN） | Internet |
| TTL | 3600 | 可缓存 1 小时 |
| RDLENGTH | 10 | RDATA 为 10 字节 |
| RDATA | `mission.scs.stanford.edu`（`scs.stanford.edu` 部分压缩） | 权威名字服务器的域名 |

注意 RDATA 里装的是一个**域名**而非 IP 地址——NS 记录只告诉你"这个区域的名字服务器叫什么名字"，至于这个服务器自己的地址，还得再查一次（由此引出 Slide 23 的 glue record 问题）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 20：DNS Wireshark 示例</span>

（本 Slide 为 Wireshark 抓包截图：展示一次真实的 DNS 查询/响应报文，可在其中看到首部标志位、Question 区的查询名字，以及 Answer 区返回的记录。）

在 Wireshark 里抓一次 DNS 交互，可以把前面学到的首部字段、资源记录、压缩指针全部对上号——建议自己抓一个 `dig` 的查询对照查看。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 21：DNS 细节（Details）</span>

（本 Slide 为分节标题页：DNS Details——DNS 细节，讨论区域遍历、glue records 与更多记录类型。）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 22：Details（图示重现）</span>

（本 Slide 为图示，与 Slide 10/12 相同的层级图：client（客户端）→ resolver（解析器）→ root（根 `.`）→ TLD → domain，经 Internet（互联网）逐级查询 "www.stanford.edu?"。）

🎥 *CS144 教授在视频中解释: "一个主要的挑战来自这个遍历区域的概念"*（字幕来源：081 - p80 5-10 DNS 3.zh.srt）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 23：遍历区域（Traversing Zones）</span>

- 引导（bootstrap）：本地名字服务器带有一个根缓存文件（指明根服务器）
  - 作为查询起点的一组名字/地址映射
- 递归问题：你怎么拿到名字服务器的地址？
  - 记住，NS 记录里装的是主机名：`argus.stanford.edu` 是 `stanford.edu` 的名字服务器
  - 你怎么联系 `argus`？
- 递归问题的解法：glue records（粘合记录）——父区域中的 A 记录
  - `.edu` 的名字服务器持有 `stanford.edu` 的 NS 记录
  - `.edu` 的名字服务器同时持有 `argus.stanford.edu` 的 A 记录

遍历区域的机制：本地名字服务器的引导靠一个根缓存文件（root cache file），它给出根服务器的地址——这是查询链的起点。

但有个递归难题：NS 记录的 rdata 是**主机名**而非 IP。`stanford.edu` 的 NS 记录说"名字服务器是 `argus.stanford.edu`"，可你怎么联系 `argus`？要查它的 A 记录，而它的 A 记录恰恰在 `stanford.edu` 区域里——如果只靠 NS 记录，查询会绕成一个环。

解法是 glue records（粘合记录）：父区域在持有子区域 NS 记录的同时，附带持有那些名字服务器的 A 记录。`.edu` 的名字服务器既持有 `stanford.edu` 的 NS 记录，也持有 `argus.stanford.edu` 的 A 记录——粘合记录打破了循环，让查询链可以逐级下探。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> glue record 解决的"鸡生蛋"问题在分布式系统里有个正式的名字：引导问题（bootstrap problem）——任何依赖查找表才能找到查找表的系统都会遇到。对照你熟悉的层级：CSAPP 里动态链接器 `ld-linux.so` 的地址写死在 ELF 的 interp 段（否则你不知道该加载哪个解释器）；6.042J 归纳法证明里必须有"最小情形"才能起步；DNS 的根缓存文件就是归纳的 base case，delegation 链是归纳步。注意 glue record 的一个微妙副作用：粘合记录是父区域的**副本**而非权威数据，子区域改了服务器地址、父区域的 glue 没同步时，就会出现"官方 NS 记录指向的地址是旧的"这种著名故障——这印证了 Slide 6 的"松一致性"是有代价的，一致性边界（zone 边界）与数据依赖边界不重合时，缓存副本就成了错误来源。🎥 *CS144 教授在视频中解释: "所以有一天，我的名称服务器有一个根缓存文件……这就只给了一些根服务器的IP地址，这就是启动过程"*（字幕来源：081 - p80 5-10 DNS 3.zh.srt）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 24：Glue Record 示例</span>

- 假设没有任何缓存，查询 `www.scs.stanford.edu`：

```text
dig +norec www.scs.stanford.edu @a.root-servers.net
dig +norec www.scs.stanford.edu @a.edu-servers.net
dig +norec www.scs.stanford.edu @argus.stanford.edu
dig +norec www.scs.stanford.edu @ns1.fs.net
```

假设无任何缓存，解析 `www.scs.stanford.edu` 的手工迭代过程是四步 `dig`（`+norec` 表示发非递归查询）：
1. 问根服务器 `a.root-servers.net`：得到 `.edu` 区域的名字服务器（ referral → `a.edu-servers.net`）；
2. 问 `.edu` 服务器 `a.edu-servers.net`：得到 `stanford.edu` 的 NS/glue（ → `argus.stanford.edu`）；
3. 问 `stanford.edu` 的服务器 `argus.stanford.edu`：得到 `scs.stanford.edu` 区域的名字服务器（ → `ns1.fs.net`）；
4. 问 `scs.stanford.edu` 的服务器 `ns1.fs.net`：得到 `www.scs.stanford.edu` 的 A 记录。

🎥 *CS144 教授在视频中解释: "想想NS记录……如果我问一个域名的名称服务器是什么，DNS记录包含一个主机名"*（字幕来源：081 - p80 5-10 DNS 3.zh.srt）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 25：CNAME 记录</span>

- 规范名字（canonical name）记录——告诉你一个名字是别名（alias）：

```text
name [TTL] [class] CNAME canonical-name
```

- 规范名下的任何记录也可以关联到别名上
- 示例：`dig www.stanford.edu`
- CNAME 排斥该名字下的任何其他记录（RR）
- 响应中可以带有规范名的其他记录

CNAME（canonical name，规范名字）记录声明"这个名字只是个别名"，格式为 `name [TTL] [class] CNAME canonical-name`。规范名（canonical name）下的任何记录，同样可以关联到别名上——示例：`dig www.stanford.edu` 会先返回 CNAME 指向真正的规范名。规则限制：一个名字一旦是 CNAME，就不能再有其他类型的记录；但响应里可以附带规范名的其他记录（如 A 记录），省得客户端再查一轮。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 26：MX 记录</span>

- 邮件交换（Mail eXchange）记录——告诉你一个域的邮件服务器：

```text
name [TTL] [class] MX preference mail-server-name
```

- 你 ping 不通 `scs.stanford.edu`，但可以把邮件发给 `scs.stanford.edu`
- MX 记录会触发对 mail-server-name 的 A 记录处理
- 示例：`dig mx scs.stanford.edu`
- 如果 mail-server-name 没有 A 记录会怎样？
  - 试一试：`dig mx bad-mx.scs.stanford.edu`

MX（Mail eXchange，邮件交换）记录告诉你一个域的邮件服务器，格式为 `name [TTL] [class] MX preference mail-server-name`，其中 preference 是优先级（数字越小越优先）。一个域可以有 MX 记录却 ping 不通——例如 `scs.stanford.edu` 本身没有 A 记录，但发往 `@scs.stanford.edu` 的邮件能正常投递，因为邮件系统查的是 MX 记录。MX 的 mail-server-name 会进一步触发 A 记录解析。示例命令 `dig mx scs.stanford.edu`；如果 mail-server-name 本身没有 A 记录会怎样？可以试试 `dig mx bad-mx.scs.stanford.edu`。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 27：许多其他类型的记录</span>

- SOA：Start of Authority（权威起始记录）
- TXT：任意文本（非常适合扩展）
- PTR：把地址映射回名字
- AAAA：IPv6 地址记录

还有许多其他记录类型：SOA（Start of Authority，权威起始，描述区域的管理元数据）；TXT（任意文本，最初给人看，后来成了不加新记录类型就引入新服务的万能口袋）；PTR（把地址反向映射回名字）；AAAA（IPv6 地址记录——四个 A 是因为 IPv6 地址是 128 位，A 记录的四倍）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> TXT 记录的演化史是"协议扩展经济学"的绝佳标本：加一个新记录类型需要改 RFC、改所有解析器/权威服务器软件、等全网部署，周期以年计；而 TXT 是"任意文本"的白板，SPF（反垃圾邮件）、DKIM（邮件签名）、域名所有权验证（Let's Encrypt 的 DNS-01 challenge）全都塞进了 TXT。这是标准的"灰市扩展"策略——在协议里预留一个自由格式口袋，生态就能绕过标准化流程创新。对比 AAAA：它老老实实走正规流程新开类型，因为二进制地址数据不适合塞文本。这两种扩展路线的取舍值得记住：数据是结构化的就加类型（类型安全、解析高效），数据是自由策略文本就用 TXT（部署快、但解析端必须理解其语义）。DNS 从 1983 年的 RFC 882/883 到今天依然是互联网的"电话簿"，靠的正是这种克制的核心 + 灵活的边缘。

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 5-8/5-9/5-10《DNS 1/2/3》的配套 slides；anycast 在课程后文（路由章节）讨论，IPv6 见 Unit 6 的 names-ipv6。</span>
