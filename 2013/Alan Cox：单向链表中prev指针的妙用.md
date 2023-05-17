# Alan Cox：单向链表中prev指针的妙用
>date: 2013-06-30T12:27:04+08:00


![Alan Cox](/assets/images/coolshell.cn/wp-content/uploads/2013/06/Alan-Cox-200x300.jpg "Alan Cox")Alan Cox


 **（感谢网友** [**@我的上铺叫路遥**](http://weibo.com/fullofbull)**投稿）**


之前发过一篇[二级指针操作单向链表](/2013/Linus%EF%BC%9A%E5%88%A9%E7%94%A8%E4%BA%8C%E7%BA%A7%E6%8C%87%E9%92%88%E5%88%A0%E9%99%A4%E5%8D%95%E5%90%91%E9%93%BE%E8%A1%A8.md)的例子，显示了C语言指针的灵活性，这次再探讨一个指针操作链表的例子，而且是一种完全不同的用法。


这个例子是linux-1.2.13网络协议栈里的，关于链表遍历&数据拷贝的一处实现。源文件是/net/inet/dev.c，你可以从[kernel.org](https://www.kernel.org/pub/linux/kernel/v1.2/)官网上下载。


从最早的0.96c版本开始，linux网络部分一直采取TCP/IP协议族实现，这是最为广泛应用的网络协议，整个架构就是经典的OSI七层模型的描述，其中dev.c是属于链路层实现。从功能上看，其位于网络设备驱动程序和网络层协议实现模块之间，作为二者之间的数据包传输通道，一种接口模块而存在——对驱动层的接口函数netif\_rx, 以及对网络层的接口函数net\_bh。前者提供给驱动模块的中断例程调用，用于链路数据帧的封装；后者作为驱动中断例程**底半部(buttom half)**，用于对数据帧的解析处理并向上层传送。


为了便于理解，这里补充一下网络通信原理和linux驱动中断机制的背景知识。从最底层的物理层说起，当主机和路由器相互之间进行通信的时候，在物理介质上（同轴、光纤等）以电平信号进行传输。主机或路由器的**硬件接口（网卡）**负责收发这些信号，当信号发送到接口，再由内置的**调制解调器(modem)**将数字信号转换成二进制码，这样才能驻留在主机的硬件缓存中。这时接口（网卡）设备驱动程序将通过**硬中断**来获取硬件缓存中的数据，驱动程序是操作系统中负责直接同硬件设备打交道的模块，硬中断的触发是初始化时通过设置控制寄存器实现的，用于通知驱动程序硬件缓存中有新的数据到来。linux卡设备驱动就是在**中断处理例程(ISR)**中将硬件缓存数据拷贝到内核缓存中，打包成数据链路帧进行解析处理，再向上分发到各种协议层。由于ISR上下文是原子性的、中断屏蔽的，整个步骤又较为繁琐，因此全部放在ISR中处理会影响到其它中断响应实时性，于是linux有实现一种bottom half的**软中断**处理机制，将整个ISR一分为二，前半部上下文屏蔽所有中断，专门处理紧急的、实时性强的事务，如拷贝硬件缓存并打包封装，后半部上下文没有屏蔽中断（但代码不可重入），用于处理比较耗时且非紧急事务，包括数据帧的解析处理和分发。下面要讲的net\_bh就属于后半部。


我们主要关心的是将链路帧分发到协议层那一段逻辑，下面摘自net\_bh函数中的一段代码：




```
526 void net_bh(void *tmp)
527 {
       ...
577
578    /*
579    * We got a packet ID.  Now loop over the "known protocols"
580    * table (which is actually a linked list, but this will
581    * change soon if I get my way- FvK), and forward the packet
582    * to anyone who wants it.
583    *
584    * [FvK didn't get his way but he is right this ought to be
585    * hashed so we typically get a single hit. The speed cost
586    * here is minimal but no doubt adds up at the 4,000+ pkts/second
587    * rate we can hit flat out]
588    */
589   pt_prev = NULL;
590   for (ptype = ptype_base; ptype != NULL; ptype = ptype->next)
591   {
592    if ((ptype->type == type || ptype->type == htons(ETH_P_ALL)) && (!ptype->dev || ptype->dev==skb->dev))
593    {
594      /*
595      * We already have a match queued. Deliver
596      * to it and then remember the new match
597      */
598      if(pt_prev)
599      {
600        struct sk_buff *skb2;
601        skb2=skb_clone(skb, GFP_ATOMIC);
602        /*
603        * Kick the protocol handler. This should be fast
604        * and efficient code.
605        */
606        if(skb2)
607          pt_prev->func(skb2, skb->dev, pt_prev);
608      }
609      /* Remember the current last to do */
610      pt_prev=ptype;
611    }
612   } /* End of protocol list loop */
613   /*
614   * Is there a last item to send to ?
615   */
616   if(pt_prev)
617     pt_prev->func(skb, skb->dev, pt_prev);
618   /*
619    *  Has an unknown packet has been received ?
620    */
621   else
622     kfree_skb(skb, FREE_WRITE);
623
      ...
640 }
```

在此稍稍解说一下数据结构，skb就是内核缓存中sock数据封装，协议栈里从链路层到传输层都会用到，只不过封装格式不同，主要是对**协议首部(header)**的由下而上层层剥离（反之由上而下是层层创建），在此你只需理解为一个链路数据帧即可。这段代码的逻辑是解析skb中的协议字段，从协议类型链表（由ptype\_base维护）中查询对应的协议节点进行函数指针func回调，以便将数据帧分发到相应的协议层（如ARP、IP、8022、8023等）。


第一眼看上去是不是有点奇怪？这段代码竟然用一个pt\_prev指针去维护ptype链表中前一个节点，从而产生了额外的条件分支判断，咋一看是否多了很多“余”了？回顾一下那篇[二级指针操作单向链表](/2013/Linus%EF%BC%9A%E5%88%A9%E7%94%A8%E4%BA%8C%E7%BA%A7%E6%8C%87%E9%92%88%E5%88%A0%E9%99%A4%E5%8D%95%E5%90%91%E9%93%BE%E8%A1%A8.md)的博文，简直完全是反其道而行之的。如果把pt\_prev去掉，代码可以精简为：



```
  for (ptype = ptype_base; ptype != NULL; ptype = ptype->next)
  {
    if ((ptype->type == type || ptype->type == htons(ETH_P_ALL)) && (!ptype->dev || ptype->dev==skb->dev))
    {
        /*
        * We already have a match queued. Deliver
        * to it and then remember the new match
        */
        struct sk_buff *skb2;
        skb2=skb_clone(skb, GFP_ATOMIC);
        /*
        * Kick the protocol handler. This should be fast
        * and efficient code.
        */
        if(skb2)
            pt_prev->func(skb2, skb->dev, pt_prev);
    }
} /* End of protocol list loop */

kfree_skb(skb, FREE_WRITE);
```

咋看一下“干净”了很多，不是吗？但我们要记住一点，凡是网上发布的linux内核源代码，都是都是经过众多黑客高手们重重检视并验证过的，人家这么写肯定有十分充足的理由，所以不要太过于相信自己的直觉了，让我们再好好review一下代码吧！看看这段循环里做了什么事情？特别是第592~611行。


由于从网络上拷贝过来skb是唯一的，而分发的协议对象可能是多个，所以在回调之前要做一次clone动作（注意这里是深度拷贝，相当于一次kmalloc）。分发之后还需要调用kfree\_skb释放掉原始skb数据块，它的历史使命到此完成了，没有保留的必要（第622行）。**注意，这两个动作都是存在内核开销的。**


然而这里为啥要pt\_prev维护一个后向节点呢？这是有深意的，它的作用就是将当前匹配协议项的回调操作延时了。举个例子，如果链表遍历中找到某个匹配项，当前循环仅仅用pt\_prev去记录这个匹配项，除此之外不做任何事情，待到下一次匹配项找到时，才去做上一个匹配项pt\_prev的回调操作，直到循环结束，才会去做最后的匹配项的回调（当然pt\_prev==NULL表示没有一次匹配，直接释放掉），所以这是一种**拖延战术**。有什么好处呢？就是比原先节省了很多不必要的操作。那么哪些操作是不必要的呢？这里我们逆向思考一下，我们看到clone是在协议字段匹配并且pt\_prev!=NULL的前提条件下执行的，而kfree是在pt\_prev==NULL的前提条件下执行的。在此可以假设一下，如果ptype链表中存在N项协议与之匹配，那么这段代码只会执行N-1次clone，而没有pt\_prev时将会执行N次clone和1次kfree，再如果ptype链表中有且仅有一项协议与之匹配，那么整个循环既不会执行到第601行的clone，也不会执行到第622行的kfree。


也就是说，**当整个链表至少有一项匹配的一般情况下，pt\_prev存在比没有时减少了一次clone和一次kfree的开销；只有全部不匹配的最差情况下，两者都只做一次kfree动作，持平。这就是延迟策略产生的效益**。


熟悉TCP/IP协议族的开发人员应该知道**MTU（最大传输单元）**这个概念，遵循不同协议的MTU值是不同的。比如以太网帧MTU是1500个字节，802.3帧MTU是1492字节，PPP链路帧MTU是269字节，而超通道MTU理论上是65535字节。要知道在一个高速吞吐量通信网络环境下，在大块数据分片传输线路里，在内核级别代码中，减少一处系统开销意味着什么？


其实我们完全可以抛开一切网络协议相关知识，这不过是一段极其普通的单向链表操作而已，逻辑并不复杂。但是看看人家顶级黑客是怎么思考和coding的，对比一下自己写过的代码，多少次数据处理是用一个简单的for循环匆匆敷衍了事而没有进一步思考其中的粗陋和不合理之处？面对真正的编程高手这种“心计”与“城府”，你是不是有种莫名不安感？你会怀疑你真的了解怎么去使用和操作C语言中基本的链表数据结构么？如果答案是肯定的，那就开始颤抖吧（哈，别误会，其实上面这段话不过是笔者的自我告解罢了）~~~


最后，让我们感谢尊敬的[Alan Cox](https://en.wikipedia.org/wiki/Alan_Cox)大大对Linux社区卓越精细、无与伦比的贡献！（Alan是图中中部戴红帽子的那位）


![Linux Kernel Team](http://old.lwn.net/images/ks/group2.jpg)


**附注：**


最新的Linux-2.6.x版本中协议栈实现部分变动很大，但/net/core/dev.c的netif\_receive\_skb函数里仍然保留了pt\_prev这种用法，目的是一样的，都是为了减少一次系统开销的优化操作。


关于Alan，他在斯旺西大学工作时，在学校服务器上安装了一个早期的linux版本，供学校使用。他修正了许多的问题，重写了网络系统中的许多部份。随后成为linux内核开发小组中的重要成员。[Alan Cox](https://en.wikipedia.org/wiki/Alan_Cox)负责维持2.2版，在2.4版上拥有自己的分支（在版本号上会冠上ac，如 2.4.13-ac1）。他的分支版本非常稳定，修正许多错误，许多厂商都使用他的版本。在他去进修工商管理硕士之前，涉入许多linux内核开发的事务，在社群中有很高的地位，有时会被视为是Linus之下的第二号领导者。


不过，今年1月28日的时候，Alan因为家庭原因宣布退出Linux项目了，下面是他Google+的声明：



> “I’m leaving the Linux world and Intel for a bit for family reasons, I’m aware that ‘family reasons’ is usually management speak for ‘I think the boss is an asshole’ but I’d like to assure everyone that while I frequently think Linus is an asshole (and therefore very good as kernel dictator) I am departing quite genuinely for family reasons and not because I’ve fallen out with Linus or Intel or anyone else. Far from it I’ve had great fun working there.”
> 
> 


（全文完）


