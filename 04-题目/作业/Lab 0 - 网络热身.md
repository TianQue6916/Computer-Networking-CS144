# CS144: Introduction to Computer Networking（计算机网络导论，Stanford Fall 2019）

## <span style="color:#2471a3;">**[lecture]**</span> Lab 0: networking warmup（Lab 0：网络热身）

> <span style="color:#7f8c8d;">Source: 实验讲义/Lab 0 - 网络热身.pdf · CS144 Fall 2019 · 行内双语翻译版</span>

> <span style="color:#7f8c8d;">脚注（源文噪声说明）：原 PDF 经 pdftotext 提取后存在系统性噪声：列表项目符号丢失、下划线丢失（如 `vm howto` 应为 `vm_howto`、`make check webget` 应为 `make check_webget`、`CMAKE BUILD TYPE` 应为 `CMAKE_BUILD_TYPE`）、终端输出与正文混排。以下译文已按上下文复原，不逐一标注。</span>

---

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">导言</span>

欢迎来到 CS144：计算机网络导论。在这个热身实验（warmup lab）中，你将在自己的计算机上安装一套 Linux，学习如何手工完成一些互联网上的任务，编写一个通过互联网抓取网页的 C++ 小程序，并（在内存中）实现网络的核心抽象之一：写者与读者之间的可靠字节流（reliable byte stream）。我们预计这个热身实验需要 2 到 6 小时完成（后续实验会占用你多得多的时间）。在动手之前先通读整个实验讲义是个好主意。

**截止时间**：9 月 30 日（周一）下午 5 点。实验课：9 月 24 日（周二）晚 7:30–10:00，地点 STLC114。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">0 合作政策（Collaboration Policy）</span>

**编程作业必须是你自己的工作**：除了我们随作业发给你的代码之外，你提交的编程作业代码必须全部自己写。请不要从 Stack Overflow、GitHub 或其他来源复制粘贴代码。如果你的代码基于网上或其他地方找到的示例，请在提交的源代码注释中注明出处 URL。

**与他人合作**：不得把你的代码给任何人看，不得看任何人的代码，也不得看往年的解答。你可以和其他同学讨论作业，但不要抄任何人的代码。如果你和另一位同学讨论了作业，请在提交的源代码注释中写出对方姓名。更多细节请参阅课程管理讲义（administrative handout），有任何不清楚的地方请在 Piazza 上提问。

**Piazza**：欢迎在 Piazza 上提问，但请不要张贴任何源代码。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">1 在你的计算机上搭建 GNU/Linux</span>

CS144 的作业需要 GNU/Linux 操作系统和支持 C++ 2017 标准的较新 C++ 编译器。请从以下三个选项中选择一项：

1. **推荐**：安装 CS144 的 VirtualBox 虚拟机镜像（安装说明见 https://stanford.edu/class/cs144/vm_howto/vm-howto-image.html）。
2. 运行 Ubuntu 18.04 LTS，然后运行我们的安装脚本。你可以在实体计算机上、VirtualBox 内，或 EC2 等虚拟机上完成（分步指南见 https://stanford.edu/class/cs144/vm_howto/vm-howto-iso.html）。
3. 使用其他 GNU/Linux 发行版，但要有心理准备：途中你可能遇到障碍，并且需要有能力调试它们。你的代码将在 Ubuntu 18.04 LTS 与 g++ 8.2.0 环境下测试，必须能在该条件下正常编译和运行。提示见 https://stanford.edu/class/cs144/vm_howto/vm-howto-byo.html。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">2 手动操作网络（Networking by hand）</span>

让我们开始使用网络。你将**手工**完成两个任务：抓取一个网页（就像浏览器那样）和发送一封电子邮件（就像邮件客户端那样）。这两个任务都依赖一个网络抽象：可靠的双向有序字节流（reliable bidirectional in-order byte stream）——你向终端输入一个字节序列，同样的字节序列最终会按相同顺序被送达另一台计算机上运行的程序（一台服务器）。服务器用它自己的字节序列回应，再传回你的终端。

#### <span style="color:#c0392b;">2.1 抓取一个网页</span>

