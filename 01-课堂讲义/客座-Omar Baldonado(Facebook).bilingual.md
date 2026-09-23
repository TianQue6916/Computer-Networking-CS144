# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Network Virtualization（网络虚拟化）—— Omar Baldonado（Facebook 网络基础设施部门，2019 年 11 月 22 日）

> <span style="color:#7f8c8d;">Source: 课堂讲义/客座-Omar Baldonado(Facebook).pdf · CS144 Fall 2019 · 行内双语翻译版</span>

> <span style="color:#7f8c8d;">说明：本文为客座讲座 slides。图示页中的 IP 地址、拓扑连线与表格按 pdftotext 残片复原；三个示例（NAT/VPN/VLAN）各配一张"报文进出设备时发生什么"的递进图（A→B→C→D 四类），在相应位置说明。</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">什么是"虚拟化"？</span>

What is "virtualization"?（什么是"虚拟化"？）

- 创造某种公共资源的虚拟版本（virtual version，虚拟版本）。你早已在三个领域见过它：
  - 虚拟内存（virtual memory，虚拟内存）——进程拥有自己的地址空间；
  - RAID 存储——进程以为自己在写一块磁盘，底下其实是多块；
  - 虚拟机（virtual machine，虚拟机）——操作系统不知道自己其实跑在另一个操作系统（而非硬件）之上。
- 本质上，<span style="color:#00838f;">虚拟化是一种共享公共资源的方式</span>（a way to share a common resource）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">走向"网络虚拟化"的漫漫长路</span>

Progress toward "network virtualization"（通往"网络虚拟化"的进展）：

- 多年来有很多不同的步骤/技术；
- 它们的共同点：<span style="color:#00838f;">做的事都多少偏离了"按分层定义的典型行为"</span>（doing something a little different from the typical layer-defined behavior）。

本次讲座用三个例子（NAT、VPN、VLAN）讲这条演化路径。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">例 1：网络地址转换（NAT）</span>

Ex 1: Network Address Translation (NAT)（例 1：网络地址转换）。

背景是一场 1980 年代末/1990 年代初的互联网大辩论——就发生在 Stanford！Steve Deering（1991 年博士毕业，IPv6 的发明人）当时指出："我们快用完 IPv4 地址空间了——我们需要 IPv6"，但"IPv6 的部署可能需要很长时间……"

于是，网络地址转换（NAT，Network Address Translation）诞生了——来自 RFC 1918（定义私有地址空间的规范文档）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">我家笔记本上的 ifconfig</span>

ifconfig on my laptop at home（我家笔记本上的 ifconfig 输出——注意内网接口的地址是 10.0.0.7，一个 RFC 1918 私有地址）：

```
ocb-mbp:~ ocb$ ifconfig lo0: flags=8049<UP,LOOPBACK,RUNNING,MULTICAST> mtu 16384
options=1203<RXCSUM,TXCSUM,TXSTATUS,SW_TIMESTAMP> inet 127.0.0.1 netmask 0xff000000 inet6 ::1 prefixlen 128 inet6 fe80::1%lo0 prefixlen 64 scopeid 0x1 nd6 options=201<PERFORMNUD,DAD>

....

en0:

flags=8863<UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST> ether 8c:85:90:95:15:4a inet6 fe80::14b2:9162:5553:8b72%en0 prefixlen 64 secured inet 10.0.0.7 netmask 0xffffff00 broadcast 10.0.0.255 inet6 2601:647:5a00:6510:c0f:3811:351b:5c4d prefixlen 64 nd6 options=201<PERFORMNUD,DAD> media: autoselect status: active

mtu 1500 scopeid 0x8 autoconf secured
```

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">家里私有，公网公开</span>

Private in home, public in Internet（家里是私有网络，互联网上是公有网络）。图示页：家里的三台设备 10.0.0.5、10.0.0.6、10.0.0.7 组成一个"私有 IP 网络"；发往互联网的分组（packets to Internet）经过 NAT 设备后，源地址被换成公网地址 73.92.1.7（public IP，公有 IP）；返回的分组（packets coming back home）再被换回内网地址。于是，在 Comcast（ISP）和互联网内部，它是一个"公有 IP 网络"。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">私有与公有之间的转换表</span>

