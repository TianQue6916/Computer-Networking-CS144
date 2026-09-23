> <span style="color:#7f8c8d;">Source: src/慕课/Unit8-Security/8-4_denial-of-service.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> <span style="color:#c0392b">Denial of Service（拒绝服务攻击）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">开场案例：2000 年 2 月的 Yahoo! 事件</span>

2000 年 2 月，Yahoo! 的路由器不断崩溃——工程师此前就与这台设备斗争过，但这次格外严重。真相是：他们正被海量 ICMP echo reply（ICMP 回显应答）淹没。此后，针对众多高知名度网站的 DDoS（分布式拒绝服务）攻击接连发生。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">基本的 DoS 攻击（Basic Denial of Service Attack）</span>

- 用过多分组使一台服务器或一段网络过载（overload）；
- 让每个分组对服务器造成的代价（CPU 与内存）最大化。

**Distributed DoS（分布式拒绝服务，DDoS）尤其有效：**

- 以半自动方式渗透大量机器；
- 把这些 host（主机）变成"僵尸"（zombies），一声令下即可发起攻击；
- 随后同时对受害者发起大范围攻击。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "过载"可以翻译成排队论的语言。CS144 在性能分析一讲给过 Little's law（利特尔法则）$L = \lambda W$：系统中平均分组数 $L$ 等于到达率 $\lambda$ 乘以平均停留时间 $W$。服务器的服务速率 $\mu$ 有限，当攻击把到达率 $\lambda$ 推过 $\mu$，队列长度便随时间增长直到缓冲区溢出——丢包率飙升、queuing delay（排队时延）暴涨，合法用户被挤出系统。你正在学的 Bertsekas 概率论里，M/M/1 排队系统在利用率 $\rho = \lambda/\mu \ge 1$ 时平均队长发散到无穷，这正是"过载"两个字的严格含义。所以 DoS 的本质不是"破坏数据"，而是"挤占共享资源"：CPU、内存、带宽、连接表，每一个都是有限资源；在先到先得的服务策略下，攻击者只需要比合法用户更快地排队。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">DoS 攻击概览（DoS Attack Overview）</span>

- 这是一类只针对 availability（可用性）的攻击。
- DoS 的动机很多：
  - 勒索（extortion）——例如"付给我们一小笔钱，否则我们就让你的离岸在线赌博网站下线"；
  - 报复（revenge）——垃圾邮件发送者永久搞垮了反垃圾邮件公司 Blue Security；
  - 炫耀（bragging rights）。
- DoS 可以发生在许多不同层：Link（链路层）、Network（网络层）、Transport（传输层）、Application（应用层）……

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">热身：简单的 DoS 攻击（Warm up: Simple DoS Attacks）</span>

- 在物理层干扰（jam）无线网络——简单到用一部现成的无绳电话（cordless phone）可能就够。
- 利用 802.11 链路层的 NAV 结构：NAV（Net Allocation Vector，网络分配向量）用于提示"网络何时可能空闲"（例如 RTS/CTS 交换之后）；攻击者反复用它把网络预留（reserve）到最大秒数，让其他节点始终没有发送机会。
- Flooding attack（洪水攻击）——例如 flood ping：`ping -f victim.com`，用 ICMP echo request（回显请求）淹没受害者。
- Amplification（放大效应）可以让攻击的威力超过攻击者自己实际拥有的资源。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">EDNS 攻击（EDNS Attack）</span>

- 某些 EDNS（RFC 2671）的 DNS 响应体积可达查询的数十倍；互联网上有约 500,000 台 open DNS resolver（开放 DNS 解析器）。
- 用 DNS 响应淹没受害者：
  - 发送伪造了源地址（看起来像受害者发出）的查询；
  - 攻击者每个查询只花 60 字节；
  - 查询发往许多不同的解析器；
  - 所有响应却都回到同一个受害者，每个 3,000 字节。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">SMURF 攻击（SMURF Attack）</span>

- ICMP echo 支持对 IP broadcast address（IP 广播地址）做 ping——本意是想了解网络里有哪些机器（它们会全部回复）。
- 对洪水攻击而言这是巨大的放大：
  - 入侵网络里的一台机器；
  - 伪造源地址为受害者 IP，去 ping 广播地址；
  - 网络里所有机器都会向受害者回复。
