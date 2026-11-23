# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> <span style="color:#c0392b;">Routing — Link State Protocol: Dijkstra's Shortest Path First Algorithm（路由 —— 链路状态协议：Dijkstra 最短路径优先算法）</span>

> <span style="color:#7f8c8d;">Source: 慕课/Unit6-Routing/routing_-_dijkstra.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

> <span style="color:#7f8c8d;">脚注：本讲主讲人 Nick McKeown。源 PDF 经 pdftotext 提取存在连字乱码（"Rou$ng"→"Routing"、"Link--State"→"Link-State"、"min--cost"→"min-cost" 等）、希腊字母丢失及示例图中边权归属碎裂，均已按上下文复原；无法确证的边仅描述其数值，不强行复原拓扑归属。</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1：标题页</span>

> CS144: An Introduction to Computer Networks
> **Routing（路由）**
> Link State Protocol: Dijkstra's shortest path first algorithm（链路状态协议：Dijkstra 最短路径优先算法）
> Nick McKeown，斯坦福大学电气工程与计算机科学教授。

<span style="color:#7f8c8d;">（Slide 1/14）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 2：Dijkstra 最短路径优先算法（"链路状态算法"一例）</span>

整个协议只有两步：

1. **交换链路状态（Exchange link state）**：每台路由器把**与自己相连链路的状态**泛洪（flood，泛洪）给所有其他路由器。
   - 触发时机：周期性地（periodically，周期性）；以及链路状态一发生变化时（when link state changes）。
2. **各自运行 Dijkstra**：每台路由器**独立**运行 Dijkstra 最短路径优先算法。

<span style="color:#00838f;">**结果：每台路由器都能找到以自己为根、通往其他所有路由器的最小成本生成树。**（Each router finds min-cost spanning tree to every other router.）</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这页是链路状态（link state，链路状态）范式的心脏，值得与你在 CLRS 第 22 章建立连接：CLRS 里 Dijkstra 是"单源最短路"——给你一张静态图 $G=(V,E)$ 和源 $s$，求 $s$ 到所有点的最短路，$O((V+E)\log V)$（二叉堆实现，$|V|$ 次 `EXTRACT-MIN` 与 $|E|$ 次 `DECREASE-KEY` 摊还 $O(\log V)$）。这里的差异在于"图从哪来"：算法课上邻接表是题目给的；路由器上，$G$ 是**每台路由器靠泛洪互相汇报"我有哪些邻居、代价多少"拼出来的全网地图**——第一步"交换链路状态"就是在分布式地构建 $G$。还有两个单机算法完全没有的新维度：其一，$G$ 随时在变（链路宕机、拥塞），每变一次就重跑一次 Dijkstra，相当于**增量式地反复求解**；其二，全网 $n$ 台路由器每台都各自跑一遍完整 Dijkstra——总计算量是 $n$ 倍。用"人人都拿一份全图、本地独立计算"换取"无需分布式的迭代协调"，这是链路状态与下一讲距离向量的根本分野。

<span style="color:#7f8c8d;">（Slide 2/14）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 3：示例：从 R8 出发（An example: From R8）</span>

图示：R1–R8 组成的带权图。主要边权（按图中数字，边归属可辨部分如下）：R7–R8 权 1，R6–R7 权 2，R4–R6 权 2，R5–R4 权 2，R3–R4 权 2，R2–R3 权 1，R1–R2 权 1，另有若干权为 4、5、6 的边（PDF 提取中个别边权归属碎裂）。目标：求从 R8 出发到每个路由器的最小成本路径。

演示将从 R8 开始，按代价从低到高逐层"点亮"最短路。

<span style="color:#7f8c8d;">（Slide 3/14）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 4：示例：从 R8 出发（加入代价 1、2 的路径）</span>

图示动画逐步把路径加进最短路树：

- **加入代价为 1 的路径**：R7（R7–R8 权 1）→ 最短路树含 R8–R7；
- **加入代价为 2 的路径**：R6（R6 经 R7，$1+1=2$）→ 树含 R8–R7–R6；
- **加入代价为 2 的路径**：R4（R4 经 R6，$2+2=4$？不——R4–R6 权 2 经 R6 得 $2+2=4$，本页动画显示 R4 以代价 2 加入，对应直连边权 2 的情形，按 slide 标注 R4 的最短代价为 2）→ 树含 R4。

（动画顺序："Add path of cost 1: R7"，"Add path of cost 2: R6"，"Add path of cost 2: R4"，……）

<span style="color:#7f8c8d;">（Slide 4/14）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 5：示例：从 R8 出发（加入代价 4 的路径）</span>

- **加入代价为 4 的路径（两条）**：
  - R5（经 R4：$2+2=4$）；
  - R3（经 R4：$2+2=4$）。