Translation table between private and public（私有与公有之间的转换表）。图示页：NAT 设备里维护一张"地址与端口转换表"（address & port translation table），形如：

| 原源 IP | 原源端口 | 新源 IP | 新源端口 | 协议 | 目的 IP | 目的端口 |
|---|---|---|---|---|---|---|
| 10.0.0.5 | 53323 | 73.92.1.7 | 45584 | TCP | 157.240.22.35 | 80 |
| 10.0.0.5 | 43023 | 73.92.1.7 | 9489 | TCP | 157.240.22.174 | 80 |
| 10.0.0.7 | 35803 | 73.92.1.7 | 49348 | TCP | 69.171.250.54 | 80 |

三台内网设备共用一个公网 IP，靠替换**端口号**来区分彼此的连接。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> NAT 违反分层的地方非常具体：按分层模型，路由器只该看三层（IP）并递减 TTL，而 NAT 竟然要改写**四层**（TCP 端口号）——表里三台内网设备共享公网 IP 73.92.1.7，只能靠重写源端口区分谁是谁。用 CSAPP ch11 的视角看：你写 socket 客户端时内核自动分配的临时端口（ephemeral port），经过 NAT 后在服务器眼里变成了另一个端口号——连接五元组（源 IP、源端口、目的 IP、目的端口、协议）在半路被偷梁换柱，"对端地址"不再是真实地址。这就是 NAT 与端到端原则（end-to-end principle，端到端原则）冲突的根源：网络中间盒持有连接状态，P2P 打洞、IPSec 这类端到端安全机制都要为此绕路。回头看那场辩论很有意思：Deering 主张"换协议（IPv6）根治地址短缺"，结果现实选择了"NAT 止血"——工程上够用的补丁往往先于正确的方案落地，而且一用就是几十年。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">例 2：虚拟专用网（VPN）</span>

Ex 2: Virtual Private Network (VPN)（例 2：虚拟专用网），1990 年代中期。

使用场景：公司有诸多"分支机构"（branches，分支，如银行网点、销售办公室）想要通过互联网连接总部（HQ，headquarters）——但互联网是"不安全"的（the "insecure" Internet）。图示页：分支机构与总部之间画出若干条"隧道"（tunnels，隧道），中间穿过接入 ISP（Access ISP）与互联网。

（下一页补充第二个场景：员工从酒店等公共网络接入——Employee in hotel，经接入 ISP、穿过不安全的互联网，隧道仍连到总部。）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">"隧道"如何工作：封装</span>

How a "tunnel" works — encapsulation（"隧道"如何工作——封装）。图示页：酒店里的笔记本（装着 IPsec VPN 软件）→ 接入 ISP → 不安全的互联网 → 总部。

源端的 IPsec VPN 软件：

- 用"隧道 IP"创建一个新分组；
- 把加密后的原始 IP 分组**作为有效载荷**（payload，有效载荷）封装进新分组；
- 把它发送到目的 IP——隧道端点（tunnel endpoint，隧道端点）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">"隧道"如何工作：解封装</span>

How a "tunnel" works — de-encapsulation（"隧道"如何工作——解封装）。图示页：总部一侧是 VPN"网关"（gateway，网关），执行 IPsec "隧道"的终点职责：

- 收到封装分组；
- 解封装——剥掉外层 IP 首部（removes outer IP header）；
- 拆开有效载荷并解密；
- 送进总部内网。

（本页之后是"报文进出设备"递进图的第三张：A. "典型"分层行为 B. 转换 C. 隧道。）

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 隧道就是把你在第 1 周学的封装（encapsulation，封装）做了一次**递归**：正常封装是"TCP 段套进 IP 分组、IP 分组套进以太网帧"，而隧道是"整个 IP 分组再套进另一个 IP 分组"——内层网络被外层网络当作一条"链路"来用。分层的套娃结构允许这种自相似：CSAPP ch11 告诉你 `socket()` 收发的 payload 对内核只是不透明的字节，隧道软件正是利用这一点，把任意内层报文（哪怕是别的协议）塞进外层分组的载荷里。工程上要留意两个代价：一是每层封装都要加首部，有效 MTU 一层层变小，内层报文稍大就会触发分片（fragmentation，分片）；二是解封装必须在隧道端点做，端点就成了故障与攻击的集中点——所以 VPN 网关通常是公司安全架构里最要紧的设备。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">例 3：虚拟局域网（VLAN）</span>

