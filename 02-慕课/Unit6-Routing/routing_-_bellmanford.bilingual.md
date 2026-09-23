# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> <span style="color:#c0392b;">Routing — Distance Vector Protocol: Bellman-Ford Algorithm（路由 —— 距离向量协议：Bellman-Ford 算法）</span>

> <span style="color:#7f8c8d;">Source: 慕课/Unit6-Routing/routing_-_bellmanford.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

> <span style="color:#7f8c8d;">脚注：本讲主讲人 Nick McKeown。源 PDF 经 pdftotext 提取存在连字乱码（"Rou$ng"→"Routing"、"Bellman--Ford"→"Bellman-Ford"、"Ini/ally"→"Initially"、"A\er"→"After"、"ligle"→"little"、"Rou/ng Informa/on Protocol"→"Routing Information Protocol"、"Solu9ons"→"Solutions"、"be^er"→"better" 等），均按上下文复原。示例表格中个别单元格在提取时碎裂，已按算法逻辑复原并注明。</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1：标题页</span>

> CS144: An Introduction to Computer Networks
> **Routing（路由）**
> Distance Vector Protocol: Bellman Ford algorithm（距离向量协议：Bellman-Ford 算法）
> Nick McKeown，斯坦福大学电气工程与计算机科学教授。

<span style="color:#7f8c8d;">（Slide 1/9）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 2：问题（Problem）</span>

<span style="color:#00838f;">**问题：路由器如何协同工作，找到最小成本生成树？**（How can routers work together to find minimum cost spanning tree?）</span>

图示：与上一讲相同的带权拓扑——端节点 A、B、C、D、X，路由器 R1–R8，各链路标注代价（如 R1 侧 1、2，R2 侧 5、4，中部 1、2、2、2，R7/R8 侧 4、3、2、3 等）。

<span style="color:#7f8c8d;">（Slide 2/9）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 3：等价问题（Equivalent problem）</span>

上问**等价于**：只在路由器之间（不含端主机）找最小成本生成树。

图示：同一张图，去掉端主机 A、B、C、D，仅保留 R1–R8 与目标 X 的子图——因为端主机不参与路由计算，树只需覆盖路由器。

<span style="color:#7f8c8d;">（Slide 3/9）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 4：分布式 Bellman-Ford 算法（The Distributed Bellman-Ford Algorithm）</span>

示例：求到 R8 的最小成本生成树。假设每台路由器知道**到各邻居的链路代价**。路由器 $R_i$ 维护一个值：到 R8 的代价 $C_i$。向量 $C = (C_1, C_2, \ldots, C_7)$ 就称为到 R8 的**距离向量**（distance vector，距离向量）。初始时，设 $C = (\infty, \infty, \ldots, \infty)$。

1. 每隔 $T$ 秒，$R_i$ 把自己的 $C_i$ 发给**邻居**；
2. 若 $R_i$ 从邻居处学到**更低的代价路径**，则更新 $C_i$；
3. 重复，直到不再变化。

把同一套流程对 R1–R7 各做一遍，即得到所有路由器的树（natural extension to calculate tree for R1–R7）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 与 CLRS 第 24 章逐点对照最能看清分布式版改了什么。CLRS 的 Bellman-Ford：单源 $s$，对全部 $|E|$ 条边做 $|V|-1$ 轮松弛，$O(VE)$，允许负权边还能检出负环。分布式版把"轮"变成了**异步消息**：松弛 $d_v \leftarrow \min(d_v, d_u + w(u,v))$ 中的 $d_u$ 不再来自同一次循环里的本地数组，而来自邻居**上一次通告的快照**（可能是陈旧的）；"$|V|-1$ 轮"变成了"$T$ 秒一次、无穷循环直到稳定"。信息传播速度由此受物理限制：每 $T$ 秒，"好消息"（代价下降）最多向外推进一跳——这与 CLRS 动态规划解释里 $d^{(k)}(v) = \min_u (d^{(k-1)}(u) + w(u,v))$（$k$ 表示"至多 $k$ 条边"）完全同构，故**最坏收敛时间约为（网络直径 × $T$）**。还能看出内存账：每路由器只需存邻居数个条目（对比链路状态要存全网 $O(V)$ 条目），这正是它在 1980 年代硬件约束下的巨大优势。至于为什么初始全 $\infty$ 却最终收敛：代价沿"从 R8 出发的一跳邻居→两跳邻居→…"逐层收紧，$\infty$ 的单调下降性保证了无环更新。

<span style="color:#7f8c8d;">（Slide 4/9）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 5：示例（An example）</span>

同一拓扑（边权同上一讲从 R8 出发的例子：R7–R8 权 1，R6–R7 权 1（源文此处为 2，按最短代价复原），R4–R6 权 2，R3–R4 权 2，R5–R4 权 2，R2–R3 权 1，R1–R2 权 1，R2–R5 权 1 等）。各路由器到 R8 的（代价，下一跳）随时间收敛的过程如下（`*` 表示该单元格在 PDF 提取时碎裂，按算法逻辑复原）：

