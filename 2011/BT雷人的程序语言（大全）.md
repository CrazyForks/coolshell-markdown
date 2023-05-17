# BT雷人的程序语言（大全）
>date: 2011-04-13T08:34:50+08:00


还记得以前本站的[BT雷人的程序语言](https://coolshell.cn/articles/1142.html "BT雷人的程序语言")吗？除了那几个[Brainfuck](http://www.muppetlabs.com/~breadbox/bf/)，[LOLCODE](http://lolcode.com/)和[WhiteSpace](http://compsoc.dur.ac.uk/whitespace/index.php)，我以为这些是比较BT的语言，但是自从这两天我在网上看到一些（见文章最后的参考一节），我发现我错了，这个世界上，只有更变态，没有最变态。不相，你看看下面这些，简直变态到了极致啊。（下面的语言变态不分排名）




目录



* [Befunge](#Befunge "Befunge")
* [Ook!](#Ook "Ook!")
* [TMMLPTEALPAITAFNFAL](#TMMLPTEALPAITAFNFAL "TMMLPTEALPAITAFNFAL")
* [INTERCAL](#INTERCAL "INTERCAL")
* [HQ9++](#HQ9 "HQ9++")
* [PerlYuYan](#PerlYuYan "PerlYuYan")
* [参考：](#%E5%8F%82%E8%80%83%EF%BC%9A "参考：")

#### Befunge


第一个变态语言[Befunge](http://quadium.net/funge/spec98.html)。[维基](https://en.wikipedia.org/wiki/Befunge)上面说——这门语言由Chris Pressey在1993年创造，本意为设计一种为难编译器的语言……结果马上出现了一批编译器。Befunge的代码是二维的。它用 < > v ^ 这四个符号来控制一个指针在代码中移动，指针经过一个字符或数字则把它压入一个栈，四则运算符号的功能就是弹出栈顶两个元素进行计算后把结果压回去。用 \_ 和 | 来表示有条件的方向选择：当栈顶元素为0时向右（上）走，否则向左（下）走。& 和 ~ 分别用于读入数字或字符并压入栈，句号和逗号分别表示将栈顶元素作为整数或字符输出。最后以一个@符号表示程序结束。Befunge代码的注释不需要任何符号标明，你可以把注释写在程序的任何地方，只要运行时指针不会经过它就行了。


下面这段Hello World代码：



```
>              v
v  ,,,,,"Hello"<
>48*,          v
v,,,,,,"World!"<
>25*,@
```

下面一个是算圆周率的代码，非常的壮观：




```
aa*          v                  +------------------------+
vp*9920p*9930<                  | Pi generator in Bef-97 |
>:09a*pa*3/1+19a*p09a*g:09b*v   |                        |
[[email protected]](/cdn-cgi/l/email-protection)# g*b90 p*b910        < p<   | 7/2/1997, Kevin Vigor  |
>19a*g:+1-29b*p19a*g::09v       +------------------------+
v*a90g*b90*g*b91: _v#p*9<
>g-#v_ 2a*+\$  v  :$
>\1-aa*ga*+v  p
v1:/g*b92p*991:<  *
>9b*p29b*g*199*g\v9
v*b92p*aa-1g*990-<9
>g2-29b*p099*g1-:0^
v -9p*b92:%ag*991  <
>#v_ 299*g1+299*p>       ^
>09b*g:#v_$v
v93p*b90-1<
>9*g199*ga/+.v
v:g*992 <p*9 92-<
v_29b*g399*p ^
>09b*g:#v_v      1
vp*b90-1    < $      g
>199*g9#v_'9,v      *
>'0, >' ,299^</pre>
通常认为Befunge是第一个基于“二维控制流”的语言，后来衍生出的一大批类似的语言都是受的Befunge影响。例如PingPong语言就是把Befunge的四种箭头符号换成正反斜杠，控制指针移动方向90度旋转，起一个反弹的作用。
<h4>Chef</h4>
<a href="http://www.dangermouse.net/esoteric/chef.html" target="_blank">Chef</a>如其名一样“主厨”(<a href="http://en.wikipedia.org/wiki/Chef_(programming_language)" target="_blank">Wiki link</a>)，这门语言主要是为了让程序代码看起来像菜谱。这可以使得我们的<a title="食客还是大厨" href="https://coolshell.cn/articles/3589.html" target="_blank">程序员更像是大厨</a>了，呵呵。该语言于2002年由David Morgan-Mar推出，核心是栈操作，特征就是——一套完整的Chef代码就是一个菜谱，程序名就是菜名，变量声明就是罗列原材料，后面一系列栈操作，就是菜肴的制作方法。把程序编写比作调和鼎鼐，有点意思，家庭主妇（或者“准家庭主妇”）试试看，权且当作人生预习。

用Chef编写Hello World代码如下：（在其网站上还有一个<a href="http://www.dangermouse.net/esoteric/chef_fib.html" target="_blank">斐波拉契数的例子</a>）
<pre style="padding-left: 30px;">Hello World Souffle.

Ingredients.
72 g haricot beans
101 eggs
108 g lard
111 cups oil
32 zucchinis
119 ml water
114 g red salmon
100 g dijon mustard
33 potatoes

Method.
Put potatoes into the mixing bowl.
Put dijon mustard into the mixing bowl.
Put lard into the mixing bowl.
Put red salmon into the mixing bowl.
Put oil into the mixing bowl.
Put water into the mixing bowl.
Put zucchinis into the mixing bowl.
Put oil into the mixing bowl.
Put lard into the mixing bowl.
Put lard into the mixing bowl.
Put eggs into the mixing bowl.
Put haricot beans into the mixing bowl.
Liquefy contents of the mixing bowl.
Pour contents of the mixing bowl into the baking dish.

Serves 1.</pre>
代码解读——原材料名显然可以随便改成别的原料，哪怕用单个字母也可以，不过少了点趣味性，但原料前面代表数量的数字不能改，那是ASCII代码。接下来菜肴制作方法就是把一个个字母和符号（都是ASCII）压入栈（就是代码中的“Put XXX into the mixing bowl”，从最后一个感叹号开始压），最后再把你做的菜托出上桌。

顺便说下，David Morgan-Mar已经设计出8种非主流编程语言了，还有一个变态的操作系统<a href="http://www.dangermouse.net/esoteric/petrovich.html" target="_blank">Petrovich</a>。  参看这位大哥的——<a href="http://www.dangermouse.net/esoteric/" target="_blank">DM's Esoteric Programming Languages</a>（下面会介绍这位老大搞出来的语言）
<h4><strong>Shakespeare</strong></h4>
<a href="http://shakespearelang.sourceforge.net/" target="_blank">Shakespeare</a>语言正如其名，其要让你的程序像“莎士比亚”的剧本一样充满艺术气息。

这个语言于2001年由Karl Hasselstrom和Jon Aslund联合推出，Shakespeare的代码完全模仿莎士比亚的戏剧。它也是一个基于栈的程序语言，程序中出场的每一个人物都代表一个栈。Shakespeare的代码自由度很高，因此同一个程序你可以写出完全不同的代码出来。

Shakespeare的Hello World代码如下（就是一部比较完整的“罗密欧与朱丽叶”的戏剧，作好心理准备）。“剧本”内容很无聊，就是一帮人在莫名其妙地称赞某些东西，里头还有古英语词汇，莎翁要是见了，可能会吐血。这里面Hello World或其ASCII码体现在全剧时不时出现的“The difference between……”句里面，根据各指代物品的好坏（比如鲜花算好的，牛粪算坏的）代表各数字，再进行各种运算最后相减（“The difference”暗指减法），得出一个字母或符号的ASCII码表。发明这个语言的人真是BT啊。
<pre style="padding-left: 60px;">Romeo, a young man with a remarkable patience.
Juliet, a likewise young woman of remarkable grace.
Ophelia, a remarkable woman much in dispute with Hamlet.
Hamlet, the flatterer of Andersen Insulting A/S.</pre>
<pre style="padding-left: 30px;">Act I: Hamlet's insults and flattery.</pre>
<pre style="padding-left: 30px;">Scene I: The insulting of Romeo.</pre>
<pre style="padding-left: 30px;">[Enter Hamlet and Romeo]</pre>
<pre style="padding-left: 30px;">Hamlet:
You lying stupid fatherless big smelly half-witted coward!
You are as stupid as the difference between a handsome rich brave hero and thyself! Speak your mind!</pre>
<pre style="padding-left: 30px;">You are as brave as the sum of your fat little stuffed misused dusty old rotten codpiece and a beautiful fair warm peaceful sunny summer's day. You are as healthy as the difference between the sum of the sweetest reddest rose and my father and yourself! Speak your mind!</pre>
<pre style="padding-left: 30px;">You are as cowardly as the sum of yourself and the difference between a big mighty proud kingdom and a horse. Speak your mind.</pre>
<pre style="padding-left: 30px;">Speak your mind!</pre>
<pre style="padding-left: 30px;">[Exit Romeo]</pre>
<pre style="padding-left: 30px;">Scene II: The praising of Juliet.</pre>
<pre style="padding-left: 30px;">[Enter Juliet]</pre>
<pre style="padding-left: 30px;">Hamlet:
Thou art as sweet as the sum of the sum of Romeo and his horse and his black cat! Speak thy mind!</pre>
<pre style="padding-left: 30px;">[Exit Juliet]</pre>
<pre style="padding-left: 30px;">Scene III: The praising of Ophelia.</pre>
<pre style="padding-left: 30px;">[Enter Ophelia]</pre>
<pre style="padding-left: 30px;">Hamlet:
Thou art as lovely as the product of a large rural town and my amazing bottomless embroidered purse. Speak thy mind!</pre>
<pre style="padding-left: 30px;">Thou art as loving as the product of the bluest clearest sweetest sky and the sum of a squirrel and a white horse. Thou art as beautiful as the difference between Juliet and thyself. Speak thy mind!</pre>
<pre style="padding-left: 30px;">[Exeunt Ophelia and Hamlet]</pre>
<pre style="padding-left: 30px;">Act II: Behind Hamlet's back.</pre>
<pre style="padding-left: 30px;">Scene I: Romeo and Juliet's conversation.</pre>
<pre style="padding-left: 30px;">[Enter Romeo and Juliet]</pre>
<pre style="padding-left: 30px;">Romeo:
Speak your mind. You are as worried as the sum of yourself and the difference between my small smooth hamster and my nose. Speak your mind!</pre>
<pre style="padding-left: 30px;">Juliet:
Speak YOUR mind! You are as bad as Hamlet! You are as small as the difference between the square of the difference between my little pony and your big hairy hound and the cube of your sorry little codpiece. Speak your mind!</pre>
<pre style="padding-left: 30px;">[Exit Romeo]</pre>
<pre style="padding-left: 30px;">Scene II: Juliet and Ophelia's conversation.</pre>
<pre style="padding-left: 30px;">[Enter Ophelia]</pre>
<pre style="padding-left: 30px;">Juliet:
Thou art as good as the quotient between Romeo and the sum of a small furry animal and a leech. Speak your mind!</pre>
<pre style="padding-left: 30px;">Ophelia:
Thou art as disgusting as the quotient between Romeo and twice the difference between a mistletoe and an oozing infected blister! Speak your mind!</pre>
<pre style="padding-left: 30px;">[Exeunt]</pre>
<h4>BIT</h4>
<a href="http://www.dangermouse.net/esoteric/bit.html" target="_blank">BIT语言</a>也是 David Morgan-Mar 搞出来的。程序员在拥有访问所有数据的全部权限。这是一款强大的编程工具。在高级程序语言中，该工具可以操作这些令人费解的数据。

看看下面这段代码，其展示了BIT的强大之处——代码和注释的完美统一。（很像BASIC）
<pre style="padding-left: 30px;">LINE NUMBER ONE CODE READ GOTO ONE ZERO
LINE NUMBER ONE ZERO CODE VARIABLE ZERO EQUALS THE JUMP REGISTER GOTO ONE ONE
LINE NUMBER ONE ONE CODE READ GOTO ONE ZERO ZERO
LINE NUMBER ONE ZERO ZERO CODE VARIABLE ONE EQUALS THE JUMP REGISTER GOTO ONE ZERO ONE
LINE NUMBER ONE ZERO ONE CODE THE JUMP REGISTER EQUALS OPEN PARENTHESIS VARIABLE ZERO NAND VARIABLE ONE CLOSE PARENTHESIS NAND OPEN PARENTHESIS VARIABLE ZERO NAND VARIABLE ONE CLOSE PARENTHESIS GOTO ONE ONE ZERO IF THE JUMP REGISTER IS EQUAL TO ONE GOTO ONE ZERO ZERO ZERO IF THE JUMP REGISTER IS EQUAL TO ZERO
LINE NUMBER ONE ONE ZERO CODE PRINT ONE GOTO ONE ONE ONE
LINE NUMBER ONE ONE ONE CODE PRINT ZERO
LINE NUMBER ONE ZERO ZERO ZERO CODE THE JUMP REGISTER EQUALS OPEN PARENTHESIS VARIABLE ZERO NAND VARIABLE ZERO CLOSE PARENTHESIS NAND OPEN PARENTHESIS VARIABLE ONE NAND VARIABLE ONE CLOSE PARENTHESIS GOTO ONE ZERO ZERO ONE IF THE JUMP REGISTER IS EQUAL TO ZERO GOTO ONE ZERO ONE ZERO IF THE JUMP REGISTER IS EQUAL TO ONE
LINE NUMBER ONE ZERO ZERO ONE CODE PRINT ZERO
LINE NUMBER ONE ZERO ONE ZERO CODE PRINT ONE</pre>
当然，对于空格和换行符，显得太冗余了，去掉他们也没有问题。
<pre style="padding-left: 30px;">LINENUMBERONECODEREADGOTOONEZEROLINENUMBERONEZEROCODEVARIABLEZEROEQUALSTHEJUMPR
EGISTERGOTOONEONELINENUMBERONEONECODEREADGOTOONEZEROZEROLINENUMBERONEZEROZEROCO
DEVARIABLEONEEQUALSTHEJUMPREGISTERGOTOONEZEROONELINENUMBERONEZEROONECODETHEJUMP
REGISTEREQUALSOPENPARENTHESISVARIABLEZERONANDVARIABLEONECLOSEPARENTHESISNANDOPE
NPARENTHESISVARIABLEZERONANDVARIABLEONECLOSEPARENTHESISGOTOONEONEZEROIFTHEJUMPR
EGISTERISEQUALTOONEGOTOONEZEROZEROZEROIFTHEJUMPREGISTERISEQUALTOZEROLINENUMBERO
NEONEZEROCODEPRINTONEGOTOONEONEONELINENUMBERONEONEONECODEPRINTZEROLINENUMBERONE
ZEROZEROZEROCODETHEJUMPREGISTEREQUALSOPENPARENTHESISVARIABLEZERONANDVARIABLEZER
OCLOSEPARENTHESISNANDOPENPARENTHESISVARIABLEONENANDVARIABLEONECLOSEPARENTHESISG
OTOONEZEROZEROONEIFTHEJUMPREGISTERISEQUALTOZEROGOTOONEZEROONEZEROIFTHEJUMPREGIS
TERISEQUALTOONELINENUMBERONEZEROZEROONECODEPRINTZEROLINENUMBERONEZEROONEZEROCOD
EPRINTONE</pre>
<h4>Haifu</h4>
<a href="http://www.dangermouse.net/esoteric/haifu.html" target="_blank">Haifu</a>程序语言也是David Morgan-Mar 搞出来的。从命名上就可以看出来它是一个汉语拼音。正是如此，作者想使用东方的哲学来创造一种编程的语言。其中还有Yin（阴）和 Yang（阳）——相当于布尔变量中的True/False，当然，也有金（Metal）木（Wood）水（Water）火（Fire）土（Earth）。呵呵。
<ul>
	<li>Wood: tree, grass, cherry, oak.</li>
	<li>Fire: flame, ash, smoke, embers.</li>
	<li>Earth: soil, mountain, rock, plain.</li>
	<li>Metal: sword, iron, plough, knife.</li>
	<li>Water: rain, snow, river, ice.</li>
</ul>
自然出现了一张关系表：
<table border="1" cellspacing="0" cellpadding="3">
<tbody>
<tr>
<th>元素关系</th>
<th>操作</th>
</tr>
<tr>
<td>B 生A</td>
<td>A+B</td>
</tr>
<tr>
<td>B 克 A</td>
<td>A-B</td>
</tr>
<tr>
<td>B 怕 A</td>
<td>A/B</td>
</tr>
<tr>
<td>B 爱 A</td>
<td>A*B</td>
</tr>
<tr>
<td>B 就是 A</td>
<td>如果A和B都是阳，则是阳，否则是阴</td>
</tr>
</tbody>
</table>
<h4>Piet</h4>
David Morgan-Mar 发明的用位图编程的<a href="http://www.dangermouse.net/esoteric/piet.html" target="_blank">Piet语言</a>也是BT到了极致，你还记得前两的那个“<a title="我有一个Hello World的C++程序编译不过" href="https://coolshell.cn/articles/4170.html" target="_blank">我的hello world编不过去</a>”文章中的那个强人用windows的画图程序编程的例子吗？呵呵Piet完全是用位图编程的语言。

下面这个图片就是其Hello World的示例：
<p style="text-align: center;"><a href="http://www.topdesignmag.com/wp-content/uploads/2011/04/Piet_hello_big.png"><img class="aligncenter" title="Piet_hello_big" src="http://www.dangermouse.net/esoteric/piet/Piet_hello_big.png" alt="" width="150" height="145" /></a></p>
<p style="text-align: left;">再看看斐波拉契数列的程序示例:</p>
<p style="text-align: center;"><a href="http://www.dangermouse.net/esoteric/piet/fibbig.gif"><img class="aligncenter" src="http://www.dangermouse.net/esoteric/piet/fibbig.gif" alt="" width="110" height="121" /></a></p>
这里还有更多的示例：<a href="http://www.dangermouse.net/esoteric/piet/samples.html">http://www.dangermouse.net/esoteric/piet/samples.html</a>
<h4><strong>Malbolge</strong></h4>
<a href="http://www.lscheffer.com/malbolge.shtml" target="_blank">Malbolge语言</a>，是最早的一个以代码丑陋为目标而设计出的程序语言，你几乎不可能读懂Malbolge的代码。它共有8条指令，所有运算都基于3进制，控制程序流的唯一指令是无条件跳转。其是BenOlmstead在1998年引进公共领域的深奥程序语言，名称来源于“the eighth circle of hell in Dante’s Inferno”，之后更名为Malbolge。

这被认为是地狱级的编程语言。

看看它的Hello World程序：
<pre style="padding-left: 30px;"><code>('&%:9]!~}|z2Vxwv-,POqponl$Hjig%[[email protected]](/cdn-cgi/l/email-protection)@>}=<M:9wv6WsU2T|nm-,jcL(I&%$#"
 ]V?Tx<uVtT[[email protected]](/cdn-cgi/l/email-protection)?]!~|4XzyTT43Qsqq(Lnmkj"Fhg${[[email protected]](/cdn-cgi/l/email-protection)></code></pre>
<h4>Unlambda</h4>
关于<a href="http://www.madore.org/~david/programs/unlambda/" target="_blank">Unlambda语言</a>，David Madore是这个语言的发明人，他于1976年8月3日生于法国，其是法国-加拿大籍数学家和计算机科学爱好者）。在unlambda里，所有东西都是函数。基本操作就是S， K， 和I三个组合子。当然，unlambda也加入一些扩展，让程序稍微好些一点。
<pre style="padding-left: 30px;">ki
 ss
     ksss
               kkk
                               .H.e.l.l.o.,. .w.o.r.l.d.!
                        k
      k
  kss`k.*
```

#### Ook!


[Ook! 语言](http://www.dangermouse.net/esoteric/ook.html)也是David Morgan-Mar 发明的，与Brainfuck类似, 但用单词“`Ook！”`，“`Ook.`” 和“`Ook?`”代替。我们来看一个Hello World的一个示例：



```
Ook. Ook? Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook.
Ook. Ook. Ook. Ook. Ook! Ook? Ook? Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook.
Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook? Ook! Ook! Ook? Ook! Ook? Ook.
Ook! Ook. Ook. Ook? Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook.
Ook. Ook. Ook! Ook? Ook? Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook?
Ook! Ook! Ook? Ook! Ook? Ook. Ook. Ook. Ook! Ook. Ook. Ook. Ook. Ook. Ook. Ook.
Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook! Ook. Ook! Ook. Ook. Ook. Ook. Ook.
Ook. Ook. Ook! Ook. Ook. Ook? Ook. Ook? Ook. Ook? Ook. Ook. Ook. Ook. Ook. Ook.
Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook! Ook? Ook? Ook. Ook. Ook.
Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook? Ook! Ook! Ook? Ook! Ook? Ook. Ook! Ook.
Ook. Ook? Ook. Ook? Ook. Ook? Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook.
Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook! Ook? Ook? Ook. Ook. Ook.
Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook.
Ook. Ook? Ook! Ook! Ook? Ook! Ook? Ook. Ook! Ook! Ook! Ook! Ook! Ook! Ook! Ook.
Ook? Ook. Ook? Ook. Ook? Ook. Ook? Ook. Ook! Ook. Ook. Ook. Ook. Ook. Ook. Ook.
Ook! Ook. Ook! Ook! Ook! Ook! Ook! Ook! Ook! Ook! Ook! Ook! Ook! Ook! Ook! Ook.
Ook! Ook! Ook! Ook! Ook! Ook! Ook! Ook! Ook! Ook! Ook! Ook! Ook! Ook! Ook! Ook!
Ook! Ook. Ook. Ook? Ook. Ook? Ook. Ook. Ook! Ook.
```

下面是一些转换器和解释器：


* [Java Ook!-to-BrainF\*\*\* 和 BrainF\*\*\*-to-Ook! 转换器](http://www.dangermouse.net/esoteric/Ook.java).
* [Ook! Ruby解释器](http://www.ruby-lang.org/en/raa-list.rhtml?name=RubyOok).
* [Ook! Python解释器](http://www.orakel.ntnu.no/~oyving/code/python/pook.py).
* [Ook!  .Net 编译器叫Ook#](http://bluesorcerer.net/esoteric/ook.html)
* [Ook! perl 解释器](http://search.cpan.org/search?module=Acme::Ook).


#### **TMMLPTEALPAITAFNFAL**


你没看错，上面这一排毫无意义的字母是一个语言的名称。它是The Multi-Million Language Project To End All Language Projects And Isn’t That A Fine Name For A Language的缩写。[TMMLPTEALPAITAFNFAL语言](http://p-nand-q.com/humor/programming_languages/tmmlpteal.html)没有固定的语法规则，每一天都是不同的语法。例如，2000年10月13日你可以使用DIV但不能使用MOD；到了10月14日时你可以使用MOD了但DIV又不能用了。因此，你今天写的程序运行起来完全正常，但是到了明天就无法编译了。下面是一个TMMLPTEALPAITAFNFAL的Hello World程序，当然现在已经无法编译了。



```
DECLARE CELL 100 AS READPOS
 DECLARE 10 AS NEWLINE
 WRITE CHAR NEWLINE
 COPY "Hello, World" TO CELL 0
 COPY 0 TO READPOS
 WHILE READPOS INDIRECT DO GOSUB 300
 WRITE CHAR NEWLINE
 RETURN
LINE 300: WRITE CHAR READPOS INDIRECT
 ADD 1 TO READPOS
 RETURN
```

#### INTERCAL


[INTERCAL语言](http://catb.org/~esr/intercal/)（[Wikipedia](https://en.wikipedia.org/wiki/INTERCAL)）全称是“Compiler Language With No Pronounceable Acronym”。自认为是“超级黑客”的人可以试试用这个语言写程序。由老牌黑客[Don Woods](https://en.wikipedia.org/wiki/Don_Woods) 和 [James M. Lyon](https://en.wikipedia.org/wiki/James_M._Lyon) 在1972年发明，其是用来讽刺当时的那些编程语言。今天 这个语言有两个版本，一个是由牛人[Eric S. Raymond](https://en.wikipedia.org/wiki/Eric_S._Raymond)维护的C-INTERCAL，另一个是Claudio Calvelli 维护的CLC-INTERCAL。（**注**：在自由软件启蒙阶段，[Eric S. Raymond](https://en.wikipedia.org/wiki/Eric_S._Raymond)以如椽之笔呼啸而出，其核心著作被业界成为”五部曲”：《黑客道简史》（A Brief History of Hackerdom）、 《大教堂和市集》（The Cathedral and the Bazaar）、《如何成为一名黑客》（How To Become A Hacker）、《开拓智域》（Homesteading the Noosphere）、《魔法大锅炉》（The Magic Cauldron）。其中最著名的当然还是《大教堂和市集》，它在自由软件运动中的地位相当于基督教的《圣经》。而用黑客们的话说，这是”黑客藏经阁”的 第一个收藏。）


来看看其Hello World的程序：



```
DO ,1 <- #13
PLEASE DO ,1 SUB #1 <- #238
DO ,1 SUB #2 <- #108
DO ,1 SUB #3 <- #112
DO ,1 SUB #4 <- #0
DO ,1 SUB #5 <- #64
DO ,1 SUB #6 <- #194
DO ,1 SUB #7 <- #48
PLEASE DO ,1 SUB #8 <- #22
DO ,1 SUB #9 <- #248
DO ,1 SUB #10 <- #168
DO ,1 SUB #11 <- #24
DO ,1 SUB #12 <- #16
DO ,1 SUB #13 <- #162
PLEASE READ OUT ,1
PLEASE GIVE UP
```

#### HQ9++


[HQ9++语言](http://www.dangermouse.net/esoteric/hq9plusplus.html)同样是David Morgan-Mar 发明的，其带有四个指令的joke语言。


* **H**: 输出 [“hello,world”](http://www.esolangs.org/wiki/Hello%2C_world%21 "Hello,world!")
* **Q**: 输出程序员的源代码
* **9**: 打印 [“99 Bottles of Beer”](http://www.esolangs.org/wiki/99_bottles_of_beer "99 bottles of beer") 的歌词
* **+**: 累加器


#### **PerlYuYan**


[PerlYuYa](https://zh.wikipedia.org/wiki/PerlYuYan)n语言是一个能令人使用中文文言文开发程式 Perl 程式的 Perl 模块，由[唐凤](https://zh.wikipedia.org/wiki/%E5%94%90%E9%B3%B3)于2002年一月发表。它是[中文编程语言](https://zh.wikipedia.org/wiki/%E4%B8%AD%E6%96%87%E7%B7%A8%E7%A8%8B%E8%AA%9E%E8%A8%80)的尝试。作者利用中文的特质，将许多指令改成以一个中国汉字来表示，因而造成了文言语法的感觉。


看看下面的这段代码，相当的文言文啊。有兴趣可以[去CPAN上下载](http://search.cpan.org/~autrijus/Lingua-Sinica-PerlYuYan-0.03/)回来玩玩。



```
#!/usr/local/bin/perl

use Lingua::Sinica::PerlYuYan;

用警兮用嚴。

印道
一至一
哉兮

印編曰雜申雜申矣
  又纖曰龍鼠矣
    又曰一矣

亂曰
國無人莫我知兮    又何懷乎故都
既莫足與為美政兮  吾將從彭咸之所居
```

还有下面这个五言。



```
# The Sieve of Eratosthenes - 埃拉托斯芬篩法
use Lingua::Sinica::PerlYuYan;

  用籌兮用嚴。井涸兮無礙
。印曰最高矣  又道數然哉。
。截起吾純風  賦小入大合。
。習予吾陣地  並二至純風。
。當起段賦取  加陣地合始。
。陣地賦篩始  繫繫此雜段。
。終陣地兮印  正道次標哉。
。輸空接段點  列終註泰來。
```

 


#### 参考：


* [Esoteric\_programming\_languages](https://en.wikipedia.org/wiki/Category:Esoteric_programming_languages)
* [Top 13 Most Absurd Programming Languages](http://www.topdesignmag.com/top-13-most-absurd-programming-languages/)
* [Befunge语言和文言文编程](http://wei.si/blog/2011/04/befunge-and-perlyuyan/)
* [疯狂的编程语言——ENGLISH，Chef，Shakespeare](http://hi.baidu.com/namekin/blog/item/9f36f21fc6be296df724e452.html)
* [DM’s Esoteric Programming Languages](http://www.dangermouse.net/esoteric/)
* [十大另类程序语言（上）](http://www.matrix67.com/blog/archives/253 "Permanent Link to 十大另类程序语言（上）")
* [十大另类程序语言（下）](http://www.matrix67.com/blog/archives/255 "Permanent Link to 十大另类程序语言（下）")


看过这些，我我还有什么好说的呢，什么C/C++/Java，神马都是浮云了……


(全文完) 

