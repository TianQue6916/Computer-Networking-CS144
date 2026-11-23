> <span style="color:#7f8c8d;">Source: 1-8-byte-order-notes.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#c0392b;">Memory, Byte Order, and Packet Formats（内存、字节序与分组格式）</span>

### <span style="color:#c0392b;">Slide 1 · Memory, Byte Order, and Packet Formats（内存、字节序与分组格式）</span>

<span style="color:#2471a3;">**[section]**</span>

通信双方要交流，就必须对彼此交换的消息达成一致。如果一方以为消息是西班牙语，另一方以为它们是柬埔寨语，双方就无法沟通。对计算机通信而言，这意味着要就以下几点达成一致：消息有哪些字段（field）、它们如何排列和格式化、以及它们如何表示。要生成一个待发送的消息，软件通常要先在内存（memory）里创建它的一份副本，再交给网卡（networking card）；类似地，当计算机收到一个消息时，网卡会把消息放进内存，软件随后才能访问。如果你想理解网络协议并编写网络协议软件，弄清这套机制以及其中可能踩到的坑就非常重要。

### <span style="color:#c0392b;">Slide 2 · Computer Memory（计算机内存）</span>

<span style="color:#2471a3;">**[section]**</span>

图示：一个简化的内存模型——地址空间从 $0$ 开始、最大 $0\text{x}0200000000$（共 8 GB），每个地址单元存放一个字节（one byte，8 bits），例如地址 $0\text{x}5\text{e}1$、$0\text{x}5\text{ff}$ 处各有一个字节。

先从一个简单的计算机内存模型说起。如今大多数计算机的内存按**字节（byte）**组织——即 8 位的内存块。每个程序有一个地址空间（address space），从地址零开始。如今大多数计算机是 64 位的：这意味着内存地址是 64 位长，所以一台计算机最多可以有 $2^{64}$ 字节，也就是 18 sextillion 字节。实践中今天的计算机并没有这么多内存：它们是吉字节（gigabyte）级别的，即 $2^{30}$ 字节。在这个例子里，我们的计算机有 8 GB 内存，所以它的最大地址就是图中所示的十六进制值。软件可以访问内存的每一个字节，也可以成组访问，比如用一条指令从内存中 8 个连续的字节单元里加载一个 64 位整数。

### <span style="color:#c0392b;">Slide 3 · Endianness（字节序/端序）</span>

<span style="color:#2471a3;">**[section]**</span>

例：$1024 = 0\text{x}0400$，两个字节 $0\text{x}00$ 与 $0\text{x}04$，在内存中谁排在前面？

- 多字节字（multibyte word）：这些字节怎么排？
- **小端（little endian）**：最低有效字节（least significant byte）放在最低地址
  - 从寻址/计算的角度看最合理
- **大端（big endian）**：最高有效字节（most significant byte）放在最低地址
  - 对人类读者最直观

但计算机如何表示一个多字节的值呢？假设要表示数字 1024，十六进制是 $0\text{x}0400$，即 4 乘 256。这个值需要 16 位，即两个字节。哪个字节在前：$0\text{x}00$ 还是 $0\text{x}04$？多字节值在内存中的排布方式称为**字节序（endianness）**，有两种选择。小端（little endian）把最低有效字节放在最低地址，也就是说最低有效字节在内存中最先出现。事实证明，从计算和体系结构的角度看，这种安排往往最合理。另一种是大端（big endian）：最高有效字节放在最低地址。大端对人类读者更友好，因为我们写数字就是先写最高位。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这一段你在 CSAPP ch2 是亲手做过的：用 `show_bytes` 观察变量在 x86-64 机器上的字节布局，会发现 $0\text{x}0400$ 确实存成 `00 04`——这就是小端。当时可能只是把它当"要背的事实"，而这里给了它一个通信视角的解释：字节序不是对错问题，而是**约定问题**——单机内部随便怎么排，但只要数据要跨机器传输，双方就必须约定同一种排法。顺带修正一个常见误解：讲义中"ARM 都是大端"的表述是简化说法，实际上现代 ARM 是双端（bi-endian）处理器，可由系统配置切换，而手机、树莓派等绝大多数实际部署都运行在小端模式；真正"纯大端"的现役主流处理器主要是 IBM 大型机的 POWER 系与部分网络专用芯片——有趣的是，早期网络设备选大端，正是"网络字节序"约定诞生的历史土壤。小端的设计动机也值得回味：把低字节放低地址，意味着同一个数按 8/16/32/64 位不同宽度取地址时，起始字节都是它的最低字节，这对变长运算的硬件更友好。

