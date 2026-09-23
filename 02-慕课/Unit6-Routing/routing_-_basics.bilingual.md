# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> <span style="color:#c0392b;">Routing Basics（路由基础）</span>

> <span style="color:#7f8c8d;">Source: 慕课/Unit6-Routing/routing_-_basics.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

> <span style="color:#7f8c8d;">脚注：本讲主讲人 Nick McKeown。源 PDF 经 pdftotext 提取时存在连字乱码（如 ti→ti 变体、/→ti、$→t 等，原文 "Rou$ng"→"Routing"、"Rou/ng"→"Routing"、"Introduc/on"→"Introduction"、"Mul/path"→"Multipath"、"Mul/cast"→"Multicast"、"hop--of--hop"→"hop-by-hop" 等）与希腊字母丢失，均已按上下文复原，不再逐处标注。</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1：Routing Basics（路由基础）</span>

> CS144: An Introduction to Computer Networks（CS144：计算机网络导论）
> **Routing Basics（路由基础）**
> Nick McKeown，斯坦福大学电气工程与计算机科学教授。

<span style="color:#7f8c8d;">（Slide 1/17）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 2：The Problem（问题）</span>

图示：网络中有两台端主机 A 与 B（中间省略号代表若干路由器与链路）。

核心问题：<span style="color:#00838f;">**来自 A 的分组应该如何到达 B？**（How should packets from A reach B?）</span>

这就是路由（routing，路由）要回答的全部问题——但答案远不止一种。

<span style="color:#7f8c8d;">（Slide 2/17）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 3：The Basics（本讲大纲）</span>

- **方法（Approaches）**：泛洪（flooding，泛洪）· 源路由（source routing，源路由）· 转发表（forwarding table，转发表）· 生成树（spanning tree，生成树）
- **度量（Metrics）**与最短路径生成树（shortest path spanning tree，最短路径生成树）
- **其他类型的路由**：多路径（multipath，多路径）· 多播（multicast，多播）

<span style="color:#7f8c8d;">（Slide 3/17）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 4：Flooding（泛洪）</span>

图示：A 发出的分组在图中向所有方向复制扩散，最终到达 B。

泛洪（flooding，泛洪）的规则极其简单：路由器把收到的每个分组复制到**除入口以外的所有接口**。代价：

- 链路使用效率低下（inefficient link usage，链路使用低效）——同一个分组会被复制成几十份、几百份；
- **分组可能永远循环**（packets can loop forever，分组可能永远循环）——图中一旦存在环，副本就会绕着环打转；
- 适用场景：当我们**不知道（或不信任）网络拓扑**时——因为泛洪不依赖任何拓扑知识，只要链路连通就一定能送到。

🎥 *CS144 教授在视频中解释: "每个路由器都会将数据包转发到所有接口……它会从所有接口发送出去，除了它到达的那个接口……你可以很快看到，在中间会形成一个循环，包会一直循环下去"*（字幕来源：085 - p84 6-1 Routing - Flooding source routing and spanning trees.zh.srt）

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 泛洪看似粗暴，却是一个值得认真对待的"基线算法"。第一性原理地看：在**对拓扑零知识**的前提下，"向所有方向复制"是唯一保证送达的策略——它把"路由问题"退化成了"覆盖问题"，相当于在 6.042J 里对一个未知连通图做全边遍历。它暴露了两个基本矛盾：其一，没有环处理机制时，副本在有环图中会无限增殖，这正是后面生成树（loop-free 子图）与 TTL 字段存在的理由；其二，带宽开销与拓扑的边数成正比，所以它只能出现在拓扑未知、拓扑极小、或可靠性压倒一切的场景——链路状态协议的链路状态公告（LSA）扩散恰恰复用了泛洪，因为那一刻"把新消息散出去"比"省带宽"重要。理解泛洪，才能理解后面每一个更聪明的算法到底在省什么、赌什么。

<span style="color:#7f8c8d;">（Slide 4/17）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 5：Source Routing（源路由）</span>

图示：A 到 B 的完整路径在 A 处就被显式指定，沿途设备只按清单转发。

源路由（source routing，源路由）的特点：

