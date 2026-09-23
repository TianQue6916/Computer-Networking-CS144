# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> TCP Setup and Teardown（TCP 连接建立与拆除）

> <span style="color:#7f8c8d;">Source: 慕课/Unit2-Transport/reliable-connections.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1：TCP Setup and Teardown（TCP 建立与拆除）</span>

本讲义共 12 页，主题：三次握手（3-way handshake）、同时打开（simultaneous open）与 TCP 状态机（TCP state machine）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 2：Problem（问题）</span>

- 可靠通信（reliable communication）通常受益于在**连接的两端各保留一些状态**（state）
- 问题一：连接建立（connection establishment）——如何建立这些状态？
- 问题二：连接拆除（connection teardown）——如何清理干净（复用端口等资源）？

<span style="color:#7f8c8d;">（Slide 2/12）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 3：TCP Header（TCP 首部回顾）</span>

本页重复给出 TCP 首部布局图：source port（源端口）、destination port（目的端口）、sequence number（序列号）、acknowledgment number（确认号）、offset、reserved 与 U A P R S F 标志位、window（窗口）、checksum（校验和）、urgent pointer（紧急指针）、options（选项）、padding（填充），按 32 bits（4 octets）一行排列。连接管理将用到其中的 SYN/FIN/ACK 标志位。

<span style="color:#7f8c8d;">（Slide 3/12）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 4：Connection Setup（连接建立）</span>

本页再次展示同一张 TCP 首部图，提示接下来的讨论聚焦在建立连接时用到的字段（SYN、ACK 标志位与序列号字段）。

<span style="color:#7f8c8d;">（Slide 4/12）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 5：3-way Handshake（三次握手）</span>

- **主动开启方（active opener）**发送第一个分组：
  - 带有序列号（sequence number）的 SYN 分组
- **被动开启方（passive opener）**回应：
  - 带有序列号的 SYN 分组
  - 并确认（ACK）主动方的 SYN 分组
- 主动开启方再回应：
  - 确认被动方的 SYN 分组
- 同时支持"**同时打开（simultaneous open）**"：
  - 两个 SYN 在网络中交错而过（two SYNs pass each other）
  - 双方各自确认对方的 SYN

<span style="color:#7f8c8d;">（Slide 5/12）</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 为什么是三次握手而不是两次？从第一性原理推：握手要同时完成两件事——双方各自同步一个初始序列号（ISN，initial sequence number），并且确认双向信道都通。两次握手只让被动方知道对方的序号是 SA，主动方却不知道"被动方的序号 SB"有没有被自己确认到位；更关键的是网络里存在上一条连接迟到的旧 SYN 分组（延迟、重放），若两次握手就进入数据传输，一个迟到的旧 SYN 会让服务器白白挂起半条连接。1985 年 Morris 的序列号预测攻击之后，各系统还把 ISN 从递增计数改为随机选择，顺带提升了安全性。"同时打开"的存在说明 TCP 并不固定"谁是客户端"——三次握手本质是两个独立的 SYN/SYN-ACK 交换的叠加。这种"用对称设计消除角色假设"的思路，与 6.042J 里验证双向关系所需信息量的论证是同一类推理。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 6：TCP Connection Setup (no data)（TCP 连接建立：不携带数据）</span>

纯图示页：两条时间线（active 与 passive 两端）按序交换 `SYN (seq=x)` → `SYN, ACK (seq=y, ack=x+1)` → `ACK (ack=y+1)` 的报文序列，随后进入 ESTABLISHED（已建立）状态，全程不携带应用数据。

<span style="color:#7f8c8d;">（Slide 6/12）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 7：TCP Connection Setup (with data)（TCP 连接建立：携带数据）</span>

纯图示页：与上一页相同的三次握手时序，但握手报文可以捎带数据——例如主动方的第三次 ACK（或 SYN 之后的首个报文）即可携带应用数据，说明三次握手完成后数据可立即在两个方向上流动。

<span style="color:#7f8c8d;">（Slide 7/12）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 8：Connection Teardown（连接拆除：首部视角）</span>

本页再次展示 TCP 首部图，提示拆除阶段将使用 FIN（结束位）与 ACK 标志位。

<span style="color:#7f8c8d;">（Slide 8/12）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 9：Connection Teardown（连接拆除）</span>

- **FIN 位表示"我不再有数据要发了"**：
  - 由对端调用 `close()` 或 `shutdown()` 触发
- 双方都必须各自发送 FIN，连接才能终止
- 典型的拆除报文交换（typical teardown exchange）：
  - A → B：FIN, seq $S_A$, ack $S_B$
  - B → A：ack $S_A + 1$
  - B → A：FIN, seq $S_B$, ack $S_A + 1$
  - A → B：ack $S_B + 1$
