# StackOverflow的404错误页
>date: 2010-06-25T08:35:41+08:00


不知道大家有没有注意到StakeOverflow的[404错误页面](http://stackoverflow.com/404)？其显示了下面的这个图片：


![](http://sstatic.net/stackoverflow/img/polyglot-404.png)


这个是一个很有意思的图片，不知道你看懂了吗？看上去像Python，又像 Ruby，还像 Perl，当然也有 C的影子，还有[Brainfuck](https://coolshell.cn/articles/1142.html)。是的，这是一个杂交程序，杂交了Python，Ruby，Perl，C，还有Brainfuck（注意其中的#号），所有的语句都是输出“404”字符串。


关于这种杂交程序，本站以前也发布过《[C语言和sh脚本的杂交代码](https://coolshell.cn/articles/1824.html)》，大家可以前往一看。这样的有趣的玩法叫“[Polyglot](https://en.wikipedia.org/wiki/Polyglot_%28computing%29)”，也就是说，把N种语言写在一个文件中，然后，该文件在任何编译器下都可以运行，上述的那段代码在Python，Ruby，Perl，Brainfuck下都可以正常运行，也可以被C和的编译器编译通过，并被运行。


下面是这个图片的字符码，以供各位试试。




```
# define v putchar
#   define print(x) main(){v(4+v(v(52)-4));return 0;}/*
#>+++++++4+[>++++++<-]>++++.----.++++.*/
print(202*2);exit();
#define/*>[[email protected]](/cdn-cgi/l/email-protection)*/exit()
```

欢迎你留下你的看法。


（全文完）



