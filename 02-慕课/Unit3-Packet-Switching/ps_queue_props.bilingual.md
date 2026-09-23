# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Packet Switching: Useful queue properties（分组交换：有用的队列性质）

> <span style="color:#7f8c8d;">Source: 慕课/Unit3-Packet-Switching/ps_queue_props.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

> <span style="color:#7f8c8d;">提取说明：源 PDF 由 pdftotext 提取后存在 "ti"→"8" 系统性乱码（如 Burs8ness = burstiness、Sta8s8cal = statistical、LiPle/Li>le = Little、proper8es = properties），以及第 14 页泊松公式碎片化；以下译文已按原意复原。</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1：标题页</span>

CS144: An Introduction to Computer Networks
Packet Switching（分组交换）
Useful queue properties（有用的队列性质）
Nick McKeown，斯坦福大学电气工程与计算机科学教授
CS144, Stanford University

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 2：随机到达过程的队列</span>

通常，到达过程是很复杂的，所以我们常常用随机过程（random process）来建模它们。<span style="color:#00838f;">研究具有随机到达过程的队列的学问，称为排队论（queueing theory）。</span>具有随机到达过程的队列有一些有趣的性质。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 3：大纲</span>

1. 突发性增加时延（burstiness increases delay）；
2. 确定性最小化时延（determinism minimizes delay）；
3. 利特尔结果（Little's Result）；
4. M/M/1 队列。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 4~5：队列的时间演化</span>

分组到达与离开；$Q(t)$ 随时间变化。FIFO 路由器队列模型：累积到达 $A(t)$、队列占用 $Q(t)$、累积离开 $D(t)$（图示：分组沿时间轴到达、进入队列、以链路速率离开，基于图示推断）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 6~8：队列性质 #1——突发性增加时延</span>

<span style="color:#c0392b;">**队列性质 #1："突发性增加时延"**</span>

- 例 1：周期性的单个到达（periodic single arrivals）——分组以规则间隔逐个到达，队列几乎不积压（图示推断）；
- 例 2：周期性的突发到达（periodic burst arrivals）——同样的长期平均速率，但分组成簇到达，队列反复堆积排空（图示推断）。

一般地，<span style="color:#00838f;">突发性增加时延。</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 9~10：队列性质 #2——确定性最小化时延</span>

<span style="color:#c0392b;">**队列性质 #2："确定性最小化时延"**</span>

- 例 3：随机到达（random arrivals）——同样的平均速率，到达时刻随机（图示推断）。

一般地，<span style="color:#00838f;">确定性最小化时延</span>。也就是说：随机到达平均而言比简单周期到达等得更久。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 性质 #1 与 #2 是一体的两面，3-5a 的比特列车例题已经给你算过具体数字：确定性等间隔到达时平均占用 5 bit，把两列车挤到一起（增加突发性）就翻倍到 10 bit；反过来，给定平均速率，越"规律"的到达序列平均时延越小。为什么？直觉上，到达间隔的方差让队列在"簇"里白白排队——固定均值下，方差越大、尾部的极端积压越频繁，而排队时延对积压是凸函数，坏情况被放大。这个原理有两个极为实用的推论：其一，**整形有价值**——把突发流在入口处匀速化（漏桶/令牌桶），就能降低自己在下游的排队时延，这正是 3-10 时延保证的出发点；其二，**调度有上限**——服务器端无论怎么聪明地排序，都消不掉到达端带来的突发，所以"把流量变规律"要尽量在源头做。你在随机过程课里将来会看到更精确的版本：对 GI/GI/1 队列，平均时延随到达间隔变异系数 $C_a^2$ 增长（Kingman 公式 $W\approx\frac{\rho}{1-\rho}\cdot\frac{C_a^2+C_s^2}{2}\cdot\frac{1}{\mu}$ 的定性内容就是这两条性质）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 11~13：队列性质 #3——利特尔结果</span>

<span style="color:#c0392b;">**队列性质 #3："利特尔结果（Little's Result）"**</span>

（标题页与过渡页；结论在下文总结中给出。）<span style="color:#00838f;">$L = \lambda\, d$：系统中平均顾客数 = 平均到达率 × 平均每位顾客的时延。</span>

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> Little's law（利特尔法则，J.D.C. Little 1961 年证明，故称 Result 而非 Law 亦可）是排队论里最令人惊叹的普适结论：$L=\lambda W$，平均队列长度等于到达率乘平均逗留时间——它对**任意**到达分布、任意服务分布、任意排队纪律都成立，唯一的隐藏前提是系统达到稳态（这意味着它连接的是"期望/稳态"层面的量，与你概率论里极限定理的地位相当）。3-5a 的验算你已经做过：$\lambda=100\ \text{bit/s}$、$W=0.05\ \text{s}$，则 $L=5\ \text{bit}$。它在你已学的 CSAPP 里也有一个孪生兄弟：流水线 CPU 的"在途指令数 = 发射率 × 每条指令时延"（第 4 章分析吞吐率与时延权衡时的关系式），以及第 11 章"并发服务器同时服务的连接数 = 连接到达率 × 每连接服务时长"的容量估算——这三个式子是同一条定律。工程上它常被反过来用：测得 $L$ 和 $\lambda$ 推 $W$，或给定时延目标 $W$ 反推系统容量。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 14：泊松过程</span>

