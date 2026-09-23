# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Dynamic Host Configuration Protocol（动态主机配置协议，DHCP）

> <span style="color:#7f8c8d;">Source: 慕课/Unit5-Applications-and-NATs/names-dhcp.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1：DHCP（Dynamic Host Configuration Protocol）</span>

本讲介绍动态主机配置协议（Dynamic Host Configuration Protocol，DHCP）：一台机器刚接入网络时，如何自动拿到通信所需的全部配置。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 2：用 IP 通信需要什么</span>

- 需要三样东西（外加一样）
  - IP 地址（IP address）
  - 子网掩码（subnet mask）
  - 网关路由器（gateway router）
  - DNS 服务器的 IP 地址也很有用（但对 IP 本身并非必需）
- 问题：这些值从哪里来？

要用 IP 通信，一台主机需要三样东西（外加一样）：自己的 IP 地址；子网掩码（subnet mask）；网关路由器（gateway router）的地址。此外，DNS 服务器的 IP 地址也很有用（不过对 IP 层本身并非必需）。问题来了：这些配置值从哪里来？

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 3：过去的日子</span>

- 系统管理员把它们发给你
  - "把这些值敲进你的网络配置控制面板……"
  - 1995 年我在布朗大学（Brown）就是这么干的（到我毕业时已经不用了）
- 许多问题
  - 机器移动了就不好使
  - 配置能用多久？
  - 垃圾回收（garbage collection）

在没有 DHCP 的年代，这些值由系统管理员手工分发："把这些值敲进你的网络配置控制面板……"（讲者回忆：1995 年在布朗大学就得这么干，到他毕业时已经不用了）。这种做法有一堆问题：机器一挪位置配置就失效；一份配置能用多久没人说得清；地址用完了或人走了谁来回收（garbage collection，垃圾回收）？

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 4：DHCP</span>

- 动态主机配置协议（DHCP），RFC 2131
  - 机器可以向 DHCP 服务器请求配置
  - 移动：再请求一次配置即可
  - 配置有期限：一个"租约"（lease），可以续约
  - 垃圾回收：租约到期时
- Discover、Offer、Request、ACK、Release

DHCP（RFC 2131）解决了这个问题：机器可以向 DHCP 服务器**请求**配置。机器移动了？再请求一次就行。配置有使用期限——称为租约（lease），到期前可以续约；租约到期自动回收，垃圾回收问题一并解决。协议的报文序列是：Discover（发现）、Offer（提供）、Request（请求）、ACK（确认）、Release（释放）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "租约"（lease）这个设计思想值得单独咀嚼，它是分布式系统里用**时间**换**状态**的经典手法：DHCP 服务器不必维护"哪些客户端还活着"的可靠状态（那需要心跳协议、失败检测器一堆机制），而是给每个配置一个过期时间——客户端要续命就得定期开口，沉默的客户端配置自动回收。这个模式你其实见过好几个亲戚：ARP 表项的老化定时器、DNS 记录的 TTL、TCP keepalive，乃至 6.824 风格分布式锁的租约——共同点是"把不可靠的显式释放问题转化为可靠的超时问题"。对比第一性原理：显式释放（Release）快但不可靠（客户端可能断电），超时回收慢但保证收敛（除非服务器自己挂了）；DHCP 两者都做，用 Release 优化快速回收、用租约兜底异常。另外注意"移动：再请求一次配置即可"——这句话是后面 Unit 8 的 DHCP 欺骗攻击（8-2b 节）的伏笔：既然谁广播谁就能拿配置，攻击者伪造一个 DHCP 服务器广播"网关是我"就能劫持整个局域网的流量。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 5：DHCP 时序</span>

（本 Slide 为时序图：Server A（服务器 A）与 Server B（服务器 B）分列两侧，Client（客户端）居中，展示 Discover 广播、A 与 B 各自 Offer、客户端选择其一发出 Request、被选服务器回 ACK 的交互过程。）

DHCP 的标准四步：客户端广播 DHCP Discover（此时它自己还没有 IP）；网络上的服务器 A、B 各自以 Offer 回应（附带可提供的地址与租约）；客户端选择其中一个（通常是先到的 Offer），广播 Request 声明"我要这个"；被选中的服务器回 ACK 确认，客户端即可使用该配置。租约到期前客户端可续约（Request/ACK 再来一轮），不用时发 Release 归还。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 6：还没有 IPv4 时怎么通信</span>

- UDP 端口 67（服务器）与 68（客户端）
- 广播 IP 地址：255.255.255.255
- 使用中继（relay）跨链路转发

鸡生蛋问题的最后一环：客户端还没有 IP 地址时，怎么跟 DHCP 服务器通信？答案：DHCP 使用 UDP 端口 67（服务器）与 68（客户端），客户端把报文发往受限广播地址 255.255.255.255（同时链路层广播），本链路上的 DHCP 服务器都能听到；若 DHCP 服务器不在本链路，则由中继代理（relay）把广播转成单播跨链路转发给远端服务器。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这页看似只有三个 bullet，其实浓缩了协议设计里最优雅的自举（bootstrap）技巧：用 IP 协议本身，构造出"不依赖任何已配置地址也能用 IP"的报文。发送端源地址填 0.0.0.0、目的地址填 255.255.255.255，这样的报文路由器默认不转发（受限广播），恰好把求助范围限定在本链路——而本链路上的 DHCP 服务器/中继正是唯一有资格回答的实体。对照 CSAPP ch11：你写的 echo 客户端先 `gethostbyname` 再 `connect`，全都建立在"机器已有配置"的前提上；而 DHCP 是那个让 `gethostbyname` 与 `connect` 得以运行的第 0 步——所有套接字（socket，套接字）通信的隐含地基。端口 67/68 的分配还体现了服务器/客户端的不对称：服务器端口固定（人人可寻址），客户端端口临时（由内核分配），这与五元组（源 IP、源端口、目的 IP、目的端口）的分用（demultiplexing）机制一脉相承——广播报文能被同链路多台服务器同时收到并各自回应，正是 UDP 无连接本质的直接利用。

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 5-11《DHCP》的配套 slides；DHCP 欺骗攻击（rogue DHCP server）在 Unit 8 的 8-2b 节完整演示；广播与 ARP 的细节见 Unit 1 的 1-11。</span>