1. 在浏览器中访问 http://cs144.keithw.org/hello 并观察结果。2. 现在，你将手工完成浏览器所做的同一件事。

(a) 在你的虚拟机上运行 `telnet cs144.keithw.org http`。这告诉 telnet 程序在你的计算机与另一台计算机（名为 `cs144.keithw.org`）之间打开一条可靠字节流，并指向那台计算机上运行的特定服务："http" 服务，即万维网（World Wide Web）所使用的超文本传输协议（Hyper-Text Transfer Protocol）。<span style="color:#7f8c8d;">¹</span> 如果你的计算机设置正确并且已接入互联网，你将看到：

```
user@computer:~$ telnet cs144.keithw.org http
Trying 104.196.238.229...
Connected to cs144.keithw.org.
Escape character is '^]'.
```

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> telnet 命令做的事，你在 CSAPP 第 11 章里用 C 代码亲手做过一遍，只是当时藏在了库函数背后：telnet 在用户态完成了 `socket()` 创建套接字、`connect()` 发起 TCP 连接（内核替你完成三次握手）、然后进入"读终端、写套接字；读套接字、写终端"的双向转发循环——这正是 CSAPP echo 客户端 `echo_client()` 的结构。此处命令第二个参数 `http` 是服务名而非端口号，系统会通过 `/etc/services` 把它解析为 TCP 端口 80；`Trying 104.196.238.229...` 展开的则是 `getaddrinfo()` 的域名解析结果。换句话说，这一节的"手工网络操作"其实是把你 CSAPP 网络编程所学剥离掉所有代码包装，直接在终端上重演：字节流两端一边是 telnet 进程，一边是 Web 服务器进程，中间的"可靠双向有序字节流"由双方内核中的 TCP 实体协作维护。

> <span style="color:#7f8c8d;">脚注 1：计算机的名字有一个数值等价物（104.196.238.229，一个互联网协议第 4 版地址，即 IPv4 地址），服务的名字也一样（80，一个 TCP 端口号，port number）。我们稍后会更多讨论这两者。</span>

如果需要退出：按住 ctrl 并按 ]，然后输入 `close`。

(b) 输入 `GET /hello HTTP/1.1`。这告诉服务器 URL 的路径（path）部分——从第三个斜杠开始的那一段。

(c) 输入 `Host: cs144.keithw.org`。这告诉服务器 URL 的主机（host）部分——`http://` 与第三个斜杠之间的部分。

(d) 再按一次回车键，发送一个空行。这告诉服务器：你的 HTTP 请求已经结束。

(e) 如果一切顺利，你将看到与浏览器相同的响应，前面带着若干 HTTP 首部（HTTP headers），告诉浏览器如何解释这个响应。

3. **作业**：既然你已经知道如何手工抓取网页了，证明给我们看！用上面的技术抓取 URL http://cs144.keithw.org/lab0/sunetid，把 `sunetid` 替换成你自己的主 SUNet ID。你将在 `X-Your-Code-Is:` 首部中收到一个秘密代码。保存好你的 SUNet ID 和这个代码，写进实验报告（writeup）里。

#### <span style="color:#c0392b;">2.2 给自己发一封电子邮件</span>

既然你已经会手工抓取网页了，现在来发送一封电子邮件——同样是通过一条可靠字节流，连接到另一台计算机上运行的服务。

1. 在 Stanford 校园网络内运行 `telnet smtp-unencrypted.stanford.edu smtp`。（如果你不在 Stanford 网络内，请先登录 cardinal.stanford.edu，然后再运行这些命令。）"smtp" 服务指的是简单邮件传输协议（Simple Mail Transfer Protocol，SMTP），用于发送电子邮件。如果一切顺利，你将看到：

```
user@computer:~$ telnet smtp-unencrypted.stanford.edu smtp
Trying 171.64.13.18...
Connected to smtp-unencrypted.stanford.edu.
Escape character is '^]'.
220 smtp-unencrypted.stanford.edu ESMTP Postfix
```

2. 第一步：向邮件服务器标识你的计算机。输入 `HELO mycomputer.stanford.edu`。等待看到 "250 smtp-unencrypted.stanford.edu"。