Ex 3: Virtual LANs (VLANs)（例 3：虚拟局域网），1990 年代末/2000 年代初。

- "以太网上的流量如今多得很——只有电子邮件那会儿没这么糟……"回忆一下 CSMA/CD（载波侦听多路访问/冲突检测）那一课：一个大广播域里冲突与广播都成灾。
- 一个大 IP 子网（IP subnet，子网）里广播太多；可要是不用一个大子网，怎么让设备跨多台交换机仍连成一片？
- 办法：<span style="color:#00838f;">在首部里引入一个"标签"（tag，标签）来创建虚拟局域网</span>（layer 2，二层）——同一个物理以太网被切成多个逻辑网。图示：销售（Sales）、市场（Marketing）、工程（Engineering）三个部门各成一个 VLAN。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">优缺点</span>

Pros and cons（优点与缺点）：

- 优点：配置超级简单（不用操心子网、路由……）；很多人想要"二层数据中心"（L2 data centers，二层数据中心）。
- 缺点：标签只有 12 位——全网最多约 4K 个 VLAN（12 bits ~ 4K networks）。

（本页之后是"报文进出设备"递进图的第四张：A. "典型"分层行为 B. 转换 C. 隧道 D. 打标签（Tagging）。）

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "12 位不够用"是一个把 6.042J 计数原理直接变成工程约束的例子：12 位二进制串共 $2^{12}=4096$ 个，抛去保留值后可用的 VLAN 数不到 4K。2000 年代初一个企业用几十个 VLAN 绰绰有余，可云计算一来，一个"租户"就要好几个隔离网络，4K 顿时捉襟见肘。后来的 VXLAN 协议把标签扩到 24 位（$2^{24}\approx 1677$ 万），正是把"计数的指数增长"当作设计参数来对待。更值得注意的是三次迭代共同的演化模式：NAT 在三层/四层边界做改写、VPN 在 IP 层外套壳、VLAN 在二层加字段——每一次都是在"分层协议没有为此预留语义"的地方私自开辟旁路，各自为政、互不兼容。这个"工具箱里一堆互不相干的零散工具"的困境，正是下一页引出统一方案（overlay）的伏笔。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">（中期小结）教训</span>

Lessons (from mid 2000s)（2000 年代中期的教训）：

- 工具箱里是一堆互不相干的零散工具（disparate tools in a toolbox）；
- 难以实现彼此兼容的标准与技术；
- 难以构建"拥有数千端点、数十万条隧道"的网络。

引一句老游戏《巨洞冒险》的话：**"You are in a maze of little twisty passages, all different."**（你置身于一座满是曲折小径的迷宫，条条不同。）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">铺垫：几股大趋势</span>

Setting the stage - some trends（铺垫：几股大趋势）：

- 数据中心达到"大规模"（@scale）——即使在公司内部也要高效利用资源；
- 2000 年代中后期托管/云提供商兴起；
- 服务器虚拟化（VMware 等）——需要编址的 VM、容器数量增加了几个数量级；
- SDN（软件定义网络）——集中式的控制/管理软件。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">最先进的网络虚拟化</span>

State-of-the-art network virtualization（最先进的网络虚拟化）：

<span style="color:#00838f;">在共享的物理网络（"underlay"，底层网络）之上，允许构建完整的虚拟网络（"overlay"，覆盖网络）。</span>

可见于各大云（Amazon、Microsoft、Google、Alibaba、Oracle 等）以及 VMware、Citrix 等的企业级方案。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">网络虚拟化的基本需求</span>

Network virtualization - basic requirements（网络虚拟化的基本需求）：

- 多租户（multi-tenancy，多租户）：客户的 VM 只能连到**自己的** VM，别人的一律不行（隔离，isolation）；
- 虚拟编址与虚拟拓扑都要独立于物理位置/物理拓扑；
- 在大规模下运营：轻松地建立、扩展、运营、拆除一个 VM 网络。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">构件：主机上的虚拟交换机</span>

