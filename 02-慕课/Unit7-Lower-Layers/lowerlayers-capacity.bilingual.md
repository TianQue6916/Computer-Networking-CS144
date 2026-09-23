# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Physical Layer: Capacity and Modulation（物理层：容量与调制）

> <span style="color:#7f8c8d;">Source: 慕课/Unit7-Lower-Layers/lowerlayers-capacity.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1：标题页</span>

本讲义共 16 页，主题为物理层的容量（capacity，容量）与调制（modulation，调制）：先讲香农极限与模拟信号的基本量，再依次介绍 ASK、FSK、PSK、QAM 各族调制方式与 I/Q 星座表示。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 2：香农极限（Shannon Limit）</span>

- 信道（channel，信道）能承载多少信息，存在一个理论上的极限（Shannon limit，香农极限）。
- <span style="color:#00838f;">信道容量（Channel capacity，信道容量）$= B\log_2(1 + S/N)$：$B$ 为带宽（Bandwidth），$S$ 为信号强度（Signal strength），$N$ 为噪声（Noise）。</span>
- 想要更高的 $S/N$，需要更低的噪声（更好、更贵的硬件），或者更强的信号（更高的电压）。
- 构建非常高的带宽的硬件是困难的。

🎥 *CS144 教授在视频中解释: "结果发现，一个给定通道的信息传输有一个理论上的限制……这个限制被称为香农极限，在克劳德·香农之后——谁建立了它"*（字幕来源：095 - p94 7-1 Physical and Link Principles - Shannon Capacity and Modulation.zh.srt）

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这条公式值得记住它的出身：1948 年，贝尔实验室的克劳德·香农（Claude Shannon）发表《A Mathematical Theory of Communication》，把"通信"第一次变成可数学化的对象——信息量用比特度量，噪声信道的极限容量由此被精确刻画。有意思的是公式里没有出现任何具体波形或电路：无论你用振幅、相位还是频率编码，$B\log_2(1+S/N)$ 就是天花板。这与你学过的排队论里的利特尔法则有同一种气质——极少数假设（噪声为高斯分布、功率受限）推出普适上界，具体实现只能逼近、不能逾越。历史顺序也值得留意：香农 1948 年给出极限，真正逼近极限的 turbo/LDPC 码要到 1993 年前后才出现，中间隔了四十多年——"知道上界"与"造得出逼近上界的系统"是两回事，这也是这一单元先讲极限、再讲实现的原因。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 3：模拟信号（Analog Signals）</span>

图示 slide：一条正弦波的标注图——振幅（Amplitude，振幅）为波峰高度，波长（Wavelength，波长）为相邻两个波峰的间距。要点：

- 频率（Frequency，频率）：$1/\text{波长}$；
- 带宽（Bandwidth，带宽）：频率范围的大小；
- 相位（Phase，相位）：波在一个波长内的时序；
- 光速 $\approx$ 每纳秒 1 英尺（1 foot/nanosecond）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 4：调制：ASK 与 FSK</span>

纯图示 slide：两条波形对比。幅移键控（Amplitude Shift Keying, ASK，幅移键控）：用振幅大小表示 1、0、1；频移键控（Frequency Shift Keying, FSK，频移键控）：用频率高低表示 1、0、1。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 5：幅移键控（Amplitude Shift Keying）</span>

- ASK 在有线网络中工作良好，因为信号强度随距离衰减不大：
  - 用于最常见的有线以太网（Ethernet，以太网）系统。
- <span style="color:#00838f;">PAM-5：五电平脉冲幅度调制（five level pulse amplitude modulation，五电平脉冲幅度调制），电平为 $(-2, -1, 0, +1, +2)$：</span>
  - 用于 100BASE-T 与 1000BASE-T 以太网（100Mbps 与千兆）。
