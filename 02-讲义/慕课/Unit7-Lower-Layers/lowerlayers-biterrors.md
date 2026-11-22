# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Physical Layer: Bit Errors and Coding（物理层：比特错误与编码）

> <span style="color:#7f8c8d;">Source: 慕课/Unit7-Lower-Layers/lowerlayers-biterrors.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1：标题页</span>

本讲义共 14 页，主题为物理层（physical layer，物理层）中的比特错误（bit errors，比特错误）与编码（coding，编码）：从信噪比与香农极限出发，到星座图与误码率，再到编码增益与 802.11n 的速率表。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 2：信噪比（SNR）与香农极限</span>

- <span style="color:#00838f;">香农极限（Shannon limit）：$C = B\log_2(1 + S/N)$，其中 $B$ 为带宽（Bandwidth，带宽），$S$ 为信号强度（Signal strength，信号强度），$N$ 为噪声（Noise，噪声）。</span>
- 可能的数据速率受信噪比（SNR, signal-to-noise ratio，信噪比）的约束——这**只是一个极限**！我们并不必然知道如何达到它。带宽通常是固定的。
- 信号越强，意味着你能更快地传输数据：更短的符号（shorter symbols，更短符号）、每符号更多比特（更密集的星座图，denser constellation，更密星座）。

🎥 *CS144 教授在视频中解释: "编码是一种在数据中引入冗余的方法……你可以保护自己免受位错误的影响。所以记住香农极限——通信的容量由带宽决定，然后是二进制对数的底，一加上信号与噪声比"*（字幕来源：096 - p95 7-2 Physical and Link Principles - Bit Errors.zh.srt）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 3：Signal with Noise（带噪声的信号）</span>

纯图示 slide：展示一条叠加了噪声的理想信号波形——原波形上出现随机毛刺，接收端看到的是"信号 + 噪声"的混合体，文字极少，按标题说明图示内容。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 4：例：16-QAM（HSPDA 使用的星座图）</span>

纯图示 slide：I-Q 平面上的 $4\times4$ 星座（constellation，星座图），共 16 个点，横轴为 I（同相）、纵轴为 Q（正交）。每个点标注 4 位二进制码，例如 $(-3,3)\to 1011$、$(-1,3)\to 1001$、$(1,1)\to 0000$、$(3,-3)\to 0111$、$(-3,-3)\to 1111$ 等，即 16-QAM 星座（该星座用于 HSPDA，一种 3G 数据标准）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这张星座图本质上是一次"编码方案"层面的双射：16 个星座点恰好对应 $2^4=16$ 个 4 比特组合，你在 6.042J 里用计数原理数过的东西在这里直接变成了工程参数。为什么是 $4\times4$ 而不是 $8\times2$ 或 $2\times8$？因为 I、Q 两个维度上电平间距越均匀、越对称，抗噪性能越平衡——几何上就是让 16 个点在正方形里尽量分散，任意两点的最小欧氏距离决定判决错误的难度。点数每翻一倍（16-QAM → 64-QAM），每符号多带 1 比特，但相邻点距离缩短到原来的 $1/2$，同样的噪声功率下误码率急剧上升。这正是香农公式里 $\log_2(1+S/N)$ 的微观体现：信噪比决定了你能把多少个点"塞进"单位面积的星座而不至于无法区分。计数给你容量上限，几何告诉你为什么不能白拿。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 5：Low-Noise Reception（低噪声接收）</span>

纯图示 slide：与 Slide 4 相同的 16-QAM 星座，但每个理想星座点周围画出实测接收点的小簇——低噪声时点云紧贴理想位置，判决几乎不会出错。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 6：High-Noise Reception（高噪声接收）</span>

纯图示 slide：同样的 16-QAM 星座，但接收点云明显散开、相邻簇彼此重叠——接收机按"最近的星座点"判决时就会判错，比特错误由此产生。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 7：SNR/BER 曲线</span>

- 对给定的调制方案与信噪比，可以计算出期望的误码率（bit error rate，误码率）：这需要对噪声做一些数学假设；这是射频（RF, radio frequency，射频）通信理论的基石原理。
- 误码率可以变得任意低，但**永远不会到达零**！
- 实践中，数学推导表明：直接以"原始比特"的形式发送分组效率极低——期望的数据吞吐量远、远低于香农极限。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "误码率永远不为零"这句话可以用你概率论前 7 章的工具算得非常具体：把每个比特的翻转看成一次伯努利试验，一个 $L$ 比特分组全对的概率是 $(1-p)^L$，哪怕 $p=10^{-9}$，一个 1500 字节（约 12000 比特）的分组完好率也只有约 $0.999988$，即每秒吉比特的链路上错误会以可感知的频率出现。单看一个比特，$p$ 很小；但大数定律保证：只要链路上的比特流足够长，错误事件就**必然**按期望频率出现——这正是"可以任意低、不能为零"的数学含义。工程上的对策因此分两层：物理层用编码把 $p$ 再压低若干个数量级，链路层再用 CRC 检测 + 重传兜底。你在排队论里学的"小概率事件 × 海量试验次数 = 不可忽略的系统级事件"，在这里就是误码率与吞吐量之间的换算。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 8：Bits vs. Symbols（比特与符号，原始数据）</span>

图示 slide：分组数据 "Hello, ..."（十六进制 `0x4865 0x6c6c 0x6f2c`）展开为比特流：

