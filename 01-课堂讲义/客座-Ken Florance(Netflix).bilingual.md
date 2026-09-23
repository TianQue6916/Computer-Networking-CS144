# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> How Netflix Works（Netflix 是如何工作的）—— Ken Florance（Netflix 产品副总裁）

> <span style="color:#7f8c8d;">Source: 课堂讲义/客座-Ken Florance(Netflix).pdf · CS144 Fall 2019 · 行内双语翻译版</span>

> <span style="color:#7f8c8d;">说明：本文为客座讲座 slides。pdftotext 提取时本 PDF 的文字层叠印严重，多处出现乱码碎片（如 "1,700+ @\"4("、"518M+ F(."、"Hcioghnqduiatiliotyns"、"c003.sfStanford…" 等水印与正文的交叠），已按可辨识残片与上下文复原并在相应位置以灰色脚注声明；图表数值按原文残片推断，不编造精确数据。纯图示页作一句说明。</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">开场：Netflix 的三个承诺</span>

No ads | Uninterrupted entertainment | All episodes at once（没有广告 | 不被打断的娱乐 | 整季一次放出）。

图示页：Watch When You Want. How You Want. Where You Want.（想在何时看、以何种方式看、在何地看，随你）。配图为多设备观看场景。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Netflix 的历程</span>

The Netflix journey（Netflix 的历程）——时间轴：

- 1997：Netflix 成立；
- 2007：流媒体视频服务在美国上线（streaming video service，流媒体视频服务）；
- 2008：扩展到加拿大——首次在美国以外上线；
- 2013：原创剧集《House of Cards》（《纸牌屋》）上线；
- 2016：进入 130 个国家（包括韩国）；此后 Netflix 覆盖 190+ 个国家。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">今天是什么规模？</span>

What about today?（今天是什么规模？）

- 覆盖 190+ 个国家；付费会员（paid membership，付费会员）超过 1.51 亿；60% 以上来自美国以外。
- 1,700+ 种不同类型的设备（different types of devices）；全球去重设备数（unique devices globally）5.18 亿+。
- （本页源文字符层叠印出乱码碎片，上述数值按可辨识残片复原。）

Netflix 的使命：把世界各地的精彩故事带给全世界的观众（Bringing great stories from around the world to audiences the world over）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">什么才是好的流媒体体验？</span>

What makes a great streaming experience?（什么造就好的流媒体体验？）

1. 快速开始（Start quickly）；
2. 高画质（Great quality）；
3. 不卡顿（No interruptions）。

Diverse networks and devices（多样的网络与设备）：接入侧五花八门——3G、有线（Cable）、LTE、卫星（Satellite）、5G、光纤（Fibre）、Wi-Fi；显示侧同样参差——4K、HDR、HD、SD。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">流媒体体验的四大构件</span>

The building blocks of the Netflix streaming experience（Netflix 流媒体体验的构件）：<span style="color:#00838f;">整条流水线由四个构件组成——制作（Production）→ 编码（Encoding）→ 部署（Deployment）→ 播放（Streaming），中间由互联网（internet）连接。</span>

这些环节都在被持续优化，以改善用户体验。此图在讲座中反复出现，用于标注当前讲到哪个环节。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">编码进化一：一刀切的码率</span>

One-size-fits-all encoding（一刀切的编码）：2011 年的做法——所有内容统一按标准清晰度（SD，Standard Definition）1050 kbps 编码。时间轴 2011—2018，展示此后各年的改进节点。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">编码进化二：按片名编码</span>

Per-title encoding（按片名编码）：不同片子的复杂度不同，各自分配不同码率——例如同一画质下，有的片名 640 kbps、有的 710 kbps、有的 910 kbps。时间轴 2011—2018。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">编码进化三：按镜头编码</span>

Per-shot encoding（按镜头编码）：再进一步，同一部片子的不同镜头（shot，镜头）单独优化。时间轴 2011—2018。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">按镜头选最优编码配方</span>

Select the best encoding recipe per-shot（按镜头挑选最佳编码配方）：

- 以最优方式分配比特（allocate the bits optimally），达到整体最佳画质；
- 同等画质下节省 64% 带宽（64% less bandwidth for the same quality）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 为什么同一画质下不同镜头的码率能差出几倍？从第一性原理看，视频压缩的本质是"给信息量分配比特"：一片纯色墙面每帧几乎不带来新信息，比特花下去也不涨画质；而暴雨、人群、胶片颗粒这类场景每帧都是新信息，熵（可压缩性的下界）天然高。传统"一刀切"码率必须按**最难**的镜头配置，结果简单场景白白浪费比特；per-shot 编码相当于把一个大优化问题拆成一堆小问题逐个求解——总比特预算固定，往边际收益最高的镜头上多投一点，整体画质才最大。这与你在概率课里"在方差大的地方多采样才降总误差"是同一个配置思想：资源该跟着不确定性走。VP9 相对 H.264 的优势则是编码器本身的进步——更精细的帧内预测与熵编码，把同样信息量用更少的比特表示出来。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">VP9 与 H.264 对比</span>

Shot-based VP9（按镜头编码 + VP9 编解码器）：270 kbps；对照——按片名编码 + H.264：750 kbps。同等画质下比特数减少 64%。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">1GB 流量能看多久？</span>

Hours of Netflix content in 1GB of data（1GB 流量能看多久的 Netflix 内容）：2011 年约 1.5 小时 → 2015 年约 2.5 小时 → 今天约 6.5 小时（约合 8 集剧集），并预告 "Coming soon"（还在继续提升）。（本页数值配对依据原文残片顺序推断。）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">构件回顾：部署</span>

The building blocks（构件回顾）：同一流水线图再次出现，当前高亮"部署（Deployment）"环节——把编码产物铺到全球分发网络（下一部分的主题）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">全球交付架构</span>

