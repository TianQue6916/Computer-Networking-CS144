# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

> <span style="color:#7f8c8d;">Source: src/慕课/Unit7-Lower-Layers/wireless-different.txt · CS144 Fall 2019 · 行内双语翻译版</span>

## <span style="color:#2471a3;">**[lecture]**</span> <span style="color:#c0392b">Wireless Networking（无线网络：Wireless Is Different 合集）</span>

原文 Slide 1 为标题页：**Wireless Networking**（无线网络）。本文件是该讲的完整合集，共 11 页：先看两张美国频谱分配图（理解无线首先要理解频谱管制），随后进入正题"Wireless Is Different（无线有所不同）"，再到信号强度、随时间变化的信道、斯坦福 SWAN 真实测试床、干扰，最后以总览收束。注：原文部分 slide 页脚编号在文本提取中丢失，小节编号按内容顺序与可见页码对齐。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 2 · Access Point Networks（接入点网络）</span>

标题页（图示：access point，接入点/AP 组成的基础设施网络）。这一页引入无线网络最常见的部署形态：终端主机（host）通过附近的接入点接入有线网络，接入点成为无线世界与有线世界的边界。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 3 · United States Frequency Allocations（美国频率分配图：全图）</span>

图示 slide：美国 FCC（Federal Communications Commission，联邦通信委员会）与 NTIA（National Telecommunications and Information Administration，国家电信与信息管理局）于 2003 年 10 月发布的 "The Radio Spectrum"（无线电频谱）海报，标题为 UNITED STATES FREQUENCY ALLOCATIONS（美国频率分配）。图中文字均为图例碎片，此处概述其要点：

- 频谱按业务（radio services，无线电业务）分配：FIXED（固定业务）、MOBILE（移动业务）、RADIOLOCATION（无线电定位/雷达）、RADIONAVIGATION（无线电导航）、BROADCASTING（广播）、AMATEUR（业余无线电）、RADIO ASTRONOMY（射电天文）、SPACE RESEARCH（空间研究）、METEOROLOGICAL AIDS（气象辅助）等数十种业务瓜分同一片频谱；
- 图例区分 government exclusive（政府专用）、non-government exclusive（非政府专用）、government/non-government shared（政府与非政府共用），以及 primary/secondary（主用/次要）地位；
- 频率轴覆盖 10 Hz 至 $10^{25}$ Hz（波长从 $3\times10^7$ m 到亚毫米量级），并标注 VLF/LF/MF/HF/VHF/UHF/SHF/EHF（甚低频至极高频）等波段代号与 INFRARED（红外）、VISIBLE（可见光）区域；
- 图上特别标出多处 ISM 频段（ISM = industrial, scientific, medical，工业/科学/医疗）：ISM 2450.0 ± 50 MHz（即 2.4 GHz）、ISM 5.8 ± 0.075 GHz、ISM 24.125 ± 0.125 GHz、ISM 245.0 ± 1 GHz、ISM 61.25 ± 0.25 GHz，并注明 "59–64 GHz 指定给 unlicensed devices（免执照设备）"。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 4 · 频谱分配图（放大视图）</span>

图示 slide：上一张频谱海报的 magnified（放大）视图——原图自注 "THE RADIO SPECTRUM 3 MAGNIFIED ABOVE"（第 3 页无线电频谱的放大）。图中文字仍是 3–30 GHz 等频段内 FIXED/MOBILE/RADIOLOCATION 等业务的密集排布碎片，此处不逐条转写。看图要点：频谱是稀缺公共资源，几乎每一赫兹都已有主，WiFi 能"免费"使用的只有那些专门留出的 ISM 缝隙。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这两页频谱图解释了一个根本问题：为什么 WiFi 恰好用 2.4 GHz 和 5 GHz？因为这些 ISM 频段不需要牌照——它们原本划给工业、科学、医疗设备（典型如工作在 2.45 GHz 的微波炉），通信只是"搭便车"。1985 年，FCC 开放 ISM 频段供免执照的扩频通信（spread spectrum）使用，这一纸行政决定直接催生了后来的 WiFi：没有频谱牌照费用，任何人都能造、都能用。但免执照也意味着"不受控"：任何人都可以在你隔壁发射同频信号，监管者只管功率上限，不管你收不收得到干扰——这就是后面反复出现的 uncontrolled medium（不受控介质）在制度层面的含义。对比一下：蜂窝运营商花几百亿美元拍下牌照，换来的是"本频段内只有我能发"的独占权；WiFi 用的是"谁都可用"的公共地，代价就是把干扰当常态来设计协议。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 5 · Wireless Is Different（无线有所不同）</span>

