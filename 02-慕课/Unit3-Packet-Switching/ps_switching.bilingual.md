# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Packet Switching: How a packet switch works (1)（分组交换：分组交换机如何工作（1））

> <span style="color:#7f8c8d;">Source: 慕课/Unit3-Packet-Switching/ps_switching.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

> <span style="color:#7f8c8d;">提取说明：源 PDF 由 pdftotext 提取后存在 "ti"→"8" 系统性乱码（如 Lookup Address 中的 Des8na8on = destination、Ac3on = action、Rou8ng = routing），以下译文已按原意复原。</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1：标题页</span>

CS144: An Introduction to Computer Networks
Packet Switching（分组交换）
How a packet switch works (1)（分组交换机如何工作（1））
Nick McKeown，斯坦福大学电气工程与计算机科学教授
CS144, Stanford University

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 2：大纲</span>

1. 分组交换机长什么样？
2. 分组交换机做什么？——以太网交换机（Ethernet switch）、互联网路由器（Internet router）
3. 地址查找如何工作——以太网交换机、互联网路由器

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 3~4：通用分组交换机</span>

通用分组交换机（generic packet switch）的内部流水线（多帧动画展示同一个分组依次通过各阶段）：

分组 `Data H` 进入 → **查找地址（Lookup Address）**：在**转发表（forwarding table）**中用首部里的目的地址（destination address）查出**出端口（egress link）** → **更新首部（Update Header）** → **排队分组（Queue Packet）**：进入**缓冲存储器（buffer memory）**中的队列等待发送。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 5：以太网交换机</span>

1. 检查每个到达帧（frame）的首部。
2. 如果以太网目的地址（Ethernet DA）在转发表中，把帧转发到正确的输出端口。
3. 如果以太网目的地址不在表中，把帧**广播**到所有端口（帧到来的那个端口除外）。
4. 表中的表项通过检查到达分组的以太网源地址（Ethernet SA）**学习**得到。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这四条规则描写了一台"零配置、自学习"的设备，妙处全在第 3、4 条的组合：交换机不认识目的地址就向所有端口广播（洪泛，flooding），同时偷看每个到来帧的源地址反向学习"哪个地址挂在哪个端口"——网络流量本身教会了交换机转发表，无需任何管理员配置。这套自学习机制代价是广播的蔓延：若两台交换机之间形成环路，广播帧会永不停止地复制增殖（广播风暴），所以以太网不得不用生成树协议（spanning tree，6.042J 的图论概念：在带环图上求一棵覆盖所有节点的树并屏蔽多余边）来逻辑断环。用 CLRS 的语言概括：交换机转发表是"精确匹配"的键值结构，学习即插入、老化即删除，哈希表是自然实现——第 8 页会印证这一点。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 6：互联网路由器</span>

1. 如果到达帧的以太网目的地址属于本路由器，接受该帧；否则丢弃。
2. 检查 IP 数据报（datagram）的版本号与长度。
3. 将 TTL（生存时间）减 1，更新 IP 首部校验和（checksum）。
4. 检查 TTL 是否等于 0。
5. 如果 IP 目的地址（IP DA）在转发表中，转发到通往下一跳（next hop）的正确出端口。
6. 找到下一跳路由器的以太网目的地址。
7. 创建新的以太网帧并发送。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 把这七步读仔细，你会看到"分层"的运行时实况：路由器是站在以太网与 IP 交界处的翻译官——每一步跳都要**拆掉旧的以太网帧、重新做一个新的**（第 1、6、7 步），因为以太网帧只在一跳内有效；而 IP 层的内容（版本号、TTL、目的地址）却跨越整条路径保持不变，只有两处被合法修改：TTL 减 1 防止路由环路让分组永生不死，校验和随之重算保持首部自洽。这正是 Unit 1 封装（encapsulation）与分层概念的动态版：层与层的"保质期"不同——链路层头部活一跳，网络层头部活端到端。对照 CSAPP 第 11 章：主机内核协议栈只收发"属于自己的帧"（目的 MAC 是自己），路由器则专门收下"路过"的帧再转发，这个差别就是"主机 vs 路由器"的全部。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 7：两种基本操作</span>

1. **查找地址（Lookup Address）**：地址如何在转发表中被查找？
2. **交换（Switching）**：分组如何被送到正确的输出端口？

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 8：查找地址：以太网</span>

以太网地址（在一台交换机中）：

| 匹配（Match） | 动作（Action） |
|---|---|
| Ethernet DA = 0xA8B72340E678 | Forward to port 7 |
| Ethernet DA = 0xB3D22571053B | Forward to port 3 |
| ... | ... |