- 是一种"端到端"方案——**不需要网络提供任何支持**；
- 分组自身携带一份**可变（且可能很长）的地址清单**，逐跳指路；
- 终端主机必须**知道拓扑并自己选路**；
- 适用场景：终端用户**想亲自控制路径**时。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 源路由把"选路智能"从网络搬到了主机，这与你在 CSAPP 第 11 章写 socket 程序时的体验形成鲜明对照：那里你对路径完全无感，只管把字节交给内核，路径由路由器逐跳决定。源路由恰好反过来——主机递上一张"途经点清单"，网络退化为不动的管道。这个设计的死穴在于**信息与控制权的错配**：要求每台主机掌握全网拓扑，在 Internet 规模下绝无可能；而允许用户指定路径，也会破坏运营商的流量工程与安全边界（伪造源路由可让流量绕过防火墙），这正是现代 IPv4 中源路由选项被路由器普遍关闭的原因。它的思想并未消亡：IPv4 的松散/严格源路由选项、SRv6 的段路由（segment routing，段路由）都是它的后裔，只是把"指定完整路径"收缩为"指定若干必经节点"。对比泛洪与源路由，你会发现一个贯穿本单元的主线：**智能放在网络里还是端系统里**，永远是一场权衡。

<span style="color:#7f8c8d;">（Slide 5/17）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 6：Forwarding Table（转发表）</span>

图示：网络 S1–S4 互连，A 发往 B 的数据沿逐跳查表前进；每台路由器维护一张"地址 → 下一跳"表。示意（S1 的表）：

| Address（目的地址） | Next-hop（下一跳） |
|---|---|
| B | S2 |
| C | S3 |
| D | S3 |

要点：

- 这是一种优化：**网络负责逐跳路由**（hop-by-hop routing，逐跳路由）——每台路由器只需回答"下一跳是谁"，不必知道全程；
- 前提是必须有人**把转发表填充好**（requires population of forwarding tables，需要填充转发表）——谁来填？这正是后面路由协议（Dijkstra、Bellman-Ford）的工作；
- 状态粒度是**每目的地址**（per-destination state，按目的地的状态），而不必是每条流（per-flow state，按流的状态）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这张 3 行的小表浓缩了 Internet 数据平面的全部秘密。注意 "per-destination, not per-flow" 的分量：若按流维护状态，状态量随并发连接数增长，而按目的前缀维护，状态量只随网络拓扑规模增长——结合 4.0 节讲过的最长前缀匹配（longest prefix match，最长前缀匹配），一条 `171.64/16` 表项就能服务 Stanford 全校几十万台主机，这是 Internet 能扩展到今天的根本原因之一，也是"网络无连接、无状态"这一设计哲学的落点。它与 CSAPP 里你见过的页表是同构的：都是"查表定下一步"，都要应对表项爆炸（页表用多级页表，路由表用前缀聚合）。表中"下一跳"而非"完整路径"的设计，则让每台路由器的知识需求降到最低——配合 Slide 5 对比：源路由让主机背全图，转发表让每个节点只背一行。至于"谁来填表"，留白恰好引出下一讲的路由算法。

<span style="color:#7f8c8d;">（Slide 6/17）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 7：Spanning Tree（生成树）</span>

图示：路由器 R1–R8 与端节点 A、B、C、D、X 组成有环图，其中加粗的子集构成一棵覆盖全部节点的树。

生成树（spanning tree，生成树）的两个条件：

- **Spanning（覆盖性）**：它能到达所有叶子节点；
- **Tree（无环性）**：它没有环。

在生成树上转发，分组永远不会循环——这就对症解决了泛洪"分组永远循环"的死穴。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这是 6.042J 图论知识的正面登场：生成树的严格定义是"包含连通图全部顶点的无环连通子图"；若图有 $n$ 个顶点，则任何生成树恰有 $n-1$ 条边——多一条必成环，少一条必不连通。这个 $n-1$ 的计数是树等价定理的直接推论（无环 + 连通 + $n-1$ 条边三者两两等价）。路由借这个纯数学对象解决纯工程问题：把有环的物理网络"剪"成无环的逻辑树，泛洪的循环问题就根除了，代价是牺牲冗余路径——树里任何一条链路断了，以它为桥的两半就失联。注意"生成"（spanning）保证的是可达性而非最优性：随便一棵生成树都可能让 B 绕远路，于是自然引出下一页的问题——既然要选树，就该有选树的标准，这就是"度量（metric）"。另外记住这里"树由中心算法算出"只是静态视角，Unit 6 后面 STP 一讲会展示**分布式地协商出一棵生成树**的完整协议。

