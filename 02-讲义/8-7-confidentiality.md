> <span style="color:#7f8c8d;">Source: src/慕课/Unit8-Security/8-7-confidentiality.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> <span style="color:#c0392b">Confidentiality（机密性）</span>

> <span style="color:#7f8c8d;">副标题：It's just between you and me（天知地知，你知我知）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 2 · 对称加密（Symmetric Encryption）</span>

- 双方（Alice 与 Bob）共享一个 secret key（密钥）$K$；
- 给定消息 $M$ 与密钥 $K$：
  - $M$ 称为 plaintext（明文）或 cleartext（清晰文本），即我们要保密的内容；
  - Encrypt（加密）：$E(K, M) \to C$，$C$ 称为 ciphertext（密文）；
  - Decrypt（解密）：$D(K, C) \to M$；
  - 攻击者无法在没有 $K$ 的情况下从 $C$ 导出 $M$（除非把每个 $K$ 都试一遍……）。
- $E$ 与 $D$ 使用同一个密钥 $K$，因此得名 symmetric encryption（对称加密）。
- 实例：AES、Blowfish、DES、RC4。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 3 · 一次一密（One-Time Pad）</span>

- 生成一段完全随机（perfectly random）的比特流 $K$；
- $E(K, M) = M \oplus K$；
- $D(K, C) = C \oplus K$（因为 $K \oplus K = 0$）；
- "Perfect" secrecy（完美保密）：
  - Information-theoretically secure（信息论安全）：给定 $C$ 但没有 $K$，$M$ 可以是任何东西！
  - 快：XOR 便宜又快；
- 完全不实用：需要一段与全部数据一样大的密钥。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 4 · 测验：Two Time Pad（两次一密）</span>

> <span style="color:#2471a3;">**[question]**</span> （测验题，保留英文原文）You generate a perfectly random key K. You send two messages, M1 and M2, encrypted with K (C1 = K ⊕ M1, C2 = K ⊕ M2). Does sending C2 leak information about M1 and M2?

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 5 · 测验解答：Two Time Pad</span>

> <span style="color:#2471a3;">**[solution]**</span> **答案：会泄漏。** 若窃听者 Eve 同时听到 $C_1$ 与 $C_2$，她就能重构 $M_1 \oplus M_2$。由 $C_1 = K \oplus M_1$、$C_2 = K \oplus M_2$ 可得：$C_1 \oplus C_2 = (K \oplus M_1) \oplus (K \oplus M_2) = M_1 \oplus M_2$。例如，若 $M_1 = M_2$，则 $C_1 = C_2$。难怪它叫 one-time pad（一次一密）——密钥只能用一次！

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这个推导在 6.042J 的代数语言下极其干净：XOR 就是模 2 加法（逐位进行的 $+ \bmod 2$），$K \oplus K = 0$ 无非是"每个比特 $x + x \equiv 0 \pmod 2$"。一次一密之所以完美，是因为 $K$ 给每条消息垫上了一层均匀随机的"位移"；密钥用第二次，就等于在未知数 $M_1, M_2$ 之间写下了线性方程 $M_1 \oplus M_2 = C_1 \oplus C_2$——$2k$ 位未知量被砍掉一半。更糟的是 $M_1 \oplus M_2$ 保留了英文文本的统计结构：对它做词频分析甚至可以分别恢复两条明文，二战中的 VENONA 项目正是利用苏联密电本对一次一密的重复使用破译了大量电报。设计教训：任何密钥或随机数的复用，都会在密文之间引入可被观测的代数关系。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 6 · 思路：计算安全（Idea: Computational Security）</span>

- 安全地分发一个较小的 $K$（例如 128 位、256 位）；
- 用 $K$ 加密大得多的 $M$；
- 给定 $C = E(K, M)$，可能的 $M$ 也许只有一个——如果 $M$ 带 redundancy（冗余性）的话；
- 但我们相信从 $C$ 找到它 computationally intractable（计算上不可行）：
  - 可以穷举所有可能的 $K$，但 $2^{128}$ 是天文数字的工作量！

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> $2^{128}$ 到底有多大？$2^{128} \approx 3.4 \times 10^{38}$。假设攻击者有一百万台机器、每台每秒尝试 $10^9$ 个密钥（合计 $10^{15}$ 次/秒，已是很激进的配置），穷举一轮约需 $3.4 \times 10^{23}$ 秒，即 $10^{16}$ 年量级——而宇宙年龄只有 $1.4 \times 10^{10}$ 年。这就是 computational security（计算安全）与上一页 information-theoretic security（信息论安全）的分野：前者只要求"多项式时间内破不动"，把安全性从"不可能"降格为"贵到没有意义"。你在 6.006 里习惯把算法效率写成输入规模的函数；密码学把这个视角反过来用——安全参数（密钥长度 $k$）每加 1 位，攻击代价翻倍，而防御方的成本只线性增长。这种攻防成本的不对称，就是"用指数换安全"的第一性原理。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 7 · 密码（Ciphers）</span>

