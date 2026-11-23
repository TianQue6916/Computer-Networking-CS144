# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Network Security（网络安全）

> <span style="color:#7f8c8d;">Source: 慕课/Unit8-Security/8-1_Security.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1：标题页</span>

**Network Security（网络安全）**——Nick McKeown，斯坦福大学电气工程与计算机科学教授。

<span style="color:#7f8c8d;">（Slide 1/8）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 2：通信如何被破坏？</span>

通信（communication）可以被怎样破坏（compromised）？三条途径：

1. **窃听（Eavesdropping）**——被动地"嗅探"（sniff）并记录网络数据（或元数据，metadata）。例如：被动分接（tap）一条电缆或光缆；监听 WiFi（课堂上我们用 Wireshark 演示过）；攻陷（compromise）一台路由器，让它复制并转发数据。
2. **修改、删除、插入（Modify, delete, insert）**——主动篡改我们的数据：修改分组（packet）的内容；把分组重定向（redirect）到另一台服务器；接管一台终端主机（end-host）的控制权。
3. **阻止通信（Prevent communication）**——通常称为"拒绝服务"（denial of service）。

<span style="color:#7f8c8d;">（Slide 2/8）</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 注意第 1 类威胁里那句"课堂上我们用 Wireshark 演示过"——这正是 CS144 第一周当堂演示过的：同一间教室里任何人都能抓到别人的分组。从 CSAPP 第 11 章的视角看这毫不奇怪：你在套接字（socket，套接字）上调用 `read()` 读到的就是线上传输的原始字节，TCP 与 IP 首部本身不含任何加密，HTTP 的请求行、Cookie、表单字段都以明文出现在有效载荷（payload，有效载荷）里。所以"窃听"不需要攻破任何一台机器，只需要物理位置在同一个广播域内（WiFi）或在传输路径上（分接光缆、入侵路由器复制转发）。这解释了为什么机密性必须由端到端的加密来提供，而不是指望链路或路由器替你保密——这也是后面 8-11 TLS 一讲的出发点。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 3：例子——窃听</span>

图示：Alice 通过 HTTP 经公共互联网（Public Internet）访问 Amazon.com，攻击者（Attacker）在路径上窃听。

攻击者学到（learns）了 Alice 的信用卡号和家庭住址等。我们会在后面的视频里学习 HTTPS（h/tps 原文如此）如何阻止这一点。

<span style="color:#7f8c8d;">（Slide 3/8）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 4：例子——中间人</span>

图示：Alice 与 Amazon.com 之间走 HTTP，攻击者位于中间路径上。

攻击者修改（changes）收货地址，或阻止通信。

<span style="color:#7f8c8d;">（Slide 4/8）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 5：例子——路由重定向</span>

图示：Alice 经公共互联网与 Amazon.com 通信，攻击者在网络中实施路由重定向。

攻击者窃取（steals）支付或信用卡信息。什么也没发货（Nothing shipped）。

<span style="color:#7f8c8d;">（Slide 5/8）</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 第 5 页的"路由重定向"例子值得提前标注：它不是黑客坐在 Alice 和 Amazon 之间，而是操纵网络的"地图"——让去往 Amazon 的分组在 IP 路由层面就走错路，攻击者只要在路径上顺流截取即可。这条线索在本单元 8-3 会正式展开：ICMP 重定向、BGP 劫持、更精确前缀（more specific prefix），本质都是污染转发面的状态。你可以把它们与第 5 单元学过的机制对上号：IP 前缀与最长前缀匹配（longest prefix match，LPM）决定了分组去哪，BGP 的 AS-Path 决定了前缀如何通告——这些机制被设计时都默认"路由器会说实话"，而安全单元的全部出发点恰恰是撤掉这个假设。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 6：我们想要什么</span>

1. **机密性（Secrecy / Confidentiality）**：没人能偷听我们的通信。我们将学习加密（encryption）。
2. **完整性（Integrity）**：我们的消息在传输途中不被篡改。我们将学习消息认证码（message authentication codes，MACs）。
3. **可认证性（Authentication）**：确认对方的身份。我们将学习数字签名（digital signatures）与证书（certificates）。
4. **不间断的通信（Uninterrupted communication）**：我们不希望有人阻止我们通信。

<span style="color:#7f8c8d;">（Slide 6/8）</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这页把安全目标列成四条，前三条就是安全领域经典的 CIA 三元组：机密性（Confidentiality）、完整性（Integrity）、真实性（Authenticity），第四条对应可用性（Availability），合称 CIA。注意它们与攻击类型是严格对偶的：窃听破坏机密性，篡改破坏完整性，伪装破坏真实性，压制破坏可用性——本单元后面的结构正是按这个对偶展开的（8-3/8-4 讲攻击，8-6 起逐一讲原则与工具）。还有一个容易忽略的要点：机密性、完整性、真实性可以靠密码学"离线"地解决，而可用性做不到——密码学无法阻止别人向你灌垃圾流量，所以 DoS 至今仍主要靠体系结构手段（复制、上游过滤）来缓解，8-6 的高可用一页会回到这一点。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 7：攻击类型</span>

在接下来几讲中我们将研究不同类型的攻击：

1. 窃听（Eavesdropping）。
2. 重定向以太网、IP 与 DNS 流量（Redirecting Ethernet, IP and DNS traffic）。
3. 劫持一条正在运行的 TCP 连接（Hijacking a running TCP connection）。
4. 拒绝服务（Denial of service）。

<span style="color:#7f8c8d;">（Slide 7/8）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 8：结束页</span>

\<The End\>（本讲结束）

<span style="color:#7f8c8d;">（Slide 8/8）</span>

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 8-1《Introduction to Network Security》的配套 slides（讲者 Nick McKeown）；原文 PDF 存在连字乱码（如 "communica/on"、"AXacker"、"man-�-in-�-the-�-middle"），已按上下文复原为 communication、Attacker、man-in-the-middle，不另逐一标注。所查视频字幕文件（110 - p109）为错配的娱乐视频，故本节不引用视频引语。</span>