<span style="color:#7f8c8d;">（Slide 7/17）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 8：What is our metric?（度量标准是什么？）</span>

同一张图，"最好"的树取决于我们如何定义"好"。候选度量（metric，度量/代价）：

- 最小距离（min distance）· 最小跳数（min hop-count）· 最小时延（min delay）· 最大吞吐量（max throughput）
- 最少负载路径（least-loaded path）· 最可靠路径（most reliable path）· 最低成本路径（lowest cost path）· 最安全路径（most secure path）· ……

🎥 *CS144 教授在视频中解释: "他们是否应该选择最短的路径？选择最不拥堵的路径？随机选择的路径？最安全、最可靠的路径？这重要吗？"*（字幕来源：085 - p84 6-1 Routing - Flooding source routing and spanning trees.zh.srt）

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这页只有一行问题，却是整条路由技术栈的"总开关"：算法（Dijkstra 还是 Bellman-Ford）只是求解器，**度量（metric）才是需求本身**。第一性原理地拆：度量必须可加（additive，可加的），"逐段代价之和最小"才有意义——跳数、时延、成本都是可加的；带宽却是"瓶颈型"（取最小值而非求和），吞吐量最优路径其实要用另一种"宽路径"算法。现实协议的选择也很务实：RIP 数跳数（简单但会把 20 ms 的一条跨城链路与 20 ms 的一条跨洋链路同权）；OSPF 用管理员配置的 cost，官方建议取与链路带宽成反比的值，让"快路"天然便宜；而 Internet 骨干上的链路大多因流自相似与统计复用呈现排队时延，纯静态度量无法反映拥塞——这正是后面多路径（multipath）与流量工程想补的课。算法课里边权只是数字，网络里**每个数字都是一个工程主张**。

<span style="color:#7f8c8d;">（Slide 8/17）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 9：Example Annotated Graph（示例：带权标注图）</span>

图示：与 Slide 7 相同的 R1–R8 拓扑（端节点 A、B、C、D、X），每条链路旁标注了代价。例如 R1 侧链路权 1 与 2，R2 侧为 5 与 4，中部 R4/R5、R6/R7 之间各为 1、2、3、2 等，R8 接目标 X。（PDF 提取中边与权值的对应关系已碎裂，此处不强行复原每条边的精确归属，只保留"每条链路带一个代价数字"这一事实。）

有了权值，"好树"终于可以定义了。

<span style="color:#7f8c8d;">（Slide 9/17）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 10：Example Annotated Graph（最小成本生成树）</span>

在同一张图上选中部分链路，得到**最小成本生成树**（minimum cost spanning tree，最小成本生成树）。本例中它恰好很简单。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "In this case, simple"（本例恰好简单）这句轻描淡写值得停一下：在一个十来个节点的图上挑最小生成树确实一眼可解，但注意本页求的是最小**生成**树（MST），而路由真正要的是**最短路径树**（shortest path tree，最短路径树）——以目的 X 为根、到每个源都走最小代价路径的树。二者在数学上不同：MST 最小化"树的总边权和"，SPT 最小化"每个节点到根的路径和"；对同一张图它们可能不同，仅当所有源到根的最短路恰好能拼成一棵公共树时才一致（Kruskal/Prim 与 Dijkstra 的区别你已在 CLRS 第 21、22 章见过：前者是贪心挑边，后者是贪心加点）。说"In this case simple"，是因为这张小图里两者碰巧重合。真实网络有数百万节点，"每台路由器各算一棵树"的成本问题，正是下两讲 Dijkstra 与 Bellman-Ford 要回答的。

<span style="color:#7f8c8d;">（Slide 10/17）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 11：How about this network...!?（那这个网络呢！？）</span>

