# C语言的谜题
>date: 2009-05-31T17:39:58+08:00


这几天，本站推出了几篇关于C语言的很多文章如下所示：


* **语言的歧义** [[酷壳链接](/2009/%E8%AF%AD%E8%A8%80%E7%9A%84%E6%AD%A7%E4%B9%89.md)] [[CSDN链接](http://blog.csdn.net/haoel/archive/2009/05/18/4197010.aspx)]
* **谁说C语言很简单？** [[酷壳链接](/2009/%E8%B0%81%E8%AF%B4C%E8%AF%AD%E8%A8%80%E5%BE%88%E7%AE%80%E5%8D%95%EF%BC%9F.md)] [[CSDN链接](http://blog.csdn.net/haoel/archive/2009/05/26/4217950.aspx)]
* **6个变态的C语言Hello World程序** [[酷壳链接](/2009/6%E4%B8%AA%E5%8F%98%E6%80%81%E7%9A%84C%E8%AF%AD%E8%A8%80Hello%20World%E7%A8%8B%E5%BA%8F.md)] [[CSDN链接](http://blog.csdn.net/haoel/archive/2009/05/26/4217565.aspx)]
* **如何加密/弄乱C源代码** [[酷壳链接](/2009/%E5%A6%82%E4%BD%95%E5%8A%A0%E5%AF%86-%E6%B7%B7%E4%B9%B1C%E6%BA%90%E4%BB%A3%E7%A0%81.md)] [[CSDN链接](http://blog.csdn.net/haoel/archive/2009/05/30/4225974.aspx)]
* **C语言的谜题** [[酷壳链接](/2009/C%20%E8%AF%AD%E8%A8%80%E6%95%B4%E5%9E%8B%E8%B0%9C%E9%A2%98.md)] [[CSDN链接](http://blog.csdn.net/haoel/archive/2009/06/01/4231029.aspx)]


我们可以看到很多C语言相关的一些东西。比如《语言的歧义》主要告诉了大家C语言中你意想不到的错误以及一些歧义上的东西。而《谁说C语言很简单》则通过一些看似你从来不可能写出的代码来告诉大家C语言并不是一件容易事情。《6个变态的hello world》和《如何弄乱C的源代码》则以一种极端的方式告诉大家，不要以为咱们自己写不出混乱的代码，每个程序员其实都有把代码搞得一团乱的潜质。通过这些文章，相信你对编程或是你觉得很简单的C语言有了一些了解。是的，很不容易吧，以前是不是低估了编程和C语言？今天是否我们又在低估C++和Java呢？


本篇文章《C语言的谜题》展示了14个C语言的迷题以及答案，代码应该是足够清楚的，而且我也相信有相当的一些例子可能是我们日常工作可能会见得到的。通过这些迷题，希望你能更了解C语言。如果你不看答案，不知道是否有把握回答各个谜题？让我们来试试。



**1、下面的程序并不见得会输出 hello-std-out，你知道为什么吗？**



```
#include <stdio.h>
#include <unistd.h>
int main()  
{
    while(1)
    {
        fprintf(stdout,"hello-std-out");
        fprintf(stderr,"hello-std-err");
        sleep(1);
    }
    return 0;
}

```

**参考答案**：stdout和stderr是不是同设备描述符。stdout是块设备，stderr则不是。对于块设备，只有当下面几种情况下才会被输入，1）遇到回车，2）缓冲区满，3）flush被调用。而stderr则不会。


**2、下面的程序看起来是正常的，使用了一个逗号表达式来做初始化。**可惜这段程序是有问题的。你知道为什么呢？



```
#include <stdio.h>

int main()
{
    int a = 1,2;
    printf("a : %d\n",a);
    return 0;
}

```

**参考答案**：这个程序会得到编译出错（语法出错），逗号表达式是没错，可是在初始化和变量声明时，逗号并不是逗号表达式的意义。这点要区分，要修改上面这个程序，你需要加上括号： int a = (1,2);


**3、下面的程序会有什么样的输出呢？**



```
#include <stdio.h>
int main()
{
    int i=43;
    printf("%d\n",printf("%d",printf("%d",i)));
    return 0;
}

```

**参考答案**：程序会输出4321，你知道为什么吗？要知道为什么，你需要知道printf的返回值是什么。printf返回值是输出的字符个数。


**4、下面的程序会输出什么？**



```
#include <stdio.h>
int main()  
{
    float a = 12.5;
    printf("%d\n", a);
    printf("%d\n", (int)a);
    printf("%d\n", *(int *)&a);
    return 0;  
}

```

**参考答案**：  

该项程序输出如下所示，  

0  

12  

1095237632  

原因是：浮点数是4个字节，12.5f 转成二进制是：01000001010010000000000000000000，十六进制是：0x41480000，十进制是：1095237632。所以，第二和第三个输出相信大家也知道是为什么了。而对于第一个，为什么会输出0，我们需要了解一下float和double的内存布局，如下：


* **float**: 1位符号位(s)、8位指数(e)，23位尾数(m,共32位)
* **double**: 1位符号位(s)、11位指数(e)，52位尾数(m,共64位)


然后，我们还需要了解一下printf由于类型不匹配，所以，会把float直接转成double，注意，12.5的float和double的内存二进制完全不一样。别忘了在x86芯片下使用是的反字节序，高位字节和低位字位要反过来。所以：


* **float版**：0x41480000 (在内存中是：00 00 48 41)
* **double版**：0x4029000000000000 (在内存中是：00 00 00 00 00 00 29 40)


而我们的%d要求是一个4字节的int，对于double的内存布局，我们可以看到前四个字节是00，所以输出自然是0了。


这个示例向我们说明printf并不是类型安全的，这就是为什么C++要引如cout的原因了。


**5、下面，我们再来看一个交叉编译的事情，下面的两个文件可以编译通过吗？如果可以通过，结果是什么？**


file1.c 



```
  int arr[80];

```

file2.c 



```
extern int *arr;
int main()  
{      
    arr[1] = 100;
    printf("%d\n", arr[1]);
    return 0;  
}

```

**参考答案**：该程序可以编译通过，但运行时会出错。为什么呢？原因是，在另一个文件中用 extern int \*arr来外部声明一个数组并不能得到实际的期望值，因为他们的类型并不匹配。所以导致指针实际并没有指向那个数组。注意：一个指向数组的指针，并不等于一个数组。修改：extern int arr[]。（参考：ISO C语言 6.5.4.2 节）


**6、请说出下面的程序输出是多少？并解释为什么？**（注意，该程序并不会输出 “b is 20″）



```
#include <stdio.h>
int main()  
{      
    int a=1;      
    switch(a)      
    {   
        int b=20;          
        case 1: 
            printf("b is %d\n",b);
            break;
        default:
            printf("b is %d\n",b);
            break;
    }
    return 0;
}

```

**参考答案**：该程序在编译时，可能会出现一条warning: unreachable code at beginning of switch statement。我们以为进入switch后，变量b会被初始化，其实并不然，因为switch-case语句会把变量b的初始化直接就跳过了。所以，程序会输出一个随机的内存值。


**7、请问下面的程序会有什么潜在的危险？**



```
#include <stdio.h>
int main()  
{      
    char str[80];
    printf("Enter the string:");
    scanf("%s",str);
    printf("You entered:%s\n",str);
    return 0;
}

```

**参考答案**：本题很简单了。这个程序的潜在问题是，如果用户输入了超过80个长度的字符，那么就会有数组越界的问题了，你的程序很有可以及会crash了。


**8、请问下面的程序输出什么？**



```
#include <stdio.h>
int main()  
{
    int i;
    i = 10;
    printf("i : %d\n",i);
    printf("sizeof(i++) is: %d\n",sizeof(i++));
    printf("i : %d\n",i);
    return 0;
}

```

**参考答案**：如果你觉得输出分别是，10，4，11，那么你就错了，错在了第三个，第一个是10没有什么问题，第二个是4，也没有什么问题，因为是32位机上一个int有4个字节。但是第三个为什么输出的不是11呢？居然还是10？原因是，sizeof不是一个函数，是一个操作符，其求i++的类型的size，这是一件可以在程序运行前（编译时）完全的事情，所以，sizeof(i++)直接就被4给取代了，在运行时也就不会有了i++这个表达式。


**9、请问下面的程序的输出值是什么？**



```
#include <stdio.h>
#include <stdlib.h>

#define SIZEOF(arr) (sizeof(arr)/sizeof(arr[0]))
#define PrintInt(expr) printf("%s:%d\n",#expr,(expr))

int main()
{
    /* The powers of 10 */
    int pot[] = {
                    0001,
                    0010,
                    0100,
                    1000
                };

    int i;
    for(i=0;i<SIZEOF(pot);i++)
        PrintInt(pot[i]);
        
    return 0;
}

```

**参考答案**：好吧，如果你对于PrintInt这个宏有问题的话，你可以去看一看《[语言的歧义](/2009/%E8%AF%AD%E8%A8%80%E7%9A%84%E6%AD%A7%E4%B9%89.md)》中的第四个示例。不过，本例的问题不在这里，本例的输出会是：1，8，64，1000，其实很简单了，以C/C++中，以0开头的数字都是八进制的。


**10、请问下面的程序输出是什么？（绝对不是10）**



```
#include 
#define PrintInt(expr) printf("%s : %dn",#expr,(expr))

int main() 
{
 int y = 100;
 int \*p;
 p = malloc(sizeof(int));
 \*p = 10;
 y = y/\*p; /\*dividing y by \*p \*/;
 PrintInt(y);
 return 0;
}

```

**参考答案**：本题输出的是100。为什么呢？问题就出在 y = y/\*p;上了，我们本来想的是 y / (\*p) ，然而，我们没有加入空格和括号，结果y/\*p中的 /\*被解释成了注释的开始。于是，这也是整个恶梦的开始。


**11、下面的输出是什么？**



```
#include <stdio.h>
int main()  
{
    int i = 6;
    if( ((++i < 7) && ( i++/6)) || (++i <= 9))
        ;

    printf("%d\n",i);
    return 0;
}

```

**参考答案**：本题并不简单的是考前缀++或反缀++，本题主要考的是&&和||的短路求值的问题。所为短路求值：对于（条件1 && 条件2），如果“条件1”是false，那“条件2”的表达式会被忽略了。对于（条件1 || 条件2），如果“条件1”为true，而“条件2”的表达式则被忽略了。所以，我相信你会知道本题的答案是什么了。


**12、下面的C程序是合法的吗？如果是，那么输出是什么？**



```
#include <stdio.h>
int main()  
{ 
    int a=3, b = 5;

    printf(&a["Ya!Hello! how is this? %s\n"], &b["junk/super"]);
    
    printf(&a["WHAT%c%c%c  %c%c  %c !\n"], 1["this"],
        2["beauty"],0["tool"],0["is"],3["sensitive"],4["CCCCCC"]);
        
    return 0;  
}

```

**参考答案**：  

本例是合法的，输出如下：



>  Hello! how is this? super  
> 
> That is C !
> 
> 


本例主要展示了一种另类的用法。下面的两种用法是相同的：



>  “hello”[2]  
> 
> 2[“hello”]
> 
> 


如果你知道：a[i] 其实就是 \*(a+i)也就是 \*(i+a)，所以如果写成 i[a] 应该也不难理解了。


**13、请问下面的程序输出什么？**（假设：输入 Hello, World）



```
#include <stdio.h>
int main()  
{ 
    char dummy[80];
    printf("Enter a string:\n");
    scanf("%[^r]",dummy);
    printf("%s\n",dummy);
    return 0;
}

```

**参考答案**：本例的输出是“Hello, Wo”，scanf中的”%[^r]”是从中作梗的东西。意思是遇到字符r就结束了。


**14、下面的程序试图使用“位操作”来完成“乘5”的操作，不过这个程序中有个BUG，你知道是什么吗？**



```
#include <stdio.h>
#define PrintInt(expr) printf("%s : %d\n",#expr,(expr))
int FiveTimes(int a)  
{
    int t;
    t = a<<2 + a;
    return t;
}

int main()  
{
    int a = 1, b = 2,c = 3;
    PrintInt(FiveTimes(a));
    PrintInt(FiveTimes(b));
    PrintInt(FiveTimes(c));
    return 0;
}

```

**参考答案**：本题的问题在于函数FiveTimes中的表达式“t = a<<2 + a;”，对于a<<2这个位操作，优先级要比加法要低，所以这个表达式就成了“t = a << (2+a)”，于是我们就得不到我们想要的值。该程序修正如下：

```
int FiveTimes(int a)  
{
    int t;
    t = (a<<2) + a;
    return t;
}

```

（全文完）




