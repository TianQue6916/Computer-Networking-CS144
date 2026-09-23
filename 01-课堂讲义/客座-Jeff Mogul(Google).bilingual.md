# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Under the covers: the networks of a cloud provider（揭开盖子：一家云提供商的网络）—— Jeff Mogul（Google 网络基础设施部门，mogul@google.com）

> <span style="color:#7f8c8d;">Source: 课堂讲义/客座-Jeff Mogul(Google).pdf · CS144 Fall 2019 · 行内双语翻译版</span>

> <span style="color:#7f8c8d;">说明：本文为客座讲座 slides。pdftotext 提取丢失了几乎全部图示（数据中心照片、地图、拓扑图、线缆对比图），各页右下角的独立数字页码（1—45）不再重复出现；纯图示页按其标题与上下文推断内容并作一句说明，不编造细节。Jeff Mogul 是 TCP 协议规范的作者之一、ACM Fellow，长期在 Google/DEC 从事系统与网络研究。</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">今天的目标：理解一点"云"里的网络</span>

今天的目标：对云网络有所了解（Goal for today: understand something about Cloud networks）。"Cloud"（云）是一个模糊的概念，但无论它指什么，它都离不开网络——而且是**大量的、各种各样**的网络。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">议程</span>

Agenda（议程）：

1. 我说的"云"是什么意思？
2. 什么是虚拟网络（virtual network，虚拟网络）？（只简要介绍）
3. 云提供商如何创建虚拟网络？（Nick 马上会讲）
4. 我们如何构建可扩展、可靠的数据中心网络（data-center network，数据中心网络）？
5. 我们如何构建可扩展、可靠的广域网（WAN，Wide Area Network，广域网）？

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">免责声明与致谢</span>

Disclaimers & Credits（免责声明与素材来源）：

- 并非所有云提供商的做法都完全一样。
- 关于 Google 有很多事情我不能告诉你。
- 我借用了许多（主要来自）Google 的报告素材：
  - B4: Experience with a Globally-Deployed Software Defined WAN（B4：全球部署的软件定义广域网的经验）——由众多 Google 工程师撰写，Amin Vahdat 在 SIGCOMM 2013 上报告；
  - Amin Vahdat 在 Open Networking Summit（ONS 2014）上的报告，录像可在 YouTube 上搜 "Vahdat ONS 2014"；
  - Jupiter Rising: A Decade of Clos Topologies and Centralized Control in Google's Datacenter Network（Jupiter 的崛起：Google 数据中心网络中 Clos 拓扑与集中式控制的十年）——Arjun Singh 等人，SIGCOMM 2015；
  - The Rise of Cloud Computing Systems（云计算系统的崛起）——Jeff Dean 在 SOSP '15 History Day Workshop 上的报告。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">什么是云，为什么要云？</span>

What's a cloud, and why?（什么是云，为什么？）——本部分标题页。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">在"云"之前：大型系统</span>

Before "cloud": large systems（在"云"出现之前：大型系统）。不少公司建造了大型系统，因为它们不得不这样做——一些资源密集型的交互式服务，例如：

- 搜索（Google）；
- 电子商务（Amazon）；
- 电子邮件（AOL、HotMail、GMail 等）。

我们学会了如何让这些系统可扩展（scalable）、可靠（reliable）且便宜（cheap）：

- "横向扩展"（Scale out，用大量廉价机器联网组成系统）而不是"纵向扩展"（Scale up，把单机做得更大）；
- 使用可靠性低但便宜的硬件，靠分布式系统（distributed system，分布式系统）获得可靠性；
- 对资源池提供高层视图，而不是"给你一堆零件"——通过 MapReduce/Hadoop、GFS/HBase 等框架实现。设计模式是：一个集中式主节点（master）负责控制，配上成千上万的 worker（工作节点）与 client（客户端），框架自动把计算/存储映射到一大簇机器上。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "用低可靠硬件换高可靠系统"这句话背后是一个漂亮的概率论证。设单台机器一年内故障的概率为 $p$（比如 4%），且各机器故障相互独立（概率论中独立事件的定义：$P(A\cap B)=P(A)P(B)$），那么 $k$ 台机器**同时**坏掉、导致服务不可用的概率就是 $p^k$——$p=0.04$、$k=3$ 时只剩 $6.4\times10^{-5}$，比单机可靠三个数量级。这就是"横向扩展"在可靠性上的数学根基：可靠性不是买来的，是用冗余"乘"出来的。这也解释了为什么云厂商敢用消费级磁盘和廉价交换芯片。你在 CSAPP ch11 写服务器时见过这种思想的单机版：`socket()` accept 循环 + 并发处理多个客户端，本质上也是"用多个廉价的处理单元分摊负载"；分布式系统只是把这个思路推到几千台机器、再用 master 把它们编排起来。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">云的起源</span>

