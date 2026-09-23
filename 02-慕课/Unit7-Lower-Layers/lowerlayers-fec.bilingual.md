# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## Physical and Link Layers: Forward Error Correction（物理层与链路层：前向纠错 FEC）

> <span style="color:#7f8c8d;">Source: src/慕课/Unit7-Lower-Layers/lowerlayers-fec.txt · CS144 Fall 2019 · 行内双语翻译版</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 1 · 标题页</span>

Physical and Link Layers: Forward Error Correction（FEC，前向纠错）。本讲回答一个问题：信道总有噪声，与其丢弃出错的包重传，能不能直接把错误"算"回来？

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 2 · SNR/BER Curves（信噪比—误比特率曲线）</span>

- 对给定的 modulation scheme（调制方案）与 SNR（signal-to-noise ratio，信噪比），可以计算出期望的 BER（bit error rate，误比特率）——对噪声做一些数学假设即可推出，这是射频（RF）通信理论的基石原理；
- <span style="color:#00838f;">BER 可以压到任意低，但永远到不了零。</span>
- 实践中算下来：把包当作裸比特（raw bits）直接发送是非常低效的——期望的数据吞吐量远低于 Shannon limit（香农极限）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 为什么裸发比特效率极低？因为不加编码时，想把 BER 压低只能提高信噪比，而 BER 随 SNR 只是指数下降、永不为零——接近零错误就要烧掉接近无限的功率。Shannon 定理（本单元另一讲）指出更聪明的出路：信道容量 $C = B\log_2(1+\mathrm{SNR})$ 之内，存在编码能让差错概率任意小。搭个脚手架：最笨的编码是"每位重复 3 次、接收端多数表决"，它以 3 倍冗余把错误率从 $p$ 压到约 $3p^2$（两次以上同时错才会判错）；好的码（如本讲的 Reed-Solomon）能用少得多的冗余做到指数级改善。所以"吞吐远低于香农极限"的补救不是硬扛噪声，而是留出一部分带宽预算交给编码——冗余不是浪费，而是逼近香农极限的唯一途径。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 3 · Coding（编码与编码增益）</span>

- 在物理层加一点点冗余，就能大幅提升链路层的有效吞吐——理论与实践皆然；
- <span style="color:#00838f;">Coding gain（编码增益/码率）：链路层比特数与物理层比特数之比。</span>
  - 1/2 码：每个链路层比特对应 2 个物理层比特；
  - 3/4 码：每 3 个链路层比特对应 4 个物理层比特；
- <span style="color:#00838f;">Forward error correction（FEC，前向纠错）：主动附加一些额外数据（冗余），使接收端能够纠正潜在的错误。</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 4 · Coding Algorithms（编码算法谱系）</span>

- 编码算法非常多，各有不同的权衡：Hamming codes（汉明码）、convolutional codes（卷积码）、LT codes、LDPC、Turbo codes、Tornado codes、Raptor codes ……
- 其中 Reed-Solomon（里德–所罗门码）纠错：
  - 应用极其广泛（如 CD、DVD、DSL、WiMax、RAID6 存储）；
  - 数学上（与其他一些码相比）相当简单。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 5 · Reed-Solomon Basic Idea（RS 码的基本思想）</span>

- 取 k 个数据块（chunk of data）；
- 把它们当作一个 $k-1$ 次多项式的系数；
- 沿多项式计算 n 个点（$n \ \ge\ k$），作为编码后的数据发送；
- <span style="color:#00838f;">n 个点中的任意 k 个，都能恢复出多项式的系数——也就是原始数据。</span>
- 麻烦之处：这 n 个点的取值必须在 finite field（有限域）中计算（比特数有限）。

图示：一条抛物线及取值示意（图片来源：http://en.wikipedia.org/w/index.php?title=File:Parabola.svg ）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> RS 的数学内核你在 6.042J 已经见过：平面上 k 个横坐标互异的点唯一确定一个次数不超过 $k-1$ 的多项式（插值唯一性）。RS 的巧思是换个方向用这条定理——把 k 个数据块当作多项式的系数，发送的是多项式在 n 个点的取值；任何 k 个取值都能反解出系数，数据天然就带了 $n-k$ 份冗余。"必须在有限域上计算"听起来吓人，其实你在 6.042J 数论里学过原型：模素数 p 的整数 $\{0,1,\dots,p-1\}$ 构成域，加减乘除都有定义且"除法不溢出"；GF($2^8$) 只是把这个想法换成"模 2 的多项式"做算术——一个字节恰好是一个域元素，硬件实现极其便宜。若直接用普通整数 0–255，加法会进位溢出、乘法超出 255、除法可能除不尽，插值与纠错的代数保证全部失效——这正是 Slide 7 说"0–255 不是域"的原因。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 6 · Details（细节：擦除与错误）</span>