3. 下一步：这封邮件是谁发的？输入 `MAIL FROM: sunetid@stanford.edu`。把 `sunetid` 替换成你的 SUNet ID。<span style="color:#7f8c8d;">²</span> 如果一切顺利，你将看到 "250 2.1.0 Ok"。

> <span style="color:#7f8c8d;">脚注 2：是的，伪造一个"发件人"地址是可能的。电子邮件有点像邮局的真信件：回信地址是否准确，基本上靠自觉（honor system）。请不要滥用这一点！</span>

4. 下一步：收件人是谁？输入 `RCPT TO: sunetid@stanford.edu`。把 `sunetid` 替换成你的 SUNet ID。如果一切顺利，你将看到 "250 2.1.5 Ok."。

5. 是时候上传邮件内容本身了。输入 `DATA`，告诉服务器你准备开始。如果一切顺利，你将看到 "354 End data with <CR><LF>.<CR><LF>"。

6. 现在你正在给自己输入一封邮件。首先输入你将来在邮件客户端里会看到的那些首部。首部写完后留一个空行。

```
354 End data with <CR><LF>.<CR><LF>
From: sunetid@stanford.edu
To: sunetid@stanford.edu
Subject: Hello from CS144 Lab 0!
```

7. 输入邮件正文——随便写什么都行。写完后，用单独占一行的句点结束：`.`。预期会看到类似 "250 2.0.0 Ok: queued" 的响应。

8. 输入 `QUIT` 结束与邮件服务器的对话。检查你的收件箱和垃圾邮件文件夹，确认收到了这封邮件。

9. **作业**：既然你已经知道如何手工发邮件了，证明给我们看。用上面的技术，从你自己出发，给 cs144grader@gmail.com 发一封邮件。

#### <span style="color:#c0392b;">2.3 监听与连接（Listening and connecting）</span>

你已经见识了 telnet 能做什么：一个向其他计算机上运行的程序发起出站连接的客户端程序。现在是时候体验一下扮演简单服务器了——那种守在那里等客户端来连接的程序。

1. 在一个终端窗口中，在你的虚拟机上运行 `netcat -v -l -p 9090`。你应该看到：

```
user@computer:~$ netcat -v -l -p 9090
Listening on [0.0.0.0] (family 0, port 9090)
```

2. 让 netcat 继续运行。在另一个终端窗口运行 `telnet localhost 9090`（同样在你的虚拟机上）。

3. 如果一切顺利，netcat 会打印出类似 "Connection from localhost 53500 received!" 的信息。

4. 现在试着在任意一个终端窗口中输入——netcat（服务器）或 telnet（客户端）都行。注意：你在一个窗口里输入的内容会出现在另一个窗口里，反之亦然。你需要按回车键，字节才会被传输。

5. 在 netcat 窗口中按 ctrl-C 退出程序。注意 telnet 程序也会立即退出。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">3 用操作系统流套接字编写网络程序</span>

在这个热身实验的下一部分，你将编写一个短小的程序，通过互联网抓取一个网页。你将用到 Linux 内核（以及大多数其他操作系统）提供的一项功能：在两个程序之间创建一条可靠的双向有序字节流——一个程序运行在你的计算机上，另一个运行在互联网另一端的一台不同计算机上（例如 Apache 或 nginx 这样的 Web 服务器，或者 netcat 程序）。

这项功能称为**流套接字**（stream socket）。对你的程序和 Web 服务器来说，套接字看起来就像一个普通的文件描述符（file descriptor）——类似磁盘上的文件，或者 stdin/stdout 这两个 I/O 流。当两个流套接字连接起来后，写入其中一个套接字的任何字节，最终都会按相同顺序从另一台计算机上的另一个套接字里出来。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> "套接字看起来就像普通文件描述符"这句话是理解 Unix 网络编程的钥匙，也直接对应 CSAPP 第 10、11 章的两条主线。第 10 章讲过 Unix 的 I/O 抽象：一切皆文件，`read()`/`write()` 对磁盘文件、管道、终端一视同仁；第 11 章则把网络接入这一统一抽象——`socket()` 返回的描述符与 `open()` 返回的描述符地位完全相同，所以 CSAPP 的 echo 服务器才能用同一套 `rio_readlineb()` 同时处理监听描述符和已连接描述符。这里还埋了一个重要事实：套接字上的"读"与"写"语义由内核里的 TCP 缓冲区支撑——写是拷进发送缓冲区后立即返回，读是从接收缓冲区里取；你在 Lab 4 之后要亲手实现的，正是这层缓冲区背后的全部逻辑。