- PAM-16：十六电平脉冲幅度调制：
  - 用于 10GBASE-T 以太网（万兆）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "有线用振幅、无线用相位"这条经验法则的根源是信道的物理性质。双绞线是受控介质：厂家保证衰减特性稳定，接收端能可靠区分 5 个甚至 16 个电平——PAM-16 意味着每符号 4 比特，电平间距只有 PAM-5 的三分之一，只有在"噪声可预测"的线上才敢这么密。而无线信道（下一单元会展开）信号强度随距离、障碍物、多径反射剧烈起伏，振幅在收发两端之间根本守不住，所以无线几乎全部改用相位（PSK/QAM 的相位维度）。这也解释了为什么 1000BASE-T 用 PAM-5 就够、10GBASE-T 要靠更复杂的编码与 DSP 才能撑住 PAM-16：每加一档电平密度，对信道线性度和噪声 floor 的要求都在指数级抬升。换个角度：PAM-5 的 5 个电平里混进一个 0（三态），既传信息又做差错控制，这是 100BASE-TX 设计里很漂亮的一步。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 6：调制：PSK</span>

纯图示 slide：相移键控（Phase Shift Keying, PSK，相移键控）波形——载波频率与振幅不变，在符号边界处波形相位发生跳变，用相位 0 与相位 $\pi$ 表示 0 与 1。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 7：相移键控（Phase Shift Keying）</span>

- PSK 在信号强度可能发生显著变化时工作良好：
  - DSL、cable modem（电缆调制解调器）、无线全都使用相移键控。
- <span style="color:#00838f;">二进制相移键控（Binary phase shift keying, BPSK，二进制相移键控）：两个相位 $(0°, 180°)$：</span>
  - 用于 1Mbps 与 2Mbps 的 802.11b（WiFi）。
- <span style="color:#00838f;">正交相移键控（Quadrature phase shift keying, QPSK，正交相移键控）：四个相位 $(0°, 90°, 180°, 270°)$：</span>
  - 用于 5.5Mbps 与 11Mbps 的 802.11b。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 8：相位切换（Shifting Phase）</span>

纯图示 slide：波形动画帧——展示符号边界处相位跳变的瞬间，前后波形频率不变、但相位差 $\pi$。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 9：I/Q 调制（I/Q Modulation）</span>

- I：同相分量（in-phase component，同相分量，$0°$）；Q：正交分量（quadrature component，正交分量，$-90°$）。
- <span style="color:#00838f;">一个符号是 I 与 Q 的线性组合（linear combination，线性组合）。</span>
  - 二进制相移键控（BPSK）：两个相位 $0°, 180°$，$(I,Q)$ 取两个值（源文此处公式碎裂，按标准星座为 $(\pm1, 0)$，见脚注）；
  - 正交相移键控（QPSK）：四个相位 $0°, 270°, 180°, 90°$。
- 电路实现：I 与 Q 分别与 $0°$ 载波（carrier，载波）、$-90°$ 载波相乘后相加，得到射频（RF, radio frequency）输出。

> <span style="color:#7f8c8d;">脚注：源文中 BPSK/QPSK 的 (I,Q) 具体取值向量在文本提取中丢失（"-(I,Q) =" 后为空），此处按 IEEE 802.11 标准 BPSK 星座 $(\pm1,0)$ 复原，仅示意。</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "符号是 I 与 Q 的线性组合"这句话，用 Strang 的语言重述就是：以 $0°$ 与 $-90°$ 两个正交载波为一组基（basis，基），任何同频带通信号都能表示为它们的线性组合，坐标 $(I,Q)$ 就是在这组基下的分量。你在第五章学过的"正交基下坐标互不干扰"在这里有硬件落地——乘法器把信号分别投影到两根基上，两路各自调幅、再相加合成，于是"调相位"被完全等价地转化成"调两个振幅"，电路瞬间变简单：不用做真正的移相器，只用两个混频器和一个加法器。星座图之所以是二维平面上的点，正是因为 $(I,Q)$ 就是这个二维向量空间的坐标；16-QAM 的 16 个点就是 16 组坐标对。线性代数第五章（特征值与正交性）讲的投影与分解，是理解一切现代调制器的钥匙。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 10：I/Q 星座图（I/Q Constellations）</span>

- 对于相移键控，可以用 I/Q 星座（I/Q constellation，星座图）来表示符号——即 IQ 取值的二维图：
  - 向量的角度：信号的相位；
  - 向量的长度：信号的振幅。