Building block: virtual switch on a host（构件：主机上的虚拟交换机）。图示页：一台服务器上跑三个 VM（VM1、VM2、VM3），它们都连到同一个虚拟交换机（virtual switch，虚拟交换机）——一个纯软件的交换机，VM 之间的流量根本不出这台物理服务器。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">物理"底层网络" + VM</span>

Physical "underlay" + VMs（物理"底层网络" + VM）。图示页：2 个机架、4 台服务器、2 个不同的"物理"子网（192.168.0.0/24 与 192.168.1.0/24），每台服务器 3 个 VM。服务器 A（192.168.0.2）与 B（192.168.0.3）在子网 .0.0；服务器 C（192.168.1.2）与 D（192.168.1.3）在子网 .1.0。底层网络对这些地址一无所知地转发。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">例 1：同一子网里的红色 VM</span>

Ex 1: Red VMs in same subnet（例 1：同一子网中的红色 VM）。图示页：三个"红色"VM 分别位于服务器 A（10.0.0.2）、B（10.0.0.3）、D（10.0.0.4）——同一个虚拟子网 10.0.0.0/24，物理位置却跨越两个物理子网。怎么把它们连起来？

- VLAN 不行——扩展不了（每个 VLAN 要贯穿全网的交换机，4K 上限也早爆了）；
- 物理交换机也记不住——交换机的内存/表项大小有限，不可能为成千上万个租户的虚拟地址各存一条表项。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">隧道与标签来救场！</span>

Tunnels & tags to rescue!（隧道与标签来救场！）图示页：主机上的虚拟交换机把发往远端 VM 的报文**封装**进隧道，跨过底层网络送到对端主机的虚拟交换机再解封装——物理子网完全不用改动。

（下一页：还记得这张图吗（Remember this picture?）？这看上去就是一台在做隧道（tunneling）的虚拟交换机！——呼应前面 VPN 的封装/解封装流程，只是执行者从 VPN 网关换成了虚拟交换机。）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">隧道与标签：分组里到底装了什么</span>

图示页：从 VM A（10.0.0.2）发往 VM D（10.0.0.4）的报文，离开服务器 A 时的双层结构：

- 外层（隧道首部）：外层源 IP 192.168.0.2，外层目的 IP 192.168.1.3，标签：Red；
- 内层（原始报文）：内层源 MAC = MAC(10.0.0.2)，内层目的 MAC = MAC(10.0.0.4)，内层源 IP 10.0.0.2，内层目的 IP 10.0.0.4。

标签（tag）告诉对端虚拟交换机：这个报文属于"红色"虚拟网络。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">红色 VM 连到一台"逻辑"交换机</span>

Red VMs connected to a "logical" switch（红色 VM 连到一台"逻辑"交换机）。图示页：从租户视角看，三个红色 VM 像是插在**同一台**交换机上；真实情况是它们各自的本机虚拟交换机经隧道两两互通，"一台逻辑交换机"是分布式软件合成的幻象。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">但还有一些问题……</span>

But some questions...（但还有一些问题……）图示页：VM 以为自己在普通 10.0.0.0/24 子网里"正常上网"，于是会问：

- 广播（broadcast，广播）怎么办？（VM 发 ARP 广播，谁能收到？）
- 路由器在哪？（VM 要跨子网时找谁做默认网关？）
- A、B、C、D 上的虚拟交换机如何互相协调？（谁来同步地址学习、隧道端点表？）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">例 2：不同子网里的绿色 VM</span>

Ex 2: Green VMs in different subnets（例 2：不同子网中的绿色 VM）。图示页：另一组"绿色"VM 分属两个虚拟子网——服务器 A、B 上的 10.0.0.x 与服务器 C、D 上的 10.0.1.x。同主机上可以同时存在红色与绿色 VM，互不干扰。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">例 2：逻辑上，绿色交换机 + 绿色路由器</span>

Ex 2: logically, green switches and green router（例 2：逻辑上，绿色的交换机和绿色的路由器）。图示页：从租户视角看，两个绿色子网之间由一台"绿色虚拟路由器"连接——它与绿色虚拟交换机一样，也是分布式软件在隧道之上合成的。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">你学过的一切，重新实现了一遍</span>