然而在现实中，互联网并不提供可靠字节流的服务。互联网真正做的唯一一件事，是对被称为互联网数据报（Internet datagram）的短小数据块"尽力而为"（best-effort）地把它们送到目的地。每个数据报包含一些元数据（首部，headers），指明诸如源地址与目的地址之类的东西——它从哪台计算机来、要到哪台计算机去——以及一些要送达目的计算机的有效载荷（payload）数据（最多约 1500 字节）。

尽管网络会尽力送达每一个数据报，实践中数据报可能 (1) 丢失，(2) 乱序送达，(3) 内容被改动后送达，甚至 (4) 被复制、多次送达。把"尽力而为的数据报"（互联网提供的抽象）变成"可靠字节流"（应用通常想要的抽象），通常是连接两端的操作系统的职责。

两台计算机必须相互协作，确保流中的每个字节最终都按其在线中的正确位置送达另一侧的流套接字。它们还必须互相告知自己准备接受多少数据，并确保不发送超过对方愿意接受的数量。这一切都通过一个 1981 年定下的、双方约定的方案完成：传输控制协议（Transmission Control Protocol），简称 TCP。

在本实验中，你只需使用操作系统对 TCP 的既有支持。你将编写一个名为 "webget" 的程序：创建一个 TCP 流套接字，连接到 Web 服务器，抓取一个页面——和你在本实验前面手工做的事情差不多。在后续实验中，你将实现这个抽象的另一侧：亲自实现传输控制协议，用不那么可靠的数据报构造出可靠字节流。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这段话是整门课的纲领，值得用端到端原则（end-to-end principle）读一遍。互联网的网络层只承诺"尽力而为"：丢包、乱序、篡改、重复全都可能发生，这些差错模式你会在 Lab 1 的重组器和 Lab 3 的重传逻辑里逐一对付。为什么不干脆让路由器保证可靠交付？因为可靠性本质上是应用级需求——文件传输要一字不差，语音通话反而能容忍丢失——把功能放在网络核心会让每一台路由器为不需要它的流量买单。TCP 在 1981 年由 RFC 793 定义，37 年后你们仍要与真实互联网上的 TCP 实现互通（Lab 4 的测试会与 Linux 内核 TCP 对话），这本身就是分层（layering）与稳定接口设计力量的最好证明。CSAPP ch11 里你只调用过 TCP，这学期你要成为实现 TCP 的人。

#### <span style="color:#c0392b;">3.1 开始——抓取并编译初始代码（starter code）</span>

1. 实验作业将使用一个名为 "Sponge" 的初始代码库。在你的虚拟机上运行 `git clone https://github.com/cs144/sponge` 抓取实验源码。
2. 可选：欢迎把你的仓库备份到一个私有的 GitHub/GitLab/Bitbucket 仓库（例如参考 https://stackoverflow.com/questions/10065526/github-how-to-make-a-fork-of-public-repository-private 的做法），但请务必确保你的工作保持私有。
3. 进入 Lab 0 目录：`cd sponge`
4. 创建一个用于编译实验软件的目录：`mkdir build`
5. 进入 build 目录：`cd build`
6. 设置构建系统：`cmake ..`
7. 编译源码：`make`（可以运行 `make -j4` 使用四个处理器并行编译）。
8. 在 build 目录之外，打开并开始编辑 writeups/lab0.md 文件。这是你实验报告（writeup）的模板，将包含在你的提交里。

#### <span style="color:#c0392b;">3.2 现代 C++：大体安全，但依然快速而底层</span>

实验作业将采用当代 C++ 风格，利用较新的（2011）语言特性尽可能安全地编程。这可能与你过去被要求书写 C++ 的方式不同。关于这种风格的参考资料，请阅读 C++ Core Guidelines（http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines）。

