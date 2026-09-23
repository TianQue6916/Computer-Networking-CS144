> <span style="color:#7f8c8d;">Source: src/慕课/Unit6-Routing/routing_-_multicast.txt · CS144 Fall 2019 · 行内双语翻译版</span>

# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## Multicast Routing（多播路由）

### <span style="color:#2471a3;">**[section]**</span> Slide 1 · 封面

Routing（路由）：Multicast Routing（多播路由）。主讲：Nick McKeown，Professor of Electrical Engineering and Computer Science, Stanford University（斯坦福大学电气工程与计算机科学教授）。

### <span style="color:#2471a3;">**[section]**</span> Slide 2 · Multicast（多播）—— 拓扑图示

图示：网络拓扑，主机 A、B、C、D、E 通过路由器 R1–R8 相连，另有一台主机 X。用于课堂演示多播分组如何沿树分发到一组接收者（slide 文字极少，图示细节以讲授为准）。

### <span style="color:#2471a3;">**[section]**</span> Slide 3 · Multicast（续）

同一拓扑图重复展示，教授在此页开始讲解分发过程（slide 文字极少）。

### <span style="color:#2471a3;">**[section]**</span> Slide 4 · Multicast — Techniques and Principles / Practice（技术与原理 / 实践）

技术与原理（Techniques and Principles）：

- Reverse Path Broadcast (RPB) and Pruning（反向路径广播与剪枝）。
- One versus multiple trees（一棵树还是多棵树）。

实践（Practice）：

- IGMP——组管理（group management）。
- DVMRP——第一个多播路由协议。
- PIM——协议无关多播（protocol independent multicast）。

### <span style="color:#2471a3;">**[section]**</span> Slide 5 · Flooding（泛洪）

纯图示 slide：演示泛洪方式下分组被复制到所有链路、重复分组泛滥的情景（slide 文字极少，图示细节以讲授为准）。

### <span style="color:#2471a3;">**[section]**</span> Slide 6 · Reverse Path Broadcast (RPB)（反向路径广播）

又名 Reverse Path Forwarding（RPF，反向路径转发）。

图示：在同一拓扑上演示 RPB——路由器只有当分组是从"通往源的最短路径上的接口"收到时，才向其他所有接口转发（slide 文字极少，图示细节以讲授为准）。

### <span style="color:#2471a3;">**[section]**</span> Slide 7 · RPB + Pruning（反向路径广播 + 剪枝）

1. 分组无环（loop-free）地送达每一个终端主机。
2. 没有感兴趣主机的路由器，向源的方向发送 prune（剪枝）消息。
3. <span style="color:#c0392b;">**得到的树，就是从源到"感兴趣主机集合"的最小代价生成树（minimum cost spanning tree）。**</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> RPB 最精彩之处在于：它只需要一个局部判据，却隐式构造出一棵全局正确的树。规则是：收到多播分组的路由器先问自己"从我到这个源的单播最短路，是否正是我收到分组的这条接口？"——如果是，说明分组恰好沿从源出发的最短路树到达了我，我向除入口外的所有接口转发；如果不是，说明我不在这棵树上，静默丢弃。用 CLRS 的视角看，单源最短路树由一次 Dijkstra 确定，RPB 相当于让全网用已有的单播路由表隐式共享这棵树，不需要专门的多播计算。第 2、3 步的 pruning 再把"广播树"剪成只保留通往感兴趣主机分支的树，使每个接收者基本都沿自己的单播最短路走。注意这里要与 6.042J 的最小生成树（MST）区分：MST 最小化"树中所有边的边权之和"且顶点事先给定；而这里要同时连接源与一组接收者、还允许借用其他中间节点，一般图上的最优版本是 NP 难的 Steiner 树问题——RPB+prune 给出的是"各接收者各自最短路"的天然近似，工程上足够好。

### <span style="color:#2471a3;">**[section]**</span> Slide 8 · One tree versus several trees?（一棵树，还是多棵树？）

图示：同一拓扑下讨论——为整个网络维护一棵共享树，还是为每个源各自维护一棵独立的树（课堂讨论 slide，文字极少，图示细节以讲授为准）。

### <span style="color:#2471a3;">**[section]**</span> Slide 9 · Multicast — Techniques and Principles / Practice（概览，重复）

技术与原理（同前）：