核心论点一页：

- <span style="color:#00838f;">无线传输介质不是一根导线（wire）；</span>
- 能量向空间辐射（radiates over space）：
  - 信号随距离衰减：按 $r^2$ 或更快；
  - 存在中间链路（intermediate links）；
- 介质不受控（uncontrolled medium）：
  - 信号强度随时间变化；
  - 来自其他发射机的干扰（interference）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> $r^2$ 可以从第一性原理推出来：发射功率 $P$ 均匀摊在半径 $r$ 的球面上，球面积 $4\pi r^2$，能流密度自然正比于 $P/r^2$——这就是 free-space path loss（自由空间路径损耗）的平方律，没有任何神秘之处，只是能量守恒。"或更快"也有实据：贴近地面的双径传播中，直射波与地面反射波相互抵消，衰减可恶化到 $r^4$ 量级。"中间链路"指的则是前几讲 A–B–C 链形拓扑的处境：$C$ 的发送对 $A$ 而言是"环境"，物理上没有任何机制把它们隔开。而"不受控"有两层含义——制度上免执照频段谁都能发射，物理上空气介质没有任何准入控制；有线世界"介质归我、干扰为零"的奢侈假设，在无线上同时被制度和物理双重否定。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 6 · Signal Strength（信号强度）</span>

- 障碍物（obstructions）会进一步削弱信号；
- 无线信号会反射（reflect）：
  - multipath（多径）：同一信号沿多条路径/反射、以不同时延到达接收方（类比：峡谷中的回声）；
- 不存在完美均匀的天线（antenna）；
- 世界在持续变化。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 7 · Changing Over Time（信道随时间变化）</span>

图示：PRR（packet reception ratio，分组接收率）与 RSSI（received signal strength indicator，接收信号强度指示）随时间（横轴 time in secs，2–10 秒）变化的曲线；PRR 纵轴为 1.0–0.0，RSSI 纵轴约 -80 至 -92（dBm）。两条曲线都在持续波动：信号强度与接收成功率在几秒的时间尺度上起伏不定。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 实测数据反复呈现一个规律：PRR 与 RSSI 的关系不是一条平滑直线，而是三段式——信号强（RSSI 高）时接收率几乎恒为 1，信号弱时几乎恒为 0，中间夹着一条又窄又不稳定的过渡带：RSSI 只差一两分贝，接收率却能从 0.9 跳到 0.1，而且时刻在抖。对协议设计的含义很直接。其一，RSSI 好 ≠ 能收全：图上存在 RSSI 平稳而 PRR 剧烈波动的时段，说明信号强度只是接收率的一个含噪代理指标。其二，速率自适应、功率控制这类机制在过渡带里会"追着抖动的目标跑"，必须加入迟滞（hysteresis）防止来回震荡——这与拥塞控制里避免 AIMD 在窗口边缘反复横跳是同一种工程智慧。用你概率论的词汇说：给定 RSSI 的取值，PRR 仍然是一个随机变量，条件期望 $E[\text{PRR} \mid \text{RSSI}]$ 本身就随时间漂移。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 8 · A Real Network: SWAN（一个真实网络：SWAN）</span>

图示：斯坦福校园局部地图，标注 Gates（盖茨计算机科学大楼）与 Packard（帕卡德电气工程大楼）两栋楼，并标注 2.5 seconds（2.5 秒）的时间尺度。