基本思想是：确保每个对象被设计成公共接口尽可能小、内部带有大量安全检查、难以被误用，并且知道如何清理自己。我们想避免"成对"的操作（例如 malloc/free，或 new/delete）——这类操作的后半部分可能不会发生（例如函数提前返回或抛出异常时）。取而代之的是：获取资源的操作放在对象的构造函数里完成，相反的操作在析构函数里完成。这种风格称为"资源获取即初始化"（Resource acquisition is initialization，RAII）。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> RAII 对你并不陌生——CSAPP ch9 的虚拟内存与动态内存分配一章实际上是在教你怎么"踩坑"：`malloc/free` 忘配对导致内存泄漏，`free` 两次触发双重释放（double free），解引用悬空指针（dangling pointer）产生段错误。RAII 的回答是把"资源生命周期"绑定到"对象生命周期"上：栈对象离开作用域时析构函数必然执行，`std::string`、`std::vector`、这里的 `FileDescriptor` 都靠这个保证在异常传播路径上也不泄漏资源。这也解释了本课禁令的来由：裸 `new`/`delete` 是手动配对操作，恰好是 RAII 要消灭的东西；而 C 风格 `char*` 字符串没有长度信息、不管理内存，是缓冲区溢出的温床。你可以把 sponge 框架视为 CSAPP 风格系统编程之上的一层现代 C++ 安全封装——底层语义不变，边界检查前移。

具体而言，我们希望你：

- 把 https://en.cppreference.com 的语言文档当作常用资源。
- 绝不使用 `malloc()` 或 `free()`。
- 绝不使用 `new` 或 `delete`。
- 基本上绝不使用裸指针（`*`），只在必要时使用"智能"指针（`unique_ptr` 或 `shared_ptr`）。（在 CS144 中你不会需要用到它们。）
- 避免模板、线程、锁和虚函数。（在 CS144 中你不会需要用到它们。）
- 避免 C 风格字符串（`char *str`）或字符串函数（`strlen()`、`strcpy()`）。它们相当容易出错。请改用 `std::string`。
- 绝不使用 C 风格强制类型转换（例如 `(FILE *)x`）。确有必要时使用 C++ 的 `static_cast`（在 CS144 中你一般用不到）。
- 优先以 const 引用传递函数参数（例如：`const Address &address`）。
- 每个变量都尽量是 `const`，除非它需要被修改。
- 每个方法都尽量是 `const`，除非它需要修改对象。
- 避免全局变量，并让每个变量的作用域尽可能小。
- 交作业之前，请运行 `make format` 统一代码风格。

**关于使用 Git**：实验以 Git（版本控制）仓库的形式分发——一种记录变更、为版本创建检查点以便调试、追踪源码出处的手段。工作过程中请频繁做小提交，并使用能说明"改了什么、为什么改"的提交信息。柏拉图式的理想是：每个提交都应该能编译通过，并稳定地向"更多测试通过"推进。小的"语义化"提交有助于调试（如果每个提交都能编译、且提交信息只清楚描述一件事，调试会容易得多），也通过记录你随时间的稳步进展来保护你免受作弊指控——这是一项在所有包含软件开发的职业中都有用的技能。评分助教会阅读你的提交信息，以理解你是如何一步步开发出实验解答的。如果你还没学过 Git，请到 CS144 办公时间（office hours）寻求帮助，或查阅教程（例如 https://guides.github.com/introduction/git-handbook）。最后，欢迎把你的代码存放在 GitHub、GitLab、Bitbucket 等私有仓库中，但请务必确保你的代码不能被公开访问。

#### <span style="color:#c0392b;">3.3 阅读 Sponge 文档</span>

为了支持这种编程风格，Sponge 的类把操作系统函数（可以从 C 调用的那类函数）包装成"现代" C++。

1. 用浏览器阅读初始代码的文档：https://cs144.github.io/doc/lab0。
2. 请特别留意 FileDescriptor、Socket、TCPSocket 与 Address 这几个类的文档。（注意：Socket 是一种 FileDescriptor，TCPSocket 是一种 Socket。）
3. 现在，找到并阅读 libsponge/util 目录中描述这些类接口的头文件：`file_descriptor.hh`、`socket.hh` 和 `address.hh`。

#### <span style="color:#c0392b;">3.4 编写 webget</span>

