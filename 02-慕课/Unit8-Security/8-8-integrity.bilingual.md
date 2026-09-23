> <span style="color:#7f8c8d;">Source: src/慕课/Unit8-Security/8-8-integrity.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> <span style="color:#c0392b">Integrity（完整性）</span>

> <span style="color:#7f8c8d;">副标题：I'd swear my life on it（我愿以性命担保）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 2 · 光有保密还不够（Secrecy Is Not Enough）</span>

- 加密只能防止别人读明文；对手仍然可以修改消息：翻转明文中的一个比特，引诱你接受垃圾数据。
- Integrity（完整性）：保护消息不被篡改与伪造。例如：
  - 那条 routing vector（路由向量）确实是该节点自己通告的；
  - 那个 bid（出价）确实是那个人出的；
  - 那条终止连接的消息确实是端点（endpoint）自己发的。
- <span style="color:#00838f;">有机密性而无完整性很少见（而且通常是糟糕设计的标志）；有完整性而无机密性则很常见。</span>
- 例外：encrypted storage（加密存储）——不在网络上、没有中间人（MitM）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 为什么"加密了还会被篡改"？看 8-7 讲的 CTR 模式：$C_i = M_i \oplus S_i$，密钥流 $S_i$ 虽然对攻击者未知，但异或的结构意味着翻转 $C_i$ 的任意一位，解密后明文的对应位必然翻转——攻击者不需要知道 $S_i$ 就能精确控制改动位置，这种性质叫 malleability（可延展性）。CBC 下翻转密文块 $C_{i-1}$ 的一位会让 $M_{i-1}$ 整块乱掉，但 $M_i$ 的对应位仍以可预测方式翻转。课程给的三个例子都很"贵"：路由向量被改可以劫持流量（呼应 8-3 讲），出价被改是直接的经济损失，伪造连接终止消息则是会话劫持——它们共同说明完整性与机密性是正交的目标。工程推论：任何"解密成功就照单全收"的协议，都默认了"密文不可篡改"这个错误假设。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 3 · 两个完整性工具（Two Integrity Examples）</span>

- **Cryptographic hashes（密码学哈希）**：验证数据未被修改的手段；不需要任何秘密——任何人都能计算；适用于数据存储场景。
- **Message authentication codes（消息认证码，MACs）**：同样验证数据未被修改；还额外验证生成者持有密钥——这提供了 authenticity（真实性）；适用于网络场景。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 4 · 密码学哈希（Cryptographic Hash）</span>

- Hash（哈希）：从任意长度输入计算出固定长度输出；典型长度 160–512 位；计算非常便宜，比网络还快。
- Cryptographic hash 还要求 collision-resistant（抗碰撞）：找到 $x \ne y$ 使 $H(x) = H(y)$ 是难解问题。
  - 当然，这样的碰撞大量存在：$2^{(2^{30} - 256)}$ 个 GB 级数据块共享同一个 256 位哈希；
  - 但多年来分析该算法的人没有一个真正找到一个。
- 今天使用 SHA-256 或 SHA-512（SHA-2 家族）；历史上最流行的哈希 SHA-1 已接近被攻破；
- 2012 年 10 月 2 日：NIST 选定 Keccak 算法作为 SHA-3——目标是一个与 SHA-2 不同的替代哈希函数。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 5 · 密码学哈希（重复页 + 设计者注记）</span>

与 Slide 4 相同，补充两处署名：SHA-2 由 NSA（美国国家安全局）设计；SHA-3 的 Keccak 由 Bertoni 等人设计（公开竞赛的获胜者）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "$2^{(2^{30}-256)}$ 个 GB 级数据块共享同一个 256 位哈希"正是 6.042J 鸽巢原理（pigeonhole principle）的直接推论：无限多的可能输入挤进 $2^{256}$ 个输出格子，碰撞必然存在——问题从来不是"有没有碰撞"，而是"多快能找到一个"。生日问题给出答案：随机取 $n$ 个输入，出现碰撞的概率约 $n^2/2^{257}$（Bertsekas 讲过的生日悖论推广），要概率过半需 $n \approx 2^{128}$——所以 256 位哈希的抗碰撞强度是 128 位，而不是直觉上的 256 位。这是密码学里"平方根攻击代价"最基本的例子：$b$ 位哈希的碰撞攻击代价是 $2^{b/2}$。SHA-1（160 位，理论碰撞代价 $2^{80}$）在 2017 年被 Google 与 CWI 的 SHAttered 攻击实际算出碰撞——生日界只是被提前兑现了。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 6 · 使用密码学哈希（Using a Cryptographic Hash）</span>

