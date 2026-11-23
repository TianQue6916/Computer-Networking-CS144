> <span style="color:#7f8c8d;">Source: src/慕课/Unit6-Routing/routing_-_Internet.txt · CS144 Fall 2019 · 行内双语翻译版</span>

# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## Routing in the Internet（互联网中的路由）

### <span style="color:#2471a3;">**[section]**</span> Slide 1 · 封面

Routing（路由）：Routing in the Internet（互联网中的路由）。主讲：Nick McKeown，Professor of Electrical Engineering and Computer Science, Stanford University（斯坦福大学电气工程与计算机科学教授）。

### <span style="color:#2471a3;">**[section]**</span> Slide 2 · Outline（大纲）

- Hierarchy and Autonomous Systems (AS)（层次结构与自治系统）。
- Interior Routing Protocols（内部路由协议）。
- The structure of the Internet（互联网的结构）。

### <span style="color:#2471a3;">**[section]**</span> Slide 3 · Hierarchy in the Internet（互联网中的层次）

图示：三个自治系统（AS）。左右两个是"单出口"（Single Exit）或 stub（残桩）AS；中间一个是"多出口"（Multi Exit）或 transit（中转）AS。每个 AS 内部运行 Interior Routing Protocol（内部路由协议，IRP），AS 之间运行 Exterior Gateway Protocol（外部网关协议，EGP）。

### <span style="color:#2471a3;">**[section]**</span> Slide 4 · Autonomous Systems（自治系统）

<span style="color:#00838f;">**AS 是互联网层次结构（hierarchy）的基本单位。**</span>

- AS 内部：由所有者（owner）决定路由怎么做。
- AS 之间：必须使用 BGP-4（Border Gateway Protocol v4，边界网关协议第 4 版）。（参见 RFC 1771）

动手找一个 AS 号：

- 先用 DNS 查 Stanford 的 IP 地址：`dig stanford.edu`，返回 `stanford.edu 1800 IN A 171.64.13.26`。
- 再查这个 IP 属于哪个 AS：`nc whois.cymru.com 43`，输入 `171.64.13.26`。
- 结果示例：Stanford (32)、AT&T (797)、Google (15169, 22859, 36039)。
- `traceroute -a <destination>` 会报告沿途的 AS 号。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这页的命令演示了一条完整的"控制平面侦探链"，值得在终端里亲手跑一遍。dig stanford.edu 做的是 DNS 查询——正是 CSAPP ch11 里 getaddrinfo() 在命令行的化身；输出中的 1800 是该 DNS 记录的 TTL（存活秒数），IN 表示 Internet 类，A 表示地址记录。拿到 IP 后向 whois.cymru.com 的 43 端口发起查询，返回的是这个 IP 所属的 AS 号：Stanford 是 32，AT&T 是 797，而 Google 拥有 15169、22859、36039 等多个 AS——大公司业务与地理分布复杂，会运营多个自治系统。traceroute -a 则把你在前面单元见过的逐跳探测（利用 IP 首部 TTL 每跳减一、超时回包的机制）与 AS 归属拼在一起，让你亲眼看见"一跳一跳"如何聚合为"一个 AS 到另一个 AS"。这套工具链的意义在于：层次结构不是教科书插图，而是可以从任何一台联网的机器上验证的现实。

### <span style="color:#2471a3;">**[section]**</span> Slide 5 · Interior Routing Protocols（内部路由协议）

**RIP（路由信息协议）**

- 使用距离向量（distance vector，即分布式 Bellman-Ford 算法），见 RFC 2453。
- 每 30 秒发送一次更新。
- 更新不做认证。
- 最早出自 BSD UNIX 的 routed 守护进程。
- 曾广泛使用多年；如今用得少了。

**OSPF（开放最短路优先）**

- 链路状态更新按需（as and when required）用泛洪（flooding）发出，见 RFC 2328。
- 每台路由器运行 Dijkstra 算法。
- 更新经过认证。
- 一个 AS 可以划分为多个 area（区域）。
- 广泛使用，实现复杂。
- IS-IS（RFC 1142）与之类似，同样广泛使用。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> RIP 与 OSPF 的差异是"用通信换计算"还是"用计算换通信"的经典案例。RIP 的距离向量就是分布式 Bellman-Ford：每台路由器只维护一张"到每个目的地的距离加下一跳"的向量，每 30 秒向邻居广播一次；用 CLRS 的视角看，这相当于让全网络异步地反复执行松弛操作，收敛可能非常慢，甚至出现"坏消息传得慢"的无穷计数问题。它连更新都不做认证——1988 年的互联网还是个熟人社会，安全不是首要目标。OSPF 则完全反过来：路由器先通过泛洪把链路状态（自己和谁相连、代价多少）同步给全网，每台路由器据有同一张地图后，本地独立跑 Dijkstra（$O((V+E)\log V)$，非负权）；泛洪的通信量大得多，但收敛快、可认证，还支持把一个大 AS 切成 area（区域）以限制泛洪范围——等于在 AS 内部又做了一层层次化。IS-IS 与 OSPF 同属链路状态家族，思想相同、报文格式不同，常见于大型运营商内网。

