# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Finite State Machines: Protocol Specification（有限状态机：协议规范）

> <span style="color:#7f8c8d;">Source: 慕课/Unit2-Transport/2-6-fsms-notes.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 1：标题页</span>

Finite State Machines（有限状态机）——Protocol Specification（协议规范）。

<span style="color:#7f8c8d;">（Slide 1/11）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 2：什么是有限状态机</span>

本讲将解释有限状态机（finite state machine，FSM）——指定网络协议与系统时最常用的工具之一，并讲解网络协议中通用的画法。最后会展示 TCP 规范中的那个有限状态机，它定义了 TCP 连接如何建立与拆除——你会看到如何用有限状态机描述像 TCP 三次握手这样的事情。

本页图示：三个圆圈标注 State 1、State 2、State 3，三个状态之间以箭头相连。顾名思义，有限状态机由**有限**个状态组成。状态（state）是系统的一种特定配置。这里从一个抽象例子开始：系统有三个状态（状态一、状态二、状态三），任一时刻系统必处于其中之一。

<span style="color:#7f8c8d;">（Slide 2/11）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 3：边——事件与动作</span>

本页图示在状态间连线上加了标注：上层为**引发状态转移的事件**（event causing state transition），下层为**转移发生时采取的动作**（actions taken on state transition）。

状态之间的边（edge）定义了如何在状态间转移。画一条边时，先写明什么**事件**（event）引发这次转移；在其下方可以写明转移发生时系统采取什么**动作**（action）。动作部分是可选的，因为并非所有转移都带动作；但如果有动作，就应当写明，否则规范不完整，别人可能无法正确地测试或实现。<span style="color:#00838f;">如果系统处于某个状态、而到达的事件在图中没有描述任何转移，那么 FSM 的行为就是**未定义**（undefined）的。</span>

<span style="color:#7f8c8d;">（Slide 3/11）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 4：一个状态可以有多条转移边——但同一事件必须唯一</span>

本页图示补充了从 State 1 出发的第二条边（标注"event action"）。

从同一个状态出发可以有**多条**转移：这里从状态 1 又引出一条转移，由不同的事件把系统带入状态 3。但对任意给定状态，某个事件对应的转移必须是**唯一**的：在这个例子里，某事件可以让状态 1 转移到状态 2，**或者**转移到状态 3，但同一个事件不能同时挂在这两条边上——否则转移就是有歧义的：事件发生时，你到底在状态 2 还是状态 3？系统只能处于一个状态。

<span style="color:#7f8c8d;">（Slide 4/11）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 5：FSM 例子——HTTP 请求</span>

来走一个例子：HTTP 请求。实践中 HTTP 请求比这复杂得多、有各种选项，这里只用非常简化的形式。把系统这样描述：起始状态是"正在查看页面"或"空闲"（Idle）；要加载新页面时，就转移到"页面请求中"（page requesting）状态——事件是"加载新页面"，动作是"打开到 Web 服务器的连接"；连接打开后便处于页面请求状态，当连接关闭、或页面上每个资源都请求完毕时，转回空闲状态。还需要一个状态来描述请求页面进行到哪一步：事件"还有更多资源要请求"触发动作"发一个 HTTP GET 请求资源"，进入"请求待决"（request pending）状态；事件"收到响应"则把我们带回页面请求状态。于是一个三状态系统就成形了：Idle（空闲）、page requesting（页面请求中）、request pending（请求待决）。

一方面，这是一个简洁漂亮的 FSM。但若真要照着实现，它遗漏了很多东西。系统里共有 4 个事件：页面请求（page request）、更多请求（more requests）、收到响应（receive response）、连接关闭（connection closed）。那么：处于"请求待决"状态时连接关闭了怎么办？"页面请求中"状态时又收到页面请求怎么办？"空闲"状态时收到响应怎么办？想做到完全显式、滴水不漏，就应该为**每个状态下的每个事件**都写明行为——但这会导致 FSM 复杂不堪、边多如牛毛。所以常见做法是只写下常见情形以便理解，再配一些关于其他转移的说明文字；有时甚至可以接受把某些行为留作未定义。例如互联网工程任务组（IETF）常常就不把每个 FSM 完全指定。其理念是：只对**互操作性**（interoperability）必需的部分做出规定，把规范的其余部分保持灵活，留给未来探索；随着协议被使用，人们会发现某些行为是否重要，重要的话再补充规范那一部分。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这页其实藏着协议工程里最重要的一个判断题：**规范该写到多完备？**从 6.042J 的视角看，一个 FSM 就是一台有限自动机，理论上把每个状态×每个事件都定义清楚（包括自环），就得到一台"全函数"的确定性机器——任何输入序列都有唯一可预测的行为，测试也可以机械覆盖所有边。但 IETF 反其道而行之：故意留白。原因在本页最后两句——留白是给**互操作**必需部分之外的行为留出实验空间；如果早期就把每个角落钉死，后来的实现者就失去了发明更优行为的自由。这与 2-4 的强端到端原则是同一种智慧的两副面孔：都拒绝为未来的未知用法过早锁定行为。代价当然也有：留白意味着两个"合规"实现可以行为不同，出了互操作 bug 没有仲裁文本。你做 Lab 时可以各取所需：对 grading 依赖的核心行为（比如重组器的边界情形）把规范写全、测试覆盖每条边；对实现自由的内部结构放心留白——**公共接口钉死、内部实现自由**，正是这套规范哲学在单个程序尺度上的翻版。