- 2000 年，这场攻击击垮了 Yahoo!、buy.com、Amazon。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 放大攻击的核心是一个成本不等式：攻防双方的"带宽账单"不对称。EDNS 反射里 60 字节进、3,000 字节出，放大系数约 50 倍；SMURF 更狠，广播让 $N$ 台机器同时回复，系数约 $N \times$（还要再乘上响应本身的大小）。这意味着一个只有百兆出口的攻击者，可以驱使数十倍流量砸向受害者——反射（reflection）还顺带隐藏了真实源地址，让过滤难以下手。应对思路正是 8-6 一讲强调的 keeping costs symmetric（保持攻防成本对称）：把"每字节攻击成本"抬高到高于"每字节防御成本"。工程上的现实解法包括：路由器不回应定向到广播地址的 ping（摧毁"放大器"名单）、DNS 解析器只对已知客户开放（消灭 open resolver）、运营商入口按 BCP 38 做源地址过滤（拒绝源地址不可能来自入接口的包）——把"伪造源地址"这个放大攻击的共同前提掐断。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">SYN 洪弹攻击（The SYN-bomb Attack）</span>

回忆 TCP 三次握手（three-way handshake）：C → S：SYN；S → C：SYN-ACK；C → S：ACK。

服务器端的实现方式：

- 服务器在一张表里插入连接状态（connection state）；
- 等待第 3 个包（一分钟超时）；
- 把每个新到的 ACK 包与已有连接比对。

问题：<span style="color:#00838f;">操作系统无法处理任意多的半开连接（partial connections）。</span>

攻击：用伪造的源地址发送 SYN 包——SYN-ACK 将发往虚空（永远等不到回应）；服务器的表被填满，停止接受新连接；每秒几百个分组就足以让大多数服务器彻底瘫痪。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这一页与你在 CSAPP 第 11 章写的网络程序直接对应：socket 编程里服务器依次调用 socket()、bind()、listen(fd, backlog)、accept()——listen 的第二个参数 backlog 约束的正是"已完成握手、等待 accept 取走"的连接队列；而内核里还有一张更靠前的半开连接表（收到 SYN、发出 SYN-ACK、还没等到 ACK 的连接），为每个半开连接维持定时器与窗口参数，槽位通常只有一两百个且按分钟级超时清理。SYN 洪弹正是用伪造 SYN 把这张表塞满，让合法握手永远进不来。工程上的经典补救是 SYN cookies（1996 年由 Daniel Bernstein 提出）：服务器不再为半开连接保存任何状态，而是把连接信息编码进 SYN-ACK 的初始序列号——等合法的第三个 ACK 回来再"解码"重建状态。这是"用无状态化对抗资源耗尽"的漂亮范例：攻击者消耗的存储被转移成了密码学校验。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">野外的 SYN 洪弹（SYN-Bombs in the Wild）</span>

- **MS Blaster worm（冲击波蠕虫）**：用 SYN 包淹没 windowsupdate.com 的 80 端口；每秒 50 个 SYN 包（每个 40 字节）；随机化源 IP 地址的最后两个字节。
- 后果：客户端无法下载补丁来修复问题。
- 微软的解决方案：把下载地址改到 windowsupdate.microsoft.com，并经由 Akamai 为老客户端分发更新。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">其他攻击（Other Attacks）</span>

- **IP Fragment flooding（IP 分片洪水）**：内核必须为不完整的分组保留 IP 分片（fragments）；用伪造的分片淹没它——与 TCP SYN 洪弹同理，耗尽的是"等待中的状态"。
- **UDP echo 端口 7**：echo 服务对收到的所有包原样回复；伪造一个源端口为 7 的包发给两台主机，它们就会互相 echo，形成乒乓循环。这一实现缺陷已在大多数系统中修复。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">应用层 DoS（Application-level DoS）</span>

- **DNS 同时由 TCP 与 UDP 承载**：TCP 方式是 16 位长度字段后跟消息；许多实现会阻塞式地读取消息体——攻击者只写入长度字段、然后让 TCP 连接一直挂着，就能拖垮 DNS 服务器。
- **SSL 要求服务器做公钥解密**：打开大量连接即可耗尽服务器的 CPU 时间；而这对客户端来说相对廉价。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 历史注：MS Blaster 爆发于 2003 年 8 月，利用 Windows RPC/DCOM 缓冲区溢出漏洞（MS03-026）自动感染数十万台主机，随后对 windowsupdate.com 发动 SYN 洪水——页面上"50 SYN/s"只是每台受感染主机的速率，几十万台僵尸叠加起来就足以淹没补丁服务器；随机化源 IP 末两字节则让按源地址过滤的黑名单失效。微软的应对（换域名 + Akamai CDN 承接流量）是"用可扩展架构吸收攻击"的早期教科书案例，与 8-6 讲的 replication（横向扩展）防御一脉相承。另外注意应用层 DoS 的共性：攻防成本严重不对称——SSL 握手中服务器要做昂贵的 RSA 私钥运算（下一讲你会看到具体开销数字），客户端却几乎零成本；这正是后来各类"耗 CPU"型攻击（如 SSL exhaustion）的根源，再次呼应 keeping costs symmetric 的设计原则。