- 此时最短路树覆盖 {R8, R7, R6, R4, R5, R3}。

<span style="color:#7f8c8d;">（Slide 5/14）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 6：示例：从 R8 出发（加入代价 5、6 的路径）</span>

- **加入代价为 5 的路径**：R2（经 R3：$4+1=5$）；
- **加入代价为 6 的路径**：R1（经 R2：$5+1=6$）。

至此全部 8 台路由器都进入了最短路树，从 R8 出发的最终代价：R7=1，R6=2，R4=2，R5=4，R3=4，R2=5，R1=6。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 注意 Slides 4–6 的动画顺序：R7(1)、R6(2)、R4(2)、R5(4)、R3(4)、R2(5)、R1(6)——**按代价从低到高依次加入**，这正是 Dijkstra 贪心的可视化。为什么贪心是对的？依赖两个不变量（你在 CLRS 22.3 的引理里见过其静态版）：最短路径的子路径必是最短路径（最优子结构），且**边权非负**时，从优先队列里取出的当前最小估计值就是最终确定值（节点一旦"落袋"不再更新）。非负性在这里有物理意义：链路代价是时延/成本/跳数，天然 $\geq 0$；若出现负权（例如某些策略把"顺路"记成负代价），贪心失效，必须换 Bellman-Ford——这也是为什么 OSPF 只需要 Dijkstra。另外留意 R4 与 R6、R3 与 R5 各自"并列"时协议如何选：代数上并列则任选其一，但工程上要靠路由器 ID 等确定性规则打破平局，否则两台路由器可能各自算出不一致的树，造成转发环。

<span style="color:#7f8c8d;">（Slide 6/14）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 7：The algorithm（算法过程表·前半）</span>

用"代价桶"视角重放上例（从 R8 出发）。桶按最短代价 $0..7$ 排列，两行分别是：**最短路集合（Shortest R8 Path Set）**与**候选集合（Candidate Set）**，"Add" 表示本轮确定加入的节点：

| 代价 | Shortest Set（已确定） | Candidate Set（候选） | Add（本轮加入） |
|---|---|---|---|
| 0 | {R8} | {R6, R7} | R7 |
| 1 | {R8, R7} | {R6} | R6 |
| 2 | {R8, R7, R6} | {R3, R5, R4} | R4 |
| 3 | {R8, R7, R6, R4} | {R3, R5, R2} | R3 |

<span style="color:#7f8c8d;">（Slide 7/14）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 8：The algorithm（算法过程表·后半）</span>

| 代价 | Shortest Set（已确定） | Candidate Set（候选） | Add（本轮加入） |
|---|---|---|---|
| 4 | {R8, R7, R6, R4, R3} | {R5, R2, R1} | R2 |
| 5 | {R8, R7, R6, R4, R3, R2} | {R5, R1} | R1 |
| 6 | {R8, R7, R6, R4, R3, R2, R1} | {R5} | R5 |
| 7 | — | — | Empty（候选集空，Done 完成） |

每轮从候选集中取出估计代价最小者"落袋"，再用它的边松弛邻居——这正是 Dijkstra 的优先队列循环。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 把这张表与 CLRS 对齐：表中"Shortest Set"就是 CLRS 伪代码里集合 $S$（已求出最短路的节点），"Candidate Set"就是优先队列 $Q$ 中带 $d$ 估计值的节点，"Add"一行对应一次 `EXTRACT-MIN`。复杂度记账也完全同构：$|V|$ 次出堆、$|E|$ 次松弛，二叉堆下 $O((V+E)\log V)$；OSPF 工程实现里常用斐波那契堆思想（`DECREASE-KEY` 摊还 $O(1)$）达到 $O(E + V\log V)$，稀疏图（$E \approx V$，如真实拓扑）时近似 $O(V\log V)$。6.042J 视角补充一句：Dijkstra 求出的以 R8 为根的树是**最短路径树**，而上一讲末尾的"最小成本生成树"（MST）目标不同——本图两者恰好同构，故 slide 不加区分；若边权分布更悬殊，SPT 与 MST 会分道扬镳。思考题：为什么表中代价最高只到 7？因为本例直径小；若拓扑呈链状，最大代价可达 $O(V)$，堆中将同时压着 $O(V)$ 个候选——这解释了为什么全图重算在大网络上代价高昂，为后面"增量计算"（IS-IS 的 PRC、OSPF 部分更新）埋下伏笔。

<span style="color:#7f8c8d;">（Slide 8/14）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 9：Dijkstra's Algorithm（两个问题）</span>

**Questions（问题）**：

