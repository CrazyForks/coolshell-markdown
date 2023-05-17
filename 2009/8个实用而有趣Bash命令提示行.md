# 8个实用而有趣Bash命令提示行
>date: 2009-09-07T17:44:22+08:00



很多人都对过命令行提示的重要性不屑一顾，甚至是一点都不关心。但是我却一点都不这么认为，一个好的命令行提示可以改变你使用命令的方式。为此，我在internet上找到一些非常实用，优秀，并有趣的bash的命令行提示。下面我将我最喜欢使用的一些命令行提示罗列如下。


注意  –  要使用下面这些提示，你可以拷贝粘贴这些以”PS1″打头的内容到你的终端上，为了使你的改变永久生效，还要将这些内容粘贴到你使用用户的~/.bashrc文件中去。



### 1. 在成功执行的命令上增加一个笑脸符号


这个命令提示行可能是这个命令行提示列表中最有趣的一个，但是它也依然有使用的价值。这个提示的想法是基于当你命令被成功执行，你将会得到一个笑脸作为你的命令行提示，一旦的命令执行失败，命令行提示将会换成一个哭脸。


例子：


![bashprompts-happyface](http://images.maketecheasier.com/2009/08/bashprompts-happyface.jpg)


代码：


PS1=”\`if [ \$? = 0 ]; then echo \[\e[33m\]^_^\[\e[0m\]; else echo \[\e[31m\]O_O\[\e[0m\]; fi\`[\[[email protected]](/cdn-cgi/l/email-protection)\h:\w]\\$ “


### **2.更改失败命令的颜色**


下面这个命令行提示是我最喜欢的命令行之一。和上一个相似，这个命令行提示的颜色会在你最后一个命令运行失败后改变，而且这个命令行长路径会缩短输入命令的空间，这个命令提示还包含了bash 每个历史命令的命令号，以方便重新提取运行。


例子：


![bashprompts-hurring](http://images.maketecheasier.com/2009/08/bashprompts-hurring.jpg)


代码：


PS1=”\[\033[0;33m\][\!]\`if [[ \$? = "0" ]]; then echo "\\[\\033[32m\\]"; else echo "\\[\\033[31m\\]"; fi\`[\u.\h: \`if [[` |wc -c|tr -d ” “ `> 18 ]]; then echo "\\W"; else echo "\\w"; fi\`]\$\[\033[0m\] “; echo -ne “\033]0;`hostname -s`:`pwd`\007″‘


### 3. 多行提示


如果你是喜欢命令行提示中包含完整信息的那一类人，那么下边就有一个适合于你的命令行提示。这个命令行提示信息中包含日期/时间，全路径，用户，主机，活动终端，甚至包含文件数和占用空间等。


例子：


  
![bashprompts-informant](http://images.maketecheasier.com/2009/08/bashprompts-informant.jpg)


代码：


PROMPT\_COMMAND=’PS1=”\n\[\033[35m\]\$(/bin/date)\n\[\033[32m\]\w\n\[\033[1;31m\]\[[email protected]](/cdn-cgi/l/email-protection)\h: \[\033[1;34m\]\$(/usr/bin/tty | /bin/sed -e ‘s:/dev/::’): \[\033[1;36m\]\$(/bin/ls -1 | /usr/bin/wc -l | /bin/sed ‘s: ::g’) files \[\033[1;33m\]\$(/bin/ls -lah | /bin/grep -m 1 total | /bin/sed ‘s/total //’)b\[\033[0m\] -> \[\033[0m\]”‘


 


### 4. 多颜色提示


这个命令行提示除了使用了不同颜色来区别不同信息外，它并没有很特别的地方。就像你看到的那样，它提供了时间，用户名，主机名，当前目录。相当少的信息，但是非常地实用。


例子：


![bashprompts-4](http://images.maketecheasier.com/2009/08/bashprompts-4.jpg)


代码：


PS1=”\[\033[35m\]\t\[\033[m\]-\[\033[36m\]\u\[\033[m\]@\[\033[32m\]\h:\[\033[33;1m\]\w\[\033[m\]\$ “


 


### 5.显示完整路径


这是一个良好，简洁，最小的2行提示(加上顶上的空行)。在第一行你能得到一个全路径信息，在第二行是一个用户名。如果你对每个命令提示行的空行不爽的话，你只要移走第一个\n就OK了


例子：


![bashprompts-5](http://images.maketecheasier.com/2009/08/bashprompts-5.jpg)


代码：


PS1=”[\[\033[32m\]\w]\[\033[0m\]\n\[\033[1;36m\]\u\[\033[1;33m\]-> \[\033[0m\]”


 


### 6. 显示后台运行任务数


这是另外的一个两行提示，但是这个两行提示具有更多的之前我们没有的信息。第一行是显示通常的[[email protected]](/cdn-cgi/l/email-protection)和全路径等信息。在第二行我们可以得到命令执行历史序号和一个后台运行任务个数信息。


例子：


 


![bashprompts-6](http://images.maketecheasier.com/2009/08/bashprompts-61.jpg)


代码：


PS1=’\[\e[1;32m\]\[[email protected]](/cdn-cgi/l/email-protection)\H:\[\e[m\] \[\e[1;37m\]\w\[\e[m\]\n\[\e[1;33m\]hist:\! \[\e[0;33m\] \[\e[1;31m\]jobs:\j \$\[\e[m\] ‘


 


### 7. 显示路径信息


这是一个非常眩的设计。我们可以从这个命令行提示信息的第一行中获取到用户/主机，运行任务数，和时间日期等信息。在第二行我们可以得到当前目录的文件数和他们占用的磁盘空间。


例子：


 


![bashprompts-7](http://images.maketecheasier.com/2009/08/bashprompts-7.jpg)


代码:


PS1=”\n\[\e[30;1m\]\[\016\]l\[\017\](\[\e[34;1m\]\[[email protected]](/cdn-cgi/l/email-protection)\h\[\e[30;1m\])-(\[\e[34;1m\]\j\[\e[30;1m\])-(\[\e[34;1m\]\@ \d\[\e[30;1m\])->\[\e[30;1m\]\n\[\016\]m\[\017\]-(\[\[\e[32;1m\]\w\[\e[30;1m\])-(\[\e[32;1m\]\$(/bin/ls -1 | /usr/bin/wc -l | /bin/sed ‘s: ::g’) files, \$(/bin/ls -lah | /bin/grep -m 1 total | /bin/sed ‘s/total //’)b\[\e[30;1m\])–> \[\e[0m\]”


### 8. My Prompt


最后这个命令提示行是我个人最喜欢的使用的命令提示行。它是#7的一个修改，这个命令提示行只包含我最希望知道的信息，因此节省了它的占用空间。我偏爱两行风格，因为这样不仅可以让我看到全路径信息，而且不影响我命令输入的可视空间。


例子:


![bashprompts-8](http://images.maketecheasier.com/2009/08/bashprompts-8.jpg)


代码:


PS1=”\n\[\e[32;1m\](\[\e[37;1m\]\u\[\e[32;1m\])-(\[\e[37;1m\]jobs:\j\[\e[32;1m\])-(\[\e[37;1m\]\w\[\e[32;1m\])\n(\[\[\e[37;1m\]! \!\[\e[32;1m\])-> \[\e[0m\]”


如果你愿意共享你的命令提示行，请在将这些命令提示代码加在下面的评论中。



PS1=”\n\[\033[35m\]\$(/bin/date)\n\[\033[32m\]\w\n\[\033[1;31m\]\[[email protected]](/cdn-cgi/l/email-protection)\h: \[\033[1;34m\]\$(/usr/bin/tty | /bin/sed


-e ‘s:/dev/::’): \[\033[1;36m\]\$(/bin/ls -1 | /usr/bin/wc -l | /bin/sed ‘s: ::g’) files \[\033[1;33m\]\$(/bin/ls -lah | /bin/grep -m 1 total | /bin/sed ‘s/total //’)b\[\033[0m\] -> \[\033[0m\]”



[出处](http://maketecheasier.com/8-useful-and-interesting-bash-prompts/2009/09/04 "原文出处")