### <span style="color:#2471a3;">**[section]**</span> Slide 6 · Routing to a single exit point（通往单一出口的路由）

只有一个出口点，所以 AS 内的路由器可以使用默认路由（default routing）：

- 每台路由器知道 AS 内的全部前缀（prefix）。
- 发往其他 AS 的分组交给默认路由器（default router）。
- 默认路由器就是通往下一个 AS 的边界网关（border gateway）。

单出口 AS 的路由表往往很小。

### <span style="color:#2471a3;">**[section]**</span> Slide 7 · Routing to multiple exit points（通往多个出口的路由）

- 多宿主（multi-homed）企业与 transit AS（中转 AS）使用。
- 每台内部路由器必须被告知：对某个目的前缀该用哪个出口。
- 需要很大的路由表，才能路由到每个前缀。
- 方案 1：Hot-potato routing（热土豆路由）——送到离自己最近的出口。
- 方案 2：选择离目的地最近的出口。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> hot-potato routing（热土豆路由）这个名字是整页最有信息量的部分：把自己手上的分组像烫手山芋一样尽快甩给下一个 AS——出口离我越近，我的网络替别人承载流量的成本就越低。这里能看到清晰的博弈结构：单出口 AS 里"往哪儿送"是纯技术问题，默认路由即可；多出口时每个 AS 都要为同一份流量在"我方传输成本低（热土豆）"与"端到端路径质量好（选离目的地近的出口）"之间做选择，而相邻 AS 的最优选择往往互相冲突——我越早甩出去，路径总长可能越长。这也是"策略"第一次正式压过"最优化"：路由不再是 6.042J 里"找一条边权和最小的路径"，而是多个自私实体之间的交互。你接下来学 BGP 会看到，这场博弈被协议正式固化成了 local preference 与客户-对等-提供商的优先级规则。

### <span style="color:#2471a3;">**[section]**</span> Slide 8 · Exterior Routing Protocol（外部路由协议）

每个 AS 之间必须使用 BGP-4 互联。BGP-4 要解决的问题：

- 拓扑（Topology）：互联网是大量 AS 组成的复杂网状（mesh），几乎没有什么整体结构。
- AS 的自治性（Autonomy）：每个 AS 用不同的方式定义链路成本，因此不可能去找全局最低成本路径。
- 信任（Trust）：有些 AS 无法信任其他 AS 会通告好路由（例如两家竞争的骨干网提供商），也无法信任对方保护自己流量的隐私（例如两个交战国家）。
- 策略（Policies）：不同 AS 的目标不同（例如走最少跳数；优先用这家供应商而不是那家）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这一页回答了一个根本问题：为什么域间路由不直接用 Dijkstra？因为最短路问题的前提在 AS 层面全部塌掉了。图论中的最短路要求边权是统一的客观量——同一张图里所有人用同一套数字；而这里每个 AS 用自己的方式定义"成本"，甚至有权拒绝通告某些路径（信任问题：两家竞争的骨干网不会如实互告负载，交战国更不会让流量经过自己）。目标函数也不再是"最短"，而是各自的政策（少跳数？走某家供应商？）。于是问题从"优化"变成了"机制设计"：协议必须足够灵活，允许每个参与者注入私有策略，同时还要保证收敛与无环。这就是下一讲路径向量与 local preference 出场的原因。CLRS 教你的是"给定一张图如何算"，BGP 要解决的是"图本身是由一批互相不完全信任的实体协商出来的"——这是网络课程里少有的、超出标准算法课的思维训练。

### <span style="color:#2471a3;">**[section]**</span> Slide 9 · Internet structure（互联网的结构）

纯图示 slide：展示由众多 AS 通过客户-提供商（customer-provider）与对等（peering）关系连接而成的互联网整体结构（slide 文字极少，为课堂演示图，细节以讲授为准）。

### <span style="color:#2471a3;">**[section]**</span> Slide 10 · Summary（小结）

- 互联网由多个 AS 组成，每个 AS 独立管理。
- 每个 AS 运行自己的内部路由协议（interior routing protocol）。
- Stub AS（残桩 AS）使用简单的默认路由。
- 有多个出口的 AS 必须决定哪个出口最佳。
- 各 AS 之间必须使用 BGP-4 协议连接。