| 路由器 | 初始（只有直连） | 第 1 轮后 | 第 2 轮后 | 第 3 轮后（收敛） |
|---|---|---|---|---|
| R1 | $(8, R3)$ | $(8, R3)$ | $(7, R2)$ | $(7, R2)$ |
| R2 | $(7, R5)$ | $(6, R4)$ | $(5, R5)$ | $(5, R5)$ |
| R3 | $4$ | $4$ | $4$ | $4$ |
| R4 | $(2, R7)$ | $(2, R7)$ | $(2, R7)$ | $(2, R7)$ |
| R5 | $6$ | $6$ | $(4, R4)$ | $(4, R4)$ |
| R6 | $2$ | $2$ | $2$ | $2$ |
| R7 | $1$ | $1$ | $1$ | $1$ |

（末行碎裂片段 "6,R4 5,R5 4 2,R7 4,R4 2 1 5" 即上述各单元格第二轮取值的拼接，按原表列序复原。）

注意观察"好消息"的传播：R7 首轮就稳定为 1，它的邻居 R4、R6 第二轮稳定，R3、R5 再下一轮……代价逐层确定，与距离向量的"$T$ 秒一跳"一致。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这张表值得当一道 6.006 追踪题亲手验算一遍，它把"异步松弛"的全过程摊开给你看。看 R1 的变化：初始 $C_1 = 8$（经 R3：$4 + 4$），第二轮 R2 通告 $C_2 = 5$，R1 立即改走 R2：$2 + 5 = 7$——注意它用的 $C_2 = 5$ 是邻居**上一轮的值**，这就是异步版的松弛。看 R5 为什么会从 6 降到 4：第二轮它收到 R4 的 $C_4 = 2$，$2 + 2 = 4$ 优于原值。再注意 R3 恒为 4、R7 恒为 1——离 R8 近的路由器**先收敛**，稳定值像波一样从目的端荡开，这正是"每 $T$ 秒传播一跳"的直观证据。若把这题放进概率论语境：$T$ 秒一轮是理想化的同步假设，真实 RIP 中通告定时器带随机抖动（防止全网同时发通告造成风暴），消息还可能丢失或乱序，于是"第几轮"本身成了随机变量——分布式算法的收敛分析因此比单机 $O(VE)$ 多了一整层时序不确定性。

<span style="color:#7f8c8d;">（Slide 5/9）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 6：分布式 Bellman-Ford（三个问题）</span>

**Questions（问题）**：

1. 算法最长要运行多久？（What is the maximum run time of the algorithm?）
2. 算法是否总能收敛？（Will the algorithm always converge?）
3. 当链路代价变化，或路由器/链路失效时，会发生什么？（What happens when link costs change, or when routers/links fail?）

<span style="color:#7f8c8d;">（Slide 6/9）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 7：Bellman-Ford 的一个问题："坏消息传得慢"（A Problem with Bellman-Ford: "Bad news travels slowly"）</span>

图示：四台路由器 R1–R2–R3–R4 依次相连，各链路代价均为 1（R3–R4 之间为最后一段）。考虑**到 R4 的距离**的计算。某时刻链路 R3–R4 失效（Link R3–R4 fails）。

| Time（时刻） | R1 | R2 | R3 |
|---|---|---|---|
| 0 | $(3, R2)$ | $(2, R3)$ | $(1, R4)$ |
| 1 | $(3, R2)$ | $(2, R3)$ | $(3, R2)$ |
| 2 | $(3, R2)$ | $(4, R3)$ | $(3, R2)$ |
| 3 | $(5, R2)$ | $(4, R3)$ | $(5, R2)$ |
| … | … | … | … |

（表中数值为"（到 R4 的估计代价，下一跳）"；部分行按源文排列，节奏为每轮代价互涨。）

R3 发现 R4 不可达后，本应把 $C_3$ 置为 $\infty$；但 R2 手里还攥着"经 R3 只要 2"的旧路径，于是 R2 告诉 R3"我这儿 2"，R3 立即采纳：$C_3 = 2 + 1 = 3$——可这条"路"绕回去还是要经过失效的链路！代价随后在 R2、R3 之间互相抬升：3 → 4 → 5 → …，**一路"数到无穷"**（"…Counting to… infinity"…"Coun/ng to Infinity"，计数到无穷）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这是距离向量范式的经典暗礁，值得用第一性原理拆透。故障前全网到 R4 的距离是"对"的：R3 直连（1），R2 经 R3（2），R1 经 R2（3）。R3–R4 断掉的那一刻，**R3 只知道"直连这条路没了"**，它无法区分"R4 彻底不可达"和"R4 换了条远路"。而 R2 递来的 $(2, R3)$ 恰好是一个陈旧快照——R2 的 2 本来就是"经 R3 得来的"，如今却反过来成了 R3 的救命稻草。这个**循环论证**（R2 信 R3、R3 信 R2）就是问题本体：Bellman-Ford 的正确性证明（CLRS 24.2）依赖"$|V|-1$ 轮内用**同一份**边权松弛完毕"，而这里边权在轮与轮之间被中间结果污染了——图在"动态生长"。于是好消息（代价变小，立即采纳）与坏消息（代价变大，被邻居的陈旧乐观值顶回来）**传播速度严重不对称**：好消息每 $T$ 秒一跳，坏消息却要两人互相抬升、一轮涨 1 地爬。极端链状拓扑下要爬到人为规定的"无穷"才罢休，期间数据包在 R2/R3 之间被来回弹——这就是"count-to-infinity 期间路由环与丢包"的真实成因。