`0100 1000 0110 0101 0110 1100 0110 1100 0110 1111 0010 1100`

（即字符 H、e、l、l、o、, 的 ASCII 编码。）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 9：Bits vs. Symbols（比特与符号，加冗余后）</span>

同样的比特流，加入冗余/纠错码（error correcting codes，纠错码）后变成物理层的符号流：图示上下两行对照，上行是链路层分组比特，下行是带冗余的符号序列——物理层真正发送的是后者。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 10：编码（Coding）</span>

- 在物理层添加**一点冗余**，可以大幅提高链路层吞吐量——理论与实践都是如此。
- <span style="color:#00838f;">编码增益（coding gain，编码增益）：链路层比特数与物理层比特数之比。</span>
  - 1/2 码：每个链路层比特用 2 个物理层比特承载；
  - 3/4 码：每 3 个链路层比特用 4 个物理层比特承载。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "花带宽买可靠性，反而提高了吞吐量"是个反直觉却可以算清的账。设物理层速率固定为 $R$，不用编码时链路层有效速率就是 $R$；但若误码率是 $p$，一个 $L$ 比特分组需要期望 $1/(1-p)^L \approx 1+Lp$ 次传输才能成功（几何分布的期望，你在概率论里已经推过），有效吞吐量被重传吃掉。改用 1/2 码后，名义速率只剩 $R/2$，但单次成功率大幅提高、重传次数锐减。哪边划算取决于 $p$：信道干净时裸发更快，信道嘈杂时编码更快——802.11 的自适应速率（下一讲的 MCS 表）本质上就是在做这个实时权衡。这与 CSAPP ch2 里讲奇偶校验是同一思想的两端：1 位奇偶位只能"发现"错误，成块的冗余码才能"纠正"错误；发现→重传是反馈式修复，纠正→继续发是前向修复，下一讲的 FEC 就是后者。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 11：例：802.15.4（QPSK）</span>

图示 slide：符号（Symbols，符号）与比特（Bits，比特）的映射表——`0000→11011001110000110101001000101110`、`0001→11101101100111000011010100100010`、`0010→00101110110110011100001101010010`、`0011→00100010111011011001110000110101`。每 **4 bits** 映射为 **16 chips（QPSK symbols）**（源文标注；图中每条序列实际长 32 chip，见脚注）。

> <span style="color:#7f8c8d;">脚注：按 IEEE 802.15.4（2.4 GHz 物理层）标准，每个 4 比特符号展开为 16 种 32-chip 伪随机序列之一；源文标注"16 chips"与图中序列长度（32）不一致，此处按源文保留并说明，疑为 slide 原文笔误或提取错位。</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 12：计算数据速率（Calculating Data Rate）</span>

```math
\text{bitrate} = \text{bits/symbol} \times \text{symbol rate} \times \text{coding rate}
```

例：802.15.4 的比特率为 250 kbps；码率（coding rate）：16 chips of 2 bits = 4 bits（源文表述）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 13：例：802.11n 的 MCS 表</span>

| MCS 索引 | 空间流（Spatial Streams） | 调制（Modulation） | 码率（Coding） | 20MHz / 800ns GI（Mbps） | 20MHz / 400ns GI | 40MHz / 800ns GI | 40MHz / 400ns GI |
|---|---|---|---|---|---|---|---|
| 0 | 1 | BPSK | 1/2 | 6.5 | 7.2 | 13.5 | 15.0 |
| 1 | 1 | QPSK | 1/2 | 13.0 | 14.4 | 27.0 | 30.0 |
| 2 | 1 | QPSK | 3/4 | 19.5 | 21.7 | 40.5 | 45.0 |
| 3 | 1 | 16-QAM | 1/2 | 26 | 28.9 | 54.0 | 60.0 |
| 4 | 1 | 16-QAM | 3/4 | 39 | 43.3 | 81.0 | 90.0 |
| 5 | 1 | 64-QAM | 2/3 | 52 | 57.8 | 108.0 | 120.0 |
| 6 | 1 | 64-QAM | 3/4 | 58.5 | 65.0 | 121.5 | 135.0 |
| 7 | 1 | 64-QAM | 5/6 | 65 | 72.2 | 135.0 | 150.0 |

表中三列自变量——调制阶数、码率、保护间隔（GI, guard interval，保护间隔）与信道带宽——共同决定数据速率，正好是 Slide 12 公式的查表化。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 14：Overview（小结）</span>

- 码片（chips，码片——物理层）对比比特（bits，比特——链路层）。
- 物理层必须应对噪声，噪声会造成码片错误（chip errors，码片错误）：
  - 更密集的调制提供更高的吞吐量；
  - 更稀疏的调制错误更少。
- 在链路层比特与物理层比特之间转换：
  - 1:1 映射几乎从不是最高效的数据表示；
  - 编码增益（L2/L1 比值）使链路能容忍一定的码片错误。
- 例：802.11n。

<span style="color:#7f8c8d;">（Slide 14/14）</span>

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 7-2《Physical and Link Principles - Bit Errors》的配套 slides；香农极限的推导与调制细节见同单元《Capacity and Modulation》，纠错码展开见《Forward Error Correction (FEC)》。源文由 pdftotext 提取，个别连接符与希腊字母丢失处已按上下文复原。</span>