- Reverse Path Broadcast (RPB) and Pruning（反向路径广播与剪枝）。
- One versus multiple trees（一棵树还是多棵树）。

实践（本页比第 4 页多一条"多播地址"）：

- Multicast addresses（多播地址）。
- IGMP——组管理。
- DVMRP——第一个多播路由协议。
- PIM——协议无关多播。

### <span style="color:#2471a3;">**[section]**</span> Slide 10 · Addresses and joining a group（地址与加入组）

- IPv4：Class D（D 类）地址被保留用于多播。
- IGMP（Internet Group Management Protocol，互联网组管理协议），见 RFC 3376：
  - 运行在主机与直连路由器（directly attached router）之间。
  - 主机请求接收属于某个多播组（multicast group）的分组。
  - 路由器周期性轮询（poll）主机，询问它们想要哪些组。
  - 若无回复，成员关系（membership）超时失效（soft-state，软状态）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "若无回复，成员关系超时"这一行背后是一种叫 soft-state（软状态）的设计哲学，与你在 TCP 里熟悉的硬状态形成鲜明对照。硬状态是"建一次、记到显式删除"：TCP 连接要挥手才真正消失，若一方突然掉电，另一方还得靠超时清理残局。软状态则干脆不追求精确记账：路由器记录"该组有成员"这个事实，但每条记录自带保质期，主机必须周期性刷新（响应轮询），一旦停止刷新就自然过期。它用一点周期性通信开销，换来了对各种故障的天然鲁棒性：主机崩溃、网络分区、报文丢失都不会留下"幽灵成员"——过一会儿自己就好了。用概率论做个粗略估计：设轮询周期为 $T$，一个成员离开后其残留记录的存活时间近似均匀分布在 $(0, T]$，期望残留 $T/2$——用可计算、有界的短暂误差，换掉了不可控的长期错误状态。这种"宁要自愈的模糊、不要精确的脆弱"的思路，在互联网设计里反复出现。

### <span style="color:#2471a3;">**[section]**</span> Slide 11 · Multicast routing in the Internet（互联网中的多播路由）

**DVMRP**

- Distance Vector Multicast Routing Protocol（距离向量多播路由协议），见 RFC 1075。
- 第一个互联网多播路由协议。
- 使用 RPB + Prune。

**PIM**

- Protocol Independent Multicast（协议无关多播）——不依赖任何特定单播路由协议。
- 两种模式：dense mode（稠密模式）与 sparse mode（稀疏模式）。
- Dense mode（RFC 3973）：与 DVMRP 类似。
- Sparse mode（RFC 4601）：构建 rendezvous point（汇聚点），分组经由汇聚点加入一小组生成树。

### <span style="color:#2471a3;">**[section]**</span> Slide 12 · Multicast in practice（实践中的多播）

多播的实际使用低于最初的预期：

- 大多数通信是个性化的（例如 time shifting，时移点播）。
- 早期实现效率不高。
- 今天用于部分 IP 电视（IP TV）与快速分发（fast dissemination）场景。
- 也出现了一些应用层多播路由（application-layer multicast）。

一些有意思的问题：

- 如何让多播可靠？
- 如何实现流量控制（flow control）？
- 如何支持不同终端用户的不同速率？
- 如何为多播会话提供安全？

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 多播的商业遇冷是检验"端到端原则"的绝佳案例。IPv4 的 D 类地址只提供一个无结构的 28 位组号——不像单播前缀可以按拓扑分层聚合，核心路由器必须为每个活跃组维护"每组的树"的状态，扩展性先天受限。更深一层，把组管理状态放进网络核心违背了 fate sharing（命运共享）：主机崩溃后，散落在沿途路由器上的组状态成了无人认领的孤儿，只能靠软状态慢慢过期。反观应用层多播：让端主机自己组成 overlay 覆盖网，一台主机收到后再转发给其他主机，网络核心只看到普通单播分组——复杂度全部回到端系统，正是 CSAPP ch11 与端到端原则推崇的位置。再看"多数通信是个性化的（如时移点播）"这一条：多播的隐含前提是"很多人在完全相同的时刻想要完全相同的字节"，而流媒体点播时代每个人暂停、快进的时刻都不同，前提失效了。技术本身没错，是时代把它的适用域挤小了——今天它活在 IP 电视与企业内部分发这类场景里。