Everything you've learned so far, but reimplemented through virtual switches, tunnels, tags, ... as an "overlay" running as software services on the hosts!（你到目前为止学过的一切——只不过全部经由虚拟交换机、隧道、标签重新实现，作为一个跑在主机上的软件服务"覆盖网络"！）

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这页是整场讲座的点题：CS144 前半学期的所有机制——交换机的 MAC 学习、子网划分、ARP 广播、路由器的最长前缀匹配——被原封不动搬进了主机上的软件，跑在隧道和标签之上。这不是重复劳动，而是"分层"威力的终极展示：只要底层能送达字节（underlay 的 IP 网络），任何"网络"都可以在其上被软件定义出来。用 CSAPP ch11 的话说，这些虚拟交换机就是普通的用户态/内核态进程，收发的报文就是普通套接字里的字节——A、B、C、D 四台主机的虚拟交换机要协调"谁是网关、广播怎么扩散"，等价于在没有中心的情况下让四个进程达成一致，这和链路状态协议里路由器泛洪同步拓扑（最终各自跑 Dijkstra 得到一致视图）是同一类分布式同步问题。这也解释了为什么这套东西要等 SDN 成熟才落地：与其让几千台主机上的软件分布式地"猜"，不如由集中控制器把隧道端点表直接下发。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">还要提供更多特性……</span>

More features left to provide...（还有更多特性有待提供……）图示页：

- 怎么访问互联网？（虚拟子网里的 VM 要出公网，NAT/边界在哪里？）
- IPv4/IPv6 怎么处理？
- 如果有不带 VM 的专用裸机（"bare metal"，裸机，没有虚拟交换机）怎么办？
- ……

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">每朵云都面临相似的设计选择</span>

Every cloud has similar design choices（每朵云都有相似的设计选择）：

- 提供哪些特性？
- 用哪个虚拟交换机？
- 虚拟交换机之间如何协调？
- 用哪种隧道/标签技术？
- 这个东西怎么调试？
- 怎么高效地做到这一切？

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">业界实例一：VMware/Nicira NSX</span>

Ex: VMware/Nicira NSX（实例：VMware 的 NSX 网络虚拟化平台）。图示页：NSX 的逻辑架构——逻辑交换、逻辑路由、逻辑防火墙叠加在物理网络上。来源：VMware NSX Network Virtualization Fundamentals（VMware NSX 网络虚拟化基础指南），https://www.vmware.com/content/dam/digitalmarketing/vmware/en/pdf/products/nsx/vmware-network-virtualization-fundamentals-guide.pdf

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">业界实例二：Amazon Virtual Private Cloud (VPC)</span>

Ex: Amazon Virtual Private Cloud (VPC)（实例：Amazon 虚拟私有云）。图示页：VPC 让每个 AWS 租户在共享物理网络上获得一张自己的私有虚拟网络（自定义子网、路由表、网关）。来源：Amazon 的 Networking @Scale 2017 报告视频，https://engineering.fb.com/networking-traffic/networking-scale-2017-recap/

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">业界实例三：Facebook 的 ILA</span>

Ex: Facebook & Identifier Locator Addressing (ILA) containers + translation（实例：Facebook 的标识符-定位符编址（ILA），用于容器 + 地址转换——而不是 VM 与隧道）。Facebook 用"标识符/定位符分离 + 内联转换"替代隧道：报文不做完整封装，而在转发路径上直接改写地址。来源：Facebook 的 Networking @Scale 2017 报告视频，https://engineering.fb.com/networking-traffic/networking-scale-2017-recap/

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">提问 / Facebook 的网络工作</span>

Questions?（有问题吗？）Networking at Facebook（Facebook 的网络工作）：

- facebook.com/ipv6（Facebook 的 IPv6 之旅）
- connectivity.fb.com/（连接性项目）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">更多信息</span>

More info（更多信息）：

- engineering.fb.com/category/networking-traffic/（Facebook 工程博客·网络板块）
- research.fb.com/category/systems-and-networking/（Facebook 研究·系统与网络板块）
- connectivity.fb.com/

PhD student? Contact Nate Lee (natelee@fb)（博士生？联系 Nate Lee）。Thanks!（谢谢！）
