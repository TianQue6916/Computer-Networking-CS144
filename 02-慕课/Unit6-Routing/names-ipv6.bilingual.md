> <span style="color:#7f8c8d;">Source: src/慕课/Unit6-Routing/names-ipv6.txt · CS144 Fall 2019 · 行内双语翻译版</span>

# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## Names and Addresses: IPv6（名称与地址：IPv6）

### <span style="color:#2471a3;">**[section]**</span> Slide 1 · 封面

Names and Addresses: IPv6（名称与地址：IPv6）。本讲讨论 IPv4 地址枯竭之后，互联网如何用 IPv6 重新回答"给每台主机一个什么样的地址"。

### <span style="color:#2471a3;">**[section]**</span> Slide 2 · Goal of Internet Protocol Addresses（IP 地址的目标）

- 把许多不同的网络拼接（stitch）在一起。
- 需要与网络无关的（network-independent）、唯一的（unique）地址。
  - 不过，如今它只能是"大体唯一"（mostly unique）——参见 NAT（网络地址转换）、anycast（任播）等机制。
- 但 IPv4 只有 $2^{32}$ 个地址。
  - 一般实际利用率约 35%。
- 需要一个地址才能通信……

### <span style="color:#2471a3;">**[section]**</span> Slide 3 · Goal of Internet Protocol Addresses（同页重复强调）

本页与上一页内容完全相同（幻灯片原样重复）。教授停留重讲，以强调核心矛盾：<span style="color:#00838f;">**互联网需要一个与网络无关的统一地址，而 IPv4 的 $2^{32}$ 个地址已经不够用了**</span>。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这页幻灯片里藏着 IPv4 最深的伤口。$2^{32}$ 约 43 亿个地址，扣除保留段后可用的只有约 37 亿；再乘上这里给出的 35% 利用率——很多地址已经分配却没有实际使用——真正在线可用的地址远少于理论值。而如今每人往往同时拥有手机、笔记本、平板等多个联网设备，43 亿明显不够。工程界的两个补丁正好对应你学过的知识：NAT（网络地址转换）让许多主机共享一个公网地址，CSAPP ch11 里你写过的 socket 程序若运行在内网 10.x.x.x 地址上，就是网关在替你改写地址与端口——代价是"唯一地址"退化为"大体唯一"，外部主机无法主动连入，端到端原则受损。anycast（任播）则让同一个地址在多地同时宣告，靠路由协议把用户自然引到最近的副本——它也没有给每台机器一个专属地址。IPv6 的 128 位地址（$2^{128}$ 个）正是回到"每个接口一个真全局地址"的初心。

### <span style="color:#2471a3;">**[section]**</span> Slide 4 · Internet Protocol, Version 6（互联网协议第 6 版）

- 1994 年开始相关工作。
- 基础协议于 1998 年发布（RFC 2460）。
- 一段沉寂之后，2003–2006 年兴趣回升。
- 如今 IETF（互联网工程任务组）内部正在大力推动部署。

### <span style="color:#2471a3;">**[section]**</span> Slide 5 · Address Structure（地址结构）

- IPv6 地址有 128 位。
  - $2^{128}$（约 $3.4 \times 10^{38}$）个地址：平均到地球每平方英寸表面约有 21 个地址！
- 地址按 RFC 4291 分为两段：subnet prefix（子网前缀，n 位）｜ interface ID（接口标识，128−n 位）。
- 书写方式：按 16 位一组、共 8 组的十六进制，用 `:` 分隔。
  - 示例（来自 market.scs.stanford.edu）：`2001:470:806d:1::9 prefixlen 64`
  - 可以用 `::` 省略**连续的一串**零（整串零只能省略一次）。
  - 在 URL 中要用方括号：`http://[2001:470:806d:1::9]:80`
  - 低 32 位可以按 IPv4 的风格书写：`64:ff9b::171.66.3.9`

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 把地址切成子网前缀加接口标识两段，本质上是一棵巨大的树。把 128 位地址逐位看成一次分支选择：走 0 进左子树、走 1 进右子树，整个地址空间就是一棵深度为 128 的二叉树；n 位前缀对应树中第 n 层的一个节点，它覆盖的叶子恰好是 $2^{128-n}$ 个接口。你在 6.042J 学过树的计数与结构，在 CSAPP ch11 里做过子网掩码的按位与运算，两边在这里合流：最长前缀匹配（LPM）就是在这棵树上找与目的地址共享最长前缀的分支，工程实现正是数据结构里的 Trie（前缀树）。还有一点值得注意：RFC 4291 并不规定 n 必须落在字节的整数倍上，前缀边界可以停在任意一位——这正是 IPv4 时代为延缓地址枯竭发明的 CIDR（无类别域间编址）精神，IPv6 完整继承了它。至于 URL 里的方括号，则是为了让浏览器能区分地址中的冒号和端口号前的冒号——历史兼容问题在语法上留下的疤痕。

### <span style="color:#2471a3;">**[section]**</span> Slide 6 · IPv6 Address Assignment（IPv6 地址分配）

- 分配规则经历了几轮迭代，随经验不断改进。
- RFC 3177：一般情形给 /48，有时给 /64，极少用 /128。
- RFC 6177（原文作 RFIC 6177，应为 RFC 6177）：至少给 /64，不要用 /128。
  - 分配多大由 RIR（区域互联网注册机构）自行决定（有点像今天 IPv4 的做法）。

### <span style="color:#2471a3;">**[section]**</span> Slide 7 · Example Approach（RFC 4291 示例方案：由 MAC 地址自动生成接口 ID）

- 可以用子网 /64 与以太网地址自动生成 IPv6 地址。
- 以太网设备有一个 48 位的唯一标识（layer 2 address，二层地址）。
  - 出厂时写死；如今通常想改也可以改。
  - 位布局：厂商码（c）与分配序号（m）；g 位为 0 表示单播 MAC 地址。

```
c c c c c c 0 g c c c c c c c c c c c c c c c c mmmmmmmm mmmmmmmm mmmmmmmm
```

- 把 48 位以太网地址转成 64 位接口 ID 的方法：把第 7 位（U/L 位）的 0 翻转为 1，并在中间插入 `fffe`：

```
c c c c c c 1 g c c c c c c c c c c c c c c c c 1 1 1 1 1 1 1 1 1 1 111 1 1 1 0 m m m m m m m m m m m m m m m m m m m m m m m m
```

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 位翻转与插入 fffe 这两个看似奇怪的动作都有明确的设计意图。MAC 地址第 7 位叫 U/L 位：0 表示出厂全球唯一地址、1 表示本地管理地址。自动生成的接口 ID 把这一位从 0 翻成 1，任何读到该地址的软件都能立刻判断"这不是出厂 MAC 原样拼进去的"；而中间插入的 16 位 fffe（二进制是十六个 1 跟一个 0，即上面位图中间那一段）则是一个标签，标记"这个 64 位 ID 来自 48 位 MAC 扩展"，避免与真正按 64 位规则分配的 ID 混淆。换句话说，RFC 4291 把"地址是怎么生成的"编码进了地址本身，让不同生成规则天然落入不同的数值区间。但这个机制也留下著名的隐私教训：MAC 地址随设备固定不变，意味着你带着笔记本走进任何网络，自动生成的 IPv6 地址都可用于跨网络追踪同一台设备，于是后来的 RFC 4941 引入了随机化、定期轮换的临时接口 ID。好的地址设计不仅要保证"唯一"，还要考虑地址与身份的关联——这是本页最值得带走的一课。