<span style="color:#7f8c8d;">（Slide 7/9）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 8：计数到无穷问题与解法（Counting to Infinity Problem — Solutions）</span>

**Solu9ons（Solutions，解法）**：

1. **把"无穷"设为一个较小的整数**（如 16）：计数数到 16 就停止，视为不可达。
2. **水平分割（Split Horizon，水平分割）**：R2 的最低代价路径是从 R3 学来的，所以 R2 **不再把到该目的的代价通告回 R3**（从谁学来的路，就不向谁宣传同一条路）。
3. **带毒 reverse 的水平分割（split-horizon with poison reverse，毒性反转）**：更狠一步——R2 向 R3 通告"到 R4 代价为无穷大"，主动封死这条错觉路径。
4. Bellman-Ford 算法还有许多其他问题与相应修法（There are many problems with (and fixes for) the Bellman-Ford algorithm）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 三个解法各是一层加固，强度与代价都不同。设 $\infty = 16$ 只是**止损**：环还会转，但转 16 轮就承认不可达——RIP 正是取 metric 上限 16（15 跳可用，16 即不可达），这同时解释了 RIP 为何不适合直径超过 15 的网络。水平分割在**逻辑上釜底抽薪**：信息"从哪来回哪去"的路不再回传，循环论证被剪断；它对两节点环有效，但注意它对**三个及以上节点的环**仍可能失效（A 学自 B、B 学自 C、C 学自 A 的三角环会绕开各自的水平分割约束）。毒性反转最彻底——宁可浪费一点带宽把"此路不通"喊出去（持久的 $\infty$ 通告），换确定性；工程上 RIP-2（RFC 2453）默认对水平分割口发毒性路由。和 6.042J 里"打破循环依赖"的思路对照很有意思：三者本质上都是给图定向或给计数封顶，把"无界等待"换成"有界失败"。但真正的教训是更一般的：**分布式系统的每个节点都在用别人的旧信息决策**，只要信息可以循环支撑，任何乐观估值都可能自我实现——BGP 后来选择"路径向量"（通告完整路径而非数字），让环检测成为本地检查，正是吸取了这一整类教训。

<span style="color:#7f8c8d;">（Slide 8/9）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 9：Bellman-Ford 在实践中（Bellman Ford in practice）</span>

- Bellman-Ford 算法是**距离向量（Distance Vector）算法**的一个实例。
- 它被用于 Internet 的**第一个路由协议——RIP（Routing Information Protocol，路由信息协议）**。
- 它在路由器上**计算量极小**、是分布式的、并且**最终会收敛**。
- 随时间推移，它逐渐被"在每台路由器上计算**整棵**生成树"的算法（即链路状态类算法）所取代。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> RIP 的兴衰就是一部"硬件约束释放后算法换代"的技术史。1980 年代它随 BSD UNIX 的 `routed` 守护进程出厂（与你在 CSAPP 第 11 章看到的 UNIX 网络栈一脉相承），跑在 UDP 520 端口上，每 30 秒全表广播一次——选 UDP 而非 TCP 正是因为路由通告要"新比全重要"，宁可丢一轮也不能让重传阻塞更新。它败给 OSPF 的三个原因恰好是它三个优点的反面：计算量小 → 每节点只存一维向量，但**全网视野为零**，无法支持按链路带宽配置代价；分布式自愈 → 30 秒定时器加计数到无穷使收敛可能拖到**分钟级**，期间持续丢包；实现简单 → 跳数度量无法表达"这条 10Gbps 链路便宜、那条 64kbps 链路昂贵"的策略需求。对照一下两条技术路线的取舍账：距离向量省的是"每节点的状态与计算"，链路状态省的是"全局收敛时间与策略表达力"——1980s 内存以 KB 计时前者赢，1990s 内存以 MB 计时后者全面胜出。今天距离向量并未绝迹：EIGRP 与 IPv6 时代的 RPL（低功耗有损网络路由）仍延续其血脉，因为"省内存"在物联网节点上重新变成了硬约束。

<span style="color:#7f8c8d;">（Slide 9/9）</span>

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 6-2《Routing - Bellman Ford》的配套 slides；与《Dijkstra》一讲对照：距离向量"人人只知邻居"vs 链路状态"人人手握全图"，是 Internet 内部路由协议（RIP vs OSPF）的分野所在。</span>