<span style="color:#7f8c8d;">（Slide 5/11）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 6：FSM 真例——TCP 连接（总览）</span>

来看一个真实的 FSM 例子——大概是互联网上最著名的状态机。这张图描述的是 TCP 的有限状态机（图源：Wikimedia Commons 的 TCP 状态图，`upload.wikimedia.org/wikipedia/commons/thumb/a/a2/Tcp_state_diagram_fixed.svg/`）。我知道它看起来非常复杂——共 12 个状态——但可以一点一点拆开看。首先，这张图其实分为四个部分：上面 4 个状态描述如何**打开**一条 TCP 连接；中间的 ESTABLISHED（已建立）状态是 TCP 收发数据时的状态——连接建立之后、关闭之前；下面 6 个状态描述连接如何关闭；最底部的 CLOSED（已关闭）状态表示连接已关闭、节点可以把它忘掉。注意最上面的状态同时也是"关闭"状态——在打开连接**之前**。回忆一下：启动 TCP 连接要用三次握手——SYN、SYN/ACK、ACK。客户端（主动开启方，active opener）向一个监听连接请求的程序发送 SYN（同步）消息；对方收到 SYN 后回复 SYN/ACK——既同步又确认最初的同步；主动开启方收到 SYN/ACK 后再回复一个 ACK。

<span style="color:#7f8c8d;">（Slide 6/11）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 7：三次握手在状态图上的行走</span>

这张状态图（图源同上）描述 TCP 在三次握手两侧的行为。**被动开启方**（passive opener）是服务器：它监听来自主动开启方（客户端）的连接请求。程序调用 `listen()` 时，套接字（socket）从橙色的 CLOSED 状态转入黄色的 LISTEN 状态——协议此时不采取任何动作、不发任何消息；若服务器在 LISTEN 状态对套接字调用 `close`，则立即转回 CLOSED 状态。接着从第一步走三次握手：客户端尝试打开连接、向服务器发送 SYN 分组——这是客户端侧从 CLOSED 到 SYN_SENT 的橙色转移，事件是客户端程序调用 `connect`，动作是发送 SYN 消息。第一个 SYN 发出后，客户端处于 SYN_SENT 状态、服务器处于 LISTEN 状态。SYN 到达服务器时触发蓝色转移：事件是"收到 SYN"，动作是"回复 SYN/ACK"，服务器进入 SYN_RCVD 状态。切回客户端：它在 SYN_SENT 状态，收到服务器的 SYN/ACK 后转入 ESTABLISHED 状态，动作是发送 ACK——SYN、SYN/ACK、ACK 握手的第三条消息。此刻客户端已可开始向服务器发数据。最后回到服务器：它在 SYN_RCVD 状态收到客户端的 ACK，转入 ESTABLISHED 状态，也可以发数据了。连接建立过程中还有几条转移，这里不必担心——视频《连接的建立与拆除》中会详细讨论。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 把三次握手"翻译"到这张状态图上，最好的方法是给每条报文做一次"双端对账"：SYN 这条报文对客户端是 `connect()` 事件的**动作**（出边 CLOSED→SYN_SENT），对服务器却是 SYN 到达的**事件**（入边 LISTEN→SYN_RCVD，动作才是发 SYN/ACK）；SYN/ACK 对服务器是动作、对客户端是事件（SYN_SENT→ESTABLISHED，动作发 ACK）；最后的 ACK 对客户端是动作、对服务器是事件（SYN_RCVD→ESTABLISHED）。一条报文总是"一端的动作 = 另一端的事件"，这个对称性一旦看破，12 个状态的大图就退化成了 2-1 里那张小时序图。再用 CSAPP 的实现视角校准一次状态与系统调用的对应：CLOSED≈套接字尚未 `socket()+bind()` 或已关闭；LISTEN≈`listen()` 之后；ESTABLISHED≈内核里连接队列中可被 `accept()` 取走的连接；你调试 `netstat`/`ss` 时看到的那些 SYN_SENT、TIME_WAIT 行，就是这台状态机的实时快照——教科书状态图与生产机器的状态统计从此对上了号。

