# 我有一个Hello World的C++程序编译不过
>date: 2011-04-02T14:33:57+08:00


在StackOverflow上有这样[一个贴子](http://stackoverflow.com/questions/5508110/why-is-this-program-erroneously-rejected-by-three-c-compilers)，楼主说，我有下面这样的一个C++程序，为什么编译不通过啊。其让我想起了以前的这两个帖子《[编程真难啊](/2009/%E7%BC%96%E7%A8%8B%E7%9C%9F%E9%9A%BE%E5%95%8A.md "编程真难啊")》和《[给我一个序列号](https://coolshell.cn/articles/1693.html "给我一个序列号")》。**仅以此篇文章祝大家假期快乐吧**。


![hello world 程序](http://i.stack.imgur.com/JQXWL.png "hello world 程序")hello world 程序
楼主还给出了相关的编译出错的信息（相信你一看就明白问题在哪里了，你应该还会发出一声“靠”！！！）


先是用Visual C++ 2010编译



```
c:\dev>cl /nologo helloworld.png
cl : Command line warning D9024 : unrecognized source file type 'helloworld.png', object file assumed
helloworld.png : fatal error LNK1107: invalid or corrupt file: cannot read at 0x5172
```

再用G++ 4.5.2编译




```
c:\dev>g++ helloworld.png
helloworld.png: file not recognized: File format not recognized
collect2: ld returned 1 exit status
```

再用clang编译



```
c:\dev>clang++ helloworld.png
helloworld.png: file not recognized: File format not recognized
collect2: ld returned 1 exit status
clang++: error: linker (via gcc) command failed with exit code 1 (use -v to see invocation)
```

不过，最强大的，有人居然给出了一个fix，靠！  

（下面的图片是一个4M大的gif动画，演示了整个过程，下载可能需要一定的时间。）


[![hello world 的解决方案](http://i.imgur.com/QlGpd.gif "hello world 的解决方案")](http://i.imgur.com/QlGpd.gif)hello world 的解决方案 （图片有点大4M，请耐心等待下载）
真是BT啊，呵呵。**仅以此篇文章祝大家假期快乐吧**。


