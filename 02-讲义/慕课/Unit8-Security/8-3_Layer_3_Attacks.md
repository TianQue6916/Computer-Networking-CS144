> <span style="color:#7f8c8d;">Source: src/慕课/Unit8-Security/8-3_Layer_3_Attacks.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> <span style="color:#c0392b">Layer 3 Attacks（第三层攻击）</span>

> <span style="color:#7f8c8d;">主讲：Nick McKeown，Professor of Electrical Engineering and Computer Science（电气工程与计算机科学教授），Stanford University</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 2 · 第三层攻击的常见类型（Common Types of Attack at Layer 3）</span>

1. **利用 ICMP 让源端主机重定向流量**：向源主机发送 ICMP redirect（ICMP 重定向）消息，诱导它把发往某前缀的流量改走别的路。
2. **BGP hijacking（BGP 劫持）**：
   - ISP 宣告（advertise）属于他人的 IP prefix（前缀），从而捕获他们的流量；
   - ISP 宣告一条无效的 ISP 路径，为流量制造 black hole（黑洞）；
   - 实施前提：伪装成某个 ISP（masquerading as ISP），或接管（take over）BGP 的 TCP 会话。
3. **More specific prefix（更精确前缀）**：插入一条更具体的前缀，把某个地址空间的一部分分流走；同样要求伪装成 ISP 或接管 BGP TCP 会话。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 3 · ICMP 重定向的正常工作方式（ICMP Redirect）</span>

图示页（按图推断）：Alice 所在局域网经 switch（交换机）连接 Router 1（默认路由器）接入 public Internet（公共互联网）；Router 1 背后还有 Router 2，通向 private intranet（专用内网）中的 Prefix A（前缀 A）与 Bob。

正常情形下，Router 1 发现发往 Prefix A 的流量经 Router 2 转发更近，于是向 Alice 发送 ICMP redirect 消息："Prefix A via IP(Router 2)"（前缀 A 经由路由器 2 的 IP 可达）。此后 Alice 的转发表变为 "IP Prefix: Prefix A → Next Hop: Router 2"（IP 前缀：前缀 A → 下一跳：路由器 2），流量直接经交换机发往 Router 2，不必再绕行 Router 1。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 4 · ICMP 重定向攻击（ICMP Redirect Attack）</span>

攻击者只需在同一个局域网里冒充路由器，向 Alice 发送伪造的 ICMP redirect："Prefix A via IP(Attacker)"（前缀 A 经由攻击者的 IP 可达）。

- 第 1 步：ICMP Redirect 使发往 Prefix A 的流量改经攻击者的 end host（终端主机）转发。
- 第 2 步：攻击者既可以阻断通信（prevent communication），也可以建立 man-in-the-middle（中间人）位置。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> ICMP redirect 由 RFC 792 于 1981 年定义，初衷是纯粹的效率优化：主机启动时往往只配了一个默认网关，路由器发现更优的下一跳时顺手告诉主机，省去绕路。问题出在信任模型上：协议没有任何认证机制，同一广播域内任何主机都能伪造"我是路由器"的重定向包。你在 CS144 的 IP 一讲见过 forwarding table（转发表）与 longest prefix match（最长前缀匹配）：重定向攻击的本质就是让受害者的转发表被别人改写——被劫持的是第 3 层的"控制流"，而不是某个数据包本身。第一性原理地看，这是 1980 年代"网络成员基本可信"这一假设的遗留物；现代操作系统已收紧策略（只接受来自当前第一跳路由器的重定向，甚至默认直接忽略），但要记住：协议设计时的信任假设会沿用到几十年后的敌对环境中。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 5 · BGP 的安全漏洞（BGP Attacks）</span>

BGP（边界网关协议）中存在三个安全漏洞（security vulnerabilities）：

1. 一个 AS（自治系统）可以宣告自己并不拥有的 IP 地址；
2. 一个 AS 无法验证收到的 AS path（AS 路径）是否正确；
3. ISP 之间通过一条普通的 TCP 会话交换 BGP 消息——没有加密，也没有认证。

