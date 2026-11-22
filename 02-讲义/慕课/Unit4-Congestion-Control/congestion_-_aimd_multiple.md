# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Congestion — AIMD with multiple flows（拥塞——多条流的 AIMD）

> <span style="color:#7f8c8d;">Source: 慕课/Unit4-Congestion-Control/congestion_-_aimd_multiple.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1：封面</span>

CS144：计算机网络导论。**Congestion（拥塞）：AIMD with multiple flows（多条流的 AIMD）**。Nick McKeown，斯坦福大学电气工程与计算机科学教授。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 2：拓扑（多条流过一台路由器）</span>

图示：多条流从左端（链路速率高于 $C$）汇入一台路由器，路由器有一个缓冲区（Router buffer，路由器缓冲区），输出链路速率恰为 $C$——这是本讲的实验台：所有流共享同一条瓶颈链路与同一个缓冲区。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 3：One flow vs multiple flows（单流与多流的对照）</span>

- **单流（One flow，单流）**：窗口 $W(t)$ 在锯齿之间时，发送速率 $R = \dfrac{W(t)}{RTT}$ 近似恒定；缓冲区占用与 $RTT$ 的曲线平滑、可预测。
- **多流（Multiple flows，多流）**：各流窗口 $W_i(t)$ 相位错开，缓冲区占用曲线变成多条锯齿的叠加；总速率 $R = \dfrac{\sum_i W_i(t)}{RTT}$ 的波动在求和中被抹平。

图示：(Zoom in) 放大后的各流 cwnd 锯齿与缓冲区占用、$RTT$ 的抖动曲线。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这页的对照图藏着大数定律的一个漂亮应用。单条流的锯齿尖锐刺眼——链路利用率在谷底与峰顶之间摇摆；但把几十条相位各异的锯齿叠加，波峰对波谷互相抵消，总速率反而平滑地贴住容量 $C$。这正是概率论里独立随机波动求和后方差趋于抵消的机制：各流的丢包时刻近似独立，$\sum_i W_i(t)$ 的相对波动随流数增多而缩小。对网络运营商来说这是天大的好消息——骨干链路汇聚的流越多，利用率越能稳定地逼近 1，而单用户感受到的速率波动也越小。不过别急着推广：这个平滑效应依赖"各流相位不同步"的假设，下一页之后会看到，当各流耦合在同一缓冲区上时，同步化反而可能发生。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 4：Simple geometric intuition（简单的几何直觉：吞吐量公式）</span>

从锯齿的几何形状直接推出 AIMD 的吞吐量。设窗口峰值为 $W_{max}$（以报文段为单位），每周期丢一包：

```math
\text{丢包率 } p = \frac{1}{A}, \qquad A = \frac{3}{8}\,W_{max}^2 \;(\text{每个锯齿周期发送的分组数})
```

```math
R = \frac{A}{\frac{W_{max}}{2}\cdot RTT} = \frac{3\,W_{max}}{4\,RTT} = \frac{\sqrt{3/2}}{RTT\,\sqrt{p}}
```

> <span style="color:#7f8c8d;">（说明：源 PDF 的数学符号在文本提取时碎裂为乱码，上式按原 slide 公式与 4-bookends 中 Phil 的口头推导"吞吐量 = $\sqrt{3/2}$ 乘以 $1/(RTT\sqrt{p})$"重构，即经典的 Mathis 公式。）</span>

