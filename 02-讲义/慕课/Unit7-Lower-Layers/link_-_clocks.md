# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## Physical Links: Clocks and Clock Recovery（物理链路：时钟与时钟恢复）

> <span style="color:#7f8c8d;">Source: src/慕课/Unit7-Lower-Layers/link_-_clocks.txt · CS144 Fall 2019 · 行内双语翻译版</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 1 · 标题页</span>

Physical Links（物理链路）——Clocks and Clock Recovery（时钟与时钟恢复）。主讲人：Nick McKeown，Stanford University 电气工程与计算机科学教授。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 2 · Outline（大纲）</span>

- 数据是用一个"时钟"（clock）发送的；
- 接收端必须知道何时对到达的数据采样（sample）；
- 异步通信（asynchronous communications）；
- 同步通信（synchronous communications）；
- 把时钟与数据一起编码；
- 恢复（recover）时钟；
- 把数据送入接收端的时钟"域"（clock domain）。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 3 · Data is transmitted using a clock（数据用时钟发送）</span>

图示：发送端用一个 D 触发器（flip-flop）驱动数据输出，时钟为 10 MHz——每 $100\,\mathrm{ns}$ 发出一个比特。发送端的每一位数据都是在时钟沿上被"推"出去的：时钟既定义了比特的节奏，也定义了比特的宽度。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 4 · If we don't know the sender's clock（如果我们不知道发送端的时钟）</span>

纯图示 slide：接收端用自己的本地时钟对到达的数据波形采样。问题在于——接收端既不知道发送端时钟的频率，也不知道它的相位，采样点与比特中心会逐渐错开。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 5 · If we don't know the sender's clock（续·漂移的定量计算）</span>

图示：TX Clock（发送时钟）与 RX Clock（接收时钟）各自驱动采样，某个比特被标为 "Missed!"（漏采）。

设 RX 时钟比 TX 时钟慢 $p$ ppm（parts per million，百万分之一），则比特时间为 $T_{Rx} = T_{Tx}\,(1+10^{-6}p)$。采样点相对数据每比特漂移 $10^{-6}p$ 个比特时间；当累积漂移达到半个比特时，采样点落到比特边界上、开始漏采——即最多经过

```math
\frac{0.5}{10^{-6}\,p}
```

个比特时间后，RX 时钟就会错过一个比特。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "两端没有共享时钟"是链路设计的第一性约束。芯片内部（你在 CSAPP 里学的数字系统）所有触发器共享同一个时钟信号，整个电路是严格同步的；而链路两端各有一块独立晶振，是两个互不知晓的时钟域。100 ppm 的漂移在人类尺度上微不足道：每天才累计差 $86400\times10^{-4}\approx8.6$ 秒；但换到比特尺度就致命——10 Mb/s 链路上采样点每比特漂 $10^{-4}$ 比特，约 5000 个比特（0.5 毫秒）就漂出半个比特、开始漏采。这解释了为什么"随数据恢复时钟"不是可选的优化，而是任何持续传输链路的必需品：本讲后面的一切机制——Manchester 编码、4b5b、时钟恢复单元、弹性缓冲——全部是这一条约束的连锁反应。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 6 · Asynchronous communication（异步通信）</span>

图示：每个包 P 个比特，以一个 Start bit（起始位，电平 0）开头、Stop bit（停止位，电平 1）结尾；TX 时钟与 RX 时钟各自独立运行。

机制：接收端平时空闲，等到起始位 $1\to0$ 的跳变沿才把本地采样时钟"对表"，然后在整个包期间保持采样；只要包足够短（P 小），时钟漂移在一个包内累积不到半个比特，就不会出错。包结束后等待下一个起始位重新对齐。

异步通信有时用于短包链路，例如红外遥控器（IR remote control）、串行链路（serial links）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 起始位/停止位机制是"把全局同步放松为逐包同步"的漂亮折中。它把漂移预算从"永远累积"切成"每个包内累积"，短包天然安全。你在单片机、树莓派上用过的串口（UART，universal asynchronous receiver/transmitter，通用异步收发器）就是这套机制：常见的 9600-8-N-1 配置表示 8 个数据位、无校验位、1 个停止位——每收一个字符就用起始位重新对一次表。代价是每个包约 2 比特的固定开销，以及速率上限（包一长漂移就出格），所以红外遥控、串口调试这类短包低速场景仍在用它，而长包高速链路全部转向下一页的同步方案。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 7 · Outline（大纲·进入后半部分）</span>

重复大纲页：同步通信之后，接下来的主题是——把时钟与数据一起编码、恢复时钟、把数据送入接收端的时钟域。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 8 · Synchronous Communication（同步通信）</span>

