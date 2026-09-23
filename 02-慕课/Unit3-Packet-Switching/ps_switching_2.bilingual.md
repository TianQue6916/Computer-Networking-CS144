# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Packet Switching: How a packet switch works (2)（分组交换：分组交换机如何工作（2））

> <span style="color:#7f8c8d;">Source: 慕课/Unit3-Packet-Switching/ps_switching_2.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

> <span style="color:#7f8c8d;">提取说明：源 PDF 由 pdftotext 提取后存在 "ti"→"8" 系统性乱码（如 Proper8es = properties、oZen = often、Loa6d0% = Load），第 10 页负载轴数字碎片化（0%/20%/40%/58%/60%/80%/100%），以下译文已按原意复原。</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1：标题页</span>

CS144: An Introduction to Computer Networks
Packet Switching（分组交换）
How a packet switch works (2)（分组交换机如何工作（2））
Nick McKeown，斯坦福大学电气工程与计算机科学教授
CS144, Stanford University

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 2：大纲</span>

把分组交换到出端口（egress port）：

- 输出排队（output queueing）与共享内存（shared memory）；
- 输入排队（input queueing）与队头阻塞（head-of-line blocking）；
- 虚拟输出队列（virtual output queues）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 3~4：输出排队分组交换机</span>

输出排队（Output Queued）分组交换机：分组在每个输入端完成查找地址、更新首部后，立即被**交换（switch）**到它要去的输出端口，并在**输出端口处**的队列里排队等待链路空闲（多帧动画展示分组穿过交换结构、汇集到输出队列，基于图示推断）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 5~6：输入排队分组交换机</span>

输入排队（Input Queued）分组交换机：分组在**输入端口处**先行排队（查找地址、更新首部之后），仅当轮到自己、且交换结构与目标输出可用时，才被交换到输出端口（多帧动画展示输入队列中的分组依次获得交换机会，基于图示推断）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 7：队头阻塞</span>

队头阻塞（Head of Line Blocking，HOL blocking）（图示 slide）：输入队列最前面的分组若其目标输出正忙，即使后面分组的目标输出空闲，它们也**不得越过队头**，只能干等（图示推断）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 8：虚拟输出队列</span>

虚拟输出队列（Virtual Output Queues，VOQs）（图示 slide）：在每个输入端为**每个可能的输出**各设一条独立队列，使被队头卡住的分组不再连累后面的分组（图示推断）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 9：输出排队是排队系统的天花板</span>

输出排队分组交换机：<span style="color:#00838f;">任何排队系统所能达到的最佳表现。</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 10：输出排队交换机的性质</span>

1. 它们是"工作保守"（work conserving）的：只要还有分组在排队，输出链路就不闲着。
2. 吞吐量最大化。
3. 期望时延最小化。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "输出排队是任何排队系统所能达到的最佳"——为什么？可以用两个角度合起来理解。工程角度：输出排队把"何时排队"的决策推迟到分组已经站在目标输出门口，分组永远只在真正是瓶颈的那个队列里等待，不存在"人在队里心已飞走"的浪费；代价是输出端缓冲存储器必须能以全部输入速率之和（$N\times R$）写入——$N$ 个输入可能同时把分组砸向同一个输出，这在高速端口上是极昂贵的内存带宽需求，类比 CSAPP 存储器层次：就像要求 L1 缓存带宽等于所有核的需求总和，做得到但太贵。排队论角度：work conserving（工作保守，即服务器只要还有活就不休息——这是 ps_queue_props 里排队论的标准术语）加上"每个输出独立 FIFO"，已把每个输出端口的平均时延做到该到达过程下的下界。于是结论很干净：OQ 是性能天花板，但硬件昂贵；IQ 便宜（所有内存在输入端、只需 $R$ 速率写入），但有队头阻塞——下一页给出 IQ 到底损失多少。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 11：输入排队、队头阻塞与 VOQ 的对比</span>

输入排队分组交换机——队头阻塞与 OQ 交换机对比；输入排队 + 虚拟输出队列（VOQs）与 OQ 交换机对比。图示为"时延—负载（load）"曲线：横轴负载 0%、20%、40%、60%、80%、100%，纯输入排队（FIFO）的时延曲线在约 **58%** 负载处开始爆炸；加入 VOQs 后曲线与 OQ 交换机几乎重合（图示推断）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 图上那个 58% 是排队论史上的著名数字，值得给你一段可复述的推导直觉：Karol、Hluchovsky 与 Tobagi 在 1987 年证明，$N\times N$ 的 FIFO 输入排队交换机在均匀随机流量下最大稳定吞吐量是 $2-\sqrt{2}\approx 58.6\%$。原因在队头：每个输入队列同一时刻只有队头分组能去竞争交换，队头被卡则全队瘫痪。粗算：均匀流量下，某输入队头锁定输出 $o$；它被卡住的概率取决于其他 $N-1$ 个输入的队头中恰好也有指向 $o$ 的……精确分析给出稳定域上界 $2-\sqrt{2}$。这组数字的工程含义写在图上：裸输入排队买不到 6 成以上的容量，必须靠 VOQs——在每个输入端为 $N$ 个输出各备一条队列，队头阻塞从此只在"该输出自身拥塞"时发生。代价是调度器每时隙要在交换结构上求解一个**二分图匹配**问题（6.042J 的图论语言：以"输入—输出"为二部、以"有分组待传"为边，找无公共端点的最大匹配）；iSLIP 等迭代算法就是 McKeown 实验室的代表作。你能在这里看到图论、排队论、体系结构在同一个问题上会师。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 12：几句结语</span>

A few last words...（几句结语）配图出自 National Complete Streets Coalition（美国"完整街道"联盟）——用道路交通的照片类比道路/车道的规划与排队（幽默收尾页，基于图示推断）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 13：总结</span>

分组交换机执行两种基本操作：

- 在转发表中查找地址；
- 交换到正确的出端口。

最简单也最慢的交换机使用输出排队，它使分组时延最小。高性能交换机常使用输入排队，并用虚拟输出队列来最大化吞吐量。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 把两讲合起来的最终图景：一台交换机 = 查表（ps_switching）+ 交换 + 排队（本讲），而"队列放在哪里"是性能与成本的枢纽——输出排队最优但需要 $N\times R$ 写入带宽，输入排队廉价但裸用只有约 58.6% 吞吐，输入排队 + VOQs + 聪明调度（二分图匹配）则鱼与熊掌兼得。这个"性能—成本—复杂度"三难是计算机系统反复出现的母题：CPU 乱序执行窗口、GPU 显存层次、数据库缓冲池，都在解同构问题。还有一个值得记的细节：本讲的输出/输入排队讨论默认"分组先查表再排队"，于是队列位置决定了你在**哪一层**排队（3-9 的速率保证、3-10 的时延保证都实现在这个队列上）——理论保证能否兑现，最终取决于交换机内部这些毫厘之间的硬件选择。产品里从低端家用交换机到数据中心核心交换机的价差，就是这三难取舍的标价。

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 3-8《Packet Switching - Practice: Switching and Forwarding (2)》的配套 slides；查表机制见 ps_switching，队列上实现的调度与时延保证见 3-9/3-10。</span>