- 小哈希值唯一地描述大数据：
  - 对文件求哈希，记住哈希值 $h_1$；
  - 之后对同一文件再求 $h_2$：若 $h_1 = h_2$，文件未被篡改；
  - 软件发布时通常随附哈希值供校验。
- Hash tree（哈希树，Merkle hash tree，默克尔哈希树）让你用对数级的步数校验大数据中的一小片：叶子是各数据块的哈希，逐层两两向上再哈希到根；图中给出了两层与三层的树结构示例（hash hash 与 data data 的逐层聚合）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 7 · HMAC（初版）</span>

- 任何人都能计算密码学哈希——但 MAC 还想保证"生成者持有共享密钥"。
- 朴素做法：由哈希与密钥生成 MAC：$\mathrm{HMAC}(K, M) = \mathrm{SHA\text{-}2}(K, M)$（把密钥拼在消息前面再哈希）；
- 发送 $\{M, \mathrm{HMAC}(K,M)\}$；
- 看起来很简单：我们就得到了一个密码学意义上强壮的 MAC！

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 8 · 错了！（WRONG）</span>

上面的朴素构造是错的。原片展示了攻击的碎片化过程：如果我们掌握了 $\{M, \mathrm{HMAC}(K,M)\}$，攻击者无需知道 $K$ 就能生成合法的 $\{M', \mathrm{HMAC}(K,M')\}$，其中 $M'$ 是在 $M$ 后面追加数据得到的消息（$M' = M \| \text{填充} \| \text{追加数据}$）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这个攻击叫 length extension attack（长度扩展攻击），根源在 SHA-2 采用的 Merkle–Damgård 结构：哈希就是把消息逐块喂进压缩函数，而输出的哈希值恰好是处理完最后一个块时的内部状态本身。于是知道 $H(K \| M)$ 的人可以把它当作"中间状态"直接续上：先补齐 $K\|M$ 的填充，再继续吸收攻击者自己的数据块，算出 $H(K \| M \| \text{pad} \| X)$——全程不需要 $K$，因为密钥只在开头被吸收过一次。HMAC 的修复（下一页）是双层结构：内层 $\mathrm{H}(K \oplus \mathrm{ipad} \| M)$ 的输出作为完整消息进入外层，外层开头再混入 $K \oplus \mathrm{opad}$——两层使用不同的常数掩码，内层的中间状态被"埋"在两次哈希之间，无法被拼接续算。教训：组合密码学原语时，组合方式本身可能引入全新的漏洞。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 9 · HMAC 修正版（HMAC, Revisited）</span>

- $\mathrm{HMAC}(K, M) = H(K \oplus \mathrm{opad},\ H(K \oplus \mathrm{ipad}, M))$
  - $H$ 是诸如 SHA-3 的密码学哈希；
  - ipad 是 0x36 重复 64 次，opad 是 0x5c 重复 64 次。
- 为什么之前的攻击失效了？图中给出嵌套结构：内层哈希 $H(K \oplus \mathrm{ipad}, M)$ 的输出作为外层的完整输入，外层再与 $K \oplus \mathrm{opad}$ 一起进入哈希——中间状态被两层计算隔开，攻击者无法从最终值续算。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 10 · 加密与 MAC 的顺序（Encryption and MACs）</span>

- 应该加密 MAC，还是对加密后的数据做 MAC？
  - 先 MAC 再加密（MAC-then-encrypt）：对 $M$ 求 MAC 后一起加密；
  - 或先加密再做 MAC（encrypt-then-MAC）：对密文求 MAC。
- <span style="color:#00838f;">"加密 MAC"并不总是安全；"对加密数据做 MAC"则总是安全。</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这条结论在密码学文献里有正式名字：encrypt-then-MAC（EtM）。Bellare 与 Namprempre（2000）证明：先加密再对密文做 MAC，组合出的方案同时满足机密性与密文完整性；反过来（MAC-then-encrypt）则有真实的攻击面。最著名的是 TLS 1.0–1.2 采用的 MAC-then-encrypt 与 CBC 的组合：CBC 需要把消息 padding（填充）到块边界，服务器解密时会区分"填充非法"与"MAC 校验失败"两类错误——攻击者构造密文探测填充是否合法（padding oracle，填充预言机），可逐字节恢复明文；2013 年的 Lucky13 攻击表明，即使把错误信息统一掉，仍能通过时间侧信道完成同样的恢复。TLS 1.3 直接改用 EtM（AEAD，如 AES-GCM），把这个设计决定固化进协议。结合上一页的结论：先验证完整性再解密，"解密成功即信任"的隐患被连根拔掉。