The genesis of cloud（云的起源）：

- 少数几家公司掌握了大规模横向扩展系统；大多数其他用户还在为基本功挣扎：电力、制冷、机器维修、升级、打补丁、网络管线等等……而这些没有一样能让你的产品与竞争对手拉开差距。Jeff Bezos 把这称为"无差异的重活"（undifferentiated heavy lifting）。
- 几十年来人们一直憧憬"计算像公用事业一样"（computing as a utility），但因为成本太高，这个愿景基本无法落地。
- "Aha!"时刻：我们可以把横向扩展计算带给每一个人——把脏活累活藏在简化接口的背后；不要试图一次解决所有人的问题；靠规模经济无情地压低成本。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">云不止一种</span>

There's more than one kind of cloud（云不止一种）：

- 基础设施即服务（IaaS，Infrastructure-as-a-Service）：提供商提供虚拟计算机/容器、存储设备与网络；客户自己提供从操作系统到应用的所有软件。例如：Amazon EC2、Google GCE。
- 平台即服务（PaaS，Platform-as-a-Service）：提供商管理高层的构建模块，并使其可靠、可扩展；客户编写代码/脚本把这些模块粘合起来（也许会搭配一些 IaaS）。例如：Google Dataflow（大数据分析即服务）。
- 软件即服务（SaaS，Software-as-a-Service）：提供商创建并运行应用程序；用户通过 Web 浏览器或 App 访问。例如：Salesforce.com（CRM）、Gmail、Google Docs。

很多云"租户"（tenant，租户）会同时使用 IaaS 和 PaaS。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">云系统的典型特征</span>

Typical characteristics of a cloud system（云系统的典型特征）：

- 大部分代码和数据都存放在提供商的基础设施里，而用户可以位于互联网上的任何地方。一些企业用云处理来增强其"本地部署"（on-premises，本地部署）的遗留系统。
- 提供商管理全部物理基础设施——客户通常可以无视硬件故障、软件升级、柴油发电机等等。
- 你只为实际使用的量付费：按小时、按 GB、按查询次数等；而且随着计算机变便宜，你要付的钱也会变少。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">SLI、SLO、SLA 与"几个 9"</span>

SLIs, SLOs, SLAs, and nines（服务等级指标、目标、协议与"几个 9"）。你付费买到的东西真的到手了吗？

- <span style="color:#00838f;">服务等级指标（SLI，Service Level Indicator）是一个精心定义的测量量</span>，例如：两台虚拟机之间的往返时延（RTT，round-trip latency），或服务的在线时间（uptime）。
- <span style="color:#00838f;">服务等级目标（SLO，Service Level Objective）是基于一个或多个 SLI 设定的目标</span>，例如：99.9% 的 RPC（远程过程调用）往返时延低于 500 微秒；或"我的存储服务在一个月内 99.95% 的时间可用"。
- <span style="color:#00838f;">服务等级协议（SLA，Service Level Agreement）是"带后果的 SLO"</span>，例如：如果你达不到时延 SLO，就得双倍退还我付的钱。