- **Stream cipher（流密码）：伪随机 pad**
  - 由密钥生成一段伪随机（pseudo-random）比特序列；
  - 像一次一密那样用 XOR 加密/解密；
  - 但它绝不是 one-time pad！谁要是对你说这话，立刻别信他！
  - 流密码在实践中惹过许多麻烦，所以我的建议是：别用它们。
  - 例：802.11 的 WEP 约 2001 年被攻破，2003 年被 WPA 取代，2004 年又有 WPA2。
- **Block cipher（分组密码）**
  - 处理固定大小的块（64 位、128 位等）；
  - 把明文块映射为密文块；
  - 今天一般应该用 AES；此外还有许多其他算法。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 8 · 密码（Ciphers，叠加演示页）</span>

本页重复 Slide 7 的内容，但幻灯片上叠加了两组交错的文字——这是教授在课堂上现场演示 two-time pad 的效果：两段英文明文被同一密钥流异或在一起后，各自的可读碎片同时浮现（页面上能同时辨认出 "BUT NOT A ONE-TIME PAD!"（但绝不是一次一密！）与 "Example: 802.11 WEP shown broken in ~2001, replaced by WPA in 2003"（例：802.11 的 WEP 约 2001 年被攻破，2003 年被 WPA 取代）的残片，其间还夹杂 "a large amount of state missing"（大量状态缺失）与警告 "Don't try it this at home!"（别在家尝试这个！））。结合 Slide 5 的测验结论看：同一密钥流加密两条消息，等于把 $M_1 \oplus M_2$ 白送给对手。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "流密码不是一次一密"的危险，WEP 给出了最完整的反面教材：802.11 WEP 用 RC4 做流密码，却配了一个只有 24 位的 IV（初始向量），且密钥长期不变——同一 IV 出现就意味着同一密钥流重复使用，恰好是 Slide 5 的 two-time pad。用 Bertsekas 概率论里的生日问题估算：24 位 IV 共 $2^{24}$ 种，发送 $n$ 个包后出现 IV 碰撞的概率约为 $1 - e^{-n^2/2^{25}}$；$n = 4096$ 时约 39%，$n = 10^4$ 时已超 95%——在繁忙的接入点上几分钟就凑齐了。2001 年 Fluhrer、Mantin、Shamir 进一步指出 RC4 密钥编排的前若干字节存在可利用的统计偏差（密钥流并非"看起来均匀"），结合 IV 的结构可在几分钟内恢复整把密钥。2003/2004 年的 WPA/WPA2 更换 IV 管理与加密算法，正是对"伪随机 ≠ 随机、短 IV = 高碰撞概率"这两课的回应。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 9 · 分组密码实例：Blowfish（Example Block Cipher: Blowfish）</span>

图示：64 位的明文块 $M$ 被切成两个 32 位半块 $L_0$ 与 $R_0$，与由密钥导出的子密钥 $P_1 \ldots P_{18}$ 及轮函数 $F$ 一起，经 16 轮 Feistel 迭代（图上标注 "13 more iterations"，还有 13 轮）后输出 64 位密文 $C$。

- 由密钥推导 $F$ 与 18 个子密钥（$P_1 \ldots P_{18}$）；
- 把明文块切成两半 $L_0$、$R_0$，逐轮迭代：
  $R_i = L_{i-1} \oplus P_i$，$L_i = R_{i-1} \oplus F(R_i)$；
- 末轮：$R_{17} = L_{16} \oplus P_{17}$，$L_{17} = R_{16} \oplus P_{18}$，输出 $L_{17}R_{17}$；
- （这只是不完整的示意描述！）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 10 · 分组密码测验（Block Cipher Quiz）</span>

> <span style="color:#2471a3;">**[question]**</span> （测验题，保留英文原文）I didn't take CS144, so I decide Blowfish is too slow and cut it to one iteration as on the left. I feed it two plaintext blocks. L0 and L2 are underlined.

> The first plaintext block is 0x0000000000000000, producing ciphertext 0x4F072DBC7B77DCA2.

> The second ciphertext is 0x4F072DBC90DCDC85.

> What is the value of L0 in the second plaintext block?

> a. EBAB0027　　b. 5414D8F7　　c. AA0A0072　　d. E6A30055　　e. E6A30057　　f. impossible to determine

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 11 · 分组密码测验（重复页）</span>