<span style="color:#7f8c8d;">（Slide 7/11）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 8：随堂测验</span>

<span style="color:#2471a3;">**[exercise]**</span>（习题保持英文原文）

> Assume there is no other documentation of the TCP finite state machine, so there's no supporting textual description which defines other state transitions.
>
> Suppose we start in the closed state, then call listen, then receive a SYN, then call close. What state will we be in: CLOSED, SYN SENT, SYN RECEIVED, ESTABLISHED, FIN WAIT 1, or undefined?
>
> Suppose we start in the closed state, then call connect, then call close. What state will we be in: CLOSED, SYN SENT, SYN RECEIVED, ESTABLISHED, FIN WAIT 1, or undefined?

这是一道随堂测验。注意题设：**假定**除这张图之外没有其他 TCP 状态机文档——也就是说没有支撑性文字定义其他状态转移，一切答案只能从图上的边推出来。

<span style="color:#7f8c8d;">（Slide 8/11）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 9：随堂测验（重复页）与解答</span>

本页与 Slide 8 为同一道测验（此处不再重复表格）。下面讲评答案。

> <span style="color:#2471a3;">**[solution]**</span>
>
> 第一题的答案是 **FIN WAIT 1**。推导：我们从 CLOSED 状态出发；用户程序调用 `listen`，转移到 LISTEN 状态；套接字收到一个 SYN，转移到 SYN_RCVD 状态；此时用户程序调用 `close`——于是沿着"以 close() 为事件"的那条边，进入 FIN_WAIT_1 状态。第二题的答案是 **CLOSED**。我们从 CLOSED 状态出发；用户程序调用 `connect`，转移到 SYN_SENT 状态；此时用户程序调用 `close`——存在一条从 SYN_SENT 出发、以 close 事件回到 CLOSED 的边，所以回到 CLOSED。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这两道题是"只认图、不认常识"的极好训练，也正是 6.042J 归纳推理思想在协议上的应用：把事件序列当作状态图上的一条路径，从起点逐步沿边推进，**严禁**用"现实里 TCP 应该怎样"的直觉跳步。第一题最反直觉的地方在于：普通人对"listen 之后突然收到 SYN 再 close"的想象是"应该回到 CLOSED"，但图上 SYN_RCVD 状态的 close 边通向 FIN_WAIT_1——这是历史上真实存在的设计（被动打开方中途放弃时也要走关闭流程、发 FIN），没有支撑文字就极易答错。第二题则展示了图的另一面：close 从 SYN_SENT 回 CLOSED 的边使得"发出去的 SYN 无疾而终"成为合法路径。做这类题时给自己立三条规矩：一，只沿图中显式画出的边走；二，每步记下"当前状态 + 触发事件"的对账清单；三，走到没有边可走时，答案才允许是 undefined。这套流程就是形式化验证里"模型检查"的手工微缩版——把协议当模型、把事件序列当反例搜索路径。

<span style="color:#7f8c8d;">（Slide 9/11）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 10：连接拆除——六状态的对称之舞</span>

