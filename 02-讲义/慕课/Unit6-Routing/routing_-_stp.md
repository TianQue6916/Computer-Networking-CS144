> <span style="color:#7f8c8d;">Source: src/慕课/Unit6-Routing/routing_-_stp.txt · CS144 Fall 2019 · 行内双语翻译版</span>

# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## Spanning Tree Protocol（生成树协议）

### <span style="color:#2471a3;">**[section]**</span> Slide 1 · 封面

Routing（路由）：Spanning Tree Protocol（生成树协议）。主讲：Nick McKeown，Professor of Electrical Engineering and Computer Science, Stanford University（斯坦福大学电气工程与计算机科学教授）。

### <span style="color:#2471a3;">**[section]**</span> Slide 2 · Outline（大纲）

- 以太网也"路由"（转发）分组。
- 我们已经知道地址是如何学到的，但环（loop）是如何防止的？
- 以太网交换机构建一棵生成树（spanning tree），分组沿它转发。

### <span style="color:#2471a3;">**[section]**</span> Slide 3 · Ethernet Switch（以太网交换机）

1. 检查每个到达帧（frame）的首部。
2. 若以太网 DA（目的地址，destination address）在转发表（forwarding table）中，把帧转发到正确的输出端口。
3. 若 DA 不在表中，把帧广播（broadcast）到所有端口（帧到达的那个端口除外）。
4. 表项通过检查到达分组的以太网 SA（源地址，source address）学习得到。

### <span style="color:#2471a3;">**[section]**</span> Slide 4 · Learning could lead to loops（学习可能导致环）

纯图示 slide：当交换机之间存在冗余链路（环）时，学习与广播会让帧在环里被反复复制，转发表不断震荡（slide 文字极少，图示细节以讲授为准）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 帧在环里被无限复制，根源要从协议层面想清楚。IP 首部里有 TTL 字段，每过一个路由器减一，减到零就丢弃——这是给分组留的"最后防线"。但以太网帧的首部（Unit 5 学过的帧格式）里没有对应字段，交换机的转发又是不打折扣的复制：只要拓扑里有环，一个广播帧就会被交换机互相放大、指数级增殖，直到占满所有链路带宽——术语叫广播风暴（broadcast storm）。更糟的是地址学习也会跟着坏掉：同一个源 MAC 的帧从不同端口陆续到达，交换机不断改写转发表（表项震荡），连普通单播都送错地方。既然帧没有"自我毁灭"机制，就必须从拓扑上消灭环——这解释了为什么防环要在"图"的层面解决（把图剪成树），而不是在"帧"的层面打补丁。这也是分层设计的典型处境：某一层的缺陷（无 TTL）必须由同一层自己的机制补救。

### <span style="color:#2471a3;">**[section]**</span> Slide 5 · Preventing loops — Spanning Tree Protocol（防环：生成树协议）

交换机组成的拓扑是一个图（graph）。<span style="color:#00838f;">**生成树协议找出一个"覆盖全部顶点、但无环"的子图（subgraph）。**</span>

- Spanning（生成/覆盖）：所有交换机都包含在内。
- Tree（树）：没有环。

这个分布式协议决定两件事：

1. 哪台交换机是树的 Root（根）；
2. 哪些端口被允许沿树转发分组。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这页有两个概念容易与 6.042J、CLRS 里的树混淆，值得精确区分。第一，STP 构造的树确实是生成树——6.042J 的定理保证连通图必有生成树，所以协议永远有解；但它不是最小生成树（MST）：Kruskal 与 Prim（CLRS ch23）最小化"树中所有边的边权之和"，而 STP 最小化的是"每台交换机到根的跳数"，实际得到的是以根桥为源的最短路径树。两者在边权不均匀的图上结果可以完全不同：一条直连链路与一条绕两跳的路径，MST 可能选后者（若边权按代价算），STP 永远选前者（跳数少）。第二，树是"协议的输出"而非"输入"：没有哪台交换机知道全网拓扑，每台只看得见自己的接口，生成树是若干台机器靠交换消息分布式协商出来的——下一页的 BPDU 选举正是这个协商过程。在分布式语境下，"计算一棵树"从一道算法题变成了一个共识问题。

