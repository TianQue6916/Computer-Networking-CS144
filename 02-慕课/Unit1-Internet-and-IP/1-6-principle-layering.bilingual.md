> <span style="color:#7f8c8d;">Source: Stanford CS144 Fall 2019 慕课讲义 · Unit1-Internet-and-IP/1-6-principle-layering.pdf · 行内双语翻译版</span>

# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> <span style="color:#c0392b;">Principle: Layering（原则：分层）</span>

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">Layering（分层）的定义</span>

> <span style="color:#2471a3;">**[definition]**</span> <span style="color:#00838f;">**Layer（层）是功能性的组件（functional components）。层与相邻的上下层依序通信。每一层利用下层提供的服务（services）加上自己的私有处理（private processing），向上一层提供定义良好的服务（well-defined service）。</span>**

（原图示意：Layer $k{+}1$ data、Layer $k$ data、Layer $k{-}1$ 三段上下堆叠，层间以服务接口衔接——数据从上层逐层下沉，每层只与紧邻层打交道。）

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这张定义里有两个关键词容易被滑过去：其一是 **functional（功能性的）**——层按"做什么功能"切分，而不是按"代码在哪台机器上"或"谁写的"切分，所以同一层的代码可以一半在主机、一半在路由器（网络层就是如此）；其二是 **well-defined service（定义良好的服务）**——它是分层的"契约"边界，上层只许依赖这份契约、不许窥探下层实现。你在 6.042J 学过的抽象（abstraction）与这里完全同构：一个结构如果只暴露操作语义而隐藏实现，那么所有实现都可互换。四层模型里"IP 之下随便换链路、之上随便换传输"的自由，全部由这两个词支付。读这份定义时建议逐句对照你写 CSAPP 的经验：`open_listenfd` 向你隐藏了 TCP 三次握手的全部细节，而你依赖的正是"返回一个可用监听描述符"这份 well-defined service。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">邮政系统里的分层</span>

（原图演示：一个信封上写着 "From: Nick / Phil Levis, Stanford, CA"——外层信封带着寄件人姓名与城市；里面还套着另一个信封 "From: Nick / Phil Levis, Stanford, CA"。）

邮政服务天然是分层的：你写信时只需关心内容与收件人；邮局只关心"从哪个分拣中心送到哪个分拣中心"；运输公司只关心"把哪批货从哪个城市运到哪个城市"。每一层都读取自己需要的地址信息、对上一层的信息一无所知——外层信封上的地址在到达目的地邮局前不会被拆开。（原图通过"信封套信封"演示了这种嵌套：内层信封的地址由收件人层面的下一跳处理者解读，外层信封的地址由运输层面的处理者解读。）

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">计算机系统里的分层</span>

（原图：一段源代码 `printf("Hello world.\n");` 依次经过四个阶段——）

- **edit（编辑）** → Source code（源代码）
- **compile（编译）** → Object code（目标代码）；编译内部又分为 lexical analysis（词法分析）、parsing（语法分析）、preprocessing（预处理）、code generation and optimization（代码生成与优化）几层
- **link（链接）** → Executable（可执行文件）；链接器把编译好的目标文件与库（libraries）链接起来
- **execute（执行）** → Run!

（原文此 slide 重复出现一次，内容相同。）

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这页是全单元把"分层"讲得最亲切的一处：编译工具链就是一套活的四层模型。`gcc hello.c -o hello` 一条命令背后是四个各自独立的 stage（阶段）：预处理器只管文本展开、编译器只管从 C 到汇编、汇编器只管汇编到目标码、链接器只管符号解析与重定位——每个 stage 向下要一个文件、向上交一个文件，接口是严格定义的文件格式（汇编文本、ELF 目标文件、ELF 可执行文件）。你甚至能在这里找到与网络完全对应的现象：换一个链接器或换一个编译器版本，程序照样能跑，只要文件格式接口不变——这就是"实现可替换、契约不可破"；而分层也带来同样的代价——`printf("Hello world.\n")` 的错误若出在链接期符号重定位，编译器无从报错，层与层之间同样存在"错误隔离但信息也隔离"的问题。CSAPP 前九章你已经把这四层全部拆开看过一遍；现在网络课程只是把同一招式用在包的旅程上：应用层产生消息，各层逐级加工封装，接收方逆序解封装——与"源码逐级变换为目标码"在结构上是同一个数学对象。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">分层的原因（六个理由）</span>

分层之所以值得，有六个理由：

1. **Modularity（模块化）**——各层是可独立构造、替换的构件；
2. **Well-defined service（定义良好的服务）**——层间契约清晰，上层无需了解下层实现；
3. **Reuse（复用）**——一个 TCP 实现可被成千上万应用共享；
4. **Separation of concerns（关注点分离）**——每一层只解决一类问题；
5. **Continuous improvement（持续改进）**——单独某一层可以独立升级演进；
6. **Peer-to-peer communications（对等通信）**——每一层与对端的同层进行逻辑上的"对话"。

（原文末页标注：CS144, Stanford University。）

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 六个理由里，"复用"与"持续改进"是互联网历史里被验证得最充分的两条。看"复用"：TCP 协议栈是 1983 年（RFC 793 于 1981 年发布、1983 年 1 月 1 日 ARPANET 全面切换到 TCP/IP）写进操作系统的公共设施，此后的 Web（1989-91 年）、P2P（1999 年的 Napster）、视频流媒体（2005 年前后的 YouTube）全部白嫖同一份实现——若没有分层，每个应用都得自带一套重传与拥塞控制，互联网大概撑不过前十年。看"持续改进"：以太网从 1980 年的 10 Mbps 演进到今天的 400 Gbps，WiFi 从 802.11b 到 Wi-Fi 7，链路层天翻地覆，而 IP 层和 TCP 语义几乎未动——因为契约稳定，下层换血零成本。但也要保持批判眼光：分层不是免费午餐，教科书（以及 2012 年后的研究文献）常批评它带来"跨层信息损失"——比如链路层明明知道这次失败是无线干扰（重传有意义）还是信号消失（重传无意义），上层却只能看到一个笼统的"包没到"。所以现代协议设计（QUIC 将传输与加密合层、数据中心里把路由信息上抛）常在做"受控的跨层穿透"。记住：分层是工程权衡的产物，不是数学定律。