是时候实现 webget 了——一个利用操作系统的 TCP 支持和流套接字抽象在互联网上抓取网页的程序，就像你在本实验前面手工做过的那样。

1. 在 build 目录中，用文本编辑器或 IDE 打开 `../apps/webget.cc`。
2. 在 `get_URL` 函数中，找到以 "// Your code here" 开头的注释。
3. 按该文件的说明实现这个简单的 Web 客户端，使用你前面用过的 HTTP（Web）请求格式。使用 TCPSocket 与 Address 类。
4. 提示：
   - 请注意在 HTTP 中，每一行都必须以 "\r\n" 结尾（只用 "\n" 或 `endl` 是不够的）。
   - 向套接字写完你的请求之后，要通过结束你的出站字节流（从你的套接字到服务器套接字的那个方向）来告诉服务器：你的请求已全部发完。方法是调用 TCPSocket 的 `shutdown` 方法并传入 `SHUT_WR` 参数。作为回应，服务器会给你发送一个应答，然后结束它自己的出站字节流（从服务器的套接字到你的套接字的方向）。你会发现你的入站字节流结束了——当你读完了来自服务器的整个字节流时，你的套接字会到达 "EOF"（文件结束）。（如果你不关闭自己的出站字节流，服务器会等一段时间看你是否还要发更多请求，也就不会结束它的出站字节流。）
   - 务必读取并打印来自服务器的全部输出，直到套接字到达 "EOF"（文件结束）——只调用一次 `read` 是不够的。
   - 我们预计你需要写大约十行代码。
5. 运行 `make` 编译你的程序。如果看到错误信息，需要先修复才能继续。
6. 用 `./apps/webget cs144.keithw.org /hello` 测试你的程序。这与你在浏览器里访问 http://cs144.keithw.org/hello 看到的结果相比如何？与 2.1 节手工操作的结果相比呢？欢迎随意实验——用任何你喜欢的 http URL 来测试它！
7. 看起来工作正常之后，运行 `make check_webget` 执行自动化测试。在实现 `get_URL` 函数之前，你应预期看到：

```
1/1 Test #25: lab0_webget ......................***Failed  0.00 sec
Function called: get_URL(cs144.keithw.org, /hasher/xyzzy).
Warning: get_URL() has not been implemented yet.
ERROR: webget returned output that did not match the test's expectations
```

完成本作业之后，你会看到：

```
4/4 Test #4: lab0_webget ...................... Passed  0.14 sec
100% tests passed, 0 tests failed out of 4
```

8. 评分助教会用与 `make check` 不同的主机名和路径来运行你的 webget 程序——所以请确保它不是只在 `make check` 所用的主机名和路径下才能工作。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 第 4 步的 `shutdown(SHUT_WR)` 提示值得展开：它只关闭"写方向"而保留"读方向"，相当于对对方说"我说完了，但我还在听"。这解决了 HTTP/1.0 时代的一个经典难题——服务器如何知道请求何时结束？答案由双方配合给出：客户端发完请求后半关闭写方向，服务器读到 EOF 便知道可以开始应答。对比 CSAPP echo 服务器里 `ctrl-D`/close 的处理：直接 `close()` 会同时关掉两个方向，客户端此后连应答都收不到。这正是后面 Lab 4 里"干净关闭"（clean shutdown）四前提、半开放状态等概念的雏形——TCP 的关闭远比建立精细，因为它要保证**两个方向各自**的字节流都完整可靠地送达。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">4 内存中的可靠字节流</span>

到现在为止，你已经看到"可靠有序字节流"这一抽象在跨互联网通信中是多么有用——即使互联网本身只提供"尽力而为"（不可靠）的数据报服务。

本周实验的最后，你将在一台计算机的内存中实现一个提供这种抽象的对象。（你在 CS 110 里可能做过类似的事情。）字节写在"输入"侧，然后可以从"输出"侧按相同序列读出。字节流是有限的：写者可以结束输入，此后不能再写入字节。当读者读到流的末端时，会遇到 "EOF"（文件结束），此后不能再读出字节。

