# 在Web上运行Linux
>date: 2011-05-19T08:35:08+08:00


一个叫Fabrice Bellard的程序员写了一段Javascript在Web浏览器中启动Linux（[原网页](http://bellard.org/jslinux/)，我把这个网页iframe在了下面），目前，你只能使用Firefox 4和Chrome 11运行这个Linux。这不是什么假的模仿Linux的东西，这是实实在在的运行一个Linux。这一举动还引起了很多很牛人的关注，包括Javascript的创建者[Brendan Eich](https://twitter.com/#!/BrendanEich/status/70393502328045568)。


清除启动开始启动




随后，Fabrice Bellard发布了相关的技术说明：<http://bellard.org/jslinux/tech.html>，从这份文档中我们可以看到：


* 这个模似器完全由Javascript写成
* CPU仿真器使用的是[QEMU](http://qemu.org/)（接近于原古的486），为了装上Linux，其做了一些改动。
* Javascript的终端本来可以使用[termlib](http://www.masswerk.at/termlib/)，但他还是自己写了一个，因为OS的按键和Web浏览器不一样（[here](http://unixpapa.com/js/key.html)）
* Linux  使用了2.6.20内核，编译配置在[这里](http://bellard.org/jslinux/config_linux-2.6.20)，并做了一些[小改动](http://bellard.org/jslinux/patch_linux-2.6.20)。
* 磁盘用的是Ram Disk，在启动的时候装载。其文件系统由[Buildroot](http://buildroot.uclibc.org/) 和[BusyBox](http://www.busybox.net/)产生。
* 在Home目录下有一个hello.c的程序，你可以使用[TinyCC](http://bellard.org/tcc)编译（tcc，参看酷壳的[这篇文章](/2009/%E7%94%A8TCC%E5%8F%AF%E4%BB%A5%E5%B9%B2%E4%BA%9B%E4%BB%80%E4%B9%88%EF%BC%9F.md "用TCC可以干些什么？")）


从这个事我有这些感触，


1. 在Web上运行一个Linux的操作系统不是问题。那么在Web上还有什么不能做的吗？
2. Linux真是性能很高，在Javascript下运行感觉也不慢啊。
3. 真是Techno-Geek。


 