- 纯图示 slide：Q-I 坐标平面上画出 OOK（开关键控）、ASK、BPSK、QPSK 四种调制的星座点分布（OOK/ASK 的点落在 I 轴上，BPSK 两点、QPSK 四点均匀分布）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 11：相位切换（Shifting Phase）</span>

纯图示 slide：又一组相位跳变波形示意，与 Slide 8 呼应，展示不同相位序列下的合成波形。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 12：符号与比特（Symbols vs. Bits）</span>

- <span style="color:#00838f;">符号（symbol，符号）是物理层的传输单位。</span>
- 一个符号可以包含多于一个比特：
  - BPSK：每符号 1 比特（0, 1）；
  - QPSK：每符号 2 比特（00, 01, 10, 11）。
- 例：有线 100BASE-T 以太网（100Mbps，Cat-5 线缆）：
  - PAM-5 的 5 个电压电平 $(-2, -1, 0, +1, +2)$。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 13：QAM（正交幅度调制）</span>

- 幅移键控与开关键控（on/off keying, OOK，开关键控）只用振幅编码符号；相移键控只用相位编码符号。
- <span style="color:#00838f;">正交幅度调制（Quadrature Amplitude Modulation, QAM，正交幅度调制）同时使用振幅与相位：</span>
  - 16-QAM：16 个符号，每符号 4 比特；
  - 256-QAM：256 个符号，每符号 8 比特。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> QAM 是把前两种调制"相乘"的产物：PSK 在单位圆上取点（只动角度），ASK 在一条轴上取点（只动半径），QAM 则在整个平面上布点——角度与半径同时携带信息。用 6.042J 的计数语言说，若角度取 $2^m$ 个相位、半径取 $2^k$ 个振幅级，符号总数就是 $2^{m+k}$，每符号恰好携带 $m+k$ 比特——16-QAM 是 $m+k=4$、256-QAM 是 $m+k=8$，双射建立得干干净净。这种"维度相乘"的套路你在组合数学里见过很多次：字符串计数、格路径计数都是把它数清楚再编码。注意代价：点数翻倍意味着同样发射功率下相邻点的最小距离按平方根缩水，误码率上升——所以 WiFi 网卡会根据实时信噪比在 BPSK 与 256-QAM 之间自适应切换，这正是 MCS 表存在的原因（见《Bit Errors》一讲 Slide 13）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 14：例：16-QAM</span>

纯图示 slide：16-QAM 星座（该星座用于 HSPDA、3G 数据标准）：$4\times4$ 网格点，每点标注 4 位二进制（如 $(-3,3)\to 1011$、$(1,1)\to 0000$、$(3,-3)\to 0111$），与《Bit Errors》一讲 Slide 4 为同一张图。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 15：当今的例子（Examples Today）</span>

- ASK/OOK：有线以太网；
- FSK：NWS "Weatheradio"（美国国家气象局气象电台）、蓝牙（Bluetooth，蓝牙）；
- BPSK：802.11abgn、WiMAX；
- QPSK：802.11abgn、802.15.4、HSPDA、LTE、WiMAX；
- 16-QAM：802.11abgn、HSPDA、LTE、WiMAX；
- 64-QAM：802.11abgn、LTE、WiMAX。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 16：Overview（小结）</span>

- 用模拟信号表示比特有很多种方式：频率、振幅、相位。
- 有线以太网使用振幅（ASK）；
- 当今大多数技术使用相位（PSK），或相位加振幅（QAM）。
- QAM 符号可以表示为同相分量（I）与正交分量（Q）的线性组合：
  - 实际电路正是这样做的；
  - 在二维图上是漂亮的可视化。

<span style="color:#7f8c8d;">（Slide 16/16）</span>

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 7-1《Physical and Link Principles - Shannon Capacity and Modulation》的配套 slides；SNR 与误码率的量化关系见同单元《Bit Errors》，纠错与交织见《Forward Error Correction (FEC)》。源文个别希腊字母（π、度数符号）在提取中丢失，已按上下文复原。</span>
