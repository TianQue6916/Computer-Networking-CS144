> <span style="color:#7f8c8d;">Source: src/慕课/Unit8-Security/8-9-public.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> <span style="color:#c0392b">Public Key Cryptography（公钥密码学）</span>

> <span style="color:#7f8c8d;">副标题：Hidden in plain sight（藏于众目睽睽之下）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 2 · 机密性（Confidentiality）</span>

三个（随机化的）算法：

- Generate（生成）：$G(1^k) \to K, K^{-1}$（随机化）；
- Encrypt（加密）：$E(K, m) \to \{m\}_K$（随机化）；
- Decrypt（解密）：$D(K^{-1}, \{m\}_K) \to m$。

提供机密性：没有 $K^{-1}$ 就无法从 $\{m\}_K$ 得到 $m$。

- $K$ 可以公开：无法从 $K$ 推出 $K^{-1}$；
- 所有人都可以共享同一个 $K$（"public" key，公钥）。

加密必须是随机化的（randomized）：同一段明文多次加密必须产生不同的密文——否则对小消息空间（"yes"、"no"）可以直接猜明文。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 3 · 完整性：签名（Integrity: Signatures）</span>

三个（随机化的）算法：

- Generate（生成）：$G(1^k) \to K, K^{-1}$；
- Sign（签名）：$S(K^{-1}, m) \to \{m\}_{K^{-1}}$（可以随机化）；
- Verify（验证）：$V(K, \{m\}_{K^{-1}}, m) \to \{\text{yes}, \text{no}\}$。

像 MAC 一样提供完整性：没有 $K^{-1}$ 就造不出合法的 $(\{m\}_{K^{-1}}, m)$ 对；但验证只需要 $K$，且无法从 $K$ 推出 $K^{-1}$——所以 $K$ 可以公开。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 4 · 常用公钥算法（Popular Public Key Algorithms）</span>

- 加密：RSA、Rabin、ElGamal；签名：RSA、Rabin、ElGamal、Schnorr、DSA……
- 警告：消息填充（message padding）至关重要——RSA 加密背后的思想很简单（大整数的模幂运算，modular exponentiation），但把消息简单映射成数字是不安全的。
- 许多密钥同时支持签名与加密，但两者使用不同的算法。常见错误：用私钥"加密"来实现签名。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 5 · 实例：RSA（Example: RSA）</span>

生成私钥 $K^{-1}$ 与公钥 $K$：

1. 选两个不同的素数（prime numbers）$p$、$q$；
2. 计算 $n = pq$；
3. 由 $p$、$q$ 计算 $K$ 与 $K^{-1}$（推导对本课而言太复杂）。

把 $n$ 与 $K$ 作为公钥公开：

- 加密：$E(K, n, m) = m^K \bmod n$；
- 解密：$D(K^{-1}, n, \{m\}_K) = m^{K \cdot K^{-1}} \bmod n$。

可以把 $n$ 当明文发送：无法从 $n$ 推出 $p$ 和 $q$。如果有人找到快速分解大整数的办法，整座大厦轰然倒塌。该问题属于 NP，被认为不在 P 内，也被认为不是 NP-complete。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "对本课太复杂"的那一步，你在 6.042J 数论单元已经完整学过：选 $e$（即公钥指数 $K$）与 $\varphi(n) = (p-1)(q-1)$ 互素，用扩展欧几里得算法求 $d \equiv e^{-1} \pmod{\varphi(n)}$，$d$ 就是私钥 $K^{-1}$。解密能还原明文靠的是欧拉定理：当 $\gcd(m, n) = 1$ 时 $m^{\varphi(n)} \equiv 1 \pmod n$，于是 $m^{ed} = m^{1 + t\varphi(n)} \equiv m \pmod n$（$\gcd(m,n) \ne 1$ 的退化情形由中国剩余定理兜底）。安全性正是 6.042J 强调的模运算单向性：由 $n = pq$ 求 $\varphi(n)$ 等价于分解 $n$，而最好的通用分解算法（数域筛）在 2048 位的 $n$ 上仍需天文数字级操作。所以"选两个大素数"不是随口一说：素数要够大（各约 1024 位）且真随机——$p$、$q$ 选得太近，还会被 Fermat 分解法（基于 $(p+q)/2$ 与 $(p-q)/2$ 的平方差）快速夹出来。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 6 · 代价（The Catch）</span>

公钥算法的开销相当可观（数据来源 http://www.cryptopp.com/benchmarks.html）：

| 算法 | Encrypt（加密） | Decrypt（解密） | Sign（签名） | Verify（验证） |
|---|---|---|---|---|
| RSA 1024 | 0.08ms | 1.46ms | 1.48ms | 0.07ms |
| RSA 2048 | 0.16ms | 6.08ms | 6.05ms | 0.16ms |
| DSA 1024 | — | — | 0.45ms | 0.83ms |
| LUC 2048 | 0.18ms | 9.89ms | 9.92ms | 0.18ms |
| DLIES 2048 | 4.11ms | 3.86ms | — | — |

相比之下，symmetric algorithms（对称算法）可以跑满线速（line speed）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 公钥为什么慢 2–3 个数量级？看运算量：RSA-2048 的私钥运算是 2048 位数的模幂，用平方-乘法（repeated squaring，6.006 与 6.042J 都练过的技巧）需要约 $\log_2 d \approx 2048$ 次大数模乘，每次又是对 2048 位数（32 个机器字）的多精度乘法，代价 $O(k^2)$ 基本位操作。而公钥操作（加密/验签）的指数 $e$ 通常很小（如 65537，仅 17 个二进制位），所以 Encrypt 只要 0.08ms——表中私钥操作（Decrypt/Sign）比公钥操作慢约 40 倍，正对应指数长度之差。AES 这类对称密码只做字节替换、移位与异或，现代 CPU 还有 AES-NI 指令每周期处理一块数据，可跑满 10 Gbit/s 线速。这张表解释了下一页混合加密为何是必然选择：公钥只负责传一把 128 位的对称密钥，重活全部交给对称算法。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 7 · 混合方案（Hybrid Schemes）</span>

- 用公钥密码加密对称密钥——协商出秘密的 session key（会话密钥）：
  - 用公钥密码建立 4 把对称密钥；
  - 客户端发给服务器：$\{\{m_1\}_{K_1}, \mathrm{MAC}(K_2, \{m_1\}_{K_1})\}$；
  - 服务器发给客户端：$\{\{m_2\}_{K_3}, \mathrm{MAC}(K_4, \{m_2\}_{K_3})\}$。
- 常见陷阱：签署"规格不全"的消息（signing underspecified messages）：
  - 例如，签名的消息里永远要写明预期的接收者；
  - 还应写明过期时间，或者更好——绑定新鲜数据（fresh data）；
  - 否则就像签了一张"给任何人、永久有效"的空白支票……

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "空白支票"陷阱对应现实中很经典的一类攻击：replay（重放）。8-6 讲里"攻击者可以重放"就预告了这一点——签名只能证明"某人在某个时刻确实认可这段字节"，却不能证明"这个认可现在仍然有效"。解法就是在被签消息里绑定 fresh data（新鲜数据）：时间戳、过期时间或一次性随机数（nonce），使每条签名只在特定语境下有效。下一讲的 TLS 握手正是这么做的：客户端与服务器先交换各自的 random，握手参数的签名里混入这些随机值——旧的握手消息被重放时对不上新的随机数，攻击自动失效。你在 8-11 的测验里还会亲手算一遍：会话密钥的强度由"所有熵源里最弱的那个"决定。新鲜性与熵，是签名之外同等重要的两根支柱。
