> <span style="color:#7f8c8d;">Source: src/慕课/Unit6-Routing/6-9_routing_bookend.txt · CS144 Fall 2019 · 行内双语翻译版</span>

# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## Unit 6 · Routing（路由）——单元收束与复习

### <span style="color:#2471a3;">**[section]**</span> Slide 1 · 封面：CS144, Stanford University（斯坦福大学）

CS144 An Introduction to Computer Networks（计算机网络导论），Unit 6: Routing（第 6 单元：路由）。本页是单元的收束页（bookend）：路由这一单元从"分组如何跨越多台路由器从 A 走到 B"出发，先建立方案（approaches）与算法（algorithms）两条主线，再进入互联网的真实世界——层次化、域间策略与多播。

### <span style="color:#2471a3;">**[section]**</span> Slide 2 · The Problem（问题本身）

图示：主机 A —— 中间节点 C —— 主机 B 连成一条链。

核心问题：<span style="color:#00838f;">**分组（packet）应该如何从 A 到达 B？**</span> 当 A 与 B 不在同一条链路上时，中间的节点必须替分组"选路"——这正是路由（routing）要回答的问题。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这个问题初看平淡，抽象成图论语言后立刻清晰：把路由器看作顶点（vertex）、链路看作带权边（edge），"A 的分组如何到 B"就是"从顶点 A 到顶点 B 沿什么路径走"。你在 6.042J 已经学过路径与连通性，在 CLRS 学过 Dijkstra 与 Bellman-Ford 两个最短路算法，但直接套用会漏掉网络独有的三个难点。其一，图是动态的——链路随时会断、路由器会宕机，预先算好的路径几秒后可能失效。其二，计算是分布式的——没有任何一台机器握有完整拓扑，每个节点只知道与自己直连的部分，答案必须靠节点之间交换消息逐步逼近。其三，"最优"不总是客观的——商业实体有权按自己的策略否决数学上最短的路径（这一点在 BGP 一讲会充分展开）。CSAPP ch11 里我们调用 connect() 时从不关心路径怎么选，正是因为 IP 层的转发表把这些复杂度封装了起来；本单元的任务就是回答：那张表是从哪儿来的。

### <span style="color:#2471a3;">**[section]**</span> Slide 3 · What you learned — Approaches（学到的方案）

- Flooding（泛洪）：收到分组就向所有接口复制转发——最简单，代价是重复分组泛滥。
- Source routing（源路由）：把完整路径写进分组首部，沿途节点照单执行。
- Forwarding table（转发表）：每个路由器查表决定下一跳。
- Spanning tree（生成树）：先把网络剪成一棵无环的树，再沿树转发。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 四种方案代表四种截然不同的设计哲学，值得逐一对比。泛洪完全不计算：分组到哪儿都复制转发，靠冗余保证"总能到"，代价是同一个分组被发送指数级多次，带宽被大量浪费。源路由把智能全部放在源头：发送者把完整路径写进首部，沿途节点退化成无脑执行者——这是"端到端原则"的极端版本，缺点是路径写死在分组里，链路故障时无能为力。转发表把状态分布在每个路由器上：查表转发，这正是 CSAPP ch11 中我们在应用层之下隐式依赖的机制。生成树则先用图论预计算把网络剪成无环的树，牺牲路径最优性换取确定性无环。6.042J 的一个定理保证了它的可行性：任何连通图都含有生成树——网络首先是一个连通图，所以"剪树"永远可行。

### <span style="color:#2471a3;">**[section]**</span> Slide 4 · What you learned — Algorithms（学到的算法）

1. Bellman-Ford（贝尔曼-福特）"distance vector"（距离向量）算法——RIP（Routing Information Protocol，路由信息协议）采用。
2. Dijkstra's shortest path first（Dijkstra 最短路优先）"link state"（链路状态）算法——OSPF（Open Shortest Path First，开放最短路优先）采用。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 两个名字对应你在 CLRS 学过的两个算法，但网络版本都做了分布式改造，思路差异非常典型。Bellman-Ford 在 CLRS 中是单机算法，复杂度 $O(VE)$，允许负权边；距离向量版本把它拆到每台路由器上：大家只和直连邻居交换"我到各目的地的当前估计"，像异步迭代一样反复松弛（relax），直到收敛。好处是通信量小、每台路由器不需要全局拓扑；代价是收敛慢，而且"坏消息传得慢"（count to infinity，无穷计数问题）。Dijkstra 在 CLRS 中要求非负权、贪心逐点确定最短路，复杂度 $O((V+E)\log V)$（用二叉堆）；链路状态版本先让所有路由器通过泛洪同步出同一张全网地图，然后每台路由器本地独立跑一遍 Dijkstra。链路状态用更多通信换收敛速度与一致的视野，RIP 与 OSPF 的几十年共存正是这两条路线的权衡史。

### <span style="color:#2471a3;">**[section]**</span> Slide 5 · What you learned — Internet routing（互联网路由）

1. Hierarchical routing（层次化路由）。
2. BGP——path vector routing（路径向量路由）与本地策略（local policies）。
3. Multicast routing（多播路由）。
4. Spanning Tree Protocol（STP，生成树协议）——以太网交换机的"路由"。

至此 Unit 6 收束：从局域网里的交换机（STP），到域内的 RIP/OSPF，再到域间的 BGP 与层次化结构，以及"一对多"的多播路由——本目录下的其余讲义将逐一展开。