本页与 Slide 10 相同（重复展示同一道题，选项加 0x 前缀；提取文本中选项 d 在两页间有 E6A30055/E6A30025 的差异，应为提取噪声）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 译者补解（原片未给答案）：答案是 **(a) 0xEBAB0027**。一轮 Blowfish 只有两条式子：$R_1 = L_0 \oplus P_1$（左半进右半）与 $L_1 = R_0 \oplus F(R_1)$（右半经 $F$ 进左半）。第一块全零：$R_1 = P_1 = \texttt{0x7B77DCA2}$，$L_1 = F(P_1) = \texttt{0x4F072DBC}$，与密文吻合。第二块密文右半给出 $L_2 \oplus P_1 = \texttt{0x90DCDC85}$，于是 $L_2 = \texttt{0x90DCDC85} \oplus \texttt{0x7B77DCA2} = \texttt{0xEBAB0027}$。更妙的看法：$L_2 = C_{2,\text{右}} \oplus C_{1,\text{右}}$——一轮 Feistel 的左半根本没经过轮函数 $F$，只被子密钥 $P_1$ 异或了一次，扩散（diffusion）为零；16 轮的意义就是把每个输出位变成所有输入位的复杂函数（雪崩效应）。这道题用一次"偷工减料"直观展示了多轮迭代的必要性。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 12 · 使用分组密码：ECB 模式（Using Block Ciphers）</span>

消息通常比一个块长。**ECB（electronic code book，电码本）模式**：

- 把明文切成块，逐块独立加密：$m_1, m_2, m_3, m_4$ 分别经 Enc 得到 $c_1, c_2, c_3, c_4$；
- 攻击者解不开任何一块，消息看似安全；
- 可以复用密钥——每块独立加密都会"安全"。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 13 · 错了！（WRONG!）</span>

- 攻击者会注意到重复出现的明文块：如果传输一个稀疏（sparse）文件，他能看出非零区域在哪里。
- 例：军事指令——大多数日子发送加密的 "nothing to report"（今日无战报）；某天发送 "attack today"（今日进攻），攻击者立刻知道有行动了。

图示：维基百科 Block cipher mode of operation 词条的著名配图——"ECB 企鹅"：用 ECB 模式加密一张企鹅图片，轮廓依然可见（source 原图 / ECB 加密后 / desired output 期望效果）。http://en.wikipedia.org/wiki/Block_cipher_mode_of_operation

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 14 · 密码块链接模式（Cipher Block Chaining, CBC Mode）</span>

- 为每条消息选取 IV（initialization vector，初始向量），IV 可以公开：
  - 当且仅当密钥只用于加密一条消息时，IV 才可以是 0；
  - 若密钥被复用，则每条消息都要随机选 IV。
- 加密式：$C_1 = E(K, M_1 \oplus IV)$，$C_i = E(K, M_i \oplus C_{i-1})$。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 15 · CBC 模式（图示页）</span>

与 Slide 14 相同，但补上了 ECB 与 CBC 的对比图（source / ECB / CBC）：ECB 的各密文块彼此独立、重复明文产生重复密文；CBC 因链式异或消除了这种规律性。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 16 · 其他许多模式！（Many Other Modes!）</span>

- **CFB（Cipher Feedback，密码反馈）模式**：$C_i = M_i \oplus E(K, C_{i-1})$——适合长度不是块大小整数倍的消息；
- **OFB（Output Feedback，输出反馈）模式**：反复加密 IV，把结果当作流密码的密钥流使用；
- **CTR（Counter，计数器）模式**：$C_i = M_i \oplus E(K, i)$——想要并行加密时非常有用；
- 测验：给定共享密钥，仅靠 CBC 模式加密，能把文件安全地传给对手吗？

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 17 · 其他许多模式！（重复页）</span>

本页与 Slide 16 相同（重复展示），要点依旧是三种模式的差异与末尾那道测验题。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这道测验的答案是否定的，而且正是 8-8 讲第一页的主题：CBC 只提供机密性，不提供完整性。看解密式 $M_i = D(K, C_i) \oplus C_{i-1}$：攻击者翻转密文块 $C_{i-1}$ 的任意一位，$M_{i-1}$ 整块会乱成垃圾，但 $M_i$ 的对应位会以完全可预测的方式翻转——他能把你文件里的某个字节从 0 改成 1 而你毫无察觉；接收方"解密成功"就照单全收，攻击即告成功。所以"加密过"不等于"安全传输"：需要下一讲的 MAC 来检测篡改，这也是 encrypt-then-MAC 组合的由来。把这道测验放在 CBC 一页的末尾，等于给下一讲埋了引子——CS144 的课件常常这样前后勾连。