图示完整架构：TX clock domain（发送端时钟域）与 RX clock domain（接收端时钟域）由 network link（网络链路）相连，两侧时钟均为 10 MHz ± 100 ppm。

- 发送端：数据在 Sender's Clock（发送端时钟）驱动下经触发器送上链路；
- 接收端：先经过 Clock Recovery Unit（时钟恢复单元），从数据波形中恢复出与发送端同步的时钟；再用恢复出的时钟在触发器处对数据采样；
- 最后由接收端本地时钟（10 MHz ± 100 ppm）把数据转入本地时钟域。

关键理解：恢复出的时钟与数据是同步的，用它采样不会漂移；真正的异步边界只有触发器那一拍——它把数据从"发送端的时序世界"搬进"接收端的时序世界"。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 9 · Encoding for clock recovery（为时钟恢复而编码）</span>

图示：数据（Data）、时钟（Clock）波形与采样点（Sampling points）的对照。

要点：<span style="color:#00838f;">如果时钟不单独用一根线传输，那么数据流本身就必须有足够的 transitions（跳变沿），接收端才能从数据中"读出"时钟的节奏。</span>一长串 0 或一长串 1 都是不行的——没有跳变就没有时钟信息。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 10 · Encoding for clock recovery — Example #1: 10Mb/s Ethernet（示例一：曼彻斯特编码）</span>

10 Mb/s 以太网采用 Manchester Coding（曼彻斯特编码）：图中数据 $0\,1\,1\,0\,1\,0\,1\,1$ 与时钟叠加——每一位的正中间必有一次电平跳变（0 为低→高，1 为高→低）。

- 优点：
  - 保证每个 bit period（比特周期）内至少一次跳变——时钟信息永远充足；
  - 保证 d.c. balance（直流平衡）——高电平与低电平的数量相等，信号无直流漂移。
- 缺点：最坏情况下所需的 bandwidth（带宽）翻倍。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> Manchester 用 100% 的带宽冗余买时钟恢复，4b5b 用 25% 的冗余买——它们是同一条"冗余—可靠"设计曲线上的两个点。更妙的是把本单元其他冗余机制放在一起看：CRC 用冗余做差错检测，FEC 用冗余做纠错，Manchester/4b5b 用冗余做时钟恢复——"以冗余换可靠"是链路层的万能模式，区别只在冗余花在哪个用途上。Manchester 的直流平衡还有工程价值：变压器与交流耦合电路无法通过直流分量，信号长期不平衡会使基线漂移、判决门限失准。再呼应本单元的 Shannon 容量：带宽翻倍等于把同信噪比下的容量预算占掉一半——编码方案的选择本质是在香农账本上做分配，这也解释了为什么后来 100 Mb/s 快速以太网改用 4b5b（配合 NRZI）：速率越高带宽越贵，冗余就得越省。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 11 · Frequency Spectrum for 10Mb/s Ethernet（10 Mb/s 以太网的频谱）</span>

两幅频谱（magnitude–freq，幅度—频率）图对照：

- 不用 Manchester 编码：能量集中在 5 MHz 以内；
- 用 Manchester 编码：能量延伸到 10 MHz。

这正是"带宽翻倍"代价的频域写照：为了让每个比特周期都带一次跳变，信号的最高频率分量翻了一倍。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 12 · Encoding for clock recovery — Example #2: 4b5b encoding（示例二：4b5b 编码）</span>

4b5b：每 4 比特数据映射为 5 比特码字，例如 $0000\to11110$、$0001\to01001$、$0010\to10100$ ……（码表精心挑选，保证不会有太长的无跳变串）。

- 优点：
  - 带宽效率更高——开销只有 25%；
  - 5 比特共 32 种码字、只用 16 种，多余的码字可用来表示控制信息（如空闲、帧边界）。
- 缺点：跳变变少，时钟恢复稍微困难一些。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 13 · Summary（小结）</span>

再次给出同步通信的完整架构图：TX 时钟域 →（网络链路）→ Clock Recovery Unit（时钟恢复单元）→ 触发器 → RX 时钟域。

一句话总结本讲前半部分：发方"带着时钟"发数据（或让收方能从数据中恢复时钟），收方用恢复出的时钟采样，再在触发器处把数据转入自己的本地时钟域。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 14 · &lt;end&gt;</span>

原讲义的视频分段结束标记，无正文内容。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 15 · Elasticity Buffer（弹性缓冲区）</span>

同一张架构图，中间多了一个新部件：elasticity buffer（弹性缓冲区）。

原因：即使时钟恢复单元工作得再好，恢复出的时钟与发送时钟之间仍有 ±100 ppm 量级的频差，接收端"写入"缓冲区的速率与本地时钟"读出"的速率并不严格相等——速率差虽小，持续累积就会溢出或掏空，必须有一个缓冲区来吸收。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 16 · Sizing an elasticity buffer（弹性缓冲区的定容·图示）</span>