🎥 *CS144 教授在视频中解释: "当网络空无一人，未拥堵时，它有空间让流量发送更多的包，以便有更多的未确认包在网络中"*（字幕来源：057 - p56 4-4 AIMD Multiple Flows.zh.srt）

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这个公式的推导本质是一次**期望计算**，值得用概率论的语言复述一遍：一个锯齿周期里，窗口从 $W_{max}/2$ 每轮加 1 爬到 $W_{max}$，累计发送的分组数是一个三角形面积 $\frac{1}{2}\cdot\frac{W_{max}}{2}\cdot W_{max} = \frac{3}{8}W_{max}^2$；每周期恰好丢 1 包，所以"平均每发 $A$ 个包丢 1 个"即 $p = 1/A$——这是把随机丢包过程按频率解释为概率的标准做法。再除以周期时长 $\frac{W_{max}}{2}\cdot RTT$（爬坡所需轮数乘以每轮一个 RTT），就得到长期平均吞吐量。整个推导只用了几何与期望，却给出了一条统治互联网二十余年的定律：吞吐量 $\propto \dfrac{1}{RTT\sqrt{p}}$。CLRS 里你分析算法期望运行时间用的是同一套手法——把随机过程按"每个阶段发生什么、概率多少"摊开求和。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 5：Interpreting the rate equation（解读速率公式）</span>

```math
R = \frac{1}{RTT}\sqrt{\frac{3}{2p}}
```

1. $RTT \to 0$ 时 $R \to \infty$？—— 是的，公式如此：RTT 越短，每个 RTT 加 1 的爬升越频繁，同样丢包率下吞吐越高；
2. $p \to 0$ 时 $R \to \infty$？—— 也是：丢包越少，锯齿爬得越高，平均窗口越大。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这两个"无穷大"都是模型边界的警报，而不是物理事实。$p \to 0$ 时 $R \to \infty$ 的真实含义是：没有丢包信号，AIMD 就没有任何刹车，窗口会一路爬升直到制造出丢包为止——所以公式的适用前提是"网络最终会把窗口压回锯齿轨道"，脱离这个前提谈极限没有意义。$RTT \to 0$ 时 $R \to \infty$ 则暴露了更微妙的问题：公式里窗口每 RTT 只加 1 个报文段，RTT 越小爬得越快，这直接导致数据中心等低 RTT 环境里 TCP 疯狂快速地逼近容量、振荡剧烈。任何数学模型都有假设域，读出公式极限行为背后的假设，比记住公式本身更重要——这也是你在 6.042J 里分析递归式时"先确认初值与边界条件"习惯的迁移。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 6：Observations for multiple flows（多流情形的观察）</span>

1. 窗口按 AIMD 规律扩张与收缩；
2. ……以此探测"管道"能装下多少字节；
3. 瓶颈缓冲区里将同时装有**许多条流**的分组；
4. 发送速率随窗口大小而变化；
5. AIMD 对丢包率（loss rate，丢包率）**极其敏感**（吞吐 $\propto 1/\sqrt{p}$）；
6. AIMD **亏待长 RTT 的流**。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 观察 6 是 AIMD 公平性理论与现实之间最大的裂缝，值得展开。吞吐公式 $R \propto \dfrac{1}{RTT\sqrt{p}}$ 说明：丢包率相同时，RTT 大一倍的流吞吐减半——比如旧金山到波士顿（约 70 ms）与旧金山到悉尼（约 200 ms）的两条流抢同一条瓶颈，后者只能拿到前者的三分之一左右。原因很直白：加性增的速度是"每 RTT 加 1"，RTT 长的流爬坡慢、被砍后再爬回也慢，而丢包是全体流共享的。这就是前面 Chiu-Jain 收敛证明中"同步丢包"假设的代价：现实中丢包不同步且偏向打击窗口大的流，于是公平退化为"RTT 平权"。观察 5 同理：吞吐对 $p$ 的平方根依赖，意味着丢包率翻十倍吞吐只缩三倍多——这让 TCP 惊人地顽固，但也意味着在无线网络把链路层误码当成拥塞丢包时，TCP 会无谓地自砍吞吐。一个极简规则在理想条件下收敛、在偏离假设时系统性地偏心——机制与目标的关系，在这里再次得到展示。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 7：结束页</span>

本讲完。CS144, Stanford University。

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 4-4《AIMD Multiple Flows》的配套 slides；吞吐量公式的完整推导见 Mathis, Ott, Semke, Mahdavi, "The Macroscopic Behavior of the TCP Congestion Avoidance Algorithm", 1997。</span>