### <span style="color:#c0392b;">Slide 4 · Quiz（随堂测验）</span>

<span style="color:#2471a3;">**[exercise]**</span>（习题保持英文原文）

Quiz
For each number, mark whether the hexidecimal representation is big endian or little endian. Don't use a calculator or other tool!

| Width | Decimal | Bytes | Big Endian | Little Endian |
|---|---|---|---|---|
| 16 bits | 53 | 0x3500 |  |  |
| 16 bits | 4116 | 0x1014 |  |  |
| 32 bits | 5 | 0x00000005 |  |  |
| 32 bits | 83,886,080 | 0x00000005 |  |  |
| 32 bits | 305,414,945 | 0x21433412 |  |  |

<span style="color:#2471a3;">**[solution]**</span>

> 下面逐行讲解答案。
>
> - 53 是**小端**。$53 = 3 \times 16 + 5$，而 $0\text{x}35$ 出现在第一个字节（最低地址）。
> - 4116 是**大端**。$4116 = 4096 + 20$，所以两个字节是 $0\text{x}10$ 和 $0\text{x}14$，其中 $0\text{x}10$ 对应更高的有效位（4096 那部分）。既然十六进制写作 $0\text{x}1014$，说明最高有效字节排在了最前——大端。
> - 5 是**大端**——最低有效字节排在最后、地址最高。
> - 83,886,080 是**小端**：它等于 $5 \times 2^{24}$，也就是说 $0\text{x}05$ 才是最高有效字节，却被放在了最前（最低地址）。
> - 最后，305,414,945 是**小端**。这题不用逐位计算，只看最低位即可：最低有效位要么属于 $0\text{x}21$，要么属于 $0\text{x}12$。若是 $0\text{x}21$，最低位是 1，数为奇数；若是 $0\text{x}12$，最低位是 0，数为偶数。305,414,945 是奇数，所以 $0\text{x}21$ 是最低有效字节——小端存储。

### <span style="color:#c0392b;">Slide 5 · Network Byte Order（网络字节序）</span>

<span style="color:#2471a3;">**[section]**</span>

- 不同处理器使用不同的字节序
  - 小端：x86；大端：ARM
- 要互操作，就必须约定多字节字段的表示方式
- <span style="color:#00838f;">**网络字节序（network byte order）采用大端**</span>

例：$1024 = 0\text{x}400$，按网络字节序应存为 $0\text{x}40$ 在前、$0\text{x}00$ 在后。

```c
uint16_t val = 0x400;
uint8_t* ptr = (uint8_t*)&val;
if (ptr[0] == 0x40) { printf("big endian\n")
} else if (ptr[1] == 0x40) {
  printf("little endian\n");
} else {
  printf("unknown endianness!\n");
}
```

那么这为什么重要？两台计算机要通信，就必须就"数字用大端还是小端表示"达成一致。而麻烦在于不同处理器的字节序不同。例如，Intel 和 AMD 的 x86 处理器是小端：最低有效字节在前；而 ARM 处理器（比如 iPhone 里的那种）则是大端：最高有效字节在前。

我们不希望两台通信的计算机还要互相探测、关心对方是大端还是小端。所以协议制定机构通常会选定一种并坚持使用。对互联网来说，这个选择就是大端。<span style="color:#00838f;">所有作为互联网规范（Internet specification）的协议都使用大端格式。</span>

