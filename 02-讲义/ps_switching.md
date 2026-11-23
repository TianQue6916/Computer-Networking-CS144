# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Packet Switching: How a packet switch works (1)（分组交换：分组交换机如何工作（1））

> <span style="color:#7f8c8d;">Source: 慕课/Unit3-Packet-Switching/ps_switching.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

> <span style="color:#7f8c8d;">提取说明：源 PDF 由 pdftotext 提取后存在 "ti"→"8" 系统性乱码（如 Lookup Address 中的 Des8na8on = destination、Ac3on = action、Rou8ng = routing），以下译文已按原意复原。</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1：标题页</span>

CS144: An Introduction to Computer Networks（CS144：计算机网络导论）
Packet Switching（分组交换）
How a packet switch works (1)（分组交换机如何工作（1））
主讲人 Nick McKeown，斯坦福大学电气工程与计算机科学教授
CS144, Stanford University（CS144，斯坦福大学）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 2：大纲</span>

1. packet switch（分组交换机）长什么样？
2. packet switch（分组交换机）做什么？——Ethernet switch（以太网交换机）、Internet router（互联网路由器）
3. address lookup（地址查找）如何工作——Ethernet switch（以太网交换机）、Internet router（互联网路由器）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 3~4：通用分组交换机</span>

一台 generic packet switch（通用分组交换机）的内部 pipeline（流水线）（多帧动画展示同一个 packet（分组）依次通过各阶段）：

一个 packet（分组）`Data H` 进入 → **Lookup Address（查找地址）**：在 **forwarding table（转发表）** 中用 header（首部）里的 destination address（目的地址）查出 **egress link（出端口）** → **Update Header（更新首部）** → **Queue Packet（排队分组）**：进入 **buffer memory（缓冲存储器）** 中的 queue（队列）等待发送。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 5：以太网交换机</span>

1. 检查每个到达的 frame（帧）的 header（首部）。
2. 如果 Ethernet DA（以太网目的地址）在 forwarding table（转发表）中，把 frame（帧）转发到正确的 output port（输出端口）。
3. 如果 Ethernet DA（以太网目的地址）不在表中，把 frame（帧）广播到所有 port（端口）（除了 frame（帧）到来的那个 port（端口））。
4. 表中的 entry（表项）通过检查到达 packet（分组）的 Ethernet SA（以太网源地址）learn（学习）得到。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这四条规则描写了一台"零配置、自学习"的设备，妙处全在第 3、4 条的组合：交换机不认识目的地址就向所有端口广播（即 flooding（洪泛）），同时偷看每个到来 frame（帧）的源地址反向 learn（学习）"哪个地址挂在哪个端口"——网络流量本身教会了交换机 forwarding table（转发表），无需任何管理员配置。这套自学习机制的代价是广播的蔓延：若两台交换机之间形成 loop（环路），广播 frame（帧）会永不停止地复制增殖（broadcast storm（广播风暴）），所以以太网不得不用 Spanning Tree Protocol（生成树协议，STP）来逻辑断环；它依赖的 spanning tree（生成树）是 6.042J（MIT 的《计算机科学中的数学》课程）里的图论概念：在带环图上求一棵覆盖所有节点的树并屏蔽多余边。用 CLRS（《算法导论》，Cormen、Leiserson、Rivest、Stein 四位作者姓氏首字母的合称）的语言概括：交换机的 forwarding table（转发表）是 exact match（精确匹配）的 key-value（键值）结构，learn（学习）即 insert（插入）、aging（老化）即 delete（删除），hash table（哈希表）是自然实现——第 8 页会印证这一点。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 6：互联网路由器</span>

