# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## Unit 7: Lower Layers（第 7 单元：底层）· 单元收尾总览

> <span style="color:#7f8c8d;">Source: src/慕课/Unit7-Lower-Layers/7-11_lower_layers_bookend.txt · CS144 Fall 2019 · 行内双语翻译版</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 1 · 标题页</span>

本页是 Unit 7 收尾课（bookend，收束性总结）的标题页：*An Introduction to Computer Networks*（计算机网络导论），Unit 7: Lower Layers（第 7 单元：底层），Stanford University。所谓 bookend，指与单元开篇相呼应的"合上书本"式总结课：把整个单元学过的链路层与物理层内容做一次盘点。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 2 · IP is the "narrow waist"（IP 是"细腰"）</span>

这是一张四层模型的 hourglass（沙漏）示意图，从上到下依次为 Application（应用层）、Transport（传输层）、Network（网络层）、Link（链路层）：

- 沙漏顶部（应用层与传输层）并列着众多协议：http、smtp、ssh、ftp …… 以及 TCP、UDP、RTP；
- 沙漏的细腰处只有一个协议：IP（Internet Protocol，网际协议）；
- 沙漏底部（链路层）并列着众多链路技术：Ethernet（以太网）、WiFi、DSL、3G ……

图示要点：<span style="color:#00838f;">上层应用再多样、下层链路技术再五花八门，彼此之间都只通过 IP 这一个"细腰"互通。</span>正是这个设计让 Internet 可以"上下两头独立演进"：发明新应用不需要改动网络，发明新链路技术也不需要改动应用——只要它能承载 packet（分组），就能接入 Internet。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 你在 CSAPP ch11 写过 socket（套接字）程序：`connect`/`bind` 的地址结构里装的就是"IP 地址 + 端口号"，也就是说，你编程时直接面对的抽象恰好骑在这条细腰的正上方。细腰的工程价值可以用一笔图论账算出来——上面 5 种应用协议、下面 4 种链路技术，若两两直接适配需要 5×4=20 种转换，而经由 IP 只需 5+4=9 种；这正是 6.042J 里完全二部图（边数 $m\cdot n$）对星型结构（边数 $m+n$）的对比，协议数一多差距会急剧拉大。CS144 前几个单元讲了细腰以上（传输、路由、拥塞控制），本单元向下钻：链路层与物理层——以太网、CSMA/CD、无线、编码与纠错，回答的都是"如何把一个 IP 分组安全地送过一段物理介质"。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 3 · What you learned（本单元学到什么·其一）</span>

本 slide 盘点链路层部分的学习成果：

- **Ethernet 和 CSMA/CD（载波侦听多路访问/冲突检测）**——多台主机如何共享同一条介质（medium），随机接入协议如何工作；
- **IP Fragmentation（IP 分片）**——当分组比下一段链路的 MTU（最大传输单元）还大时，IP 如何把大包切开、到终点再重组；
- **Wireless networks（无线网络）**——为什么无线与有线本质不同、信道如何共享，以及 WiFi 的机制。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 4 · What you learned（本单元学到什么·其二）</span>

本 slide 盘点物理层部分的学习成果：

- **Bit errors and coding（比特错误与编码）**——噪声如何让链路出错；
- **Error correcting codes（纠错码）**——如何用少量冗余把错误"算"回来；
- **Shannon Capacity（香农容量）**——一条有噪声信道在理论上最多能可靠传多快。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 把两张"学习清单"合起来看，Unit 7 的知识地图是一条自底向上的链：物理介质会出错（比特错误）→ 用冗余换可靠性（纠错码，Shannon 容量给出收益上界）→ 多台主机要抢同一条线（MAC 协议、CSMA/CD）→ 包太大过不去窄链路（IP 分片）→ 无线介质还要额外处理共享与衰减。这与 CSAPP 的组织方式异曲同工：你在 ch11 之前学的机器级表示与存储器层次，本质也是"信号如何变成可靠的字节"，只不过 CSAPP 讲芯片内部，本单元讲芯片之间。学完本单元再回头看 ch11 里"一条 `write()` 就把字节送出去"的 socket 抽象，你就能说清这行调用之下每一层各做了什么——抽象的好处与代价，只有钻到抽象底下看过一次才算真正理解。