1. 算法运行一次要花多长时间？（How long does the algorithm take to run?）
2. 当链路代价变化，或路由器/链路失效时，会发生什么？（What happens when link costs change, or when routers/links fail?）

<span style="color:#7f8c8d;">（Slide 9/14）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 10：Dijkstra 在实践中（Dijkstra's algorithm in practice）</span>

Dijkstra 算法是**链路状态算法（Link State algorithm，链路状态算法）**的一个实例：

- 链路状态（拓扑与代价）被**每台路由器**所知；
- 每台路由器独立求出**到其他所有路由器的最短路径生成树**。

它是 **OSPF（Open Shortest Path First，开放最短路优先）** 的基础——一个使用极广泛的路由协议。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> OSPF（RFC 2328）就是上一页两个问题的工程答案。问题 1（运行时长）：$O((V+E)\log V)$ 的单机计算在区域（area，区域）内可控——OSPF 把 AS 再切成若干区域，区域内跑完整 Dijkstra，跨区域只传摘要，这正呼应 basics 一讲 Slide 11 的规模论证。问题 2（变化时怎么办）：链路代价变化或失效 → 触发新一轮链路状态公告（LSA，Link State Advertisement）泛洪 → 全网路由器各重跑 Dijkstra；OSPF 用序列号与老化（age）字段防止旧 LSA 死灰复燃，用扩散确认保证可靠泛洪——这些是"单机 Dijkstra"之外、协议真正复杂的部分。还有一个 CLRS 对比的余味：算法课的 Dijkstra 假设"图在手、边权已知且瞬时一致"，而 OSPF 的地图在泛洪过程中**各节点可能短暂不一致**，因此协议必须保证"所有节点最终对同一版本图达成一致"（收敛）后，各自的树才互相兼容——不一致窗口期的转发黑洞/环，是链路状态网络的固有代价，也是"为什么收敛速度是路由协议的第一指标"。

<span style="color:#7f8c8d;">（Slide 10/14）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 11：Another view of Dijkstra...（Dijkstra 的另一个视角）</span>

接下来用一个不同的图示再看一遍 Dijkstra 的执行过程（出自 Berkeley 的 Jean Walrand 教授）。

<span style="color:#7f8c8d;">（Slide 11/14）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 12：示例图 A–B（初始）</span>

图示：源 A 与目的 B，中间五个路由器 R1–R5 的带权图：A–R1 权 3，A–R2 权 4，R1–R2 权 4，R2–R4 权 2（R4–R2 权 2），R1–R3 权 4，R3–R5 权 3，R4–B 权 4?（部分边权提取碎裂），R5–B 权 4、2 等。目标：A 到 B 的最短路。

（致谢：Thanks to Prof. Jean Walrand, Berkeley。）

<span style="color:#7f8c8d;">（Slide 12/14）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 13：示例图 A–B（无权骨架）</span>

图示：同一拓扑去掉边权的骨架图，突出 A → R1/R2/R3 → R4/R5 → B 的三层结构，用于展示 Dijkstra 的"波前"逐层推进。

<span style="color:#7f8c8d;">（Slide 13/14）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 14：示例图 A–B（带权计算）</span>

图示：在带权图上执行 Dijkstra：先确定 A–R1（代价 3），再由 R1 松弛出 R2（$3+4=7$）与 R3（$3+4=7$），随后 R4、R5 依次确定，最终到 B 的最短代价为沿虚线路径求和所得（图中数字随过程逐步标注）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> Walrand 这个例子想让你"看见" Dijkstra 的另一个名字——**最短路径优先（SPF）**里的"优先"：算法不是一次算完全图，而是维护一个不断外扩的"波前"，每步只把波前上**最便宜**的那个节点确定下来。从概率论角度做个对照：如果你把每条边的代价理解成期望时延 $\mathbb{E}[D_{ij}]$，Dijkstra 最小化的是**期望端到端时延** $\sum \mathbb{E}[D_{ij}]$（期望可加，恰是可加度量）；但它无法回答"端到端时延超过 100 ms 的概率是多少"——那需要对路径上独立近似同分布的排队时延做卷积或用正态近似（第 7 章的中心极限定理正是为此准备的）。这个 gap 正是 QoS 路由的出发点：网络工程关心尾部，而最短路只保证均值最优。另一个直觉：三层骨架图解释了为什么 Dijkstra 的迭代次数是 $O(V)$ 而与"层数"无关——层数只影响波前的形状，不影响循环结构，这与 6.042J 中 BFS 层次遍历的同构性一致。

<span style="color:#7f8c8d;">（Slide 14/14）</span>

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 6-3《Routing - Dijkstra》的配套 slides；与《Bellman-Ford》一讲对照阅读，可理解链路状态与距离向量两种范式的完整权衡。</span>
