# C语言中史上最愚蠢的Bug
>date: 2011-08-26T10:17:48+08:00


本文来自“[The most stupid C bug ever](http://www.elpauer.org/?p=971)”，很有意思，分享给大家。我相信这样的bug，就算你是高手你也会犯的。你来看看作者犯的这个Bug吧。。


首先，作者想用一段程序来创建一个文件，如果有文件名的话，就创建真正的文件，如果没有的话，就调用?[tmpfile()](http://linux.die.net/man/3/tmpfile)?创建临时文件。他这段程序就是HTTP下载的C程序。code==200就是HTTP的返回码。



```
else if (code == 200) {     // Downloading whole file
    /* Write new file (plus allow reading once we finish) */
    g = fname ? fopen(fname, "w+") : tmpfile();
}
```

但是这个程序，只能在Unix/Linux下工作，因为 Microsoft 的?[tmpfile()的实现](http://msdn.microsoft.com/en-us/library/x8x7sakw.aspx)?居然选择了 C:\ 作为临时文件的存放目录，这对于那些没有管理员权限的人来说就出大问题了，在Windows 7下，就算你有管理员权限也会有问题。所以，上面的程序在Windows平台下需要用不同的方式来处理，不能直接使用Windows的tmpfile()函数。


于是作者就先把这个问题记下来，在注释中写下了FIXME：



```
else if (code == 200) {     // Downloading whole file
    /* Write new file (plus allow reading once we finish) */

    // FIXME Win32 native version fails here because
    //   Microsoft's version of tmpfile() creates the file in C:\
    g = fname ? fopen(fname, "w+") : tmpfile();
}
```

然后，作者觉得需要写一个跨平台的编译：



```
FILE * tmpfile ( void ) {
#ifndef _WIN32
    return tmpfile();
#else
    //code for Windows;
#endif
}
```

然后，作者觉得这样实现很不好，会发现名字冲突，因为这样一来这个函数太难看了。于是他重构了一下他的代码——写一个自己实现的tmpfile() – w32\_tmpfile，然后，在Windows 下用宏定义来重命名这个函数为tmpfile()。（陈皓注：这种用法是比较标准的跨平台代码的写法）




```
#ifdef _WIN32
  #define tmpfile w32_tmpfile
#endif

FILE * w32_tmpfile ( void ) {
    //code for Windows;
}
```

搞定！编译程序，运行。靠！居然没有调用到我的w32\_tmpfile()，什么问题？调试，单步跟踪，果然没有调用到！难道是问号表达式有问题？改成if – else 语句，好了！



```
if(NULL != fname) {
    g = fopen(fname, "w+");
} else {
    g = tmpfile();
}
```

问号表达式不应该有问题吧，难道我们的宏对问号表达式不起作用，这难道是编译器的预编译的一个bug？作者怀疑到。


现在我们把所有的代码连在一起看，并比较一下：


**能正常工作的代码**



```
#ifdef _WIN32
#  define tmpfile w32_tmpfile
#endif

FILE * w32_tmpfile ( void ) {
    code for Windows;
}

else if (code == 200) {     // Downloading whole file
    /* Write new file (plus allow reading once we finish) */
    // FIXME Win32 native version fails here because
    //     Microsoft's version of tmpfile() creates the file in C:\
    //g = fname ? fopen(fname, "w+") : tmpfile();
    if(NULL != fname) {
        g = fopen(fname, "w+");
    } else {
        g = tmpfile();
    }
}
```

**不能正常工作的代码**



```
#ifdef _WIN32
#  define tmpfile w32_tmpfile
#endif

FILE * w32_tmpfile ( void ) {
    code for Windows;
}

else if (code == 200) {     // Downloading whole file
    /* Write new file (plus allow reading once we finish) */
    // FIXME Win32 native version fails here because
    //    Microsoft's version of tmpfile() creates the file in C:\
    g = fname ? fopen(fname, "w+") : tmpfile();
}
```

也许你在一开始就看到了这个bug，但是作者没有。所有的问题都出在注释上：



```
/* Write new file (plus allow reading once we finish) */
// FIXME Win32 native version fails here because
//     Microsoft's version of tmpfile() creates the file in C:\

```

**你看到了最后那个C:\吗？在C中，“\” 代表此行没有结束，于是，后面的代码也成了注释。这就是这个bug的真正原因**！


而之所以改成if-else能工作的原因是因为作者注释了老的问号表达式的代码，所以，那段能工作的代码成了：



```
/* Write new file (plus allow reading once we finish) */
// FIXME Win32 native version fails here because Microsoft's version of tmpfile() creates the file in C:    //g = fname ? fopen(fname, "w+") : tmpfile();
if(NULL != fname) {
    g = fopen(fname, "w+");
} else {
    g = tmpfile();
}
```

我相信，当作者找到这个问题的原因后，一定会骂一句“妈的”！我也相信，这个bug花费了作者很多时间！


最后，我也share一个我以前犯的一个错。


我有一个小函数，需要传入一个int\* pInt的类型，然后我需要在我的代码里 把这个int\* pInt作除数。于是我的代码成了下面的这个样子：



> float result = num/\*pInt;  
> 
> ….
> 
> 
> /\*  some comments \*/
> 
> 
> -x<10 ? f(result):f(-result);
> 
> 


因为我在我当时用vi编写代码，所以没有语法高亮，而我的程序都编译通过了，但是却出现了很奇怪的事。我也不知道，用gdb调式的时候，发现有些语句直接就过了。这个问题让我花了很多时间，最后发现问题原来是没有空格导致的，TNND，下面我用代码高亮的插件来显示上面的代码，



```
float result = num/*pInt;
....

/*  some comments */

-x<10 ? f(result):f(-result); 
```

Holly Shit!  我的代码成了：


`float result = num-x<10 ? f(result):f(-result);`


妈的！我的这个错误在愚蠢程度上和上面那个作者出的错误有一拼。


（全文完）