方法：把地址存进哈希表（可用二路哈希 2-way hash）；在哈希表中查找**精确匹配（exact match）**。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 9：查找地址：IP</span>

IP 地址（在一台路由器中）：

| 匹配（Match） | 动作（Action） |
|---|---|
| IP DA = 127.43.57.99 | Forward to 56.99.32.16 |
| IP DA = 123.66.44.X | Forward to 22.45.21.126 |
| IP DA = 76.9.X.X | Forward to 56.99.32.16 |
| ... | ... |

查找是**最长前缀匹配（longest prefix match）**，而不是精确匹配。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 10：最长前缀匹配</span>

前缀集合：65.0.0.0/8、128.9.176.0/24、128.9.16.0/21、128.9.172.0/21、128.9.0.0/16、142.12.0.0/19。例：目的地址 65.14.24.120 命中 /8 前缀；目的地址 128.9.16.14 同时命中 128.9.0.0/16 与 128.9.16.0/21，选更长的那条。

<span style="color:#00838f;">路由查找：在所有匹配目的地址的前缀中，找出最长匹配前缀（即最具体的路由）。</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 11：最长前缀匹配查找——二进制 trie</span>

用二进制 trie（binary tries）做最长前缀匹配。表项与前缀：a=00001、b=00010、c=00011、d=001、e=0101、f=011、g=100、h=1010、i=1100、j=11110000；以 0/1 为边的二叉树组织这些前缀，标记节点即为表项（图示推断）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 12：最长前缀匹配查找——TCAM</span>

用三态内容寻址存储器（Ternary Content Addressable Memory，TCAM）做最长前缀匹配。每个表项存"二进制值 + 掩码（mask）"，如 a=00001XXX（掩码 11111000）、d=001XXXXX（掩码 11100000）……j=11110000（掩码 11111111）。

<span style="color:#00838f;">路由查找：把地址与每一个带掩码的表项**同时**比较。</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 同一个问题（查表），交换机与路由器走向了两条完全不同的技术路线，这页对照着看最精彩。以太网查找是精确匹配 → 哈希表 $O(1)$，是你在 CLRS 学过的标准散列；IP 查找是"在所有匹配里挑最长前缀" → 天然是树问题：二进制 trie 把地址当 32 位路径走一遍，一步一个分支，32 步内必达叶节点——6.042J 里"前缀码/二叉树"的直接应用。但每步一次内存访问太慢（骨干链路每纳秒就到几个分组），于是硬件路线登场：TCAM 把"值+掩码"写进每个存储单元，让**所有表项在单个时钟周期内并行比对**，硬件自动把最长匹配排到优先输出——用电路面积换时间，比软件树快几个数量级，也贵几个数量级。这个"算法（trie）vs 硬件（TCAM）"的权衡是体系结构课的经典案例，CSAPP 里"存储器层次用 SRAM 换速度"是它的近亲。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 13：查找地址：通用化</span>

通用化/抽象化的查找：**\<Match, Action\>**（匹配，动作）对。

| 匹配（Match） | 动作（Action） |
|---|---|
| IP DA = X | Forward to port 7 |
| Eth DA = Y AND IP DA = Z | Drop packet |

这是对交换机、路由器、防火墙等设备中"查找 + 转发动作"的推广。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 把"查表"抽象成\<Match, Action\>对，看似只是措辞升级，实际上是整个软件定义网络（SDN）运动的理论起点：既然所有网络设备（交换机、路由器、防火墙、负载均衡器）本质都在执行"匹配若干首部字段、执行某个动作"，那么查找规则本身就可以从设备的硬件里**抽出来**，变成由中央控制器统一下发、可编程的流表。这一抽象在 2008 年演化为 OpenFlow 协议（匹配字段更多、动作集更丰富），后来又催生了 P4 这类可编程数据平面语言——McKeown 正是这两项工作的核心作者，讲义里这页就是他留给后文的伏笔。用你熟悉的语言说：这是把"查找逻辑"从固件里解耦出来做成了"策略即数据"，和数据库把查询优化器与存储引擎分离、操作系统把系统调用表做成表驱动，是同一种架构智慧。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 14：总结</span>

分组交换机执行两种基本操作：

- 在转发表中查找地址；
- 交换（switching）到正确的出端口。

在高层面上，以太网交换机与互联网路由器执行相似的操作。但地址查找在交换机与路由器中非常不同。

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 3-7《Packet Switching - Practice: Switching and Forwarding (1)》的配套 slides；"交换到正确端口"的内部机制（输出/输入排队）见 ps_switching_2。</span>
