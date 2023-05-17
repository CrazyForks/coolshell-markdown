# bash 函数级重定向
>date: 2009-10-14T23:47:25+08:00


![bash 函数级重定向](/assets/images/coolshell.cn/wp-content/uploads/2009/08/bash.jpg "bash 函数级重定向")相信每一个人对于操作系统的重定向不会陌生了。就是>, >>, <, <<，关于重定向的基本知识我就不说了。这里主要讨论bash的重定向中的一个鲜为人知的东西，那就是bash脚本的函数也可以定义相关的重定向操作。这可不是命令级的重定向，这是函数级的重点向。这并不是一个新的东西，我只是想告诉大家一个已经存在了多年但却可能不被人常用的功能。


关于bash的这个函数级的重定向的语法其实很简单，你只需要在函数结尾时加上一些重定向的定义或指示符就可以了。下面是一个示例：



```
function mytest()
{
        ...
} < mytest.in > mytest.out 2> mytest.err
```

现在，只要是test被调用，那么，这个函数就会从mytest.in读入数据，并把输出重定向到mytest.out文件中，然后标准错误则输出到mytest.err文件中。是不是很简单？



因为函数级的重定向仅当在被函数调用的时候才会起作用，而且其也是脚本的一部分，所以，你自然也可以使用变量来借文件名。下面是一个示例：



```
#!/bin/bash

function mytest()
{
    echo Hello World CoolShell.cn
} >$out

out=mytest1.out
mytest
out=mytest2.out
mytest
```

这样一来，标准输出的重定向就可以随$out变量的改变而改变了。在上面的例子中，第一个调是重定向到mytest1.out，第二个则是到mytest2.out。



```
$ bash mytest.sh; more mytest?.out
::::::::::::::
mytest1.out
::::::::::::::
Hello World CoolShell.cn
::::::::::::::
mytest2.out
::::::::::::::
Hello World CoolShell.cn
```

正如前面所说的一样，这里并没有什么新的东西。上面的这个示例，转成传统的写法是：



```
#!/bin/bash

function mytest()
{
    echo Hello World CoolShell.cn
}
mytest >mytest1.out
mytest >mytest2.out
```

到此为此，好像这个feature并没有什么特别的实用之处。有一个可能比较实用的用法可能是把把你所有代码的的标准错误重定向到一个文件中去。如下面所示：



```
#!/bin/bash

log=err.log
function error()
{
    echo "$*" >&2
}
function mytest1()
{
    error mytest1 hello1 world1 coolshell.cn
}

function mytest2()
{
    error mytest2 hello2 world2 coolshell.cn
}

function main()
{
    mytest1
    mytest2
} 2>$log

main
```

运行上面的脚本，你可以得到下面的结果：



```
$ bash mytest.sh ;cat err.log
mytest1 hello1 world1 coolshell.cn
mytest2 hello2 world2 coolshell.cn
```

当然，你也可以不用定义一个函数，只要是{…} 语句块，就可以使用函数级的重定向，就如下面的示例一样：



```
#!/bin/bash

log=err.log
function error()
{
    echo "$*" >&2
}
function mytest1()
{
    error mytest1 hello1 world1 coolshell.cn
}

function mytest2()
{
    error mytest2 hello2 world2 coolshell.cn
}

{
mytest1
mytest2
} 2>$log
```

你也可以重定向 (…) 语句块，但那会导致语句被执行于一个sub-shell中，这可能会导致一些你不期望的行为或问题，因为sub-shell是在另一个进程中。


如果你问，我们是否可以覆盖函数级的重定向。答案是否定的。如果你试图这样做，那么，函数调用点的重定向会首先执行，然后函数定义上的重定向会将其覆盖。下面是一个示例：



```
#!/bin/bash

function mytest()
{
    echo hello world coolshell.cn
} >out1.txt
mytest >out2.txt
```

运行结果是，out2.txt会被建立，但里面什么也没有。


下面是一个重定向标准输入的例子：



```
#!/bin/bash

function mytest()
{
    while read line
    do
        echo $line
    done
} <<EOF
hello
coolshell.cn
EOF
mytest
```

下面是其运行结果：



```
$ bash mytest.sh
hello
coolshell.cn
```

(全文完)


