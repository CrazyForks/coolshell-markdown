# State Threads 回调终结者
>date: 2014-10-12T22:48:57+08:00


**（感谢网友**[**@我的上铺叫路遥**](http://weibo.com/fullofbull)**投稿）**


上回写了篇[《一个“蝇量级”C语言协程库》](https://coolshell.cn/articles/10975.html "一个“蝇量级” C 语言协程库")，推荐了一下[Protothreads](http://dunkels.com/adam/pt/ "Protothreads")，通过coroutine模拟了用户级别的multi-threading模型，虽然本身足够“轻”，杜绝了系统开销，但这个库本身应用场合主要是内存限制的嵌入式领域，提供原生态组件太少，使用限制太多，比如依赖其它调用产生阻塞等。


这回又替大家在开源界淘了个宝，推荐一个轻量级网络应用框架**State Threads**（以下简称ST），总共也就3000行C代码，跟Protothreads不同在于ST针对的就是**高性能可扩展服务器**领域（值得一提的是Protothreads官网[参考链接](http://dunkels.com/adam/pt/links.html "参考链接")上第一条就是ST的官网）。在其[FAQ](http://state-threads.sourceforge.net/docs/faq.html "FAQ")页面上一句引用”Perfection is achieved not when there is nothing more to add, but rather when there is nothing more to take away.”可以视为开发人员对ST源码质量的自信。




目录



* [历史渊源](#%E5%8E%86%E5%8F%B2%E6%B8%8A%E6%BA%90 "历史渊源")
* [基于事件驱动状态机（EDSM）](#%E5%9F%BA%E4%BA%8E%E4%BA%8B%E4%BB%B6%E9%A9%B1%E5%8A%A8%E7%8A%B6%E6%80%81%E6%9C%BA%EF%BC%88EDSM%EF%BC%89 "基于事件驱动状态机（EDSM）")
* [基于Mult-Threading范式](#%E5%9F%BA%E4%BA%8EMult-Threading%E8%8C%83%E5%BC%8F "基于Mult-Threading范式")
* [基于多核环境](#%E5%9F%BA%E4%BA%8E%E5%A4%9A%E6%A0%B8%E7%8E%AF%E5%A2%83 "基于多核环境")
* [使用限制](#%E4%BD%BF%E7%94%A8%E9%99%90%E5%88%B6 "使用限制")
* [总结](#%E6%80%BB%E7%BB%93 "总结")
* [参考](#%E5%8F%82%E8%80%83 "参考")

#### 历史渊源


首先介绍一下这个库的历史渊源，从代码贡献者来看，ST不是个人作品，而是有着雄厚的商业支持和应用背景，比如服务器领域，在[这里](http://state-threads.sourceforge.net/news.html)你可以看到ST曾作为Apache的多核应用模块发布。其诞生最初是由网景（Netscape）公司的MSPR（Netscape Portable Runtime library）项目中剥离出来，后由SGI（Silicon Graphic Inc）还有Yahoo!公司（前者是主力）开发维护的独立线程库。历史版本方面，作为[SourceForge](http://sourceforge.net/projects/state-threads/files/ "SourceForge")上开源项目，由2001年发布v1.0以来一直到2009年v1.9稳定版后未再变动。在平台移植方面，从Makefile的配置选项中可知ST支持多种Unix-like平台，还有专门针对Win32的源码改写。源码例子中，提供了web server、proxy以及dns三种编程实例供参考。可以说代码质量应该是相当的稳定和可靠的。



至于许可证方面，有必要略作说明。出于历史原因，网景最初发布时选择了MPL1.1许可证，而后SGI在维护中又混进了GPLv2许可证，照理说这两种许可证是互不兼容的（MPL1.1后续版本是GPL兼容的），也就是说用双许可证打包发布理论上是非法无效的，见GNU官网上[MPL兼容性](http://www.gnu.org/licenses/license-list.html#MPL "GPL兼容")一节。但这里有值得商榷的地方，因为文中又提及，根据MPL1.1中某条款第13节，如果整段或部分代码允许采用另一许可证作为备用（alternate）选择，比如GPL及其兼容，那么整个库的许可证就可视为GPL兼容的。如此一来所谓GPL兼容性一般解释为你不能在GPLv2的代码中混入MPL1.1，而不是说你不能在MPL1.1代码中混入GPLv2，也就是说GPLv2在MPL1.1之后是可以接受的，事实上SGI就采用了后面的做法，尚未引起版权上的纠纷。为此我还考证了一下FAQ上[license](http://state-threads.sourceforge.net/docs/faq.html "license")一节的说法，说ST既可以在MPL和GPL之间选择一种，也可以继续用双许可证，还补了一句在non-free项目使用上也没有限制，但对ST源码所做改动必须对用户可见。在源码文件中的SGI的附加声明还解释了将ST转为GPL代码的做法，就是可以删除前面MPL的声明，否则后续用户仍可以在两者之间二选一。个人觉得既然SGI都这样发话了，那么可解释为反之删除GPL的声明继续采用MPL也是可以接受的，如果你对双许可证承诺仍不放心的话。


#### 基于事件驱动状态机（EDSM）


好了，下面该进入技术性话题了。前面说了ST的目标是**高性能可扩展**，其技术特征一言以蔽之就是



> **“It combines the simplicity of the multi-threaded programming paradigm, in which one thread supports each simultaneous connection, with the performance and scalability of an event-driven state machine (EDSM) architecture.”**
> 
> 


我们先来纵向比较ST与传统的EDSM区别，再来横向比较与其它线程库（比如Pthread）的区别（注：以下图片全部来自[State Threads Library FAQ](http://state-threads.sourceforge.net/docs/faq.html "ST FAQ")）。


传统EDSM最常见的方式就是I/O事件的**异步回调**。基本上都会有一个叫做dispatcher的单线程主循环（又叫event loop），用户通过向dispatcher注册回调函数（又叫event handler）来实现异步通知，从而不必在原地空耗资源干等，在dispatcher主循环中通过select()/poll()系统调用来等待各种I/O事件的发生，当内核检测到事件触发并且数据可达或可用时，select()/poll()会返回从而使dispatcher调用相应的回调函数来对处理用户的请求。所以异步回调与其说是通知，不如说用委托更恰当。


整个过程都是单线程的。**这种处理本质上就是将一堆互不相交（disjoint）的回调实现同步控制，就像串联在一个顺序链表上。**见图1，黑色的双箭头表示I/O事件复用，回调是个筐，里面装着对各种请求的处理（当然不是每个请求都有回调，一个请求也可以对应不同的回调），每个回调被串联起来由dispatcher激活。这里请求等价于thread的概念（不是操作系统的线程），只不过“上下文切换”（context switch）发生在每个回调结束之时（假设不同请求对应不同回调），注册下一个回调以待事件触发时恢复其它请求的处理。至于dispatcher的执行状态（execute state）可作为回调函数的参数保存和传递。


![EDSM](https://coolshell.cn/wp-content/uploads/2014/10/edsm.gif)


异步回调的缺陷在于**难以实现和扩展**，虽然已经有libevent这样的通用库，以及其它actor/reacotor的设计模式及其框架，但正如Dean Gaudet（Apache开发者）所说：“其内在的复杂性——**将线性思维分解成一堆回调的负担**（breaking up linear thought into a bucketload of callbacks）——仍然存在”。从上图可见，**回调之间请求例程不是连续的，比如回调之间的切换会打断部分请求，又比如有新的请求需要重新注册。**


**ST本质上仍然是基于EDSM模型，但旨在取代传统的异步回调方式。**ST将请求抽象为thread概念以更接近自然编程模式（所谓的linear thought吧，就像操作系统的线程之间切换那样自然）。ST的调度器（scheduler）对于用户来说是透明的，不像dispatcher那种将执行状态（execute state）暴露给回调方式。每个thread的现场环境可以保存在栈上（一段连续的大小确定的内存空间），由C的运行环境管理。从图2看到，**ST的threads可以并发地线性地处理I/O事件，模型比异步回调简单得多。**


![State Threads](https://coolshell.cn/wp-content/uploads/2014/10/st_edsm.gif)


这里稍微解释一下ST调度工作原理，ST运行环境维护了四种队列，分别是IOQ、RUNQ、SLEEPQ以及ZOMBIEQ，**当每个thread处于不同队列中对应不同的状态（ST顾名思义所谓thread状态机）。**比如polling请求的时候，当前thread就加入IOQ表示等待事件（如果有timeout同时会被放到SLEEPQ中），当事件触发时，thread就从IOQ（如果有timeout同时会从SLEEPQ）移除并转移到RUNQ等待被调度，成为当前的running thread，相当于操作系统的就绪队列，跟传统EDSM对应起来就是注册回调以及激活回调。再比如模拟同步控制wait/sleep/lock的时候，当前thread会被放入SLEEPQ，直到被唤醒或者超时再次进入RUNQ以待调度。


**ST的调度具备性能与内存双重优点**：在性能上，ST实现自己的setjmp/longjmp来模拟调度，无任何系统开销，并且context（就是jmp\_buf）针对不同平台和架构用底层语言实现的，可移植性媲美libc。下面放一段代码解释一下调度实现：



```
/*
 * Switch away from the current thread context by saving its state 
 * and calling the thread scheduler
 */
#define _ST_SWITCH_CONTEXT(_thread)       \
    ST_BEGIN_MACRO                        \
    if (!MD_SETJMP((_thread)->context)) { \
      _st_vp_schedule();                  \
    }                                     \
    ST_END_MACRO

/*
 * Restore a thread context that was saved by _ST_SWITCH_CONTEXT 
 * or initialized by _ST_INIT_CONTEXT
 */
#define _ST_RESTORE_CONTEXT(_thread)   \
    ST_BEGIN_MACRO                     \
    _ST_SET_CURRENT_THREAD(_thread);   \
    MD_LONGJMP((_thread)->context, 1); \
    ST_END_MACRO

void _st_vp_schedule(void)
{
    _st_thread_t *thread;

    if (_ST_RUNQ.next != &_ST_RUNQ) {
        /* Pull thread off of the run queue */
        thread = _ST_THREAD_PTR(_ST_RUNQ.next);
        _ST_DEL_RUNQ(thread);
    } else {
        /* If there are no threads to run, switch to the idle thread */
        thread = _st_this_vp.idle_thread;
    }
    ST_ASSERT(thread->state == _ST_ST_RUNNABLE);

    /* Resume the thread */
    thread->state = _ST_ST_RUNNING;
    _ST_RESTORE_CONTEXT(thread);
}

```

如果你熟悉setjmp/longjmp的用法，你就知道当前thread在调用MD\_SETJMP将现场上下文保存在jmp\_buf中并返回返回0，然后自己调用\_st\_vp\_schedule()将自己调度出去。调度器先从RUNQ上找，如果队列为空就找idle thread，这是在整个ST初始化时创建的一个特殊thread，然后将当前线程设为自己，再调用MD\_LONGJMP切换到其上次调用MD\_SETJMP的地方，从thread->context恢复现场并返回1，该thread就接着往下执行了。**整个过程就同EDSM一样发生在操作系统单线程下，所以没有任何系统开销与阻塞。**


**其实真正的阻塞是发生在等待I/O事件复用上，也就是select()/poll()，这是整个ST唯一的系统调用。**ST当前的状态是，整个环境处于空闲状态，所有threads的请求处理都已经完成，也就是RUNQ为空。这时在\_st\_idle\_thread\_start维护了一个主循环（类似于event loop），主要负责三种任务：1.对IOQ所有thread进行I/O复用检测；2.对SLEEPQ进行超时检查；3.将idle thread调度出去，代码如下：



```
void *_st_idle_thread_start(void *arg)
{
    _st_thread_t *me = _ST_CURRENT_THREAD();

    while (_st_active_count > 0) {
        /* Idle vp till I/O is ready or the smallest timeout expired */
        _ST_VP_IDLE();

        /* Check sleep queue for expired threads */
        _st_vp_check_clock();

        me->state = _ST_ST_RUNNABLE;
        _ST_SWITCH_CONTEXT(me);
    }

    /* No more threads */
    exit(0);

    /* NOTREACHED */
    return NULL;
}
```

这里的me就是idle thread，因为\_st\_idle\_thread\_start就是创建idle thread的启动点，每从上次\_ST\_SWITCH\_CONTEXT()切换回来的时候，接着在\_ST\_VP\_IDLE()里轮询I/O事件的发生，一旦检测到发生了别的thread事件或者SLEEPQ里面发生超时，再用\_ST\_SWITCH\_CONTEXT()把自己切换出去，如果此时RUNQ中非空的话就切换到队列第一个thread。这里主循环是不会退出的。


在内存方面，**ST的执行状态作为局部变量保存在栈上，而不是像回调需要动态分配，**用户可能分别这样使用thread模式和callback模式：



```
/* thread land */
int foo()
{
    int local1;
    int local2;
    do_some_io();
}

/* callback land */
struct foo_data {
    int local1;
    int local2;
};

void foo_cb(void *arg)
{
    struct foo_data *locals = arg;
    ...
}

void foo()
{
    struct foo_data *locals = malloc(sizeof(struct foo_data));
    register(foo_cb, locals);
}

```

#### 基于Mult-Threading范式


同样基于multi-threading编程范式，ST同其它线程库又有和有点呢？比如Posix Thread（以下简称PThread）是个通用的线程库，它是**将用户级线程（thread）同内核执行对象（kernel execution entity，有些书又叫lightweight processes）做了1:1或m:n映射，**从而实现multi-threading模式。**而ST是单线程（n:1映射），它的thread实际上就是协程（coroutine）。**通常的网络应用上，多线程范式绕不开操作系统，但在某些特定的服务器领域，线程间的共享资源会带来额外复杂度，锁、竞态、并发、文件句柄、全局变量、管道、信号等，面对这些Pthread的灵活性会大打折扣。**而ST的调度是精确的，它只会在明确的I/O和同步函数调用点上发生上下文切换，这正是协程的特性，如此一来ST就不需要互斥保护了，进而也可以放心使用任何静态变量和不可重入库函数了**（这在同样作为协程的Protothreads里是不允许的，因为那是stack-less的，无法保存上下文），极大的简化了编程和调试同时增加了性能。


对于同样用户级线程如GNU Pth和MIT Phread比起来呢？有两点，一是ST的thread是**无优先级的非抢占式调度**，也就是说ST基于EDSM的，每个thread都是事件或数据驱动，迟早会把自己调度出去，而且调度点是明确的，并非按时间片来的，从而简化了thread管理；二是ST会**忽略所有信号处理**，在\_st\_io\_init中会把sigact.sa\_handler设为SIG\_IGN，这样做是因为将thread资源最小化，避免了signal mask及其系统调用（在ucontext上是避免不了的）。但这并不意味着ST就不能处理信号，实际上ST建议将信号写入pipe的方式转化为普通I/O事件处理，示例详见[这里](http://state-threads.sourceforge.net/docs/notes.html#signals "signal handling")。


这里顺便说一句，**C语言实现的协程据我所知只有三种方式**：Protothread为代表利用switch-case语义跳转，以ST为代表不依赖libc的setjmp/longjmp上下文切换，以及依赖glibc的ucontext接口（[云风的coroutine](https://github.com/cloudwu/coroutine "云风的coroutine")）。第一种最轻，但受限最大，第三种耗资源性能慢（陈皓注：glibc的ucontext接口的实现中有一个和信号有关的系统调用，所以会慢，估计在一些情况下会比pthread还慢），目前看来ST是最好使的。


#### 基于多核环境


下面来聊聊ST在多核环境下的应用。服务器领域多核的优势在于实现了物理上真正的并发，所以如何充分利用系统优势也是线程库的一大难点。这对ST来说也许正是它的拿手好戏，前面提及ST曾作为Apache的多核引擎模块发布。这里要补充一下前面漏掉的ST的一个重要概念——**虚拟处理器**（virtual processor，简称vp），见图3，多个cpu通过内核的SMP模拟出多个“核”（core），一个core对应一个内核任务（kernel task），同时对应一个用户进程（process），一个process对应ST的一个vp，每个vp下就是ST的thread（是协程不是线程），结合前面所述，vp初始化先创建idle thread，然后根据I/O事件驱动其它threads，这就是ST的多核架构。


![multi-core](https://coolshell.cn/wp-content/uploads/2014/10/st_app.gif)


这里要指出的是，**ST只负责自身thread调度，进程管理是应用程序的事情，**也就是说由用户来决定fork多少进程，每个进程分配多少资源，如何进行IPC等。这种架构的好处就是每个vp有自己独立的空间，避免了资源同步竞态（比如杜绝了多进程里的多线程这样混乱的模型）。我们知道这种**基于进程的架构是非常健壮的，一个进程奔溃不会影响到其它进程，同时充分利用多核硬件的高并发。**同时对于具体逻辑业务使用vp里的thread处理，这是基于EDSM的，如此一来做到了**逻辑业务与内核执行对象之间的解耦**，没必要因为1K个连接去创建1K的进程。这就是ST的扩展性和灵活性。


#### 使用限制


ST的主要限制在于，应用程序所有I/O操作必须使用ST提供的API，因为只有这样thread才能被调度器管理，并且避免阻塞。


另一个限制在于thread调试，这本身不容易，好在v1.9的ST提供了DEBUG参数，使用TREADQ以及\_st\_iterate\_threads接口检测thread调度情况，用户还可自定义\_st\_show\_thread\_stack接口dump每个thread的栈，在GDB使能\_st\_iterate\_threads\_flag变量，这些都在Readme中对调试方法有具体说明。按下不表。


#### 总结


这篇文章写得有点短了，主要是通过对比来介绍ST的，其实还有大段原理可以讲，大段源码以及实战用例可以贴，但这一下子又写不过来，ST还是有点技术含量的。说白了，**ST的核心思想就是利用multi-threading的简单优雅范式胜过传统异步回调的复杂晦涩实现，又利用EDSM的性能和解耦架构避免了multi-threading在系统上的开销和暗礁。**学习ST告诉我们一个道理：**未来技术的趋势永远都是融合的。**


#### 参考


* 在[SourceForge](http://sourceforge.net/projects/state-threads/files/ "sourceforge源码")以及[github](https://github.com/winlinvip/state-threads "github源码")上的源码：前者有历史版本及win32版本，后者只有v1.9。


* [State Threads for Internet Applications](http://state-threads.sourceforge.net/docs/st.html "State Threads for Internet Applications")：介绍原理的，值得一看，[这里](http://blog.csdn.net/win_lin/article/details/8242653 "中文翻译")有篇中文翻译附加单元测试（在单CPU 512M内存上创建数万个thread，CPU占用率约5%，内存约4.3K/thread）。


* [State Threads Library FAQ](http://state-threads.sourceforge.net/docs/faq.html "State Threads Library FAQ")：本文基于此而写。


* [Complete reference](http://state-threads.sourceforge.net/docs/reference.html "API手册")：API完全手册。


* [Programing Notes](http://state-threads.sourceforge.net/docs/notes.html "注意事项")：编程注意事项，包括信号处理，IPC，非网络I/O事件等。


（全文完）


