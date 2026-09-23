# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Worked Example（例题精讲：单流 AIMD 的数值演算）

> <span style="color:#7f8c8d;">Source: 慕课/Unit4-Congestion-Control/4-3a_single_AIMD_flow_worked_example.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">题目场景（Nick McKeown，斯坦福大学电气工程与计算机科学教授）</span>

Alice 正从 San Francisco 的一台远程服务器上以 $10\ \text{Mb/s}$ 流式播放一段高清视频。所有分组（packet，分组）长 $250$ 字节。她测量到服务器的 ping 时间，测得的最小值为 $5\ \text{ms}$。一旦 AIMD 窗口达到稳态，在视频的剩余时间里，锯齿波（sawtooth，锯齿波）就在恒定的最小值与最大值之间振荡。缓冲区（buffer，缓冲区）被配置得恰到好处——刚好大到不会变空。

🎥 *CS144 教授在视频中解释: "一旦IMD窗口达到稳定状态，对于视频的其余部分，锯齿波在常数、最小和最大值之间振荡"*（字幕来源：056 - p55 4-3a Single AIMD flow worked example.zh.srt）

围绕这一场景共 6 问。

---

### <span style="color:#2471a3;">**[example]**</span> <span style="color:#c0392b;">问题 1：AIMD 窗口的最小值是多少（以字节计）？</span>

**解**：$5\ \text{ms}$ 的最小 ping 时间对应的正是"缓冲区为空、但瓶颈链路被填满"的时刻。以 $10\ \text{Mb/s}$ 计，此时"管道"里恰有 $50{,}000$ 比特。也就是说，AIMD 锯齿的最小值（谷底，trough）是 $50{,}000$ 比特，即 $6{,}250$ 字节。

> <span style="color:#7f8c8d;">**答**：6,250 字节。</span>

### <span style="color:#2471a3;">**[example]**</span> <span style="color:#c0392b;">问题 2：AIMD 窗口的最大值是多少（以字节计）？</span>

**解**：当缓冲区与瓶颈链路**都**装满时，$RTT$ 从 $5\ \text{ms}$ 翻倍为 $10\ \text{ms}$。按 $10\ \text{Mb/s}$ 计，这对应 $100{,}000$ 比特：其中 $50{,}000$ 比特在"管道"里（in flight，在途），$50{,}000$ 比特排在缓冲区里。因此 AIMD 锯齿的最大值（峰顶，peak）是 $100{,}000$ 比特，即 $12{,}500$ 字节。

> <span style="color:#7f8c8d;">**答**：12,500 字节。</span>

### <span style="color:#2471a3;">**[example]**</span> <span style="color:#c0392b;">问题 3：路由器中的分组缓冲区有多大（以字节计）？</span>

**解**：问题 2 已经判定：装满时缓冲区恰好容纳一个最小 $RTT$ 的量，即 $50{,}000$ 比特。

> <span style="color:#7f8c8d;">**答**：6,250 字节。</span>

### <span style="color:#2471a3;">**[example]**</span> <span style="color:#c0392b;">问题 4：一个分组被丢弃后，窗口要过多久才能重新到达最大值？</span>

**解**：分组长 $2{,}000$ 比特，所以窗口每轮（每个 RTT）增加 $2{,}000$ 比特。要补回 $50{,}000$ 比特、重新填满缓冲区，需要 $25$ 个 RTT。平均 $RTT$ 为 $7.5\ \text{ms}$，因此需要 $187.5\ \text{ms}$。

> <span style="color:#7f8c8d;">**答**：187.5 ms。</span>

### <span style="color:#2471a3;">**[example]**</span> <span style="color:#c0392b;">问题 5：若 Alice 改从澳大利亚的一台服务器观看同一视频（RTT 为 250 ms），在网络仍是 10 Mb/s 的前提下，路由器缓冲区应该多大（以字节计）？</span>

**解**：最小 $RTT$ 为 $250\ \text{ms}$，因此缓冲区至少需要 $250\ \text{ms} \times 10\ \text{Mb/s} = 2.5 \times 10^6$ 比特。由于存储习惯上按 2 的幂表达，答案是 $2.4\ \text{Mbit}$，即 $298\ \text{KB}$（Kbytes）。

> <span style="color:#7f8c8d;">**答**：298 Kbytes。</span>

### <span style="color:#2471a3;">**[example]**</span> <span style="color:#c0392b;">问题 6：服务器在澳大利亚时，一个分组被丢弃后，窗口要过多久才能重新到达最大值（以秒计）？</span>

**解**：与之前一样，窗口每轮增加 $2{,}000$ 比特。要填满缓冲区需要 $\dfrac{2.5 \times 10^6}{2 \times 10^3} = 1{,}250$ 轮。平均 $RTT$ 为 $375\ \text{ms}$，所以重新到达最大值的时间是 $468.75$ 秒——差不多 8 分钟！

> <span style="color:#7f8c8d;">**答**：468.75 秒。</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这 6 问串起来其实是一门"量纲思维"的微缩课。第 1、2 问的关键动作是给"管道"记账：谷底 = 带宽-时延积（BDP，bandwidth-delay product）$= 10\ \text{Mb/s} \times 5\ \text{ms} = 50{,}000$ 比特；峰顶 = 管道 + 缓冲区，而 $RTT$ 翻倍正好暴露了缓冲区的容量。做这类题时最容易犯的错是单位混乱——Mb/s 与 ms 相乘得到的是**比特**，再除以 8 才是字节；slide 原文把 "$250\ \text{bytes}$" 的分组在第 4 问里换算成 $2{,}000$ 比特也是同一件事。另一处细节：平均 RTT 取 $(5+10)/2 = 7.5\ \text{ms}$ 而非 5 或 10，因为爬坡过程里 RTT 从谷底值连续涨到峰顶值，取中值是合理的近似。这套记账法你在 CSAPP 里其实练过——_cache 容量、块大小、组数之间的换算是同一种"单位守恒"的推理。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 第 6 问的 468.75 秒（约 8 分钟）值得单独驻足：它就是前文"AIMD 亏待长 RTT 流"的定量化身。旧金山本地（RTT 5 ms）丢包后不到 0.2 秒就能满血复活，跨洋到悉尼（RTT 250 ms）却要 8 分钟——因为加性增的速度是"每 RTT 加 1 个报文段"，RTT 变大 50 倍，恢复时间也放大 50 倍。这就是高带宽-时延积网络的通病：卫星链路、洲际专线上的 TCP 启动慢、恢复慢，工程师为此发明了各种"窗口自动调优"（window scaling、初始窗口增大等）补丁。再往前一步：如果路由器缓冲区配得远超 BDP（早年厂商以为"大缓冲 = 好性能"），队列会长到把 RTT 推高几百毫秒，这个 modern 网络的顽疾叫 bufferbloat（缓冲区膨胀），它的解药之一（CoDel、FQ-CoDel 等主动队列管理）正是从"缓冲区该多大"这个本讲问题出发的。

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 4-3a《Single AIMD flow worked example》的配套 slides；源文本中 "boWleneck"（bottleneck）、"Aaer"（After）、"Introduc8on"（Introduction）等为 PDF 提取乱码，已按上下文还原。</span>
