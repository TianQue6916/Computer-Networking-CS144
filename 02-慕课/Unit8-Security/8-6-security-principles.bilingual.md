> <span style="color:#7f8c8d;">Source: src/慕课/Unit8-Security/8-6-security-principles.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> <span style="color:#c0392b">Security Principles（安全原则）：Confidentiality, Integrity, and Availability（机密性、完整性与可用性）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 2 · 基本问题（Basic Problem）</span>

近似地说（to first approximation），<span style="color:#00838f;">攻击者控制着整个网络</span>：他们可以 snoop（窥探）、replay（重放）、suppress（抑制/丢弃）、send（伪造发送）。

我们如何防御？

- 在不安全的网络上实现安全通信——cryptography（密码学）；
- 在大网络不安全的前提下，保护网络的小片段；
- 设计能在攻击下依然良好扩展（scale well）的系统。

两条路线：cryptography（密码学）与 scalable system design（可扩展系统设计）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 3 · 密码学（Cryptography）</span>

- 密码学是一套用于保护通信的数学原理与思想。
- 小心：它极易被用错！经常被误用！必须理解它保证什么、不保证什么。
- 密码学如何提供帮助：
  - Confidentiality（机密性）：我们可以私密地通信（encryption，加密）；
  - Integrity（完整性）：防止篡改（hashes 哈希、signatures 签名、MACs 消息认证码）；
  - Authenticity（真实性）：你确实是你说的人（certificates 证书、MACs、signatures）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 4 · 密码学（重复页）</span>

本页与 Slide 3 内容完全相同（原课件重复展示），要点不变：密码学的三大贡献是 Confidentiality（机密性）、Integrity（完整性）、Authenticity（真实性），而"密码学极易被误用"的警告值得再读一遍。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 5 · 机密性（Confidentiality）</span>

我想秘密地告诉你一件事，除你之外没人知道我说了什么："My credit card number is...."（我的信用卡号是……）。

**Perfect confidentiality（完美机密性）：one-time pad（一次一密）**

- 你我共享一段完全随机的 0/1 密钥 $K$，其他任何人都拿不到；
- 我把消息 $M$ 与 $K$ 做 XOR（异或），得到 $C$，发送 $C$（$C = M \oplus K$）；
- 你把 $C$ 与 $K$ 做 XOR，就重构出 $M$（$M = C \oplus K$）。

优点：information-theoretically secure（信息论安全）且快速——给定任意 $C$，任何 $M$ 都同样可能。

缺点：需要一个与"我此生可能发送的全部数据"一样长的 $K$。

出路：设法交换一个较小的 $K$（$k$ 位），让攻击者至多只剩 $2^k$ 种可能的候选——这正是下一讲 computational security（计算安全）的起点。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 为什么"给定任意 $C$，任何 $M$ 都同样可能"？用 Bertsekas 概率论第一二章的条件概率就能严格证明：设 $K$ 是均匀分布的 $k$ 位随机串，则对任意明文 $m$ 与密文 $c$，$P(C = c \mid M = m) = P(K = m \oplus c) = 2^{-k}$，与 $m$ 无关。再由贝叶斯公式，$P(M = m \mid C = c) = P(M{=}m)\,P(C{=}c \mid M{=}m) / P(C{=}c)$，分子分母中的条件项相消，后验等于先验——看到密文完全不改变对明文的信念。这就是 Shannon（1949）定义的 perfect secrecy（完美保密）：信息论安全连"无限算力的攻击者"都无计可施，与后面要讲的 computational security（只要求多项式时间内破不动）是两个强度不同的概念。注意上述推理依赖 $K$ 均匀且只用一次——这正是下一讲 two-time pad 测验要击破的前提。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 6 · 完整性（Integrity）</span>

- 我想确保你收到的消息未被更改、未被篡改："I recalculated his grade and it is a C..."（我重算了他的成绩，是 C……）、"在你的计算机上运行这个程序"——这类内容容不得改动。
- 我还想确保消息确实是你发的：Nick 说："I said it was alright if he handed in the assignment late"（我说过他晚交作业没关系）——这句话真的出自他之口吗？

**Cryptographic hash（密码学哈希）：$H(M)$**

- 把任意长度的输入映射成固定长度的哈希值；
- Collision-resistance（抗碰撞性）：给定 $x \ne y$，要找到 $H(x) = H(y)$ 在计算上不可行。

**Message authentication code（消息认证码）：$\mathrm{MAC}(M, K)$**

- 用密钥 $K$ 生成 $\mathrm{MAC}(M,K)$，同样用 $K$ 校验；
- 没有 $K$，就无法生成合法的 $\mathrm{MAC}(M,K)$。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 机密性与完整性是两个独立的目标，加密本身不提供防篡改——这是本页背后最容易被忽视的一点。第一性原理地看：基于 XOR 的加密（下一讲的 CTR/OFB 模式，$C = M \oplus S$）天然是可延展的（malleable）——攻击者翻转密文第 $i$ 位，解密后恰好是明文第 $i$ 位被翻转；他虽然不知道明文内容，却能精确控制改动的位置。所以"只有机密性没有完整性"是糟糕设计的标志；而"只要完整性不要机密性"却很常见——路由更新、软件发布签名、DNSSEC 记录都需要"不可篡改 + 可认证"，内容却完全公开。例外是加密存储（不联网、没有中间人），此时机密性可以单独存在。这也是本单元先讲 hash/MAC、再讲加密与 MAC 如何组合的原因。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 7 · 真实性（Authenticity）</span>

- 我想确认你确实是你说的人："This is the provost... I need your credit card number."（我是教务长……我需要你的信用卡号。）"你应该把我的所有流量都经由这个第三方转发。"
- 如果我们事先交换过密钥 $K$：直接校验 $\mathrm{MAC}(\text{“This is the...”}, K)$ 即可。
- 如果没有：chain of trust（信任链）——
  - 我们按设计（by design）信任 Verisign（root of trust，信任根）；
  - Verisign 说："这是给 Stanford 的密钥"；
  - Stanford 说："这是给教务长的密钥"。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 信任链解决的是一个无穷倒退问题：验证签名需要签名者的公钥，而这个公钥也可能是假的，又需要另一张证书为它背书，递归没有尽头。6.042J 里你已经体会过同样的结构：任何证明都必须从一组公理出发，公理本身不被证明、只被接受——root of trust（信任根）就是信任体系里的"公理"，由浏览器与操作系统出厂时硬编码（hard-baked）进本地，构成不证自明的初始信任。注意 Verisign → Stanford → 教务长这条链与 8-9 讲的签名机制完全同构：每一环都是"用上一环的公钥验证下一环的签名"，工程形态就是证书一讲的证书链。它同时解释了"误用密码学"为何如此危险：链条上任何一环被攻破，其下游的全部信任随之失效——信任没有局部故障隔离。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 8 · 高可用设计（High Availability Design）</span>

- DoS（拒绝服务）与 DDoS（分布式拒绝服务）；
- 攻击种类繁多，防御也很多样：
  - Replication（多副本 / scale-out，横向扩展）；
  - Keeping costs symmetric（保持攻防成本对称）；
  - Upstream filtering（上游过滤）——"路由器，别再放那些 ping 进来了！"；
- 这是一场持续的军备竞赛（arms race），本课不再深入。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 9 · 密码学（重复收束页）</span>

本页再次重复 Slide 3 的内容，作为本讲的收束：密码学通过 Confidentiality（机密性）、Integrity（完整性）、Authenticity（真实性）三条主线支撑安全通信，且务必理解它保证什么、不保证什么。
