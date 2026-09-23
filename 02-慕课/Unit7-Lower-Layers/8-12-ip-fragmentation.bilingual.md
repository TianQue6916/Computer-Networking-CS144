# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## Fragmentation and Assembly（分片与重组）

> <span style="color:#7f8c8d;">Source: src/慕课/Unit7-Lower-Layers/8-12-ip-fragmentation.txt · CS144 Fall 2019 · 行内双语翻译版</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 1 · 标题页</span>

本讲主题：Fragmentation and Assembly（分片与重组）——当上层数据单元对下层来说"太大"时，网络如何把它切开，并在目的地拼回原样。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 2 · Basic Problem（基本问题）</span>

这是一张两主机通信示意图：`myth.stanford.edu`（斯坦福的教学服务器）与 `www.google.com` 之间隔着多段不同类型的链路。

问题根源在于：不同链路允许的帧长不同——每段链路有自己的 MTU（Maximum Transmission Unit，最大传输单元）。一条从源到目的的路径可能同时经过 MTU=1500 字节的以太网和 MTU 只有几百字节的串行链路，于是"一次性装满一个大包"的策略会在窄链路上卡住。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 3 · Fragmentation and Assembly（分片与重组·定义）</span>

- **问题何时发生**：当 higher layer（上层）的数据单元对 lower layer（下层）来说太大时。
- <span style="color:#00838f;">**Fragmentation（分片）**：把一个大数据单元拆成若干较小的块（chunk）。</span>
- <span style="color:#00838f;">**Assembly（重组）**：把这些块重新合并成原来的数据单元。</span>
- 三个层次都有分片的例子：
  - **Transport（传输层）**：TCP 把字节流（stream of bytes）切成 TCP segment（报文段）；
  - **Network（网络层）**：IP 把对某条链路来说过大的 packet（分组）切成 IP fragment（IP 分片）；
  - **Link（链路层）**：6lowpan 在需要时把 IPv6 packet 切成 link fragment（链路层分片）——6lowpan 是面向低功耗网络（如传感器网络）的 IPv6 适配层。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 分片是一把"不得已才用"的钝刀：任何一片丢失，整个原包就作废。若原包切成 3 片、每片独立丢包率为 $p$，则整包成功率从 $1-p$ 掉到 $(1-p)^3$——丢包率被近似放大 3 倍，且片数越多放大越狠。正因为代价高，IP 把重组责任只放在最终目的主机、中间路由器只切不拼：路由器无须为任何一个经过的包保留状态，这与本课程讲过的端到端原则（end-to-end principle）和命运共享（fate sharing）一脉相承——状态放在通信两端，中间路由器崩溃重启也不会弄丢正在重组的数据。你在 CSAPP ch11 调 `write()` 发大块数据时，TCP 会按 MSS（maximum segment size，最大报文段大小，以太网上典型为 1460 字节）预先切分，让每个报文段恰好装进一个 MTU=1500 的帧，从源头避免走到 IP 分片这一步。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 4 · Fragmentation Example: IP（IP 分片示例·场景）</span>

图示场景：一个 1400 字节的 IP packet 要穿越三段链路——

1. Ethernet（以太网），MTU 1500 字节；
2. PPP（点对点协议链路），MTU 576 字节；
3. Ethernet，MTU 1500 字节。

1400 字节的包装得进第一段以太网的帧，但对 MTU 只有 576 字节的 PPP 链路来说太大了，必须在中间路由器上被切分。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 5 · Fragmentation Example: IP（示例·第一段以太网）</span>

第一段链路上的封装：原始 1400 字节的 IP 分组外面套一个以太网首部，构成帧 `ETH | IP 1400`——以太网帧内是完整的 IP 包，MTU 1500 足够容纳，无需分片。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 6 · Fragmentation Example: IP（示例·PPP 链路上切三片）</span>

到达 MTU 576 的 PPP 链路时，1400 字节的 IP 包被切成三片，每片套上 PPP 首部：

- `PPP | IP 512`
- `PPP | IP 512`
- `PPP | IP 376`

图中数字 512+512+376=1400 为示意画法；实际实现中每一片都要新增 20 字节的 IP 首部，且只有第一片携带原包上层的首部（如 TCP 首部）——分片本身会带来额外的首部开销。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 7 · Fragmentation Example: IP（示例·第二段以太网）</span>

