# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Demo: DHCP Masquerade Attack（演示：DHCP 伪装攻击）

> <span style="color:#7f8c8d;">Source: 慕课/Unit8-Security/8-2b_DHCP_Masquerade_Attack.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1：标题页</span>

**Demo: DHCP Masquerade Attack（演示：DHCP 伪装攻击）**——Nick McKeown，斯坦福大学电气工程与计算机科学教授。

<span style="color:#7f8c8d;">（Slide 1/4）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 2：DHCP 与 DNS 伪装——实验拓扑</span>

本页是课堂演示的拓扑图（DHCP and DNS Masquerade，DHCP 与 DNS 伪装），各要素如下：

| 图中元素 | 身份 | 说明 |
|---|---|---|
| h1 / Alice | 受害主机 | 接在交换机上，准备通过 DHCP 自动配置网络 |
| Good DHCP Server | 正牌 DHCP（动态主机配置协议）服务器 | 提供合法的地址配置 |
| s1 Switch | 交换机 | 连接主机与服务器 |
| Router | 路由器 | Alice 的（本应正确的）默认网关 |
| Public Internet | 公共互联网 | Alice 真正要去的地方 |
| Good DNS Server | 正牌 DNS（域名系统）服务器 | 提供合法的域名解析 |
| Rogue DHCP / Rogue DNS / Rogue Web Server | 流氓 DHCP / DNS / Web 服务器 | 三台"evil"服务器，同属 Eve the Attacker（攻击者 Eve） |

<span style="color:#7f8c8d;">（Slide 2/4）</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 先把拓扑读清楚：Alice（主机 h1）接在交换机 s1 上，网络里既有"好"的 DHCP 服务器、好 DNS 服务器和路由器，也有 Eve 的三台流氓服务器。回忆第 5 单元学过的 DHCP 四步交互（Discover–Offer–Request–Ack）：客户端发的是广播 Discover，而服务器用 Offer 竞争应答，协议本身不认证服务器身份——谁先回一个像样的 Offer，客户端就按谁提供的参数配置自己的默认网关和 DNS。因此攻击者的取胜条件仅仅是"比真 DHCP 服务器先应答"，这在软件里只是个循环发包的小程序，完全不需要攻破任何一台机器。这就是把第 5 单元的"便利协议"放进对抗模型后的第一课：便利与信任是一体两面。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 图中三台流氓服务器是一条完整的攻击链，而不是三个孤立道具：流氓 DHCP 先在地址配置阶段把"默认网关"和"DNS 服务器"两个参数指给 Eve；此后 Alice 的每一次域名查询都会先经过流氓 DNS，被解析到流氓 Web 服务器；Alice 以为自己在和真网站说话，账号口令却交给了 Eve。这实际上是本单元开头"伪装（spoofing）"与"中间人（man-in-the-middle）"两种威胁的组合拳：DHCP 负责把流量引过来（链路层的重定向），DNS 与 Web 负责在不破解任何密码的情况下完成凭据窃取。对应的防御思路是把"谁来当 DHCP 服务器"变成交换机可验证的策略（如 DHCP snooping：只允许连接合法服务器的端口转发 Offer 报文），这也是 8-6 里"保护网络的小块"这一原则的具体化。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 3：演示画面</span>

本页为课堂演示（demo）的实际画面，slides 上无文字内容：演示中 Eve 的流氓 DHCP 服务器抢答，使 Alice 的主机被配置了指向攻击者的网关与 DNS，随后流量被引向流氓 Web 服务器。

<span style="color:#7f8c8d;">（Slide 3/4）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 4：结束页</span>

\<The End\>（本讲结束）

<span style="color:#7f8c8d;">（Slide 4/4）</span>

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 8-2b《DHCP Attack Demo》的配套 slides（讲者 Nick McKeown），原文标题 "A>ack" 为连字乱码（Attack）；页 3 为演示画面。所查视频字幕文件（113 - p112）仅含音乐轨、无讲解旁白，故本节不引用视频引语；演示细节以上下文推断为准，未编造画面内容。</span>