1. 如果到达 frame（帧）的 Ethernet DA（以太网目的地址）属于本 router（路由器），接受该 frame（帧）；否则丢弃。
2. 检查 IP datagram（数据报）的版本号与长度。
3. 将 TTL（生存时间）减 1，更新 IP header（首部）的 checksum（校验和）。
4. 检查 TTL（生存时间）是否等于 0。
5. 如果 IP DA（IP 目的地址）在 forwarding table（转发表）中，转发到通往 next hop（下一跳）的正确 egress link（出端口）。
6. 找到 next hop（下一跳）router（路由器）的 Ethernet DA（以太网目的地址）。
7. 创建新的 Ethernet frame（以太网帧）并发送。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 把这七步读仔细，你会看到 layering（分层）的运行时实况：router（路由器）是站在以太网与 IP 交界处的翻译官——每一跳都要**拆掉旧的 Ethernet frame（以太网帧）、重新做一个新的**（第 1、6、7 步），因为 Ethernet frame（以太网帧）只在一个 hop（跳）内有效；而 IP layer（层）的内容（version（版本号）、TTL（生存时间）、destination address（目的地址））却跨越整条路径保持不变，只有两处被合法修改：TTL（生存时间）减 1 防止 routing loop（路由环路）让 packet（分组）永生不死，checksum（校验和）随之重算保持 header（首部）自洽。这正是 Unit 1 里 encapsulation（封装）与 layering（分层）概念的动态版：层与层的"保质期"不同——link layer（链路层）的 header（头部）活一个 hop（跳），network layer（网络层）的 header（头部）活 end-to-end（端到端）。对照 CSAPP（《深入理解计算机系统》）第 11 章：host（主机）内核的 protocol stack（协议栈）只收发"属于自己的 frame（帧）"（destination MAC（目的 MAC 地址）是自己），router（路由器）则专门收下"路过"的 frame（帧）再转发，这个差别就是"host（主机）vs router（路由器）"的全部。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 7：两种基本操作</span>

1. **Lookup Address（查找地址）**：地址如何在 forwarding table（转发表）中被查找？
2. **Switching（交换）**：packet（分组）如何被送到正确的 output port（输出端口）？

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 8：查找地址：以太网</span>

Ethernet address（以太网地址）（在一台 switch（交换机）中）：

| Match（匹配） | Action（动作） |
|---|---|
| Ethernet DA = 0xA8B72340E678 | Forward to port 7 |
| Ethernet DA = 0xB3D22571053B | Forward to port 3 |
| ... | ... |

方法：把地址存进 hash table（哈希表）（可用 2-way hash（二路哈希））；在 hash table（哈希表）中查找 exact match（精确匹配）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 9：查找地址：IP</span>

IP address（IP 地址）（在一台 router（路由器）中）：

| Match（匹配） | Action（动作） |
|---|---|
| IP DA = 127.43.57.99 | Forward to 56.99.32.16 |
| IP DA = 123.66.44.X | Forward to 22.45.21.126 |
| IP DA = 76.9.X.X | Forward to 56.99.32.16 |
| ... | ... |

（表中 X 是通配符，表示该字节可为任意值。）查找是 longest prefix match（最长前缀匹配），而不是 exact match（精确匹配）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 10：最长前缀匹配</span>

prefix（前缀）集合：65.0.0.0/8、128.9.176.0/24、128.9.16.0/21、128.9.172.0/21、128.9.0.0/16、142.12.0.0/19（斜线后的数字 N 表示取前 N 位作为 prefix（前缀），其余位任意）。例：destination address（目的地址）65.14.24.120 命中 /8 前缀；destination address（目的地址）128.9.16.14 同时命中 128.9.0.0/16 与 128.9.16.0/21，选更长的那条。

<span style="color:#00838f;">路由查找：在所有匹配目的地址的 prefix（前缀）中，找出 longest matching prefix（最长匹配前缀）（即最具体的 route（路由））。</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 11：最长前缀匹配查找——二进制 trie</span>

用 binary trie（二进制 trie，一种用字符串共享前缀来组织键的树，中文常称字典树或前缀树）做 longest prefix match（最长前缀匹配）。entry（表项）与 prefix（前缀）：a=00001、b=00010、c=00011、d=001、e=0101、f=011、g=100、h=1010、i=1100、j=11110000；以 0/1 为边的 binary tree（二叉树）组织这些 prefix（前缀），标记节点即为 entry（表项）（图示推断）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 12：最长前缀匹配查找——TCAM</span>