可用性 SLA 常用"几个 9"（nines）来表述：例如"5 个 9"表示 SLA 保证 99.999% 的在线时间——即每年 5.26 分钟的停机，或每天 864 毫秒。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "几个 9"可以直接用期望来核算：一年约有 $365\times24\times3600=3.15\times10^7$ 秒，乘以不可用概率 $10^{-5}$ 得 315 秒，正好对应"5.26 分钟/年"——SLO 本质上是对随机变量（停机时长、时延）的分布提出的量化要求，这正是概率课里"用期望刻画随机现象"的直接应用。更要紧的是注意 SLO 是**分位数约束**而不是均值约束："99.9% 的 RPC 时延低于 500 µs"允许 0.1% 的请求慢得离谱。为什么尾部这么难管？拿你在概率论里见过的 M/M/1 排队系统说：泊松到达、指数服务时，等待时间的尾部是指数衰减的 $P(W>x)=\rho e^{-(\mu-\lambda)x}$，平均负载 $\rho$ 一高，尾部就又长又肥——均值明明还行，第 99.9 百分位却已经爆炸。这就是为什么云厂商把 SLO 定义在分位数上，也是为什么后面 B4 要给高优先级流量"零丢包 + 最短路"的特殊待遇。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">客户为什么喜欢用云</span>

Why customers like using the cloud（客户为什么喜欢用云）：

- 往往比自建系统更便宜：云提供商能利用规模经济；而且把"资本支出"（CapEx，capital expense）变成了"运营支出"（OpEx，operating expense）。
- 需要快速扩张时可以快速扩张，也可以快速收缩。
- 能快速上线新应用——通过弹性资源和不断增长的 PaaS 组件。

但也有客户对云感到不安：我能相信提供商可靠且诚实吗？政府会不会拿搜查令来窥探我的数据？我能预测每月账单有多大吗？如果我想换一家提供商怎么办（供应商锁定）？

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">虚拟网络</span>

Virtual networks（虚拟网络）——本部分标题页。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">什么是"虚拟网络"？</span>

What's a "virtual network"?（什么是"虚拟网络"？）

<span style="color:#00838f;">虚拟网络之于真实网络，就像虚拟机之于真实计算机。</span>两种情形下，这个抽象都：

- 从用户视角保留重要的方面；
- 隐藏底层"真实"实现中无聊的细节；
- 让提供商能在租户之间高效分配资源；
- 支持租户之间的隔离（安全与性能）。

典型地，一个 IaaS 租户拥有若干台虚拟机（VM，virtual machine，虚拟机），由一个虚拟网络连接。提供商把这个结构映射到底层真实网络上——这种映射很少是 1:1 的。PaaS 租户通常也有虚拟网络，用来连接 PaaS 服务。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">虚拟机与虚拟网络的运作示意</span>

VMs and virtual networks in action（实际运行中的 VM 与虚拟网络）。图示页：机房里有多台 ToR（Top-of-Rack，机架顶部交换机），各机架划分出物理子网 10.1.1/24、10.1.2/24、10.1.3/24、10.1.4/24；而三块"VNET"（虚拟网络）10.1.1/24、192.168.32/24、5.4/16 覆盖在其上——每台主机里的 VMM（虚拟机监视器）承载若干 VM，同一个 VNET 里的 VM 可以分布在不同物理子网。要点：虚拟网络的地址空间与物理子网可以完全解耦。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">我们需要从虚拟网络得到什么？</span>

What do we need from virtual networks?（我们需要从虚拟网络得到什么？）

- 几乎所有 IaaS 和 PaaS 租户都需要把多个东西连起来：VM（或"容器"，本讲座为简化不展开）、PaaS 服务、互联网用户、本地部署系统。
- 云租户想要：可预测的高性能与高可用；虚拟网络灵活的伸缩与重排。
- 云提供商需要：在租户之间强制隔离，保护租户和自己免受攻击；兑现网络可用性与性能的 SLA；收集计费相关信息。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">数据中心网络</span>

Data center networks（数据中心网络）——本部分标题页。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">我们的数据中心很大……</span>

Our datacenters are big ...（我们的数据中心很大……）——三张连续的图示页（数据中心建筑与内部实景照片），无正文文字，展示 Google 数据中心的规模。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">数据中心网络的巨大挑战：带宽孤岛</span>

Grand challenge for datacenter networks（数据中心网络的巨大挑战）：数以万计的服务器按集群（cluster，集群）互联。

