> <span style="color:#7f8c8d;">Source: src/慕课/Unit6-Routing/routing_-_bgp.txt · CS144 Fall 2019 · 行内双语翻译版</span>

# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## BGP（边界网关协议）

### <span style="color:#2471a3;">**[section]**</span> Slide 1 · 封面

Routing（路由）：BGP。主讲：Nick McKeown，Professor of Electrical Engineering and Computer Science, Stanford University（斯坦福大学电气工程与计算机科学教授）。

### <span style="color:#2471a3;">**[section]**</span> Slide 2 · Border Gateway Protocol (BGP-4) — Basics（基础）

- BGP 既不是链路状态（link-state）、也不是距离向量（distance-vector）路由协议，而是所谓 "Path vector"（路径向量）。
- BGP 路由器通告完整的路径（一个 AS 列表）——也叫 AS_PATH（这就是路径向量）。
- 通告示例："网络 171.64/16 可以经由路径 {AS1, AS5, AS13} 到达"。
- 带环的路径在本地被检测出来并忽略。
- 由本地策略（local policies）在候选路径中挑选偏好者。
- 当链路/路由器故障时，路径被"撤回"（withdrawn）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 路径向量可以理解为距离向量的"诚实升级版"。距离向量只传播一个数字（到目的地的距离），路由器无从知道这个数字背后藏着哪条路径——于是既有环检测难题，又有"我不知道这条路经过谁"的策略盲区。路径向量的做法是把完整的 AS 列表随距离一起传播：AS_PATH 就是路径本身，环检测因此变得平凡——收到通告时看一眼自己的 AS 号是否已在列表里，在则丢弃。这正是 6.042J 图论里"路径中顶点重复即构成环"的直接应用，检测代价只有 $O(\text{路径长度})$。策略层面也顺带解决：既然整条路径可见，每个 AS 都可以按自己的规则否决经过特定邻居的路径。故障处理则对应"撤回"（withdraw）消息——链路断了，曾通告过这条路径的 AS 沿途收回它。可以看到：BGP 放弃了链路状态的全网拓扑同步，换来的是"路径信息本身就是策略载体"。

### <span style="color:#2471a3;">**[section]**</span> Slide 3 · Customers and Providers（客户与提供商）

纯图示 slide：展示客户 AS 与提供商 AS 之间的连接关系（下一页给出文字说明）。

### <span style="color:#2471a3;">**[section]**</span> Slide 4 · Customers and Providers（客户与提供商）

图示：provider（提供商）与 customer（客户）相连，IP traffic（IP 流量）在链路上流动。

<span style="color:#00838f;">**客户向提供商付费，请它承载自己的分组（Customer pays provider to carry its packets）。**</span>

### <span style="color:#2471a3;">**[section]**</span> Slide 5 · Customer-Provider Hierarchy（客户-提供商层次）

纯图示 slide：多组客户-提供商关系向上堆叠，形成互联网商业上的层次结构（课堂图示，slide 文字极少）。

### <span style="color:#2471a3;">**[section]**</span> Slide 6 · Customer-Provider Hierarchy（续）

图示：provider（提供商）与 customer（客户）之间的链路承载 IP traffic（IP 流量）。

### <span style="color:#2471a3;">**[section]**</span> Slide 7 · The Peering Relationship（对等关系）

图示：peer（对等方）→ provider（提供商）→ customer（客户）的中转被允许（traffic allowed）；peer 与 peer 之间的中转不被允许（traffic NOT allowed）。

- Peer 为各自 customer 之间的流量提供中转。
- Peer 不为 peer 提供中转。
- Peer（通常）不交换钱。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这三条规则是整份材料里最"经济学"的一页，值得从激励角度读一遍。transit（中转）是收费服务：你的 provider 替你把分组送到互联网任何角落，你为它的承载付费；peering（对等）则是对等互换：我们各自把自己的 customer 交给对方，谁也不给谁付钱——所以第三条"peer 通常不结账"是前两条的自然推论。第二条"peer 不为 peer 提供中转"防的是搭便车：假设 A 与 B 对等、B 又与 C 对等，若 A 能经由 B 触达 C 的 customer，那 A 等于免费获得了 B 的 transit 服务，B 的 transit 收入就被稀释了。注意图中的箭头语义：peer→provider→customer 这条组合链路被明确"允许"，peer→peer 则"不允许"——规则不仅靠君子协定，还会写进路由策略（下一页的 local preference）。理解了这套商业结构，后面"路由选择"为什么把客户路由排在第一优先就顺理成章：把流量尽量从付费路上引走，是纯粹的成本理性。

### <span style="color:#2471a3;">**[section]**</span> Slide 8 · BGP Messages（BGP 报文）

- Open：建立 BGP 会话（session）。
- Keep Alive：按固定间隔握手保活。
- Notification：关闭一个对等会话（peering session）。
- Update：通告新路由，或撤回先前通告的路由。

<span style="color:#00838f;">**一条 BGP 通告 = 前缀 + 路径属性（path attributes）。**</span>

路径属性包括：next hop（下一跳）、AS Path（AS 路径）、local preference（本地优先级）、Multi-Exit Discriminator（多出口鉴别器，MED）等，用于在多条候选路径之间做选择。