纯图示 slide：展示 The Opte Project（Opte 项目）绘制的 Internet 全网可视化图——一张密如星云、由数十万条边与节点构成的巨大图。文字信息仅此出处标注；该图旨在说明真实 Internet 的规模与复杂度远超前面几张十几个节点的小图。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> Opte 项目（Barrett Lyon 于 2003 年发起）用 traceroute 数据把 Internet 的一角画成一张发光的星系图，本页放它只有一个目的：**让你对"规模"产生生理性敬畏**。前面 Slides 4–10 的图不过十来个节点，而 Opte 描绘的是数十万个 IP 前缀之间的连接。做一道数量级估算：若对 $n$ 个路由器逐对交换链路状态，状态量约 $O(n^2)$——$n = 10^6$ 时就是 $10^{12}$ 量级，任何单点都无法承载；即使每台路由器只存全网 $n$ 个节点的链路状态库，$10^6$ 条目也在内存与泛洪风暴的双重压力极限附近。这个量级论证正是 Internet 下一页大纲里"层次化路由（hierarchical routing，层次化路由）与自治系统（AS，自治系统）"的入场券：当一张图大到算不动时，唯一出路是把它**拆成可独立求解的子图**。这与 6.042J 里"分治将大图问题化为小块"的直觉一致，也预告了 routing_-_Internet 一讲的核心思想。

<span style="color:#7f8c8d;">（Slide 11/17）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 12：The Basics（大纲回看）</span>

回到开头的大纲，到此我们已覆盖前两块：

- **方法**：泛洪 · 源路由 · 转发表 · 生成树
- **度量** 与最短路径生成树
- **其他类型的路由**：多路径 · 多播 ←—— 接下来展开这一块

<span style="color:#7f8c8d;">（Slide 12/17）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 13：Multipath（多路径）</span>

图示：A 到 X 的流量不再只走一棵树上的唯一路径，而是在多条路径上同时分发。

多路径（multipath，多路径）路由的思想：利用泛洪/生成树方案中**被砍掉的那些冗余链路**，把负载摊到多条路径上，以提高总吞吐与容错性。

<span style="color:#7f8c8d;">（Slide 13/17）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 14：Multicast（多播）</span>

图示：同一拓扑，源 A 需要把同一份内容送达多个接收者（图中多个端点）。

多播（multicast，多播）路由：让一个源把同一分组**高效地送达多个终端主机**——理想做法是只在"分叉处"复制分组，每条链路上最多出现一份副本。

<span style="color:#7f8c8d;">（Slide 14/17）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 15：Multicast（多播，树形态）</span>

图示：与 Slide 14 相同的拓扑，但高亮出以源 A 为根、通往所有接收者的一棵多播分发树（multicast distribution tree，多播分发树）。

同一分组沿这棵树向下分发，在需要分流的地方才复制——这正是"最小成本"思想从单播到群发的推广。

<span style="color:#7f8c8d;">（Slide 15/17）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 16：Summary（总结）</span>

- 跨网络为分组选路有多种方法，从最简单的**泛洪**开始。
- 实践中我们用**路由算法**（即路由协议，routing protocols，路由协议）来计算路由。
- 算法通常计算的是**到目的地的最小成本生成树**。
- 其他类型的路由包括：把负载摊到多条链路上的**多路径**，以及把内容送达多个终端主机的**多播**。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 把本讲收束成一条主线：泛洪（零知识、必达、但环与冗余）→ 源路由（智能在主机、网络零支持、但不可扩展）→ 转发表（智能在路由器、逐跳、需要"填表人"）→ 生成树（消除环、但要挑树就要挑度量）。四个方案其实是同一问题的四个设计维度：**谁掌握拓扑知识、状态放在哪里、如何对付环、以什么为目标函数**。后面两讲（Dijkstra 与 Bellman-Ford）回答"填表人怎么算"，Internet 一讲回答"图太大怎么办"，BGP 一讲回答"当'代价'因产权而无法统一时怎么办"。你在 6.006 里学过这些算法在单机上怎么跑；网络课真正新增的视角是：同一个最短路问题，被**拆散到几十台各自为政、只与邻居通话、且随时可能宕机的机器上异步求解**——分布式的引入会带来收敛、震荡、计数到无穷这些单机算法里根本不存在的毛病。

<span style="color:#7f8c8d;">（Slide 16/17）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 17：（收尾页）</span>

空白收尾页（仅含 "CS144, Stanford University" 页脚）。

<span style="color:#7f8c8d;">（Slide 17/17）</span>

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 6-1《Routing - Flooding, source routing and spanning trees》的配套 slides；下一讲《Dijkstra》与《Bellman-Ford》分别展开链路状态与距离向量两类路由算法。</span>