- 10 年前："带宽孤岛"（islands of bandwidth）曾是 Google 的瓶颈——机架内 1 Gbps/机，小集群内 100 Mbps/机，整个数据中心只有 1 Mbps/机。
- 工程师们不得不费力优化"带宽局部性"（bandwidth locality），把计算和数据凑到同一机架。
- 结果是"搁浅"（stranded）的计算/内存资源，阻碍了应用扩展。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">巨大挑战（续）：扁平带宽的理想</span>

Grand challenge (cont.)（巨大挑战·续）：挑战在于——若所有服务器都有扁平（均匀）的带宽画像（flat bandwidth profile，X Gbps/机），就能：

- 去掉对局部性的需求，简化作业调度（job scheduling）；
- 通过更好的装箱（bin-packing）节省大量资源；
- 让应用扩展得更好。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">动机：为什么不用传统架构</span>

Motivation（动机）：

- 传统网络架构：成本高得令人望而却步；跟不上我们的带宽需求；"以盒子为中心"（box-centric）部署带来运维复杂性。
- 机会：数据中心是单一管理域（administrative domain，管理域）——同一个组织设计、部署、控制、运营这张网络，往往连服务器也是它自己管的。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">指引我们的三大支柱</span>

Three pillars that guided us（指引我们的三大支柱）：

1. 商用芯片（merchant silicon，商用现货芯片）：通用、商品化价格、即买即用的交换部件；
2. Clos 拓扑（Clos topology，Clos 拓扑）：容纳低端口数（radix）的交换芯片，通过增加级数扩展到几乎任意的规模；
3. 集中式控制/管理（centralized control / management）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> Clos 拓扑值得用你已经会的图论语言重述一遍。它本质是一个多层二部图（bipartite graph，二部图）的堆叠：服务器挂在边缘汇聚块上，边缘块连到多个 spine（脊）块，任意"左半"与"右半"之间都有大量**边不相交**的路径——这正是 6.042J 里 Menger 定理式的直觉：两点间不相交路径越多，同时可用的"通道"越多。而"对分带宽"（把服务器均分两半时跨越割的带宽）恰好是图论意义上的最小割：CLRS 的最大流最小割定理说，一个流网络能同时承载的最大流量等于最小割容量——在这里"流"就是服务器两两之间的并发通信总量。传统树形拓扑的对分带宽是树根那条"细腰"，而 Clos 用并列的多级结构把腰加粗。三大支柱里"低 radix 芯片 + 加级数扩展"还有一个计数视角：一层 $m\times m$ 的交换级数增加一级，可支撑的端口数近似再乘一轮，这就是"几乎任意规模"的代数来源。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">2004 年的水平：4-Post 集群网络</span>

2004 State of the art: 4 Post cluster network（2004 年的最高水平：4-Post 集群网络）。图示页：4 台集群路由器（Cluster Router 1—4）在上层，每台以 2×10G 上联、总计 1000T 容量级；下层每机架一台 ToR 交换机（1G 上行、共 100T），挂若干服务器。缺点：规模上限 2 Tbps（受限于最大的那台路由器）；纵向扩展（scale up）——升级路由器时得"叉车式"整体搬移集群（forklift cluster）。右下角时间轴（2004—2013，对数刻度）预告后面带宽的指数增长。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">数据中心带宽增长要求高得多</span>

DCN bandwidth growth demanded much more（数据中心带宽的增长要求比这多得多）——过渡页，指向下一页的演进图。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Google Clos 网络的基本模式</span>

Basic pattern for Google's Clos networks（Google Clos 网络的基本模式）。图示页：上层是 Spine Block 1—M（脊块），中层是 Edge Aggregation Block 1—N（边缘汇聚块），底层是带 ToR 交换机的服务器机架。流量从机架经边缘汇聚块上行到脊块、再下行到目的机架——任何两台机架之间都有多条等价路径。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">五代演进：从 4-Post 到 Jupiter</span>

Watchtower Evolution over five generations（五代演进，对数刻度时间轴 2004—2013）：