你的字节流还将是**流量控制的**（flow-controlled）：它以一个特定容量（capacity）初始化——即它愿意在自己内存中存储的最大字节数。字节流会限制写者所能写入的字符串长度，以确保流不超过其存储容量。当读者读出字节、把它们从流中排空（drain）时，写者获准写入更多。

你的字节流只在单线程中使用——你不必担心并发读写者、加锁或竞态条件。

以下是写者侧的接口：

```cpp
// Write a string of bytes into the stream. Write as many
// as will fit, and return the number of bytes written.
size_t write(const std::string &data);

// Returns the number of additional bytes that the stream has space for
size_t remaining_capacity() const;

// Signal that the byte stream has reached its ending
void end_input();

// Indicate that the stream suffered an error
void set_error();
```

以下是读者侧的接口：

```cpp
// Peek at next "len" bytes of the stream
std::string peek_output(const size_t len) const;

// Remove ``len'' bytes from the buffer
void pop_output(const size_t len);

// Read (i.e., copy and then pop) the next "len" bytes of the stream
std::string read(const size_t len);

bool input_ended() const;  // `true` if the stream input has ended

bool eof() const;  // `true` if the output has reached the ending

bool error() const;  // `true` if the stream has suffered an error

size_t buffer_size() const;  // the maximum amount that can currently be peeked/read

bool buffer_empty() const;  // `true` if the buffer is empty

size_t bytes_written() const;  // Total number of bytes written

size_t bytes_read() const;  // Total number of bytes popped
```

请打开 `libsponge/byte_stream.hh` 和 `libsponge/byte_stream.cc` 文件，实现一个提供该接口的对象。在开发你的字节流实现的过程中，可以用 `make check_lab0` 运行自动化测试。

**接下来是什么？** 在接下来的四周里，你将实现一个系统，它提供同样的接口——只是不再在内存中，而是跨越一个不可靠的网络。这就是传输控制协议。

> <span style="color:#1e8449;">**[note] Note（译者注）:**</span> 这个 ByteStream 是未来四周一切工作的地基，值得从设计角度多看两眼。它本质上是一个有界缓冲区（bounded buffer）：`write` 是生产者接口，`peek/pop` 拆成了两步——先窥视（peek）再弹出（pop）——这比直接 `read` 更灵活，Lab 1 的重组器就会利用"先 peek 判断前缀、再 pop 提交"的顺序来保证字节只会按序进入流。`remaining_capacity()` 返回的剩余容量就是后续 TCP 接收窗口（receive window）的计算原料：Lab 2 里窗口大小 = capacity 减去 ByteStream 中暂存的字节数，这正是"流量控制"从抽象落地为代码的方式。`end_input()`/`eof()` 这对状态则是 TCP FIN 标志的内存版雏形。换句话说，你在 CSAPP ch9 里见过的"缓冲区管理"问题（容量有限、生产消费速率不匹配），在这里以网络流量控制的形式再次登场。

### <span style="color:#2471a3;">**[section]**</span> <span style="color:#c0392b;">5 提交（Submit）</span>

1. 提交时，请只修改 webget.cc 以及 libsponge 顶层的源码文件（byte_stream.hh 和 byte_stream.cc）。请不要修改任何测试或 libsponge/util 中的辅助代码。
2. 交任何作业之前，请按顺序运行：
   (a) `make format`（规范代码风格）
   (b) `make`（确保代码能编译）
   (c) `make check_lab0`（确保自动化测试通过）
3. 完成编辑 writeups/lab0.md，填入本作业花费的小时数以及任何其他意见。
4. 准备好提交时，请按照 https://cs144.github.io/submit 上的说明操作。请确保在提交之前已经 commit 所有你打算提交的内容。
5. 如有任何问题，请尽快在周二晚间的实验课上告知课程组，或在 Piazza 上发帖提问。祝你好运，欢迎来到 CS144！

---

> <span style="color:#7f8c8d;">脚注：本讲义为 Lab 0（热身实验）原始 handout。第 2 节的手工操作（telnet/netcat/SMTP）对应慕课 Unit 2 的 socket 与应用层内容；第 4 节的 ByteStream 是 Lab 1（StreamReassembler）、Lab 2（TCPReceiver）、Lab 3（TCPSender）的共同地基。</span>
