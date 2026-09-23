> <span style="color:#7f8c8d;">Source: src/慕课/Unit8-Security/8-10-certificates.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> <span style="color:#c0392b">Certificates（证书）</span>

> <span style="color:#7f8c8d;">副标题：Establishing a chain of trust（建立信任链）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 2 · 问题（Problem）</span>

想与一台服务器（如 www.amazon.com）安全通信：

- 用公钥密码学，给定公钥 $K$，我们能验证另一个程序确实持有对应的私钥 $K^{-1}$：$V(K, \{m\}_{K^{-1}}, m) \to \{\text{yes}, \text{no}\}$；
- 再用公钥密码学交换对称密钥。

真正的麻烦是 key management（密钥管理）：

- 我们怎么拿到服务器的公钥？
- 我们凭什么确定那就是服务器的公钥？

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 3 · 攻击：中间人（Attack: Man in the Middle）</span>

图示：client（客户端）与 server（服务器）之间的路径上站着 attacker（攻击者）。

- 攻击者冒充服务器，把自己的公钥交给客户端；
- 发动 man-in-the-middle（中间人）攻击：
  - 在客户端看来他与服务器毫无二致（除了公钥不同）；
  - 攻击者看到敏感流量、再加密（re-encrypt）后转发；
  - 攻击者还可以插入新流量。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这一页点破了一个第一性问题：密码学验证的是"某实体持有 $K^{-1}$"，而非"该实体就是 amazon"。公钥只是一串数字，它与身份之间没有任何天然绑定——$V(K, \{m\}_{K^{-1}}, m)$ 通过，只说明对面有人握有与 $K$ 配对的私钥；这个人是 Amazon、攻击者还是你的室友，数学管不着。用 6.042J 的图论视角看中间人：攻击者把自己插到 client–server 之间的每条通信路径上，成为图的割点（cut vertex）——客户端与服务器各自只与"邻居"（攻击者）建立一条加密隧道，彼此从未直接对话，所以两边都毫无察觉。出路只有两条：要么存在攻击者无法触碰的带外信道（out-of-band，例如当面交换密钥），要么引入双方都信任的第三方作证——后者就是证书。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 4 · 证书（Certificate）</span>

<span style="color:#00838f;">Certificate（证书）：由某个私钥 $K_1^{-1}$ 签署的一份文档，把公钥 $K_2$ 与一个身份/名字 $N$ 绑定在一起。</span>

- "The public key of www.ebay.com is ...."（www.ebay.com 的公钥是……）；
- "The public key of axess.stanford.edu is..."（axess.stanford.edu 的公钥是……）。

如果我们信任签署方、并且知道他们的公钥 $K_1$，那么与 $N$ 通信时就可以放心使用 $K_2$。图中示例：axess.stanford.edu 的证书——公钥 $K_2$ 由 $K_1$ 签署。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 5 · 坏证书！（Bad Certificate!）</span>

纯图示页：浏览器遇到无效证书（域名不匹配、签名无法验证、已过期等）时弹出的红色警告页面——用户被明确告知连接不是私密的，需自行决定是否继续。这正是证书校验失败在工程上的呈现方式。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 6 · 信任的自举（Bootstrapping Trust）</span>

- "所有人"都信任少数几个签署机构（signing authorities），并预知它们的公钥：
  - 硬编码（hard-baked）在你的浏览器或操作系统里；
  - 例如：Verisign、Microsoft、Google。
- 根机构（root authorities）签署证书（比如给 Stanford 的）；
- 这些证书又可以用来签署更多证书："chain of trust（信任链）"——GeoTrust → Google → www.google.com。
- 证书只说明"有人作证某台主机持有这个密钥"！链条上的每一步都必须信任。
- 今天的 TLS/HTTPS 就是这样工作的（浏览器地址栏上的那把锁）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 信任链用 6.042J 的图论语言说，就是一张有向图上"从根到叶子"的路径：顶点是证书，边是"用上一张证书的公钥验证下一张的签名"，信任沿路径单向传递。这个设计的脆弱点同样在图上看得清楚：浏览器与操作系统预装上百个根 CA（Mozilla 根证书库常年在 150 个左右），而任何根都能给任何域名签证书、没有范围限制——"入边"太自由，意味着单点失陷就是全局失陷。2011 年的 DigiNotar 事件是惨痛注脚：这家荷兰 CA 被入侵后为 google.com 签发了伪造证书，中间人在伊朗借此拦截 Gmail 流量；事件曝光后各大浏览器吊销其根证书，公司当月破产。工业界的后续补救是 Certificate Transparency（证书透明度，2013 年起标准化）：所有签发的证书必须进入公开可审计的日志，域名所有者能发现"谁在替我签证书"——用公开性约束信任链上的每一步。