一个到达过程是**泊松过程（Poisson process）**，如果：

1. 在 $t$ 秒的区间内到达 $k$ 个的概率为

```math
P_k(t) \;=\; \frac{(\lambda t)^k}{k!}\,e^{-\lambda t}
```

2. 长度为 $t$ 的区间内到达数的期望 $E[\text{区间 } t \text{ 内的到达数}] = \lambda t$；
3. 相继的到达间隔相互独立（即不突发）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 15：为什么用泊松过程？</span>

它为"许多独立随机事件的聚合"建模，例如：

- 电话交换机上新呼叫的到达；
- 许多独立核粒子的衰变；
- 电路中的"散粒噪声（shot noise）"。

它让数学变容易。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这三条定义你应该感到亲切——它们正是 Bertsekas 概率论教材泊松过程一章的标准刻画：条件 1 的 $P_k(t)$ 就是泊松分布的 PMF（在长度 $t$ 的窗口内、速率为 $\lambda$），条件 2 说窗口内计数期望 $\lambda t$（均值与窗口长度成正比），条件 3 的独立间隔等价于你学过的"独立增量"，而由此自动得到指数分布的到达间隔与无记忆性。条件 3 的括号注释"即不突发"把性质 #1 接了回来：泊松到达的间隔是纯随机的、无后效的，比周期到达更突发。至于"为什么到处都是泊松"——答案是极限定理：大量独立的稀有事件（每人偶发的呼叫、每颗原子的衰变）叠加后，计数过程收敛到泊松（这是你学过的泊松定理/特征函数论证的推广），所以"聚合创造泊松"。排队论本身则诞生于 1909 年：丹麦哥本哈根电话公司的 A.K. Erlang 为计算电话交换机需要多少线路建立了第一个排队模型——本页第一条例子就是它 115 年前的原版。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 16：警告</span>

请注意：

1. 网络流量非常具有突发性！
2. 分组到达不是泊松的。
3. 但它能相当好地建模**新流（new flow）的到达**。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "分组到达不是泊松的"这句话背后有一篇里程碑式的测量研究：Leland 等人 1994 年对以太网流量的测量发现，聚合分组流量在很宽的时间尺度上都呈现自相似（self-similar）与长程相关——突发嵌套突发，放大时间轴后长得还像自己；其根源是文件大小的重尾分布（大文件少数但贡献巨大流量）。这与泊松假设（独立、无后效）直接冲突，也宣判了"用 M/M/1 精确算分组时延"的死刑。但注意第 3 条的精妙分工：**新流的到达**（一个用户发起一次网页/视频会话）确实近似泊松——因为用户行为彼此独立、各自稀有，聚合正是泊松的诞生条件；而流**内部**的分组到达是相关的、突发的。所以正确的分层用法是：流级建模用泊松（M/M/1 算并发流数、服务器负载），分组级建模用确定性图或测量。这种"哪一层用什么模型"的判断力，比记住任何单个公式都值钱。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 17：大纲回顾</span>

1. 突发性增加时延；2. 确定性最小化时延；3. Little's Result；4. M/M/1 队列。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 18：M/M/1 队列</span>

M/M/1 队列（图示）：到达过程为泊松（Poisson）到达，服务时间为指数（Exponential）分布，单个服务窗口（图示推断：分组到达队列、以单服务器逐个服务后离开）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 19：图示</span>

本页为图示 slide（文字极少），展示 M/M/1 队列的到达—服务—离开示意。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> M/M/1 的命名来自 Kendall 记号 A/B/C：A 描述到达间隔分布、B 描述服务时间分布、C 是服务器个数；字母 M 表示无记忆性（Memoryless），即到达是泊松过程、服务时间是指数分布。这两个 M 的组合让整个系统成为一个马尔可夫链（你已学过），状态是队列里的分组数，于是稳态分布可以用生灭过程闭式解出。补充两个标准结论供你验算直觉：令 $\rho=\lambda/\mu$（利用率），系统中平均分组数 $L=\dfrac{\rho}{1-\rho}$，平均逗留时延 $W=\dfrac{1}{\mu-\lambda}$。最有教育意义的是 $\rho\to 1$ 的行为：时延在利用率逼近 1 时爆炸（$\rho=0.5$ 时 $L=1$，$\rho=0.9$ 时 $L=9$，$\rho=0.99$ 时 $L=99$）——这就是为什么工程师把服务/链路利用率红线画在 70%~80%，为什么"服务器 CPU 90% 就开始报警"。确定性三角形告诉你排队怎么长，M/M/1 告诉你平均长多快，两者合起来就是排队论的全部入门装备。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 20：总结</span>

队列性质：

- 突发性增加时延；
- 利特尔结果：$L = \lambda d$。

分组到达不是泊松的……但有些事件是，例如网页请求和新流的到达。

M/M/1 队列是一个简单的队列模型。

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 3-6《Queueing Model Properties》的配套 slides；确定性模型的例题见 3-5a 与 ps_delay_model，M/M/1 的假设在真实流量中的适用边界见第 16 页与译者注。</span>
