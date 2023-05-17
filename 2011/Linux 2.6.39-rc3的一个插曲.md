# Linux 2.6.39-rc3的一个插曲
>date: 2011-04-27T08:39:26+08:00


2011年4月12日，Linux 2.6.39-rc3发布了，Linus Torvalds写了一个[发布邮件](http://thread.gmane.org/gmane.linux.kernel/1124982)，其中包含了一个长长的为这个版本做过贡献的人员名单，这个名单中有很多看上去应该是中国人的名字，我挺为他们感到骄傲的（不知道你是否还记得以前本站的”[Linux是由谁写的](https://coolshell.cn/articles/1360.html "谁写了Linux")“）。


不过，没过一会，发现了一个bug，经过大家的调查（2.6.38版没有发现这个问题），很快，找到了原因，是因为一个内存地址的问题，一个叫Yinghai Lu的人（看其名字应该是中国人，其邮件是@kernel.org）[找到了原因](http://thread.gmane.org/gmane.linux.kernel/1124982/focus=1126082)—— radeon card使用了一个不正确的内存地址[0xa0000000 – 0xc000000]。Joerg Roedel跟贴说，这个地址超出了4GB的内存，然后他和Alex Deucher聊了一会，觉得不应该是这个问题，因为这个地址应该是GPU的，而不是系统内存的。


好像，Yinghai Lu没有理会他们说的不应该是这个问题，[给出了个fix](http://thread.gmane.org/gmane.linux.kernel/1124982/focus=1126133)：



```
diff --git a/arch/x86/kernel/aperture_64.c b/arch/x86/kernel/aperture_64.c
index 86d1ad4..3b6a9d5 100644
--- a/arch/x86/kernel/aperture_64.c
+++ b/arch/x86/kernel/aperture_64.c
@@ -83,7 +83,7 @@ static u32 __init allocate_aperture(void)
 	 * so don't use 512M below as gart iommu, leave the space for kernel
 	 * code for safe
 	 */
-	addr = memblock_find_in_range(0, 1ULL<<32, aper_size, 512ULL<<20);
+	addr = memblock_find_in_range(0, 1ULL<<32, aper_size, 512ULL<<21);
  	if (addr == MEMBLOCK_ERROR || addr + aper_size > 0xffffffff) {
 		printk(KERN_ERR
 			"Cannot allocate aperture memory hole (%lx,%uK)\n",

```

看到这个fix，Linus Torvalds不高兴了，他回贴问道：


* 为什么全都是Magic Numbers？
* 为什么0x80000000就那么特殊？
* 为什么我们这样改就行？


还说了这样一句话——




> This kind of “I broke things, so now I will jiggle things randomly until they unbreak” is not acceptable. 这种“我把事搞砸了，就随意地调整直到事情又工作”的方式是不可接受的。
> 
> 


还说，这里即没有说明为什么我们fix在了正确的地方（也没有解释那些Magic Number是什么），也没有回滚那个有问题的patch。还说——



> Don’t just make random changes. There really are only two acceptable models of development: “think and analyze” or “years and years of testing on thousands of machines”. Those two really do work.
> 
> 
> 不要乱改。那里只有两个可行的开发模式：“思考和分析” 或是 “数年数年地不断地在几千台机器上测试”。这两个方式才是真正可行的。
> 
> 


当然，Yinghai Lu对[其做了解释](http://thread.gmane.org/gmane.linux.kernel/1124982/focus=1126154)，说我们的确调查过了，老的代码用的内存地址是0x80000000，新的则是用0xa0000000，而0xa0000000不工作。这又引发了 Linus Torvalds 的[不满的回贴](http://thread.gmane.org/gmane.linux.kernel/1124982/focus=1126216)。Linus说——



> Yinghai, we have had this discussion before, and dammit, you need to understand the difference between “understanding the problem” and “put in random values until it works on one machine”.
> 
> 
> Yinghai，我们以前谈过这个事，该死的，你真的需要明白“理解一个错误”和“设一个随意的值直到其正常工作”的区别。
> 
> 
> There was absolutely \_zero\_ analysis done. You do not actually understand WHY the numbers matter. You just look at two random numbers, and one works, the other does not. That’s not “analyzing”. That’s just “random number games”.
> 
> 
> 这里就根本没有分析。你没有直正的明白**为什么**这些数字能行。你只看了两个随机的数，一个能行，另一个不行。这不是“分析”，这叫“随机数游戏”。
> 
> 
> If you cannot see and understand the difference between an actual analytical solution where you \_understand\_ what the code is doing and  why, and “random numbers that happen to work on one machine”, I don’t know what to tell you.
> 
> 
> 一个解决方案真正经过分析了那段代码干什么的为什么的，另一个是“随机数字可以让其在一台机器上运转”，如果你不能看到和理解他们之间的不同，那我不知道要和你说什么了。
> 
> 


然后，Linus Torvalds进行了谆谆教导——（相当的受用啊）



> Let me repeat my point one more time.
> 
> 
> 让我再一次重复一下我的观点
> 
> 
> You have TWO choices. Not more, not less:
> 
> 
> 你有两个选择，不多也不少：
> 
> 
> – choice #1: go back to the old allocation model. It’s tested. It doesn’t regress. Admittedly we may not know exactly \_why\_ it works, and it might not work on all machines, but it doesn’t cause regressions (ie the machines it doesn’t work on it \_never\_ worked on).
> 
> 
> – **选择一**：回滚到老的分配模式。那是测试过的。它过了回归测试。诚然，我们也许不知道**为什么**那样能行，并且，即使是那样也不一定能在所有的机器上工作，但是其没有让回归测试有问题（这个代码**永不可能**在不能运行的系统上运行）
> 
> 
> And this doesn’t mean “old value for that \_one\_ machine”. It means “old value for \_every\_ machine”. So it means we revert the whole bottom-down thing entirely. Not just “change one random number so that the totally different allocation pattern happens to give the same result on one particular machine”.
> 
> 
> 这并不代表“老的值只能在一台机器上工作”。这代表“老的值可以工作在每一台机器上”。所以，我们需要回滚整个代码改动。而不只是“为了一个特别的机器去修改一个和以前完全不一样的随机数”。
> 
> 
> – Choice #2: understand exactly \_what\_ goes wrong, and fix it analytically (ie by \_understanding\_ the problem, and being able to solve it exactly, and in a way you can argue about without having to resort to “magic happens”).
> 
> 
> – 选择二：真正搞清楚为什么会错，并且有分析地修改他（理解问题才能真正解决之，并且，只有没有“魔法发生”的时候你才可以来争论）
> 
> 
> Now, the whole analytic approach (aka “computer sciency” approach), where you can actually think about the problem without having any pesky “reality” impact the solution is obviously the one we tend to prefer. Sadly, it’s seldom the one we can use in reality when it comes to things like resource allocation, since we end up starting off with often buggy approximations of what the actual hardware is all about (ie broken firmware tables).
> 
> 
> 现在，整个分析方法（亦称作“计算机科学”的方法）应该是你可以在没有在外界干扰下真正思考这个问题而得到的解决方案，这很明显是我们推崇的。只有在极罕见地情况下我们可以在有外界干扰下分析这种资源分配的事，因为我们只有了解倒底是什么样的硬件，我们才能最终远离bug（如：错误的固件表）
> 
> 
> So I’d love to know exactly why one random number works, and why another one doesn’t. But as long as we do \_not\_ know the “Why” of it, we will have to revert.
> 
> 
> 所以，我希望你能知道为什么一个随机数能行，而另一个不行。只要我们不知道，那么我们就不得和回滚整个改动。
> 
> 
> It really is that simple. It’s \_always\_ that simple.
> 
> 
> 这真的是很简单，而且这**一直**是那么简单。
> 
> 
> So the numbers shouldn’t be “magic”, they should have real explanations. And in the absense of real explanation, the model that works is “this is what we’ve always done”. Including, very much, the whole allocation order. Not just one random number on one random machine.
> 
> 
> 所以，那些数不应该是“magic”的，他们应该有真正的说明。在有真正的说明的情况下，我们的开发模式才会工作。其包括了整个分配顺序。不只是那个在任意机器上的随机数。
> 
> 
> Linus
> 
> 
> 


后面的事不用说了。我没有想到Linux 内核组会有像Yinghai这样工作的方式，毕竟这是一个黑客级的开发团队。我个人对这个乱写代码的人执零容忍的态度，不管你干过什么，不管你哪里毕业的，不管你简历怎么样，不求甚解随意写代码的人我无法接受。我不知道Yinghai Lu会怎么样想，他/她会像我在“[程序员那些悲催的事儿](https://coolshell.cn/articles/3980.html "程序员那些悲催的事儿")”中谈我经历那样知耻而后勇吗？能得到Linus的教导真是一件很不错的事。虽然，Linus教导的这些东西，都应该是程序员最最最基本的技能。**fix bug一定要fix在root cause上啊**，**了解一个问题，不但要知其然，还要知其所以然啊**，这都是老生长谈了。本站有很多提高程序员能力的文章，比如，[这篇](https://coolshell.cn/articles/222.html "优秀程序员的十个习惯")，[这篇](https://coolshell.cn/articles/1007.html "优质代码的十诫")，还有[这篇](https://coolshell.cn/articles/2606.html "五个方法成为更好的程序员")。


各位朋友，我真心希望你能从这个小插曲中明白点什么。


**—– 更新2011/04/27**—–


从本贴的回复中可以看到有朋友说如果时间紧，没有办法只能在不求甚解的地去fix bug，因为老板催。我认为这是老板的“急功近利”的问题。我想和大家说一下，你得想清楚你属于下面那种人：


1. 你的老板给你压力，让你不得不乱fix，
2. 你认同只要时间紧bug是可以乱fix的。


如果你属于1），那我觉得还情由可原，这是管理问题。但这不能成为你对乱fix bug的理由。一般这种问题怎么解决：**首先，给一个hot fix去救火，然后，有时间去调查root cause，最后经过分析和测试，给出一个final 的 offical fix**。这就是应急的做法，根本不存在什么可以乱fix bug的做法。


如果你属于2），那么我只能“过激”地说你没有成为程序员的资质！


另外，**快速地fix bug，并不等于，不求甚解的fix bug**。大家不要把这两件事等同。


 


**（请勿用于商业用途，转载时请注明作者和出处）**



