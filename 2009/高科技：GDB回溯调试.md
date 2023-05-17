# 高科技：GDB回溯调试
>date: 2009-09-28T17:14:08+08:00


也许大家知道，GDB 版本7.0 (2009年9月release) 会是第一次开始支持Reversable Debugging （回溯调式技术），这是一种可以让在debug程序时当我们运行单步调试或是运行到断点时，可以以逆向执行程序的一种技术。（这是GNU的[新闻链接](http://www.gnu.org/software/gdb/news/reversible.html)）


下面是GDB7.0版本所支持的回溯调试的命令，其中包括，continue，step，以及调试方向的设置。


- **reverse-continue** (‘rc’) — 继续程序运行到断点，但是是逆向运行程序。

- **reverse-finish** — 逆向运行程序直到跳出本层函数。

- **reverse-next** (‘rn’) — 语句单步向后跟踪程序。

- **reverse-nexti** (‘rni’) — 指令单步向后一条指令。

- **reverse-step** (‘rs’) — 向后执行一条语句，单步进入。

- **reverse-stepi** — 向后执行一条指令，单步进入。

- **set exec-direction (forward/reverse)** — 设置程序执行方向，向前或向后。


在网上查了一下，发现VS2010好像也准备要支持这个东西，微软叫这个东西为“[Historical Debugging](http://blogs.msdn.com/ianhu/archive/2009/05/13/historical-debugging-in-visual-studio-team-system-2010.aspx)”。


这个东西，对于我这个老家伙来说比较新鲜，而且还有点诡异。我有点没跟上这个技术，不知道这个技术主要是用来干什么？对于程序的运行的回滚？这样一来，如果，我某条语句创建了一个线程，或是一个文件，逆向执行回去，莫非它还能把这些程序创建出来的资源回收啦？就算是能回收，要是我的某个程序向网络发了些数据出去，莫非它还能给我再收回来？也许我想得太极端了，不过好像目前对这个技术的原始需求的说明不是很多，所以真不知道这个技术除了很酷，还有什么？也许是我理解错了，希望大家指点一下。