三个分片各自作为独立的 IP 包穿过第二段以太网：`ETH | IP 512`、`ETH | IP 512`、`ETH | IP 376`。注意：这段以太网 MTU 1500 足够大，因此分片在此**不会被合并**——中间路由器从不重组，分片一直保持"片"的状态直到终点。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 8 · Fragmentation Example: IP（示例·目的地重组）</span>

在目的主机上，三个分片被重组成原来的 1400 字节 IP packet——图中箭头从三个 `ETH | IP 512 / 512 / 376` 指回一个完整的 1400 字节包。reassembly（重组）只发生在最终接收端。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 9 · IP Fragmentation（IP 分片的字段机制）</span>

**分片前（Before fragmentation）**：一个 1400 字节的 IP 包，首部中与分片相关的字段为 `ident=x`、`MF=0`、`offset=0`，其余为首部的常规字段。

- <span style="color:#00838f;">目的 IP 地址加 ident（identification，标识符）字段共同标识"属于同一个原包"的各分片。</span>
- <span style="color:#00838f;">MF（More Fragments，还有更多分片）位：除最后一个分片外均为 1。</span>
- <span style="color:#00838f;">offset（片偏移）字段指出该片在原包中的位置，以 8 字节为单位。</span>
- 因此，除最后一片外，所有分片的长度都必须是 8 字节的倍数。

**分片后（After fragmentation）**：三片分别为——

- 第一片 512 字节：`ident=x`、`MF=1`、`offset=0`；
- 第二片 512 字节：`ident=x`、`MF=1`、`offset=64`（即从第 $64\times8=512$ 字节处开始）；
- 第三片 376 字节：`ident=x`、`MF=0`、`offset=128`（即从第 $128\times8=1024$ 字节处开始）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> offset 字段只有 13 位：若以字节为单位，最多只能表示 8191 字节，装不下最长 65535 字节的 IP 包；改以 8 字节为单位后，表示范围变成 $8191\times8=65528$ 字节，恰好覆盖最大包长。这是一个典型的"用分辨率换表示范围"的定点编码权衡——与你在 CSAPP ch2 学的定点数完全同构：固定字长之下，把小数点右移一格，表示范围翻倍、精度减半。配套的约束"非末片长度必须是 8 的倍数"就是这种编码的必然代价：偏移格点对齐到 8 字节，片长也必须落在格点上，否则下一片的 offset 将无法表达。协议设计中这类"字长—粒度"取舍到处都是，看到奇怪的"必须是 N 的倍数"规则，先猜字段位数往往就能解释。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 10 · Fragmentation Example: TCP（TCP 如何避免分片）</span>

- **一般规则：能避免 IP 分片就避免。**
- TCP 可以主动选择 segment size（报文段大小）来避免分片；
- **DF（Don't Fragment，禁止分片）位**：置位后若路由器仍需分片，会丢弃包并回送 ICMP error（差错报文），发送方由此得知路径 MTU 太小；
  - 可以用 binary search（二分搜索）试探合适的包大小——但代价昂贵；
  - 更好的做法：直接尝试常见 MTU 尺寸（如以太网的 1500）；
  - 细节见 RFC 1191。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这页描述的就是 Path MTU Discovery（路径 MTU 发现，RFC 1191，1990）的原型：置上 DF 位发大包，收到 ICMP "fragmentation needed" 就把包改小重试。讲义说二分搜索 expensive，值得算一笔账：每次试探失败都要丢一个完整包并等一个 RTT（往返时延），探测 $k$ 次的代价就是 $k$ 个包加 $k$ 个 RTT；而"先猜常见尺寸"利用了真实路径 MTU 分布高度偏斜的先验——绝大多数路径就是 1500（以太网）或 1492/1480（PPPoE）这几档，一次命中。这与你熟悉的算法选择逻辑一致：CLRS 里二分搜索虽是 $O(\log n)$，但前提是"单次查询便宜"；当单次查询要付出丢包加 RTT 的代价时，带先验的启发式查表反而更优。现代 Linux 还会把探测到的路径 MTU 缓存在路由缓存里，让这次学习的成果被后续连接复用。
