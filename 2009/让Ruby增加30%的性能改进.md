# 让Ruby增加30%的性能改进
>date: 2009-05-05T23:44:55+08:00



#### 一切都和 `--enable-pthread` 有关


问一下 Ruby 黑客怎么简单地增加一个线程的Ruby应用程序的性能。也许，这些黑客会告诉你，“**小伙，每个人都知道在编译Ruby的时候你需要使用`configure` 的 `--disable-pthread`参数**”。没错，在`configure` `--disable-pthread` 可以让你得到大约 30% 性能提高。但是，这是为什么呢？


所有的这一些我们需要使用 [strace](http://timetobleed.com/hello-world/) 工具，这个工具可以打出所有的真实的操作系统的调用。


下面，是一段我们测试的例程：




```
def make_thread
  Thread.new {
    a = []
    10_000_000.times {
      a << "a"
      a.pop
    }
  }
end

t = make_thread
t1 = make_thread

t.join
t1.join

```

如果我们使用 `strace` 工具去测试 `configure` `--enable-pthread` 版本的Ruby引擎，那么我们可以得到下面这样的结果：



```
22:46:16.706136 rt_sigprocmask(SIG_BLOCK, NULL, [], 8 ) = 0 <0.000004>
22:46:16.706177 rt_sigprocmask(SIG_BLOCK, NULL, [], 8 ) = 0 <0.000004>
22:46:16.706218 rt_sigprocmask(SIG_BLOCK, NULL, [], 8 ) = 0 <0.000004>
22:46:16.706259 rt_sigprocmask(SIG_BLOCK, NULL, [], 8 ) = 0 <0.000005>
22:46:16.706301 rt_sigprocmask(SIG_BLOCK, NULL, [], 8 ) = 0 <0.000004>
22:46:16.706342 rt_sigprocmask(SIG_BLOCK, NULL, [], 8 ) = 0 <0.000004>
22:46:16.706383 rt_sigprocmask(SIG_BLOCK, NULL, [], 8 ) = 0 <0.000004>

```

你会发现上面的sigprocmask 系统调用一页一页又一页地没完没了的。如果你用 `strace -c，你会发现`一共大约**20,054,180** 个`sigprocmask系统调用。但是，如果你是在``--disable-pthread` 的Ruby版本下运行，你会发现根本没有那么多的`sigprocmask` 系统调用（只有 **3** 次，简直就是**天壤之别**）


#### 查看一下源代码


我们知道 `configure` 是一个脚本，其主要用来创建一个 `config.h` 文件，其中有一大堆宏定义 `define`s ，这些宏定义决定了使用什么样的函数。所以，让我们来比较一下版本 `./configure --enable-pthread` 和版本`./configure --disable-pthread的不同之处吧。`



```
$ diff config.h config.h.pthread
> #define _REENTRANT 1
> #define _THREAD_SAFE 1
> #define HAVE_LIBPTHREAD 1
> #define HAVE_NANOSLEEP 1
> #define HAVE_GETCONTEXT 1
> #define HAVE_SETCONTEXT 1

```

好的，现在我们再 `grep` 一下Ruby的源代码，我们可以看到只要`HAVE_[S/G]ETCONTEXT` 被设置了，Ruby 就会调用`setcontext()` 和`getcontext()` 这两个系统调用来存取context 的状态，以便异常处理时的切换（通过`EXEC_TAG）。`


而如果 `HAVE_[S/G]ETCONTEXT` **没有被定义** `的情况下，`Ruby 会使用 `_setjmp/_longjmp这两个系统调用。`


`我们来看看 `_setjmp/_longjmp` 的man page:`

> … The \_longjmp() and \_setjmp() functions shall be equivalent to longjmp() and setjmp(), respectively, with the additional restriction that \_longjmp() and \_setjmp() shall not manipulate the signal mask…
> 
> 


还有`setcontext /getcontext的` man page:



> … uc\_sigmask is the set of signals blocked in this context (see sigprocmask(2)) …
> 
> 


我们可以看到 `getcontext` 调用每次都要调用`sigprocmask` 但是`_setjmp` 不会。


#### 补丁


请点击 **[这里](https://github.com/ice799/matzruby/commit/0b9b69f9653782a33aee2b8937d405eae245b60c)**获取补丁


这个补丁增加了一个configure 的参数 `--disable-ucontext` 其可以让你关闭使用 `setcontext或getcontext，你只需要像如下方式使用就好了。`



```
./configure --disable-ucontext --enable-pthread

```

如果你以这种方式编译Ruby，那么，你的程序的性能在同等条件下可能会有30%左右的提升。


文章：[来源](http://timetobleed.com/fix-a-bug-in-rubys-configurein-and-get-a-30-performance-boost/)