上面是一段 C 代码，能告诉你你的计算机是大端还是小端。它取一个 16 位的值，把指向它的指针强制转换成字节指针，从而逐个查看字节。如果索引 0 处的字节是 $0\text{x}40$，说明最高有效字节在前——大端；如果索引 1 处是 $0\text{x}40$，则是小端；如果两者都不是，那就有奇怪的事情发生了。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这段检测代码的思路你在 CSAPP ch11 写 socket 代码时用过它的"产品版"：`htonl(INADDR_ANY)`、`htons(port)` 这类转换函数必须出现在 `bind()`、`connect()` 的参数里，本质动机完全相同——网络协议规定了线上字节排列，而你的 CPU 有自己的排列，两者相遇就需要翻译。注意这里的分层逻辑：**字节序约定本身也是一种协议**，它独立于"消息内容"而存在，这就是为什么它能被统一固定下来（互联网选择大端，即 network byte order），而不必像两端程序那样动态协商。一个帮助你记忆的模型：把"发送一个多字节数字"想象成"把一串字节写进字节流"——TCP/UDP 给你的本来就是无结构的字节流（CSAPP ch11 的 echo 程序里你已经体会过字节流没有"消息边界"），多字节数字必须由应用自己拆成字节、按约定顺序放入，`htons`/`ntohs` 只是替你完成"拆与拼"的标准件。

### <span style="color:#c0392b;">Slide 6 · Portable Code（可移植的代码）</span>

<span style="color:#2471a3;">**[section]**</span>

- 你必须把网络字节序的值转换成你的主机序（host order）
- 例如：分组里有一个按网络字节序存放的 16 位端口号，你用 struct 来访问它，想在 x86 处理器上判断它是不是 80 端口

```c
uint16_t http_port = 80; // Host order
if (packet->port == http_port) { ...
// Network vs. host order
```

按主机序，80 的两个字节是 $0\text{x}00$、$0\text{x}50$；而分组里按网络序存放的 80 是 $0\text{x}50$、$0\text{x}00$——直接比较，结果为 `!=`（不相等），测试失败。

- 辅助函数：`htons()`、`ntohs()`、`htonl()`、`ntohl()`
  - htons = "host to network short"，ntohl = "network to host long"
  - 需 `#include <arpa/inet.h>`

```c
uint16_t http_port = 80;            // Host order
uint16_t packet_port = ntohs(packet->port);
if (packet_port == http_port) { ...  // OK
```

且慢——这就带来了一个麻烦。你要求分组是大端格式，可如果你的处理器是小端怎么办？举个例子：你想把一个 TCP 段的端口号（port number）设为 80，即 HTTP 端口。一个直观做法是定义一个 C 结构体（struct），让其中的 `port` 字段正好落在正确的偏移上。但如果你用数值 80 与 `port` 字段比较，这个 80 在你的机器上是按小端存的——第一个字节是 $0\text{x}50$。而大端格式要求 $0\text{x}50$ 存在第二个字节。所以尽管段里的端口确实是 80，这个比较仍会失败。

为了让这件事容易些，C 的网络库提供了在主机序与网络序之间转换的工具函数。例如 `htons()` 接受一个主机序的 short（16 位）值作为参数，返回网络序的值。还有把网络序 short 转主机序的函数，以及针对 long（32 位值）的对应函数。所以判断分组端口是否为 80 的正确做法是：读取分组结构体的 `port` 字段，调用 `ntohs` 把它从网络序转成主机序，然后与 80 比较，就能得到正确结果。在小端体系结构上，`ntohs` 和 `htons` 会把两个字节顺序对调；在大端体系结构上，它们原样返回该值。

### <span style="color:#c0392b;">Slide 7 · Be careful whenever you handle network data!（处理网络数据时务必小心！）</span>

<span style="color:#2471a3;">**[section]**</span>

> 处理网络数据时务必小心！否则你会浪费许多（本可避免的）小时去调试代码——仅仅因为忘记转换，或者转换了两次。

这些函数给了你编写"与处理器体系结构无关"的网络代码的机制。但要小心！这一点怎么强调都不过分。处理网络数据时务必小心。如果你对"何时在主机序与网络序之间转换"没有原则、不够严谨，你就会给自己带来巨大的头痛：忘了转换、或者不小心转换了两次，于是你的协议行为突然不对，触发各种怪异的 bug。

### <span style="color:#c0392b;">Slide 8 · Packet Formats（分组格式：RFC 791 的 IPv4 首部图）</span>

<span style="color:#2471a3;">**[section]**</span>

