# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> TCP Header（TCP 首部）

> <span style="color:#7f8c8d;">Source: 慕课/Unit2-Transport/tcp-header.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1：TCP Header（TCP 首部）</span>

本讲义共 2 页，给出 TCP 首部（TCP header）的逐字段布局图。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 2：TCP Header 字段布局</span>

首部按 32 位（4 个八位组，octets）一行、共 5 行绘制，各字段如下（按图中从左到右、自上而下的顺序）：

| 字段（原文） | 中文 | 说明（基于图与上下文） |
|---|---|---|
| source port | 源端口 | 16 位，标识发送方应用 |
| destination port | 目的端口 | 16 位，标识接收方应用 |
| sequence number | 序列号（sequence number，序列号） | 本段第一个数据字节的编号 |
| acknowledgment number | 确认号 | 期望对方发送的下一个字节编号 |
| offset | 数据偏移 | 首部长度（以 4 字节为单位） |
| reserved | 保留位 | 未使用，置 0 |
| U A P R S F | 六个标志位 | URG、ACK、PSH、RST、SYN、FIN |
| window | 窗口 | 接收方通告的流量控制窗口 |
| checksum | 校验和 | 覆盖伪首部 + 首部 + 数据的差错检测 |
| urgent pointer | 紧急指针 | 配合 URG 标志使用 |
| options / padding | 选项 / 填充 | 可选扩展字段，padding 补齐到 32 位边界 |

图注：每个字段上方标注 "32 bits (4 octets)"，即首部按 4 字节对齐；没有选项时标准首部长度为 20 字节。

<span style="color:#7f8c8d;">（Slide 2/2）</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 前两个 16 位字段是源端口与目的端口，这是整个 TCP 首部里与 CSAPP 第 11 章联系最直接的部分：你在 socket 编程里调用 `bind()`/`connect()` 填进内核的端口号（port number，端口号），最终就写在这两个字段里。端口的意义是分用（demultiplexing）：IP 首部只把分组送到主机，主机再靠"（协议，源 IP，源端口，目的 IP，目的端口）"五元组把字节流分发到正确的套接字（socket，套接字）——CSAPP echo 服务器里 `accept()` 返回一个新的已连接描述符，本质就是内核为这个五元组新建了一条套接字表项。16 位意味着端口范围是 $0 \sim 65535$，其中 $0 \sim 1023$ 是需要 root 权限的知名端口（如 HTTP 的 80）。注意端口是"连接级"而非"主机级"概念：同一台服务器的 80 端口可以同时服务成千上万条连接，只要各连接的源端口互不相同。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 序列号与确认号是前面停等（stop-and-wait）与滑动窗口（sliding window）两节全部理论的落点。TCP 不给分组编号，而是给**字节**编号：序列号是"本段第一个数据字节"的编号，确认号是"期望对方发送的下一个字节"——这正呼应滑动窗口一节的提醒：收到 1、2、3、5 时 TCP 回 ack 4 而不是 ack 3。首部里的 window 字段就是流量控制的接收窗口：发送方的发送上限被压到 $LAR + \text{window}$。🎥 *CS144 教授在视频中解释: "窗口字段是流量控制窗口……如果你说看到二十万的窗口，这意味着在那个方向上，不能存在超过二十万的未确认字节"*（字幕来源：035 - p34 2-10 Reliable comm --- TCP header.zh.srt）。也就是说，将来拥塞控制里的拥塞窗口（congestion window，cwnd）与这里的接收窗口（receive window，rwnd）最终在发送方汇合成同一个不等式：在途字节数不超过二者的较小值。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 中间那排单字母 U A P R S F 是六个标志位（URG、ACK、PSH、RST、SYN、FIN），它们是连接管理一节 TCP 状态机的"燃料"：SYN/FIN 驱动连接建立与拆除，ACK 置位表示确认号字段有效，RST 用于异常终止——回看 2-6 节的 TCP 状态机图，每条状态转移边上标的就是这些标志的组合。checksum（校验和）字段则连接回 2-5 节的差错检测：TCP 的校验和不仅覆盖首部与数据，还覆盖一个从 IP 首部借来若干字段的**伪首部（pseudo-header）**，其目的是让 TCP 也能发现"分组被送错主机/端口"这类 IP 层差错。options 与 padding 解释了为什么需要 offset（数据偏移）字段：无选项时首部恰为 20 字节，有选项时按 4 字节对齐、靠 padding 凑整。用 20 字节的首部在不可靠的 IP 之上实现可靠字节流（reliable byte stream），正是端到端原则（end-to-end principle）代价核算的教科书案例。

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 2-10《Reliable comm --- TCP header》的配套 slides；各字段的行为细节分别在 2-1（服务模型）、2-8（窗口）、2-11（SYN/FIN/ACK）中展开。</span>
