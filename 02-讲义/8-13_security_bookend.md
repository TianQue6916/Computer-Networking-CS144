> <span style="color:#7f8c8d;">Source: src/慕课/Unit8-Security/8-13_security_bookend.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> <span style="color:#c0392b">Unit 8: Security（第 8 单元：安全）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 2 · 问题（The Problem）</span>

纯图示页：一幅魔鬼卡通形象（象征网络中的攻击者），来源：http://commons.wikimedia.org/wiki/File:Devil_cartoon_charactor.gif

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 3 · 本单元学到的：攻击（What you learned: Attacks）</span>

- Eavesdropping（窃听）、tampering（篡改）、suppression（抑制/丢弃）；
- Spoofing（欺骗/冒充）、man-in-the-middle（中间人）；
- 第 2、3、4 层上的 redirection/hijacking（重定向/劫持）；
- Denial of service（拒绝服务）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 4 · 本单元学到的：原则（What you learned: Principles）</span>

- Confidentiality（机密性）；
- Integrity（完整性）；
- Authenticity（真实性）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 5 · 本单元学到的：工具（What you learned: Tools）</span>

- Block ciphers（分组密码：ECB 模式、CBC 模式）；
- Cryptographic hashes（密码学哈希：collision resistance 抗碰撞性）；
- Message authentication codes（消息认证码）；
- Public key encryption/decryption（公钥加密/解密）；
- Signatures and certificates（签名与证书）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 把本单元收束成一张映射表，各讲就串成了一条线：机密性由对称分组密码（CBC 等模式）承担；完整性由哈希与 MAC 承担（HMAC 的双层结构说明"怎么组合"与"用什么原语"同样重要）；真实性由签名与证书承担——公钥密码解决密钥分发，证书把密钥与身份绑定；而 TLS 把三类工具组装成一个会话层协议，正是 8-6 讲"密码学路线"的终点。另一条路线（可扩展系统设计）由 DoS 一讲与高可用三件套（replication、成本对称、上游过滤）承担。两条路线的分工并非偶然：密码学保护"单条消息的字节"，系统设计保护"服务整体的可用性"——一个对付能读能改的对手，一个对付能量更大的对手。8-8 讲过"机密性缺完整性是糟糕设计"，其实对整个单元都成立：三条原则缺任何一条，另外两条都会被连带打穿。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 6 · 最重要的教训（Most Important Lesson）</span>

- 犯错很容易（It's easy to make a mistake）；
- 使用现有的开源实现（use existing, open source implementations）；
- 小心谨慎，遵循最佳实践（be careful and follow best practices）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "别自己发明密码学"有真实的教训可查。最著名的一例：2008 年 5 月，Debian/Ubuntu 的 OpenSSL 维护者为消除 valgrind 工具的警告，注释掉了两行看似"多余"的代码，结果进程随机性几乎归零——可预测的密钥空间骤降到 32768 种（CVE-2008-0166），全球所有用该版本生成的 SSH 与 SSL 密钥数小时内就能全部枚举，业界被迫批量吊销重签海量证书与主机密钥。更早的 1996 年，Goldberg 与 Wagner 证明 Netscape SSL 1.x 用当日时间加进程号做随机种子，随机性形同虚设，几分钟就能猜出会话密钥。这两例都不是"密码算法被攻破"，而是"密码算法被用错"——恰好印证 8-6 的警告：必须先读懂密码学保证什么、不保证什么，再动手。用久经审计的开源库并遵循最佳实践，本质上是在复用别人踩过的全部坑。