原文介绍（翻译）：SWAN（Stanford Wireless Access Network，斯坦福无线接入网络）是斯坦福校园内的一个 802.11b/g testbed（测试床），也是该校与 KAUST（King Abdullah University of Science and Technology，阿卜杜拉国王科技大学）研究合作的一部分。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 教授为什么专门放一页"真实网络"？因为仿真器里的信道是理想化的：固定参数的理想 $r^2$ 衰减、平稳的高斯噪声，而 SWAN 这类校园测试床会把"世界在变"原样喂给你——人走动、门开关、微波炉启动、隔壁 AP 换频道，全都写进实测曲线。做无线研究的人从仿真换到实测，第一课就是：实测 PRR 的方差远大于任何仿真模型的预测。与 KAUST 的合作让同一套协议在不同地域、不同建筑结构下被重复检验，结论不至于只对斯坦福成立。图上标注的 2.5 seconds 提示了观察的时间尺度：信道在秒级就有明显起伏，这意味着速率自适应等机制必须工作在亚秒级，否则永远慢信道一拍。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 9 · Research Space（研究区域图）</span>

图示 slide：SWAN 研究区域的放大地图。原文本中该页仅残留 "Research Space"（研究空间/研究区域）字样，其余为图形内容，无法从文本转写更多细节。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 10 · Interference（干扰）</span>

图示 slide：再次使用放大频谱图，这次突出 ISM 频段——图中标注 ISM 2450.0 ± 50 MHz（即 2.4 GHz）、ISM 24.125 ± 0.125 GHz、ISM 5.8 GHz 等多处，并写有 Interference（干扰）字样。要点：免执照频段上所有设备共享同一片频谱，彼此之间的干扰不受任何人控制。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 2.4 GHz ISM 频段是名副其实的公共广场：微波炉泄漏（其磁控管恰好工作在 2.45 GHz 附近，且功率是 WiFi 的成百上千倍）、蓝牙、无线鼠标、老式无绳电话、邻居家的 WiFi，全都挤在图中那道 ±50 MHz 的窄条里。频谱图想让你看到的是：这些业务在纸面上各有名分，在空气里却没有任何隔离，微波炉一开机，附近的 802.11b/g 立刻出现丢包尖峰。这就是 uncontrolled medium 最直观的体现——上一页讲的是"信道自己会变"，这一页讲的是"别人还能让它变得更坏"。工程上的应对是把"躲"自动化：动态选信道、速率自适应降档、以及前几讲的随机退避与链路层确认，都是在干扰不可消除的前提下，把不可控的损失折算成可控的期望值。

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b">Slide 11 · Overview（总览）</span>

- 无线网络日益成为个人通信的最后一跳（last hop），但总体表现通常不如有线；
- 无线的行为与有线截然不同，涌现出许多复杂行为：
  - 信号随距离衰减；
  - 信号受环境影响；
  - 中间链路（intermediate links）；
  - 外部干扰（external interference）；
- 行为不同，导致协议与算法也随之不同。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这一页给整个无线模块收口："不同行为 → 不同协议"不是口号，而是你在本组 slides 里逐个验证过的因果链。以太网 CSMA/CD 的"边发边听"在无线失效（半双工、近端信号淹没远端），于是有了 CSMA/CA 的"先退避、后确认"；载波侦听只代表发送方视角，隐藏终端因此不可见，于是有了 RTS/CTS 让接收方把视角广播出来；误码率高且原因无法区分（冲突还是低信噪比），于是有了逐帧的链路层确认；各代 PHY 速率差距悬殊而前导码固定低速，于是 MAC 全部以时间而非字节计价。这种"物理特性向上顶穿协议设计"的动力，常被概括为跨层设计（cross-layer design）的动机；但值得注意 802.11 的实际做法恰恰是反向的——用 duration/NAV 与"时间"抽象把 PHY 差异挡在 MAC 之下，维持了分层。泄漏与抽象两种力量的拉扯，正是链路层设计的全部戏剧性所在。