### <span style="color:#2471a3;">**[section]**</span> Slide 6 · Example Spanning Tree（生成树示例）

图示：九台交换机 S1–S9 组成的网状拓扑。

做法：1. 选出唯一的根（root）。2. 只在"到根的跳数（hop count）最短"的端口上转发分组。

### <span style="color:#2471a3;">**[section]**</span> Slide 7 · Resulting Spanning Tree（得到的生成树）

图示：S1–S9 修剪后的树形结构，冗余链路被置为阻塞（slide 文字极少，图示细节以讲授为准）。

### <span style="color:#2471a3;">**[section]**</span> Slide 8 · How it works（工作原理）

1. 所有交换机周期性广播 BPDU（Bridge Protocol Data Unit，桥协议数据单元），内容包括（发送者 ID、根 ID、发送者到根的距离）。
2. 初始时，每台交换机都宣称自己是根：把距离字段设为 0。
3. 每台交换机持续广播，直到听到"更好"的消息：
   - 根 ID 更小；或
   - 根 ID 相同，但距离更短；或
   - 前两者相同，而发送者 ID 更小（平局由更小的发送者 ID 打破）。
4. 听到更好消息的交换机，转发该消息（距离加 1）。

- Root port（根端口）：交换机上离根最近的端口。
- Designated port（指定端口）：邻居们约定用来到达根的端口。
- 其余端口被阻塞（blocked），不转发分组（但仍收发 BPDU）。

最终效果：

- 只有根发起配置消息（其余交换机负责转发它们）。
- 本地而言，交换机只在（树上的）端口转发。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> BPDU 的选举本质上是分布式系统里的"选唯一代表"（leader election），用 6.042J 的语言可以讲得很干净。每台交换机维护一个三元组（根 ID，到根距离，发送者 ID），消息之间按字典序比较——这是一个全序（total order）；全序保证任何两条消息必分高下、不存在循环平局，于是"更好的消息"才有良定义，全网最终必然收敛到同一个最小元：ID 最小的交换机当根。选举的动态过程像一场传话：初始人人自称根（distance=0），每听到更好的消息就把它转发出去（distance+1），相当于把"现任最优者"沿图扩散——消息至少要走遍网络的每个角落，所以收敛时间的下界是图的直径（diameter，任意两点最短跳数的最大值）。root port 与 designated port 的分工则是把"选边"落实到每条链路：一条链路的两端必须恰好有一个方向被采纳，否则环又回来了。把三元组换成别的比较键，这套模式可以套用到任何"分布式选唯一代表"的场景。

### <span style="color:#2471a3;">**[section]**</span> Slide 9 · A brief history（简史）

- 1985：STP 被提出；1990 年成为 IEEE 标准。至今仍被非常广泛地使用。
- 2004：STP 被收敛更快的 RSTP（Rapid Spanning Tree Protocol，快速生成树协议）取代。不过 RSTP 对网络的利用仍然低效（树之外的链路闲置）。
- 2012：以太网交换机引入了新标准：Shortest-Path Bridging（SPB，最短路径桥接，IEEE 802.1aq）。它是像 OSPF 那样的链路状态协议。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这三行年表浓缩了协议演化的两个教训。1985 年提出 STP 的是 Radia Perlman——分布式算法界的传奇人物，"互联网之母"之一；1990 年它成为 IEEE 802.1D 标准，至今大量交换机仍默认开启。但 STP 收敛慢：拓扑变化后可能要等 30 到 50 秒才恢复转发，期间网络近乎瘫痪，2004 年的 RSTP 把收敛压缩到秒级，思想没变、消息机制更聪明。真正的范式转变发生在 2012 年的 SPB：干脆放弃"分布式选举一棵树"，改用与 OSPF 同构的链路状态协议——先泛洪全网拓扑，再各自本地计算最短路树。回头看正好呼应本单元的主题：以太网交换机绕了一大圈，最终也走到了 RIP/OSPF 时代的同一分岔口，选择了链路状态。设计思想的跨层迁移（把 IP 层验证过的方案搬进链路层）在工程史上反复出现，能识别出"这是个老问题的新马甲"，是很有价值的工程直觉。