Global Netflix Delivery Architecture（全球交付架构）。图示页：Netflix 的服务器（OCA，Open Connect Appliance，开放连接设备）部署在两类地点——全球约 100 个量级的互联网交换点（IX，Internet Exchange，互联网交换点）站点，以及全球约 10,000 个量级的 ISP 站点，内容从 OCA 流向各 ISP 网络。

- 接入长途容量（long-haul capacity，长途链路容量）是昂贵的——对内容提供商和 ISP 都是，所以要尽量少用；
- "缓存在你睡觉时填充"（caches fill while you sleep）——利用网络低谷时段预填缓存；
- 网络按峰值容量设计（dimensioned for peak capacity），低谷填缓存意味着无需额外扩容。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">交付网络的规模数字</span>

规模统计（源文数字清晰可辨）：

- 26 个国家 85 个互联网交换点（IX）位置；
- 140 个国家 3,546 个 ISP 位置；
- 生产环境中 13,263 台服务器；
- 105,000 网络英里（network miles，网络英里）——约合绕地球 4.2 圈。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "网络按峰值设计、缓存在低谷填充"这句话可以直接用你概率课的语言算账。网络需要按 $\lambda(t)$ 的**峰值** $\lambda_{\max}$ 配容量（否则高峰期队列爆炸），但一天里大部分时间 $\lambda(t)$ 远低于峰值——容量在闲置。Netflix 把"把片库搬到 OCA 缓存"这件不着急的事挪到夜间低谷去做，等于把一个大小为 $C$ 的负载从 $\lambda$ 高峰处搬走：M/M/1 平均队长 $L=\lambda/(\mu-\lambda)$ 对 $\lambda$ 是**非线性**的，高峰处少一份负载省下的排队时延，远大于低谷处加同一份负载多付出的——削峰填谷在数学上稳赚。这与利特尔法则（Little's law，$L=\lambda W$）一样，是"平均负载≠平均体验"的又一例证。此外注意 OCA 直接连进 ISP 机房的做法，物理上把内容推到离用户一跳以内，绕开了昂贵的长途段——这就是 CDN 的"驻地化"路线。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">构件回顾：播放</span>

The building blocks（构件回顾）：流水线图再次出现，当前高亮"播放（Streaming）"环节——自适应码率播放（下一部分的主题）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">网络状况会变化</span>

Network conditions change（网络状况会变化）。图示页：横轴为时间（秒），纵轴为带宽（kb/s）——可用带宽随时间剧烈波动的曲线。Wi-Fi 换房间、家里有人开始下载、 cellular 信号漂移，都会让"管子粗细"随时改变。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">播放端如何应对：自适应引擎的三个旋钮</span>

图示页（源文此页乱码碎片较多，已按残片复原）：播放器内部有一个自适应流媒体引擎（adaptive streaming engine，自适应流媒体引擎），<span style="color:#00838f;">它的职责是在"视频画质"与"重缓冲风险"这两个相互挤压的量之间做动态权衡</span>：

- 视频画质（video quality）——调高它消耗更多带宽；
- 重缓冲风险（rebuffer risk，重缓冲风险）——带宽不够还硬扛高码率，就会卡顿。

还有一个"播放延迟"（playdelay，起播延迟）参数：宁可先缓冲一小段再开播，换取后续更稳的体验。引擎的职责：在画质、卡顿风险与起播速度之间动态权衡。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 注意 Netflix 把"画质—卡顿"权衡放在**应用层**解决，而不是指望网络。这背后是 CS144 反复强调的端到端原则（end-to-end principle，端到端原则）：TCP 已经在传输层提供了可靠字节流（reliable byte stream，可靠字节流）——你在 CSAPP ch11 用 `socket()` 收发的正是它——但"可靠"只保证数据不丢不乱，**不保证及时**：拥塞时 TCP 会重传、退避，应用只会感到"字节迟迟不来"。播放器对"及时性"有硬需求（每秒必须喂够一帧的比特），这个需求只有应用自己知道、也只有应用自己能权衡，所以自适应码率逻辑必须放在端上。实现上它把视频切成几秒长的分段（segment，分段），每段预先用不同码率编码（这呼应前面 per-title/per-shot 编码），播放器按"当前测得带宽 × 保守系数"选下一分段的码率——本质是在不确定信道上做在线决策，和你在概率课里学的"基于当前观测更新策略"同构。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">根据变化的网络状况调整视频画质</span>

Adapt video quality based on changing conditions（根据变化的网络状况调整视频画质）。图示页：视频被切成 Segment 1—5（分段 1—5）；带宽高时各段取高画质（Mid quality，中画质），带宽跌落时切换到低画质（Low quality，低画质）——同一段内容存在多条码率版本，播放器逐段挑选，"..."表示时间轴继续。

（下一页重复同一图示，强调"逐段切换"的机制。）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">调整后的最终体验</span>

Adapt video quality based on changing conditions（根据变化的网络状况调整视频画质）：Resulting experience（最终得到的体验）——图示把逐段选出的码率拼接起来：画质随带宽起伏上下浮动，但播放**从不中断**；对比不做自适应的情形（一条高码率扛到带宽跌落 → 缓冲区清空 → 卡死）。这就是"画质让位于流畅"的工程答案。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">技巧与彩蛋</span>

tips and tricks（技巧与彩蛋）：

- 在 Netflix 播放页按 `Ctrl+Alt+Shift+D` 可以调出统计面板（码率、缓冲、吞吐等实时数据）；
- fast.com 是 Netflix 提供的网速测试网站——它直接用一段视频下载来测你到 Netflix 网络的真实吞吐。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">提问与致谢</span>

questions?（有问题吗？）Thank you!（谢谢！）