图示：cumulative bits（累积比特数）随时间变化的两条直线。

- 当 $R_{rx} > R_{tx}$（写入快于读出）：缓冲水位持续上升，趋向溢出；
- 当 $R_{tx} > R_{rx}$（读出快于写入）：水位持续下降，趋向下溢（underflow）；
- $B$ 为缓冲区容量，标称工作点应停在半满 $B/2$，两个方向都留有余地。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 17 · Sizing an elasticity buffer（定容的四条规则）</span>

1. 让缓冲区标称停在 $B/2$（半满）：新包开始时允许缓冲充到 $B/2$；或者确保新包到来之前缓冲已排空回 $B/2$。
2. 选择 $B$，使得缓冲区在一个包传完之前既不溢出（overflow）也不下溢（underflow）。
3. 当 $R_{tx} > R_{rx}$ 时：给定 inter-packet gap（包间间隙），按"不溢出"来确定 $B/2$。
4. 当 $R_{rx} > R_{tx}$ 时：按最大长度包（max length packet），选取 $B/2$ 保证不下溢。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 18 · Preventing overflow（防溢出·图示）</span>

图示：cumulative bits（累积比特数）—时间图。一个最大包 Pmax 以 $R_{\max}$ 的速率到达，而缓冲区以 $R_{\min}$ 的速率向本地时钟域排出；两条曲线之间的缺口在包结束时最大；包间间隙（interpacket gap）期间缺口回落。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 19 · Preventing overflow（防溢出·条件）</span>

要防止溢出，需满足（原公式以图片嵌入，此处按图中几何关系复原：缺口宽度为 $P_{\max}/R_{\max}$、高度为 $(R_{\max}-R_{\min})\cdot P_{\max}/R_{\max}$）：

```math
B/2 \ \ge\ P_{\max}\,\frac{R_{\max}-R_{\min}}{R_{\max}}
```

含义：接收一个最大包期间，"快写慢读"最多净攒下这么多比特，半满缓冲必须吞得下。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 20 · Preventing underflow（防下溢·图示）</span>

对偶情形的图示（单位换成 cumulative bytes，累积字节数）：最大包 Pmax 以 $R_{\min}$ 慢速到达，而排出以 $R_{\max}$ 快速进行；包间间隙期间两条曲线继续分离。received bytes（接收字节）与 transmitted bytes（发送字节）两条曲线的最大缺口不能超过 $B/2$，否则缓冲被掏空、无数据可读。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 21 · Preventing underflow（防下溢·条件）</span>

要防止下溢（原文公式在文本提取中损坏，按图示几何关系：最大缺口 $\approx P_{\max}\,(R_{\max}-R_{\min})/R_{\min}$，与防溢出条件同量级）：

```math
B/2 \ \gtrsim\ P_{\max}\,\frac{R_{\max}-R_{\min}}{R_{\min}}
```

由于 $R_{\max}\approx R_{\min}$（只差百万分之几百），两个方向的界在数值上几乎相同——半满设计让一个界同时保护两边。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 22 · Sizing an elasticity buffer — Example（定容示例）</span>

已知：最大包 $P_{\max}=4500$ 字节（IP 数据报的上限量级）；时钟容差 ±100 ppm。

两个时钟的最坏相对速率差为 $|R_{\max}-R_{\min}|/R = 200\times10^{-6}$，于是

```math
B \ \ge\ 2\,(4500\times8\times200\times10^{-6}) = 14.4\ \text{bit}
```

讲义取缓冲大于 14 比特。结论三条：

1. 缓冲区大于 14 比特；
2. 开始排出（读出）之前至少先攒 7 比特——保证从半满启动；
3. 包间间隙至少 7 比特——保证下个包到来前水位已排回半满。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这页的 14 比特是纯最坏情形（worst-case）分析，与概率论里的排队思维形成有趣对照。排队论（如 M/M/1）用到达过程的期望与方差定缓冲容量，靠大数定律把波动"平均掉"；而两个晶振的频率差是系统性偏差（bias）：恒为正或恒为负、均值不为零——大数定律帮不上忙，样本再多也不会抵消，只能按上界给确定性保证。注意界与包长线性相关：$B\ge 2P_{\max}\times200\times10^{-6}$，是最大包 4500 字节直接决定了缓冲大小。工程中这两套思维并存：随机波动用期望与分位数去对付（排队、拥塞、CDN 容量规划），确定性偏差用最坏上界去封死（时钟、对齐、协议字段宽度）。"先攒 7 比特再排"与"间隙至少 7 比特"则分别从启动和周期两个方向，把水位拉回半满这个安全工作点。