用 TCAM（Ternary Content Addressable Memory，三态内容寻址存储器）做 longest prefix match（最长前缀匹配）。每个 entry（表项）存"binary value（二进制值）+ mask（掩码）"，如 a=00001XXX（mask（掩码）11111000）、d=001XXXXX（mask（掩码）11100000）……j=11110000（mask（掩码）11111111）。（mask（掩码）中为 1 的位要求匹配、为 0 的位不关心，所以 00001XXX 配 11111000 表示"只匹配前 5 位"。）

<span style="color:#00838f;">路由查找：把地址与每一个带 mask（掩码）的 entry（表项）同时比较。</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 同一个问题（查表），交换机与路由器走向了两条完全不同的技术路线，这页对照着看最精彩。以太网查找是 exact match（精确匹配）→ hash table（哈希表）$O(1)$，是你在 CLRS（《算法导论》）里学过的标准 hashing（散列）；IP 查找是"在所有匹配里挑最长前缀" → 天然是树问题：binary trie（二进制 trie）把地址当 32 位路径走一遍，一步一个分支，32 步内必达 leaf node（叶节点）——6.042J（MIT 的《计算机科学中的数学》课程）里"prefix code（前缀码）/ binary tree（二叉树）"的直接应用。但每步一次 memory access（内存访问）太慢（backbone link（骨干链路）每纳秒就到几个 packet（分组）），于是硬件路线登场：TCAM 把"值 + 掩码"写进每个 memory cell（存储单元），让所有 entry（表项）在单个 clock cycle（时钟周期）内并行比对，硬件自动把最长匹配排到优先输出——用 circuit area（电路面积）换时间，比软件树快几个数量级，也贵几个数量级。这个"trie（字典树）算法 vs TCAM 硬件"的权衡是 computer architecture（计算机体系结构）课的经典案例，CSAPP（《深入理解计算机系统》）里"memory hierarchy（存储器层次）用 SRAM（静态随机存取存储器）换速度"是它的近亲。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 13：查找地址：通用化</span>

generalization（通用化）/ abstraction（抽象化）的查找：**\<Match, Action\>**（匹配，动作）对。

| Match（匹配） | Action（动作） |
|---|---|
| IP DA = X | Forward to port 7 |
| Eth DA = Y AND IP DA = Z | Drop packet |

这是对 switch（交换机）、router（路由器）、firewall（防火墙）等设备中"lookup（查找）+ forwarding action（转发动作）"的推广。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 把"查表"抽象成 \<Match, Action\> 对，看似只是措辞升级，实际上是整个 SDN（Software-Defined Networking，软件定义网络）运动的理论起点：既然所有网络设备（switch（交换机）、router（路由器）、firewall（防火墙）、load balancer（负载均衡器））本质都在执行"匹配若干 header field（首部字段）、执行某个动作"，那么查找规则本身就可以从设备的硬件里抽出来，变成由 central controller（中央控制器）统一下发、可编程的 flow table（流表）。这一抽象在 2008 年演化为 OpenFlow 协议（一种开放流表协议，定义控制器如何把 flow table（流表）下发给交换机），后来又催生了 P4 这类可编程数据平面语言——McKeown 正是这两项工作的核心作者，讲义里这页就是他留给后文的伏笔。用你熟悉的语言说：这是把"lookup logic（查找逻辑）"从 firmware（固件）里解耦出来做成了"policy-as-data（策略即数据）"，和 database（数据库）把 query optimizer（查询优化器）与 storage engine（存储引擎）分离、operating system（操作系统）把 system call table（系统调用表）做成 table-driven（表驱动），是同一种架构智慧。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 14：总结</span>

packet switch（分组交换机）执行两种基本操作：

- 在 forwarding table（转发表）中查找 address（地址）；
- switching（交换）到正确的 egress link（出端口）。

在高层面上，Ethernet switch（以太网交换机）与 Internet router（互联网路由器）执行相似的操作。但 address lookup（地址查找）在交换机与路由器中非常不同。

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 3-7《Packet Switching - Practice: Switching and Forwarding (1)》的配套 slides（幻灯片）；"交换到正确端口"的内部机制（output（输出）/ input（输入）queuing（排队））见 ps_switching_2。</span>