- 也可能发生"**同时关闭（simultaneous close）**"
- 问题：A 和 B 在收到最后一条报文后，能否立刻忘掉（forget）这个已关闭的套接字（closed socket）？

<span style="color:#7f8c8d;">（Slide 9/12）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 10：Cleaning Up Safely（安全地清理）</span>

- 已关闭套接字（closed socket）的问题：
  - 如果**最后的 ACK 在网络中丢失**怎么办？
  - 如果**同一对端口立即被新连接复用**怎么办？
- 解决方案：**主动关闭方进入 TIME-WAIT 状态**
  - 主动关闭（active close）指先于收到 FIN 之前发送 FIN 的一方
  - 将套接字保留 $2\,\mathrm{MSL}$（两倍的"最大分段生存时间"，maximum segment lifetime）
- 这可能给服务器带来麻烦：
  - 操作系统中过多的 TIME-WAIT 套接字会拖慢系统
  - 变通手段（hack）：发送 RST 直接删除套接字；把套接字选项 `SO_LINGER` 设为 0
  - 操作系统不允许重启服务器，因为端口仍被占用（`SO_REUSEADDR` 选项允许重新绑定已用过的端口号）

<span style="color:#7f8c8d;">（Slide 10/12）</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> TIME-WAIT 是把序列号空间思想扩展到"连接生命周期"上的结果。设 MSL（maximum segment lifetime，最大分段生存时间）为任何分组在网络中的最长存活时间（历史上取 2 分钟）：一个 MSL 保证旧连接的残余分组自然死亡，再一个 MSL 保证若最后的 ACK 丢失、对端重传 FIN 时自己还在场、能重新应答——否则对端只会收到 RST，关闭就不干净。这个问题在 CSAPP 第 11 章有非常实用的对应物：重启服务器时常见的 "Address already in use" 报错，就是上一条连接的 TIME-WAIT 套接字还占着端口；解决办法正是幻灯片列出的 `SO_REUSEADDR`（允许绑定处于 TIME-WAIT 的端口），而 `SO_LINGER` 置 0 则是"发 RST 立即销毁套接字"的硬手段——你在 CSAPP 里见过 `setsockopt()` 的名字，这里是它们存在的完整原因。高并发服务器上成千上万个 TIME-WAIT 套接字拖慢系统，正是这条安全设计的代价。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 11：Connection Teardown（连接拆除：状态图）</span>

纯图示页：时间线图展示双方从 ESTABLISHED 出发，分别经过 `FIN → ACK → FIN → ACK` 的报文交换，各自进入关闭相关状态（如 FIN-WAIT、TIME-WAIT、CLOSED）的完整过程。

<span style="color:#7f8c8d;">（Slide 11/12）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 12：Full TCP FSM（完整 TCP 状态机）</span>

纯图示页：完整的 TCP 状态机图，包含 LISTEN、SYN-SENT、SYN-RECEIVED、ESTABLISHED、FIN-WAIT-1、FIN-WAIT-2、CLOSE-WAIT、CLOSING、LAST-ACK、TIME-WAIT、CLOSED 等状态及带标注的转移边。图片来源：http://en.wikipedia.org/wiki/File:Tcp_state_diagram_fixed.svg

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这页状态机值得用 6.042J 的图论眼光重看一遍：每个状态是顶点（vertex），每条带"事件/动作"标注的转移是有向边（edge），三次握手是从 LISTEN/SYN-SENT 到 ESTABLISHED 的路径，拆除则是经 FIN-WAIT、TIME-WAIT 回到 CLOSED 的另一条路径。它与 2-6 节停等 FSM 的差别在状态数：停等只有 2 个状态，TCP 有 11 个，多出来的状态大多不为正常路径服务，而是为异常路径（同时打开、同时关闭、半打开连接）准备的——把协议写成显式状态机的好处是可以机械验证：每条边的事件、动作、守卫条件写清楚后，"两个 FSM 组合是否会死锁"就能像图的可达性分析一样处理，这正是 6.042J 状态机不变式方法的用武之地。另外注意状态转移由**到达的报文**驱动而非由"角色"驱动，所以同时打开/同时关闭都有一条合法路径——对称设计让 TCP 不依赖"谁是客户端"的假设，与端到端原则的边缘智能立场一脉相承。

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 2-11《Reliable comm --- Connection setup and teardown》的配套 slides；TCP 状态机与 CS144 Lab 的 SYNH/ack 机制直接相关。</span>