### <span style="color:#2471a3;">**[section]**</span> Slide 9 · BGP Route Selection Summary（BGP 路由选择小结）

路由器按顺序依次比较（前面的标准能分出胜负就不再看后面）：

1. Highest Local Preference（本地优先级最高）——执行商业关系，例如客户路由优先于对等路由（Enforce relationships）。
2. Shortest AS_PATH（AS 路径最短）——属流量工程（Traffic Engineering）范畴。
3. Lowest MED（多出口鉴别器最低）。
4. i-BGP < e-BGP（经外部 BGP 学来的路由优先于内部 BGP 学来的）。
5. Lowest IGP cost to BGP egress（到 BGP 出口的 IGP 开销最低）。
6. Lowest router ID（路由器 ID 最小）——实在无法区分，就摊手打破平局（Throw up hands and break ties）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这张决策表是一个按字典序比较的贪心过程，可以类比"比较两个元组"：先比第一位（local preference，纯本地策略），大的赢；打平再比第二位（AS_PATH 越短越好）；再往后依次是 MED（多出口鉴别器）、i-BGP/e-BGP 之别、到 BGP 出口的 IGP 开销，最后用 router ID 强行打破平局——连客观的随机化手段都不需要，每个维度都有明确的语义。它与你熟悉的 Dijkstra 有本质区别：Dijkstra 在一张统一边权的图上计算全局最优，而这里是多个局部决策的串联，每一步都把商业关系（customer > peer > provider）硬编码在最高优先位。换句话说，BGP 的路由选择不是"算出来的最短路"，而是"字典序筛出来的可用路"。这个设计的妙处在于可解释性：任何一条被选中的路由都能逐级说出"为什么是它"——在多方博弈的系统里，可解释往往比最优更重要。

### <span style="color:#2471a3;">**[section]**</span> Slide 10 · ASPATH Attribute（AS_PATH 属性）

图示：前缀 135.207.0.0/16（标注 Prefix Originated，即"前缀起源处"）在 AS 6341、AS 7018、AS 1239、AS 1755、AS 1129、AS 12654、AS 3549 之间的传播关系（课堂图示，布局以原 slide 为准）。

### <span style="color:#2471a3;">**[section]**</span> Slide 11 · ASPATH Attribute（续）

图示：各 AS 学到的前缀与路径（按原图文字）：

- AS 6341：135.207.0.0/16（前缀起源处，路径为空）。
- AS 7018：AS Path = 7018 6341
- AS 1239：AS Path = 1239 7018 6341
- AS 1755：AS Path = 1755 1239 7018 6341
- AS 1129：AS Path = 1129 1755 1239 7018 6341
- AS 12654：AS Path = 7018 6341
- AS 3549：AS Path = 3549 7018 6341

决策规则：<span style="color:#c0392b;">**挑更短的 AS 路径（pick shorter AS path）**</span>。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 把左图的 AS_PATH 增长过程倒过来读会很清楚：前缀 135.207.0.0/16 由 AS 6341 起源（originated），它对外通告时路径列表里只有自己；AS 7018 收到后把自己的号 prepend（前置）到列表头部再向外传播，路径变成 7018 6341……每经过一个 AS，列表头部就多一个号。于是任何一台路由器看到的 AS_PATH，恰好是从自己到源 AS 的反向路径，"挑更短的 AS 路径"就等价于在 AS 图上跳数更少。这正是路径向量的妙处：路由信息自带完整履历，环检测、路径比较、策略过滤全部在本地 $O(\text{路径长度})$ 内完成，不需要任何全局计算。你可以在公共 BGP looking glass（路由镜）网站上实时查看任意前缀的真实 AS_PATH——上一讲我们查过 Stanford 是 AS32，哪天你查到的路径里出现 32，就是这套机制在替你记录来路。

### <span style="color:#2471a3;">**[section]**</span> Slide 12 · So Many Choices...（选择这么多……）

图示：peer（对等）、provider（提供商）、customer（客户）三类邻居环绕着 AS 4——"Frank's Internet Barn"（Frank 的互联网谷仓）。前缀 13.13.0.0/16 位于 AS 1。问题：Frank 应该选哪条路由去往 13.13.0.0/16？

### <span style="color:#2471a3;">**[section]**</span> Slide 13 · Frank's Choices...（Frank 的选择）

规则：<span style="color:#00838f;">**从 customer 学到的路由，优先于从 peer 学到的；后者又优先于从 provider 学到的。**</span>

- provider 路线：local pref = 80
- peer 路线：local pref = 90
- customer 路线：local pref = 100

设置合适的 local preference（本地优先级）来反映偏好：**数值越高越优先**。

### <span style="color:#2471a3;">**[section]**</span> Slide 14 · Summary（小结）

- 互联网上所有 AS 必须使用 BGP-4 连接。
- BGP-4 是路径向量算法，环可以被轻易检测出来。
- BGP-4 提供丰富而复杂的接口，让每个 AS 选择本地私有的策略。
- 每个 AS 自行为流量工程、安全以及任何私有偏好决定本地策略。