- 两类错误：
  - <span style="color:#00838f;">Erasure（擦除）：错误的位置已知（值被"擦掉"了）。</span>
  - <span style="color:#00838f;">Error（错误）：位置未知（例如比特翻转）。</span>
- 取 k 个数据块编成 n 块（$n \ \ge\ k$）；
- Reed-Solomon 最多可纠正 $n-k$ 个 erasure（剩下的好点仍有 k 个）；
- Reed-Solomon 最多可纠正 $(n-k)/2$ 个 error。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 为什么纠 error 的能力恰好是纠 erasure 的一半？用线性方程组的语言（Strang 的方向）看最清楚：k 个系数是 k 个未知数，n 个点给出 n 个方程。Erasure 是"知道哪个方程坏了"——直接扔掉它，只要剩下至少 k 个好方程就能解，所以能容忍 $n-k$ 个。Error 是"方程被篡改了、但不知道是哪一个"——每个未知位置的错要同时回答"谁错了"和"错成什么"两个问题，一个消耗 2 个冗余方程，所以只能纠 $(n-k)/2$ 个。这个 2 倍差距在存储系统里一目了然：RAID6 用双校验容忍 2 块"知道哪块坏了"的盘（2 个 erasure），但对"静默损坏"（数据错了却报正常，位置未知）就只能容忍 1 块——这也是为什么工程上宁可花代价给每个块记录位置信息。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 7 · Conceptual Reed-Solomon Code（概念性的 RS 码示例）</span>

- 取 223 个 8-bit 值，作为 222 次多项式 $p$ 的系数；
- 计算 $p(0), p(1), p(2), p(3), p(4)\ldots p(254)$，共 255 个 8-bit 值（在一个域上计算）；
- 发送这 255 个值——这是一个 (255,223) 码：每 255 个 codeword（码字）来自 223 个 data word（数据字）；
- 可以从至多 32 个 erasure 或 16 个 error 中恢复（$255-223=32$，$32/2=16$）；
- 注意：今天的实际工程并不这么做——
  - 0–255 并不构成域（代数上不成立，必须用 GF($2^8$) 这样的域）；
  - 按这个方式直接解码太复杂；
  - 但它给了你基本直觉。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 8 · Reed-Solomon Example（RS 编码示例·数据侧）</span>

图示：以字符串 "Hello, ..." 为例——前 6 个字节的十六进制为 0x4865、0x6c6c、0x6f2c（即 "Hel" "lo,"），对应比特串

`0100 1000 0110 0101 0110 1100 0110 1100 0110 1111 0010 1100`

这是 48 比特的 packet data（分组数据，以比特为单位）。下方是编码后"带冗余/纠错码的数据（以 symbols，符号为单位）"，与原始比特上下对照。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 9 · Reed-Solomon Example (7,5)（RS 示例：(7,5) 码）</span>

图示流程：48 比特数据（图中标注 48 bits、45 + 3）末尾补 12 个零，得到 60 比特；切成 3-bit 符号后，每 5 个数据符号经 (7,5) 码编成 7 个 3-bit codeword（码字）符号——即 5 个数据符号加 2 个冗余符号。图中上下两排分别是原始的 packet data（比特）与编码后带冗余的数据（symbols，符号），各分成若干组"7 个 3-bit 码字"。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 10 · Interleaving（交错）</span>

- 按讲义记号：一个 (n+k, n) 的 Reed-Solomon 码可抗 k 个 erasure 或 k/2 个 error——对 (7,5) 码即 2 个 erasure 或 1 个 error；
- 物理介质上的错误往往是 burst errors（突发错误）——一错一串；
- 可以通过 interleaving（交错）让编码更健壮；
- 示例：12 个 (7,5) 码块，每块 21 比特（7 个 3-bit 码字）；符号记作 A0–A20、B0–B20、C0–C20、……、L0–L20——发送时按"行"读出、不同码字的符号穿插排列，突发错误就被摊薄到多个码字上。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 交错解决的是错误的相关性问题。划痕、电火花、脉冲噪声造成的错误是成串的 burst：一次能吞掉相邻几十个符号，单个码字的纠错能力（2 个 erasure）瞬间被击穿。交错把 A、B、C……各码字的符号按列写入、按行读出，让物理上相邻的符号属于不同的码字——突发到达时每个码字只摊到一两个错，回到纠错能力之内。用概率论的话说：交错把强相关的错误过程"白化"成近似独立的小错误，而独立性正是大数定律、排队模型这些工具成立的前提——交错是物理层主动对齐统计假设的工程手段。这对组合无处不在：CD 上一条划痕不会毁掉整首歌、RAID6 两块盘同时坏不丢数据、二维码破损一半仍能扫出来，背后都是"Reed-Solomon + 交错"。