- 2004：4-Post，1 Tbps；
- 2005 左右：Firehose 1.0，100 Tbps；随后 Firehose 1.1，10 Tbps（图示刻度交错）；
- Saturn（土星）、Watchtower（瞭望塔）等中间代；
- 2013：Jupiter，1.3 Pbps（拍比特每秒）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Jupiter 的构件</span>

Jupiter building blocks（Jupiter 的构件）：对主机提供 40G 接入；横向扩展到 1.3 Pbps。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">自建方案时我们面对的挑战</span>

Challenges we faced in building our own solution（自建方案时我们面对的挑战）：

- 拓扑与部署：把新网络引入生产环境；多得管不过来的线缆/光纤；集群外突发带宽需求。
- 控制与管理：在巨大规模下运营；路由可扩展性/海量多路径下的路由；与外部厂商设备的互操作。
- 性能与可靠性：片上缓冲区很小；要靠廉价/低可靠组件做出高可用。

（下一页重复同内容，并注明）时间有限只能讲其中一部分——其余请看 "Jupiter Rising" 论文（I only have time for some of these）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">线缆管理</span>

Cable management（线缆管理）。图示页对比"不捆扎"与"捆扎"两种布线方式：捆扎线缆节省 40% 的 TCO（总拥有成本），并把需要部署的光纤条数减少 10 倍。规模一大，物理层的"针线活"也是一等公民。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">连接到其他网络</span>

Connecting to other networks（连接到其他网络）。图示页：

- 通过边界路由器（border router，边界路由器）对外连接；
- 支持巨大的对外突发带宽；
- 使跨集群 MapReduce 之类成为可能；
- 不再需要保留旧的"集群路由器"。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">用便宜、低可靠的组件做出高可靠？</span>

High reliability from cheap, low-reliability components?（用便宜、低可靠的组件做出高可靠性？）

- 利用多路径（Clos）网络拓扑中的冗余；
- 为"多样性"设计拓扑：例如，不要把"冗余"的两条链路接到同两个交换芯片上；
- 只实现我们需要的特性；
- 从我们的故障停机（outage）中学习如何构建可靠的控制平面（control plane，控制平面）；
- 在虚拟化测试床里测试控制平面（大规模实测太贵了！）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">广域网</span>

Wide-area networks（广域网）——本部分标题页。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">广域网是另一回事</span>

Wide Area Networks are different（广域网是不同的）：

- 所有类型的云（IaaS/PaaS/SaaS）都需要 WAN 连接。WAN 面临数据中心网络没有的挑战：
  - WAN 链路贵得多；
  - WAN 链路更脆弱——挖掘机、农民、鲨鱼（会咬海底光缆）、喝醉的猎人、间谍机构等等；
  - WAN 拓扑高度不规则：受地理约束；有些路径比其他路径贵得多或脆弱得多；
  - WAN 有大量"边缘"问题：路由域之间的边界、所有者之间的边界；设备常放在偏远/受限地点（"POP"，存在点）；每条链路、每个端口可能都需要独一无二的配置。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Google 维护两张不同的 WAN</span>

Google maintains two distinct WAN networks（Google 维护两张不同的广域网）：

- 面向用户的 WAN：把我们的数据中心连到互联网对等方（ISP，互联网服务提供商）；必须对接来自众多厂商的五花八门的设备；连到全世界的 POP；也把数据中心连到我们遍布全球的边缘缓存（CDN，内容分发网络）；承载用户的请求与对它们的响应。
- 数据中心到数据中心的 WAN（"B4"）：只在我们的数据中心之间连接；用于分布式存储、大数据集复制、用户数据复制等。

我会讲 B4，因为它是技术上更有意思的那个。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">B4 的全球覆盖</span>

B4's world-wide scope（B4 的全球覆盖范围）——图示页：世界地图上标出 B4 连接的各数据中心站点，横跨各大洲。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">B4 的设计目标</span>

B4 design goals（B4 的设计目标）——与我们其他 WAN 相比，B4 的设计动机有些不同：

- 带宽需求巨大，但能容忍偶发的带宽下降；
- 站点相对较少；
- 我们控制着所有端点上的应用与操作系统；
- 我们宁可降低成本，而不是靠超额配置（overprovisioning）去实现：低丢包率、低链路故障率。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">B4 的设计原则</span>