图示：RFC 791 中"Example Internet Datagram Header（互联网数据报首部示例）"的 ASCII 排版图（32 位宽、5 行必需字段）：Version（版本）、IHL、Type of Service（服务类型）、Total Length（总长度）、Identification（标识）、Flags（标志）、Fragment Offset（分片偏移）、Time to Live（TTL，生存时间）、Protocol（协议）、Header Checksum（首部校验和）、Source Address（源地址）、Destination Address（目的地址）、Options（选项）、Padding（填充）。右侧配了一张更简洁的示意版，标注"32 bits (4 octets)"。

既然我们知道了互联网规范按网络序（大端）排布多字节值，就可以看看互联网规范是如何描述其分组格式的。出于历史原因，互联网规范是用纯 ASCII 文本写成的。左边的文字块逐字取自征求意见稿（Request for Comments, RFC）791——它定义了 IP 协议第 4 版，即 IPv4。图的顶部标出第 0 到 31 位——分组图按 4 字节宽绘制。由于 IPv4 有 5 行必需字段，这意味着 IPv4 首部至少有 20 字节长。McKeown 和 Levis 展示分组时常用一种更简单的视觉格式，就是右边那种。

拿它举例：IPv4 分组的总长度（Total Length）字段长 2 字节，即 16 位。这意味着 IPv4 分组不能超过 65,535 字节。该字段按大端存放。一个长度为 1400 字节的分组存为 $0\text{x}0578$，所以这个长度的 IP 分组的第 3 个字节（字节下标 2）是 $0\text{x}05$。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "RFC 用 ASCII 字符画报文图"这件事本身就值得一注：RFC 791 发布于 1981 年，那时没有图形界面，协议规范必须能在电传打字机和纯文本终端上阅读，于是形成了这套每行 32 位、用竖线分隔字段的"ASCII 艺术"，并沿用至今——你以后读 TCP（RFC 793）、UDP 的规范时会看到一模一样的排版传统。另一个值得记住的推导是"5 行必需字段 ⇒ 首部至少 20 字节"：首部长度不是拍脑袋定的，而是由 IHL（互联网首部长度）字段以 4 字节为单位显式声明的，IPv4 首部可变的正是"选项"部分。最后，$1400 = 0\text{x}0578$ 这个例子建议你手算一遍：$5 \times 256 + 7 \times 16 + 8 = 1400$。这种"十六进制 ↔ 十进制"的换算会在你做 CS144 Lab 抓包解析、乃至日后阅读真实报文时反复用到——网络工程师的日常之一就是盯着 hex dump 逐字节核对字段，正如本讲末尾 Wireshark 演示中底部那排字节。

### <span style="color:#c0392b;">Slide 9 · Packet Formats（分组格式：Wireshark 验证）</span>

<span style="color:#2471a3;">**[section]**</span>

图示：同上一页的 RFC 791 IPv4 首部 ASCII 图与简化示意图，用于对照 Wireshark 中的真实字节。

我们在 Wireshark 里看看。我打开 Wireshark 开始抓包。第一个分组属于一个叫 TLS（传输层安全协议，Transport Layer Security）的东西——网页浏览器用它建立安全连接（https）。TLS 把分组的数据内容对我们隐藏了，但我们仍然能看到它的首部。用 Wireshark 可以看到：TLS 的载荷装在一个目的端口（destination port）为 443 的 TCP 段里（443 是标准的 TLS 端口），这个 TCP 段又装在一个 IPv4 首部里。细看 IPv4 首部，可以看到分组的总长度（total length）字段是 1230。1230 的十六进制是 $0\text{x}04\text{ce}$：1024（即 $0\text{x}04$）乘 256，加 106（即 $0\text{x}\text{ce}$）。在底部，Wireshark 显示了分组的实际字节——看，就是 `04 ce`，大端，即网络序。

你已经看到不同处理器排布数字的方式不同。但网络协议必须达成一致，所以协议规范决定了数字的排布方式——它可能与你的处理器不同。为此，C 网络库提供了在主机序与网络序之间转换的辅助函数。但请谨慎使用！随意乱用会让你在调试上损失无数个小时——只要一开始就小心、并为你的代码定下有原则的转换方法，这些损失完全可以避免。