结论：<span style="color:#00838f;">几乎任何一个 ISP 都能让整个互联网瘫痪——无论出于事故还是恶意。</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> BGP 诞生于 1989 年（RFC 1105），当时首要目标是把分属不同机构的网络连通，安全性完全不在需求清单上。用你在 6.042J 图论里的话说：互联网在 AS 层面是一张图，BGP 通告就是每个节点在"口述"自己见过哪些边、沿哪条边能到谁——而图论算法（比如 CLRS 里的 Dijkstra）默认图的边是给定的、可信的输入；BGP 打破了这个前提：图本身由参与者自由声明构建，任何节点都可以宣布一条不存在的边（幻影路径）或冒领别人的顶点（未拥有的前缀）。三条漏洞恰好对应"节点可以撒谎、路径无法验证、承载通道不设防"三个层面。后续的 S-BGP/RPKI 方案试图用密码学签名补上"谁拥有哪个前缀"的绑定，但部署至今仍不彻底——协议生态的社会惯性，有时比技术难度本身更难克服。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 6 · 真实案例（Some Examples）</span>

- **2008：Pakistan Telecom（巴基斯坦电信）**——本想封锁国内对 YouTube 的访问，却把伪造的 BGP 通告无意间传播到了全球，导致 YouTube 大面积不可达。
- **2004：DataOne（马来西亚）**——劫持了 Yahoo 位于 Santa Clara 的两个前缀；普遍认为这是恶意行为（目的是封锁 Yahoo）。
- **2003：垃圾邮件发送者劫持 Northrop Grumman**——劫持一段未使用的 IP 地址块，用来发送 spam（垃圾邮件）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 历史注（补 Slide 6 的细节）：2008 年 2 月 24 日，Pakistan Telecom 为执行国内的封网指令，把 YouTube 的前缀 208.65.153.0/24 宣告进了自己的域内——这本身只是 Slide 9 要讲的"更精确前缀"手法，因为 /24 比 YouTube 官方持有的 /22 更长、按最长前缀匹配必赢；问题在于这个错误通告被上游运营商 PCCW 原样输出到全球，几小时内世界各地的路由器都认为"去 YouTube 该走巴基斯坦"，YouTube 实际瘫痪约两小时。这次事故让 route leak（路由泄露）进入大众视野，也直接推动了 RPKI（资源公钥基础设施，用证书签名绑定前缀归属）的加速落地——你在证书一讲将学到的 chain of trust（信任链）思想，正是这类修复方案的原型。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 7 · 真实案例（续）（Some Examples）</span>

- **2004：土耳其 ISP —— TTNet**：TTNet 把完整的 BGP 路由表发了出去，宣称"经土耳其到任何地方都是最优路径"；结果几乎整个互联网的流量都改经土耳其路由，大部分互联网瘫痪了几个小时。
- **2008：巴西**：CTBC 发送的完整 BGP 路由表几乎劫持了其他运营商的流量；所幸一个 BGP monitor（BGP 监测系统）及时发现了异常。（原文注记认为其出于恶意。）
- ……还有更多案例！

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 8 · BGP 劫持的机制（BGP Hijacking）</span>

图示：Alice 在 AS1，Bob 在 AS3。正常路径上 AS3 通过 BGP 宣告："171.64/16 can be reached via the path {AS2, ..., AS3}"（171.64/16 可经由路径 {AS2, …, AS3} 到达）。恶意的 AS Evil 则宣告："171.64/16 can be reached via the path {AS Evil}"（171.64/16 可经由路径 {AS Evil} 到达），把发往 Bob 前缀的流量劫持到自己。

- 第 1 步：伪造的 BGP AS_Path 通告把发往 Bob 的流量重定向到攻击者；
- 第 2 步：攻击者可以阻断通信，或建立 man-in-the-middle（中间人）位置。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 9 · 用更精确前缀只劫持一部分流量（More Specific Prefix）</span>

同样的拓扑，攻击者只需宣告一条更长的前缀："171.64.74/24 can be reached via the path {AS2, AS Evil}"（171.64.74/24 可经由路径 {AS2, AS Evil} 到达）——利用 longest prefix match（最长前缀匹配）规则，这条 /24 路由会压过 Bob 正常宣告的 /16，于是只有发往 171.64.74/24 这一小段地址的流量被劫走：攻击面更小，也更难被察觉。2008 年 Pakistan Telecom 对 YouTube 的劫持（见 Slide 6 的案例注）用的正是这一手法。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 10 · 结束页（The End）</span>

本讲到此结束（原图仅为 "<The End>" 结束标记）。