B4 design principles（B4 的设计原则）——传统做法 vs B4 做法：

| 传统做法 | B4 做法 |
|---|---|
| 传统路由器：管理成千上万台独立的盒子 | 更简单的交换机 + SDN 控制器（software-defined networking，软件定义网络） |
| 分布式、非确定性的路由协议 | 逻辑上集中的控制，配以流量工程（traffic engineering，流量工程） |
| 所有分组同等重要 | 按应用层优先级分配资源 |
| TCP 流由"公平份额"机制调节 | 在端点上测量需求、整形（shape）流量 |

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">这让我们能把网络跑到 100% 利用率</span>

This allows us to run the network at 100% utilization（这让我们能把网络跑到 100% 利用率）。

- 100% 利用率？那也太疯了吧！
- 在传统网络里，这会导致可怕的丢包。
- 但它真的省钱，所以我们把它做成了。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "100% 利用率太疯了吧"在排队论里是一条定理级的事实。把到达看成随机流（比如泊松到达），M/M/1 系统的平均队长是 $L=\dfrac{\lambda}{\mu-\lambda}$：当到达率 $\lambda$ 趋近服务率 $\mu$（利用率 $\rho=\lambda/\mu\to 1$）时分母趋于零，队长和排队时延一起爆炸——所以传统网络靠把峰值负载压在 70% 以下来留安全余量，本质上是在为随机性交"保险费"。B4 敢跑 100% 的关键，是它把"随机的到达过程"改造成了**受控**的过程：端点的 Bandwidth Enforcer 先测量并整形流量（这正是期末复习里"漏桶调节器"那种 $(r,b)$ 约束思想的工程化），再由集中控制器给每条流分配确定性的隧道带宽——确定性模型里 $Q(t)=A(t)-D(t)$ 完全可算，不需要为泊松尖峰留余量。代价是低优先级流量必须自己容忍丢包与容量波动。这是"用确定性换随机性、用复杂度换容量"的经典设计权衡。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">如何在 100% 利用率下生存</span>

How to live with 100% utilization（如何在 100% 利用率下生存）：

- 区别对待高优先级流量与低优先级流量：高优先级（Hi-pri）力争零丢包，走最短路径（时延最低）路由；低优先级（Lo-pri）的应用必须容忍丢包、时延和容量波动。
- B4 上大部分流量都是低优先级。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">构建 B4 网络</span>

Building the B4 network（构建 B4 网络）：

- 我们自己用商用芯片做的、SDN 可控的交换机：特性比商用路由器少得多——所以便宜得多；缓冲区更小、路由表更小、硬件容错更少。
- 高优先级流量走最短路径路由；低优先级流量走隧道化（tunnelled，隧道化）、经流量工程规划的路由。
- SDN 控制器小心地管理这些流量工程隧道（TE 算法与协议详见 SIGCOMM 2013 论文）。
- "Bandwidth Enforcer"（带宽执行器）在所有端主机上整形流量（BwE 细节见 SIGCOMM 2015 论文）。
- 专门的应用：能容忍高丢包率与高时延，并能随带宽容量变化调整发送速率。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">B4 的硬件</span>

B4 hardware（B4 的硬件）：

- 用商用芯片构建：数百个无阻塞 10GE 端口；
- 支持 OpenFlow；BGP、ISIS 用开源路由栈；
- 每站点多机箱（chassis），通过冗余实现容错；
- 可扩展到多个 Tbps。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">来自赞助商的话……</span>

A word from our sponsor ...（来自赞助商的话……）——招聘部分标题页。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">招聘链接</span>

Short links for jobs（招聘短链接）：

- 实习：PhD 软件工程学生：截止 2020 年 2 月 7 日；本科/硕士软件工程：截止 2019 年 12 月 13 日；硬件工程：截止 2020 年 1 月 31 日；大一大二 "STEP" 实习：截止日为 11 月 1 日。用 google.com/students 了解更多。
- 全职：g.co/research/networks（我们团队），careers.google.com（更广泛）。
- 如果你投了简历：请告诉我（mogul@google.com），以便我跟进。有问题？stanfordstudents@google.com
