# 用TCC可以干些什么？
>date: 2009-05-10T15:22:15+08:00


Tiny C Compiler 是一个微型的 C 语言编译器，支持 Windows 和 Linux 平台。其项目主页是： <http://bellard.org/tcc/> 。你可以使用这个不到100K的编译器编译你的C文件，其支持C的预处理，编译，机器码汇编和链接。编译速度也超过了gcc，而且它支持ISO C99标准，并且，tcc还包括了一些内存和数组边界的检查。其还可以编译Linux的内核。


不过，TCC 最有趣的特性是可以用 UNIX 系统上常见的 #!/usr/bin/tcc 的方式来执行 ANSI C 语言写就的源程序，省略掉了在命令行上进行编译和链接的步骤，而可以直接运行 C 语言写就的源程序。这样就能做到像任何一种其它的脚本语言比如 Perl 或者是 Python 一样，显著的加快开发步调。可以像编写 Shell 脚本一样的使用 C 语言，随便想一想都觉得是一件奇妙的事情。但是 TCC 还有一些其它的特性呢！



在TCC这个超小型的C语言编译器下，我们还可以干得更多，比如这个开源项目：C in Python，项目主页是：<http://www.cs.tut.fi/~ask/cinpy/>，这个项目主要是让你可以在Python中直接使用C的源码。呵呵。


Cinpy 是一个Python的库，它可以让你在Python的模块中实现C的函数。在前些天，酷壳向大家介绍过《[Python调用C语言函数](/2009/Python%E8%B0%83%E7%94%A8C%E8%AF%AD%E8%A8%80%E5%87%BD%E6%95%B0.md)》——这主要是通过调用动态链接库的方式调用C的函数。而Cinpy则是直接在Python中写C语言。


我们来看一个示例：（部分代码）



```
import ctypes
import cinpy

# Fibonacci in Python
def fibpy(x):
    if x<=1: return 1
    return fibpy(x-1)+fibpy(x-2)

# Fibonacci in C
fibc=cinpy.defc("fib",
                ctypes.CFUNCTYPE(ctypes.c_long,ctypes.c_int),
                """
                long fib(int x) {
                    if (x<=1) return 1;
                    return fib(x-1)+fib(x-2);
                }
                """)

# ...and then just use them...
# (there _is_ a difference in the performance)
print fibpy(30)
print fibc(30)

```

源代码这里下载：[cinpy-0.10.tar.gz](https://coolshell.cn/wp-admin/cinpy-0.10.tar.gz)


