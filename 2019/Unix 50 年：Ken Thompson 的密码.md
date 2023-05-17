# Unix 50 年：Ken Thompson 的密码
>date: 2019-11-03T14:12:54+08:00


![](/assets/images/coolshell.cn/wp-content/uploads/2019/11/ken.dennis-300x186.jpeg)50年前，除了Apollo上天之外，还有一个大事的发生，就是Unix操作系统的诞生，若干年前我写过《Unix的传奇，[上篇](https://coolshell.cn/articles/2322.html)，[下篇](https://coolshell.cn/articles/2324.html)》，Unix是我入行前十年伴我成长的操作系统，虽然现在Linux早已接过了Unix的时代交接棒，但是，Unix文化对我个人的技术观影响是非常大的（注：《[Unix编程艺术](https://book.douban.com/subject/1467587/)》是一本对影响我很深的书），而对于 [Ken Thompson](https://en.wikipedia.org/wiki/Ken_Thompson) 和 [Dennis Ritchie](https://en.wikipedia.org/wiki/Dennis_Ritchie) 这两位 Unix 的缔造者，也是计算机圈中的神一般的人物。今天，Dennis已经去逝，Ken在Google里跟 Rob Pike和 Robert Griesemer 这两位大神在开发Go语言。


P.S. 今年，我一直想写篇Unix 50周年纪念的文章，但一直无从下手，因为不想写过大的命题，如果能写个轶事最好不过。正好过完国庆节，技术圈里有个“热搜”——Ken Thompson的密码。但一直没有时间，所以拖到今天才写下来。


正文开始，2014年，有个叫Leah Neukirchen的程序员（[blog](https://leahneukirchen.org/blog/)）在 BSD 3 的源代码中的 `[/etc/passwd](https://leahneukirchen.org/blog/archive/2019/10/ken-thompson-s-unix-password.html)` 看到了早年Unix黑客们的被 hash了的密码，该文件如下所示：




```
root:OVCPatZ8RFmFY:0:10:Ernie Co-vax,4156427925:/:
daemon:*:1:1:The devil himself:/:
bill:.2xvLVqGHJm8M:8:10:& Joy,4156424948:/usr/bill:/bin/csh
ozalp:m5syt3.lB5LAE:40:10:& Babaoglu,4156423806:/usr/ozalp:/bin/csh
sklower:8PYh/dUBQT9Ss:2:10:Keith &,4156424972:/usr/staff/sklower:/bin/csh
kridle:4BkcEieEtjWXI:3:10:Bob &,4156426744:/usr/staff/kridle:/bin/csh
kurt:olqH1vDqH38aw:4:10:& Shoens,4156420572:/usr/staff/kurt:/bin/csh
schmidt:FH83PFo4z55cU:7:10:Eric &,4156424951:/usr/staff/schmidt:/bin/csh
hpk:9ycwM8mmmcp4Q:9:10:Howard Katseff,2019495337:/usr/staff/hpk:/bin/csh
tbl:cBWEbG59spEmM:10:10:Tom London,2019492006:/usr/staff/tbl:
jfr:X.ZNnZrciWauE:11:10:John Reiser:/usr/staff/jfr:
mark:Pb1AmSpsVPG0Y:12:10:& Horton,4156428311:/usr/staff/mark:/bin/csh
dmr:gfVwhuAMF0Trw:42:10:Dennis Ritchie:/usr/staff/dmr:
ken:ZghOT0eRm4U9s:52:10:& Thompson:/usr/staff/ken:
sif:IIVxQSvq1V9R2:53:10:Stuart Feldman:/usr/staff/sif:
scj:IL2bmGECQJgbk:60:10:Steve Johnson:/usr/staff/scj:
pjw:N33.MCNcTh5Qw:61:10:Peter J. Weinberger,2015827214:/usr/staff/pjw:/bin/csh
bwk:ymVglQZjbWYDE:62:10:Brian W. Kernighan,2015826021:/usr/staff/bwk:
uucp:P0CHBwE/mB51k:66:10:UNIX-to-UNIX Copy:/usr/spool/uucp:/usr/lib/uucp/uucico
srb:c8UdIntIZCUIA:68:10:Steve Bourne,2015825829:/usr/staff/srb:
finger::199:199:The & Program:/usr/ucb:/usr/ucb/finger
who::199:199:The & Program:/usr/ucb:/bin/who
w::199:199:The & Program:/usr/ucb:/usr/ucb/w
mckusick:AAZk9Aj5/Ue0E:201:10:Kirk &,4156424948:/usr/staff/mckusick:/bin/csh
peter:Nc3IkFJyW2u7E:202:10:& Kessler,4156424948:/usr/staff/peter:/bin/csh
henry:lj1vXnxTAPnDc:203:10:Robert &,4156424948:/usr/staff/henry:/bin/csh
jkf:9ULn5cWTc0b9E:209:10:John Foderaro,4156424972:/usr/staff/jkf:/bin/csh
fateman:E9i8fWghn1p/I:300:10:Richard &,4156421879:/usr/staff/fateman:/bin/csh
fabry:d9B17PTU2RTlM:305:10:Bob &,4156422714:/usr/staff/fabry:/bin/csh
network:9EZLtSYjeEABE:501:50:*:/usr/net/network:/usr/net/network/nsh
tty::504:50::/:/bin/tty我
```

（注，以前Unix是一个服务器，所有人都用一个终端到服务器上进行操作，于是，这个服务上的 `/etc/passwd` 下保存着所有的人的登录密码，能让所有的人都能读到，为了不让别人猜到，这个文件中的密码保存（第二列）被做过哈希处理）


这位程序员一看，这些个用户不就是[Dennis Ritchie](https://en.wikipedia.org/wiki/Dennis_Ritchie), [Ken Thompson](https://en.wikipedia.org/wiki/Ken_Thompson), [Brian W. Kernighan](https://en.wikipedia.org/wiki/Brian_Kernighan), [Steve Bourne](https://en.wikipedia.org/wiki/Stephen_R._Bourne), [Bill Joy](https://en.wikipedia.org/wiki/Bill_Joy) 这些神人的密码吗？！于是，他想看看这些人用什么样的密码。考虑到当时的加密算法用的是基于DES的 [crypt(3)](https://minnie.tuhs.org/cgi-bin/utree.pl?file=V7/usr/man/man3/crypt.3) 算法（这个算法今天还在用，像Perl/PHP/Python/Ruby都提供`crypt()` 函数），而且当时的密码最长只支持8个长度，所以，感觉还是很容易暴力破解的。


一般来说，暴力破解的这种hash密码的工具主要是用[hashcat](https://hashcat.net/) 或 [john](https://www.openwall.com/john/) ，很快，Leah 破解了大多数人的密码，因为大多数都使用的是比较弱的密码，比如： [Brian W. Kernighan](https://en.wikipedia.org/wiki/Brian_Kernighan) （`bwk`）使用了 `/.,/.,` 这样的密码，而 [Dennis Ritchie](https://en.wikipedia.org/wiki/Dennis_Ritchie) （`dmr`）则使用了 `dmac` 这样的密码。然后，在破解到 Ken Thompson的密码时，搞不定了，花了好几天穷举完了所有的小写字母+数字都没有找到。


因为这个`crypt`的算法也是Ken Thompson 和 Robert Morris 写的，他们在40年前就发现，原来的hash算法太快了，这样很容易被暴力穷举，于是在第七版的Unix（1979年发布），他们把算法改成DES的算法，就是要让这个算法变慢。详细地说，用户密码被截断为八个字符，每个字符仅被压缩为7位。这形成56位DES密钥。然后，该密钥用于加密全零位块，然后再次使用相同的密钥对密文进行加密，依此类推，总共进行了25次DES加密。感觉跟区块链的“挖矿”有点像。**在最早的Unix计算机上，这个算法需要花了整整一秒钟的时间来计算密码哈希**。


这几十年来，计算机的计算速度根据摩尔定律至少double了20次，所以，DES算法已经很容被攻击了，然而，对于Ken Thompson的密码，在2014年还是很不容易被破解的，因为，**如果要加上所有的大小写字符数字和其它特殊字符，那么，在2014年，就算用最快的GPU来穷举所有的8位长度的密码，也需要花上至少2年以上的时间**。


在2019年10月份，在 [The Unix Heritage Society](https://www.tuhs.org/) 这个社区中，[这个事又被人问起来](https://inbox.vuxu.org/tuhs/6dceffe228804a76de1e12f18d1fc0dc@inventati.org/)，说以前有个人破解这些密码，不知道有没有全破解出来了？于是Leah看到了，就回应说，那个人是我，但是还是没干出来……于是好些人进来留言。


5天后，2019年10月08日，一个来自澳大利亚的程序员Nigel Williams说，[Ken的密码我破解出来了](https://inbox.vuxu.org/tuhs/CACCFpdx_6oeyNkgH_5jgfxbxWbZ6VtOXQNKOsonHPF2=747ZOw@mail.gmail.com/)，哈希串`ZghOT0eRm4U9s` 明文是 `p/q2-q4!`（果然是有数字有特殊字符），小伙说，我在 AMD Radeon Vega 64 的 GPU上运行了 `hashcat` 这个命令，干了我 4天多，每秒钟的“配速”是930MH/s （每秒钟9亿3千万次hash运算）。然后，[Ken Thompson 也留言到 “恭喜”](https://inbox.vuxu.org/tuhs/CAG=a+rj8VcXjS-ftaj8P2_duLFSUpmNgB4-dYwnTsY_8g5WdEA@mail.gmail.com/) ，这样，Ken 的密码在40年后被破解了……


马上，就有人问到，这个密码是不是国际象棋的走棋？嗯，很像中国象棋中的“车五进一”，“马三退一”，这个密码中的 `p` 代表 `pawn` 小兵，从 `q2` 的位置走到 `q4`，这个看来是国际象棋中的开局进兵——用来做登录密码，非常合适。而且，Ken Thompson 在 Unix中写下的一个国际象棋的程序 [Belle](https://en.wikipedia.org/wiki/Belle_(chess_machine))，在1978年首次参加[计算机协会的北美计算机国际象棋锦标赛](https://en.wikipedia.org/wiki/North_American_Computer_Chess_Championship)时，它获得了第一个冠军头衔，其搜索深度为八层。之后又赢得了四次冠军。1983年，它也成为第一台获得国际象棋“大师”称号的计算机。所以，Ken用这个做密码相当make sense!


![](/assets/images/coolshell.cn/wp-content/uploads/2019/11/ken.chess_.jpg)Ken在贝尔实验室调程序（图片来源：[IEEE SPECTRUM](https://spectrum.ieee.org/tech-history/silicon-revolution/in-1983-this-bell-labs-computer-was-the-first-machine-to-become-a-chess-master)）


当然，还有一个人的密码是所有人里最难破解的，这个人就是[Bill Joy](https://en.wikipedia.org/wiki/Bill_Joy)，他最初作为加州大学伯克利分校的研究生，在校期间着手改进Unix 内核，并管理BSD发行版。他最著名的贡献是ex和vi编辑器以及C shell。在Sun公司成立6个月后，他正式成为公司的联合创始人，他在Sun公司的推动了NFS，SPARC处理器，以及Java语言。他还是一个风险投资人员。


在Ken的密被破解后两周（2019年10月19日），有人号称已经破解了Bill的密码，他在[邮件组中这样写到](https://minnie.tuhs.org/pipermail/tuhs/2019-October/019124.html)：



> 一开始，我使用了大小写字符和数字，8位长度来破解所有的组合，花了我6天的时间，失败了。然后，我开始尝试只用小写字母和控制字符，结果在40分钟内就破解了。但是因为Bill现健在，所以，只要bill同意他才公布这个密码。
> 
> 


在密码里存控制字符？这脑洞，Ctrl+C么？破解者还说，他在一个有三个结点的DELL 的HPC集群上完成这个工作，每个结点包括两个 Tesla V100 nVidia GPU 的显卡，一共30720个CUDA核…… 关于这个显卡多少钱，你可以上网搜吧…… 相当于一块劳力士吧……（我估计这组机器平时是用来挖矿的……[狗头]）


好了，我们来看一下这个 `/etc/passwd` 中的这些人的密码是什么样的，**但最主要的是向这些为人类做过巨大贡献的程序员科学家们致敬**！


* **[Ken Thompson](https://en.wikipedia.org/wiki/Ken_Thompson)**  

除了是Unix、B语言和Go语言作者之外，他还贡献过正则表达式，QED/ed编辑器，UTF-8编码定义，以及计算机国际象棋Belle……

|  |  |  |
| --- | --- | --- |
| 登录名 | **哈希串** | **密码** |
| `ken` | `ZghOT0eRm4U9s` | `p/q2-q4!` |
* **[Dennis Ritchie](https://en.wikipedia.org/wiki/Dennis_Ritchie)**  

Unix和C语言之父，与Ken于1983年获图灵奖，1990年美国国家海明奖章，于2011年去世。

|  |  |  |
| --- | --- | --- |
| 登录名 | **哈希串** | **密码** |
| `dmr` | `gfVwhuAMF0Trw` | `dmac` |
* **[Brian W. Kernighan](https://en.wikipedia.org/wiki/Brian_Kernighan)**  

AWK的作者，是AWK中的“K”，也是与Dennis写的K&C的C语言编程书中的“K”，他还编写了很多Unix的其它程序，如：`ditroff`，而且，设计了著名的[启发式算法](https://en.wikipedia.org/wiki/Heuristic)。

|  |  |  |
| --- | --- | --- |
| 登录名 | **哈希串** | **密码** |
| `bwk` | `ymVglQZjbWYDE` | `/.,/.,` |
* **[Stephen R. Bourne](https://en.wikipedia.org/wiki/Stephen_R._Bourne)**  

Bourne shell（`sh`）的作者，Unix Shell作者，同时也是Unix调试器的作者。

|  |  |  |
| --- | --- | --- |
| 登录名 | **哈希串** | **密码** |
| `srb` | `c8UdIntIZCUIA` | `bourne` |
* **[Eric Schmidt](https://en.wikipedia.org/wiki/Eric_Schmidt)**  

你可能知道他是Google的CEO，苹果的董事，但是你可能不知道，他当年是是贝尔实施室的实习生，他对Unix的词法分析器 Lex 进行为了完全的重写。他的密码是中的wendy应该是他的妻子。

|  |  |  |
| --- | --- | --- |
| 登录名 | **哈希串** | **密码** |
| `schmidt` | `FH83PFo4z55cU` | `wendy!!!` |
* **[Stuart Feldman](https://en.wikipedia.org/wiki/Stuart_Feldman)**  

他除了是Unix系统小组的成员，他还是第一个Fortran 77 编译器的作者，也是 `make` 的作者。他还是楼上Shmidt慈善基金会的科学负责人，在Google/IBM Research任过职，也担任过ACM的主席。

|  |  |  |
| --- | --- | --- |
| 登录名 | **哈希串** | **密码** |
| `sif` | `IIVxQSvq1V9R2` | `axolotl` |
* **[Mark Horton](https://en.wikipedia.org/wiki/Mary_Ann_Horton)**  

Unix贡献者，包括vi和curses，后来变性为女性，新的名字叫Mary Ann Horton。原来的照片在[Unix Guru Universe](http://www.ugu.com/sui/ugu/show?I=info.Mark_R._Horton)

|  |  |  |
| --- | --- | --- |
| 登录名 | **哈希串** | **密码** |
| `mark` | `Pb1AmSpsVPG0Y` | `uio` |
* **[Kirk McKusick](https://en.wikipedia.org/wiki/Marshall_Kirk_McKusick)**  

BSD贡献者，主要负责文件系统UFS以及fsck命令，同时也是`gprof`的贡献者，公开的同性恋者。

|  |  |  |
| --- | --- | --- |
| 登录名 | **哈希串** | **密码** |
| `mckusick` | `AAZk9Aj5/Ue0E` | `foobar` |
* **[Richard Fateman](https://en.wikipedia.org/wiki/Richard_Fateman)**  

他在伯克利的VAX UNIX系统的开发工作中发挥了重要作用，以及开发了 [Franz Lisp](https://en.wikipedia.org/wiki/Franz_Lisp)。

|  |  |  |
| --- | --- | --- |
| 登录名 | **哈希串** | **密码** |
| `fateman` | `E9i8fWghn1p/I` | `apr1744` |
* **Peter Kessler**  

这位老兄能在网上查到的资料基本没有，可以查到他是 `gprof` 的贡献者，以及有名字的[gprof的一篇论文](https://web.eecs.umich.edu/~weimerw/2009-4610/reading/graham-gprof.pdf)

|  |  |  |
| --- | --- | --- |
| 登录名 | **哈希串** | **密码** |
| `peter` | `Nc3IkFJyW2u7E` | `...hello` |
* **Kurt Shoens**  

BSD电子邮件开发者。Unix早期版本中使用 `uux` 和 `sendmail` 来进行远程消息传递，1978年，Kurt为Unix编写了一个邮件用户代理 Berkeley Mail。相关的历史可以参看[这篇文章](http://heirloom.sourceforge.net/mailx_history.html)。

|  |  |  |
| --- | --- | --- |
| 登录名 | **哈希串** | **密码** |
| `kurt` | `olqH1vDqH38aw` | `sacristy` |
* **[John Foderaro](https://franz.com/about/press_room/foderaro_2-2-2015.lhtml)**  

他为Berkeley的Lisp语言编写原始的编译器，Lisp语言是一种类似于数据代数的语言，在计算机历史上有和C语言一样的作用。后来他成立了Franz公司，主要开发和部署图形搜索解决方案。

|  |  |  |
| --- | --- | --- |
| 登录名 | **哈希串** | **密码** |
| `jkf` | `9ULn5cWTc0b9E` | `sherril.` |
* **[Peter J. Weinberger](https://en.wikipedia.org/wiki/Peter_J._Weinberger)**  

他就是AWK中的那个“W”，同时也是Fortan编译器f77的贡献者，后来是[Renaissance Technologies](https://en.wikipedia.org/wiki/Renaissance_Technologies) （一家对冲基金）的CTO，现在在Google工作，

|  |  |  |
| --- | --- | --- |
| 登录名 | **哈希串** | **密码** |
| `pjw` | `N33.MCNcTh5Qw` | `uucpuucp` |
* **John Reiser**  

他主要工作是将Unix和C移植到了DEC VAX上，这个机器在学术界相当流行（陈皓注：我在1994年上大学的时候，就是在这个机器上学习的C语言）。这扩大了Unix和C的影响力。

|  |  |  |
| --- | --- | --- |
| 登录名 | **哈希串** | **密码** |
| `jfr` | `X.ZNnZrciWauE` | `5%ghj` |
* **[Steve Johnson](https://en.wikipedia.org/wiki/Stephen_C._Johnson)**  

曾在贝尔实验室和AT＆T工作近20年。他以Yacc，Lint，spell和Portable C编译器而闻名。后来他去了硅谷，加入了一些创业公司，主要从事编译器的工作，以及2D和3D图形，大规模并行系统和嵌入式系统的开发工作。现在他在Wave Computing从事机器学习的工作。

|  |  |  |
| --- | --- | --- |
| 登录名 | **哈希串** | **密码** |
| `scj` | `IL2bmGECQJgbk` | `pdq;dq` |
* **Bob Kridle**  

这位老兄的资料在没有太多，只能在 [Berkeley Unix 20 年](https://www.oreilly.com/openbook/opensources/book/kirkmck.html_original) 上看到他跟Ken Thompson混过一段时间。

|  |  |  |
| --- | --- | --- |
| 登录名 | **哈希串** | **密码** |
| `kridle` | `4BkcEieEtjWXI` | `jilland1` |
* **[Keith Sklower](https://people.eecs.berkeley.edu/~sklower/)**  

BSD 的一个程序员。从他的主页上可以看到他目前在Berkeley大学，信息分析师，主要研究一些网络通信相关的技术。

|  |  |  |
| --- | --- | --- |
| 登录名 | **哈希串** | **密码** |
| `sklower` | `8PYh/dUBQT9Ss` | `theik!!!` |
* **Robert Henry**  

网上的资料不多，只在[Life with Unix](https://www.tuhs.org/Archive/Documentation/Books/Life_with_Unix.pdf)这本电子书中查到，他写了 `error`

|  |  |  |
| --- | --- | --- |
| 登录名 | **哈希串** | **密码** |
| `henry` | `lj1vXnxTAPnDc` | `sn74193n` |
* **Howard Katseff**  

网上的资料不多，只在[Life with Unix](https://www.tuhs.org/Archive/Documentation/Books/Life_with_Unix.pdf)这本电子书中查到，他写了 `sdb` 和 `last`

|  |  |  |
| --- | --- | --- |
| 登录名 | **哈希串** | **密码** |
| `hpk` | `9ycwM8mmmcp4Q` | `graduat;` |
* **[Özalp Babaoğlu](https://en.wikipedia.org/wiki/%C3%96zalp_Babao%C4%9Flu)**  

土耳其计算机科学家，1981年在Berkeley担任 BSD Unix的首席设计师，曾经与Sun的创造人Bill Joy在BSD上实现了虚拟内存。

|  |  |  |
| --- | --- | --- |
| 登录名 | **哈希串** | **密码** |
| `ozalp` | `m5syt3.lB5LAE` | `12ucdort` |
* **[Bob Fabry](https://en.wikipedia.org/wiki/Bob_Fabry)**  

他主要推动美国国防部高级研究计划局DARPA采用了Unix系统

|  |  |  |
| --- | --- | --- |
| 登录名 | **哈希串** | **密码** |
| `fabry` | `d9B17PTU2RTlM` | `561cml..` |
* **Tom London**  

他和John Reiser在把Unix移植到了VAX-11机上。

|  |  |  |
| --- | --- | --- |
| 登录名 | **哈希串** | **密码** |
| `tbl` | `cBWEbG59spEmM` | `..pnn521` |


最后，再首尾呼应一下，在我的技术生涯中，Unix文化对我个人的技术观影响是非常大的，**我个人认为 Unix 就像摇滚乐一样，上世纪60年代-80年代，是整个人类最经典最光亮的时代，值得我们每个人向那个时代的人和事致敬！**


————————————————————————


P.S.


你可以浏览 Github 的 [unix-history-repo](https://github.com/dspinellis/unix-history-repo/tree/BSD-3-Snapshot-Development) 目录（注：本文给的这个链接不在master分支上），这个repo是40年前的代码，涵盖了从1970年创建时的2.5万行内核和26条命令到2017年为止广泛使用的2700万行系统。1.1GB的存储库包含大约一百万次提交和两千多次合并。通过[这个链接](http://www.dmst.aueb.gr/dds/pubs/jrnl/2016-EMPSE-unix-history/html/unix-history.html)你可以了解一下这个代码的历史！


下载这些代码需要你的1.5GB的硬盘空间，你可以查看各个大神写的代码，包括 Ken Thompson 和 Dennis的，以及相关的注释。


根据这些，你还可以找到 Ken Thompson的 Github账号 <https://github.com/ken> 以及别人为dmr建的github帐号 <https://github.com/dmr-1941-2011>


P.S.S


下面是一些和Unix相关的维基百科资料


* [History of Unix](https://en.wikipedia.org/wiki/History_of_Unix)
* [List of Unix systems](https://en.wikipedia.org/wiki/List_of_Unix_systems)
* [List of Unix commands](https://en.wikipedia.org/wiki/List_of_Unix_commands)
* [List of Unix daemons](https://en.wikipedia.org/wiki/List_of_Unix_daemons)
* [Research Unix](https://en.wikipedia.org/wiki/Research_Unix)
* [Berkeley Software Distribution](https://en.wikipedia.org/wiki/BSD_Unix)
* [Unix philosophy](https://en.wikipedia.org/wiki/Unix_philosophy)


还有Unix的社区：TUHS: The Unix Heritage Society – [The Unix Tree](http://minnie.tuhs.org/cgi-bin/utree.pl)


（全文完）