现在我们的套接字都处于 ESTABLISHED 状态、正在交换数据。蓝色方框里的六个状态（图源同上）描述 TCP 如何"拆除"（teardown）一条连接、也就是如何关闭它。之所以有时说"拆除"而不说"关闭"，是因为 "close" 这个词在系统调用的语义里有特定含义——正如我们将会看到的，一方"close"之后，连接还继续存在。TCP 建立连接与拆除连接的方式之间存在**对称性**：连接建立用同步（SYN）分组，连接拆除用结束（FIN）分组。连接的一方调用 `close` 时，沿左图的边进入 FIN_WAIT_1 状态——这使它向连接对端发送一个 FIN 分组；它是"主动关闭方"（active closer），因为操作由它发起。对端收到 FIN，沿右图的蓝边进入 CLOSE_WAIT 状态，并停留在此，直到它一侧的程序调用 `close`，届时它发出自己的 FIN。这里开始有点复杂了：TCP 连接是**双向**的。主动关闭方已关闭自己这个方向，不能再写数据；但被动关闭方可能还有数据要发——它可以继续发送，由主动关闭方接收并确认；它也可以关闭自己这一侧；它甚至可能几乎同时决定关闭连接——于是两个 FIN 分组在网络中交叉而过。对处于 FIN_WAIT_1 的主动关闭方而言有三种可能的结局。第一，被动关闭方确认了 FIN 但不发自己的 FIN：它处于 CLOSE_WAIT 状态并可继续发数据，主动关闭方走最下面的边进入 FIN_WAIT_2 状态。第二，被动关闭方也关闭自己这一侧，确认 FIN 并发出自己的 FIN：走中间的边，进入 TIME_WAIT 状态。第三，双方几乎同时主动关闭、互发 FIN：双方都在 FIN_WAIT_1，各自都会看到一条"没有确认自己 FIN 的 FIN"，此时转入 CLOSING 状态；当自己的 FIN 被确认后，与中间那条边一样转入 TIME_WAIT。TCP 在收到对端 FIN 时从 FIN_WAIT_2 转入 TIME_WAIT，然后在 TIME_WAIT 停留一段时间，直到可以安全转入 CLOSED。最后一条蓝色的边（从 LAST_ACK 到 CLOSED）在被动关闭方的 FIN 被确认时发生。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 讲稿只说"在 TIME_WAIT 停留一段时间"，这里补上"为什么必须停"——它是这套状态机里最精彩的第一性原理设计。回忆 2-1 讲过：分段可能在网络里滞留很久（卡在路由器缓冲或环路里），而 2-1 还讲了连接靠五元组标识、序列号可能回绕。现在把这三件事串起来：主动关闭方发出最后的 ACK 后，如果立刻销毁状态、让同一五元组立刻被新连接复用，那个滞留在网里的旧 FIN/旧数据就可能"转世投胎"进新连接。TIME_WAIT 期的存在就是**让网里的旧报文先死透**（超过其 TTL 生存时间），才允许五元组复活；同时它还保证"最后的 ACK 丢了"时有第二次机会——对方重发的 FIN 到达时，TIME_WAIT 侧还能重发 ACK。你看，这个状态把 2-1 的两个孤立知识点（旧分段滞留、随机初始序列号）全部收拢成了一个机制，而且解释了为什么拆除比建立复杂：建立时双方都在等对方，拆除时却要处理"每方任意时刻都可以走"的异步性——12 个状态里近一半服务于关闭，不是 TCP 过度设计，而是双向异步关闭的组合复杂度本就如此。🎥 *McKeown 在视频中解释： "TCP 连接是双向的"*（字幕来源：031 - p30 2-6d Finite state machines 4.zh.srt）——半关闭的可能性，正源于这句话。

<span style="color:#7f8c8d;">（Slide 10/11）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Slide 11：总结——从口语描述到精确规范</span>

一方面，这里确实有很多细节：12 个状态、覆盖大量情形。但你可以看到，这个 FSM 把此前那些口语化的描述变得细节化、精确化了。试图凭那些口语描述实现一个可正确互操作的 TCP 会非常困难；而这张图精确地指定了 TCP 的行为，因此极其有用。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 🎥 *McKeown 在视频中解释： "有十二个状态，覆盖了许多情况……这个图表精确地指定 TCP 的行为，因此，它非常有用"*（字幕来源：031 - p30 2-6d Finite state machines 4.zh.srt）。收尾这句"从口语到精确"正是本讲的题眼，也回答了一个你在写 Lab 时一定会遇到的问题：为什么规范要用状态机而不是文字？因为文字有歧义而图没有——"TCP 收到对方的 FIN 后可以继续发数据"这句人话，落到图上就是"CLOSE_WAIT 状态下存在数据发送（无需边）且由 close 事件触发 FIN 出边"的精确组合，容不得第二种解读。这套表示法的能力还不止于阅读：状态图让"覆盖所有转移"成为可机械检查的测试目标，让 6.042J 的归纳法可以逐状态展开论证，让两个实现团队可以对着同一张图逐边对账——互操作性就是这么来的。学完本讲，建议你做一件小事：用 `ss -tan` 观察你机器上真实的 TCP 连接各处于哪个状态、数量多少（TIME_WAIT 往往最多），把这张图与你正在运行的系统对上号——从"看懂图"到"在系统里认出图"，才算真正掌握协议规范这种技术。

<span style="color:#7f8c8d;">（Slide 11/11）</span>

---

> <span style="color:#7f8c8d;">脚注：本讲稿为视频 2-6《Finite state machines 1–4》的配套 notes（11 页）；TCP 状态图源自 Wikimedia Commons（Tcp_state_diagram_fixed.svg）；连接建立/拆除的报文时序详见视频 2-11《Reliable comm --- Connection setup and teardown》配套讲义。</span>
