# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Reading an RFC（阅读 RFC）

> <span style="color:#7f8c8d;">Source: 慕课/Unit4-Congestion-Control/3-10-reading-rfc-2.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1：封面</span>

Reading an RFC（阅读 RFC）——本讲是"技能课"（Skills）：教你读懂互联网标准的原始文本。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 2：History (RFC 2555)（历史，据 RFC 2555）</span>

- **RFC 1："Host Software"（主机软件，1969 年）**：

> <span style="color:#7f8c8d;">（RFC 2555 引 Steve Crocker 回忆 RFC 1 的原话，保留英文原文供阅读练习）</span>

> *"Mindful that our group was informal, junior and unchartered, I wanted to emphasize these notes were the beginning of a dialog and not an assertion of control."*
>
> （译文：考虑到我们这个群体是非正式的、初出茅庐的、没有正式授权的，我想强调这些"注释"是一段对话的**开端**，而不是控制权的宣示。）

- **格式的标准化**：结构与风格、知识产权（intellectual property rights，知识产权）、术语（RFC 2119）；安全考虑、IANA（互联网号码分配局）；
- **RFC 的种类**：提议标准（proposed standard，提议标准）、标准进程中的（standards-track，标准进程）、信息性的（informational，信息性的）、实验性的（experimental，实验性的）、当前最佳实践（best current practice，BCP，当前最佳实践）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> RFC（Request for Comments，征求意见稿）这个名字本身就是互联网文化的宣言：第一份 RFC 写于 1969 年 4 月 7 日，作者 Steve Crocker 当时只是加州大学洛杉矶分校的一名研究生，参与的 ARPAnet 项目组如此年轻、如此没有官方授权，以至于他刻意把文档叫做"征求意见"而不是"规范"。上面引的那句回忆写于 1999 年的 RFC 2555（30 周年纪念文集）：一句"对话的开端，而非控制权的宣示"，定义了此后五十余年互联网标准制定的精神气质——自下而上、靠技术说服力而非行政命令推进。对比一下你在其他领域见到的标准化组织（如电信业的 ITU、国际标准的 ISO），它们是会员制、代表团投票、自上而下的；而 IETF（互联网工程任务组）的原则是那句名言"rough consensus and running code"（粗略共识与可运行代码）——谁实现了、谁的代码跑得通，谁就有发言权。这份草莽出身与今天互联网的开放性之间的因果，是技术史课的绝佳案例。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 3：RFC Process (simplified)（RFC 流程——简化版）</span>

一份 RFC 的诞生之路（以 trickle 协议草案为例）：

- 从一份草案（draft，草案）开始：`draft-levis-roll-trickle-00`；
- 修订：`draft-levis-roll-trickle-XX`（作者个人版本）；
- 被工作组（working group，工作组）接受：`draft-ietf-roll-trickle-00`；
- 修订：`draft-ietf-roll-trickle-XX`；
- 被工作组主席接受、准备发布；
- 工作组最后征询（working group last call）、IETF 全体最后征询（IETF last call）；
- IESG（互联网工程指导组）审查；
- 批准成为一份 RFC。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 注意这份流程里藏着的"版本命名学"：`-levis-` 段是作者名（本课程讲义作者 Philip Levis 正是 trickle 协议的作者之一），`-ietf-` 段表示已被工作组收编，末尾的 `00`、`XX` 是版本号——读一个草案的文件名就能说出它的成熟度。整个流程与你在 CLRS 里熟悉的"算法从伪代码到被标准库收录"不同，它更像一个开放源码项目的治理流程：任何人可提交（草案公开可评），先在小社区（工作组）内打磨，再经过两层"最后征询"（last call）全社区公开评审，最后由 IESG 这个 15 人左右的指导组把关。没有投票、没有否决权博弈，异议要么被技术论证化解、要么记录在案（文档里可以有 Dissent 小节）。对即将进入工程行业的学生，这是一个重要事实：**互联网的"标准"是这样一份份由志愿者反复修改的文本**，而不是某个权威机构一次性颁布的法令。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 4：Terminology（术语：RFC 2119 的关键词）</span>

RFC 2119 规定了一组**全大写**关键词，在任何标准文本中含义严格固定：

- **MUST, REQUIRED, SHALL（必须）**：绝对要求。

> <span style="color:#7f8c8d;">（RFC 2119 对 SHOULD/MAY 的定义原文，保留英文供阅读练习）</span>

> *"SHOULD, RECOMMENDED: mean that there may exist valid reasons in particular circumstances to ignore a particular item, but the full implications must be understood and carefully weighed before choosing a different course."*
>
> （译文：SHOULD、RECOMMENDED（应当、推荐）：指在特定情形下可能存在合理理由忽略某个条目，但在选择另一条路之前，必须理解并仔细权衡其全部影响。）

> *"MAY, OPTIONAL: mean that an item is truly optional."*
>
> （译文：MAY、OPTIONAL（可以、可选）：指某个条目是真正的可选项。）

**讲稿要点**：读 RFC 时，一个词的大小写决定它的约束力——`MUST` 不实现就是不符合标准；`SHOULD` 可以不实现，但你最好能说出为什么；`MAY` 随你便。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> RFC 2119 是 1997 年发布的、全文只有一页多的 BCP 文档，却可能是被引用次数最多的 RFC——因为所有标准都要借用它的词汇表。这个设计非常聪明：把"强制力等级"编码进单词的大小写里，让自然语言文本获得了接近形式化规约（specification）的精确性。从设计思路看，这是对"协议实现者之间互相扯皮"这一现实问题的回应：早期许多互操作失败（A 家的实现与 B 家的实现对不上）源于对"should"这种模糊词的理解差异，于是干脆规定全大写 `SHOULD` 是一种带辩护义务的弱要求、`MUST` 是无例外的强要求。你在 6.042J 里见过把自然语言命题翻译成逻辑符号的练习——RFC 2119 干的是同一件事，只是把符号系统放进了英语的大写字母里。作为读者，拿到任何一份 RFC 先看它用的关键词等级，就能立刻分辨"协议的灵魂"（MUST 部分）与"可选的点缀"（MAY 部分）。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 5：Example: RFC5681（例子：RFC 5681）</span>

纯图示 slide：展示 RFC 5681 文档首页的截图。

**配套说明**：RFC 5681《TCP Congestion Control》正是本单元所学内容的**标准文本**：慢启动与拥塞避免、快速重传/快速恢复的算法（IW、cwnd、ssthresh 的操作规则）都定义其中。学完本单元，你已经具备直接阅读这份 RFC 的全部背景：用它检验你对 cwnd 更新规则（每确认 $\text{cwnd} \mathrel{+}= MSS \cdot \frac{MSS}{\text{cwnd}}$、丢包减半）的记忆，注意它把哪些写成了 MUST、哪些写成了 SHOULD。

---

> <span style="color:#7f8c8d;">脚注：本讲义为视频 4-9《Skills - Reading and RFC》的配套 slides；RFC 2555（30 Years of RFCs）、RFC 2119（Key words for use in RFCs）、RFC 5681（TCP Congestion Control）均可从 ietf.org 免费获取。</span>
