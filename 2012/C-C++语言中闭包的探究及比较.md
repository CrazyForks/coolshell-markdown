# C-C++语言中闭包的探究及比较
>date: 2012-09-20T08:17:07+08:00


（**感谢投稿人 [@思禽饮霜](http://weibo.com/jasonmblog)**）


这里主要讨论的是C语言的扩展特性[block](https://en.wikipedia.org/wiki/Blocks_(C_language_extension))。该特性是Apple为C、C++、Objective-C增加的扩展，让这些语言可以用类Lambda表达式的语法来创建[闭包](https://en.wikipedia.org/wiki/Closure_(computer_science))。前段时间，在对CoreData存取进行封装时（让开发人员可以更简洁快速地写相关代码），我对block机制有了进一步了解，觉得可以和C++ 11中的Lambda表达式相互印证，所以最近重新做了下整理，分享给大家。




目录



* [0. 简单创建匿名函数](#0_%E7%AE%80%E5%8D%95%E5%88%9B%E5%BB%BA%E5%8C%BF%E5%90%8D%E5%87%BD%E6%95%B0 "0. 简单创建匿名函数")
* [1. 从语法上看如何捕获外部变量](#1_%E4%BB%8E%E8%AF%AD%E6%B3%95%E4%B8%8A%E7%9C%8B%E5%A6%82%E4%BD%95%E6%8D%95%E8%8E%B7%E5%A4%96%E9%83%A8%E5%8F%98%E9%87%8F "1. 从语法上看如何捕获外部变量")
* [2. 从语法上看如何修改外部变量](#2_%E4%BB%8E%E8%AF%AD%E6%B3%95%E4%B8%8A%E7%9C%8B%E5%A6%82%E4%BD%95%E4%BF%AE%E6%94%B9%E5%A4%96%E9%83%A8%E5%8F%98%E9%87%8F "2. 从语法上看如何修改外部变量")
* [3. 从实现上看如何捕获外部变量](#3_%E4%BB%8E%E5%AE%9E%E7%8E%B0%E4%B8%8A%E7%9C%8B%E5%A6%82%E4%BD%95%E6%8D%95%E8%8E%B7%E5%A4%96%E9%83%A8%E5%8F%98%E9%87%8F "3. 从实现上看如何捕获外部变量")
* [4. 从实现上看如何修改外部变量（\_\_block类型指示符）](#4_%E4%BB%8E%E5%AE%9E%E7%8E%B0%E4%B8%8A%E7%9C%8B%E5%A6%82%E4%BD%95%E4%BF%AE%E6%94%B9%E5%A4%96%E9%83%A8%E5%8F%98%E9%87%8F%EF%BC%88_block%E7%B1%BB%E5%9E%8B%E6%8C%87%E7%A4%BA%E7%AC%A6%EF%BC%89 "4. 从实现上看如何修改外部变量（__block类型指示符）")
* [5. 背后的内存管理动作](#5_%E8%83%8C%E5%90%8E%E7%9A%84%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86%E5%8A%A8%E4%BD%9C "5. 背后的内存管理动作")
	+ [5.1 拷贝block结构体](#51_%E6%8B%B7%E8%B4%9Dblock%E7%BB%93%E6%9E%84%E4%BD%93 "5.1 拷贝block结构体")
	+ [5.2 拷贝捕获的变量（\_\_block变量）](#52_%E6%8B%B7%E8%B4%9D%E6%8D%95%E8%8E%B7%E7%9A%84%E5%8F%98%E9%87%8F%EF%BC%88_block%E5%8F%98%E9%87%8F%EF%BC%89 "5.2 拷贝捕获的变量（__block变量）")
	+ [5.3 \_\_forwarding指针的作用](#53_forwarding%E6%8C%87%E9%92%88%E7%9A%84%E4%BD%9C%E7%94%A8 "5.3 __forwarding指针的作用")
* [参考资料：](#%E5%8F%82%E8%80%83%E8%B5%84%E6%96%99%EF%BC%9A "参考资料：")

#### 0. 简单创建匿名函数


下面两段代码的作用都是创建匿名函数并调用，输出Hello, World语句。分别使用Objective-C和C++ 11：


`^{ printf("Hello, World!\n"); } ();`  

`[] { cout << "Hello, World" << endl; } ();`


Lambda表达式的一个好处就是让开发人员可以在需要的时候临时创建函数，便捷。


在创建闭包（或者说Lambda函数）的语法上，Objective-C采用的是上尖号^，而C++ 11采用的是配对的方括号[]。


不过“匿名函数”一词是针对程序员而言的，编译器还是采取了一定的命名规则。


比如下面Objective-C代码中的3个block，



```
#import <Foundation/Foundation.h>

int (^maxBlk)(int , int) = ^(int m, int n){ return m > n ? m : n; };

int main(int argc, const char * argv[])
{
    ^{ printf("Hello, World!\n"); } ();

    int i = 1024;
    void (^blk)(void) = ^{ printf("%d\n", i); };
    blk();

    return 0;
}

```

会产生对应的3个函数：




```
__maxBlk_block_func_0
__main_block_func_0
__main_block_func_1

```

可见函数的命名规则为：\_\_{$Scope}\_block\_func\_{$index}。其中{$Scope}为block所在函数，如果{$Scope}为全局就取block本身的名称；{$index}表示该block在{$Scope}作用域内出现的顺序（第几个block）。


#### 1. 从语法上看如何捕获外部变量


在上面的代码中，已经看到“匿名函数”可以直接访问外围作用域的变量i：



```
int i = 1024;
void (^blk)(void) = ^{ printf("%d\n", i); };
blk();

```

当匿名函数和non-local变量结合起来，就形成了闭包（个人看法）。  

这一段代码可以成功输出i的值。


我们把一样的逻辑搬到C++上：



```
int i = 1024;
auto func = [] { printf("%d\n", i); };
func();

```

GCC会输出：错误：‘i’未被捕获。可见在C++中无法直接捕获外围作用域的变量。


以BNF来表示Lambda表达式的上下文无关文法，存在：



```
lambda-expression : lambda-introducer lambda-parameter-declarationopt compound-statement
lambda-introducer : [ lambda-captureopt ]

```

因此，方括号中还可以加入一些选项：



```
[]        Capture nothing (or, a scorched earth strategy?)
[&]       Capture any referenced variable by reference
[=]       Capture any referenced variable by making a copy
[=, &foo] Capture any referenced variable by making a copy, but capture variable foo by reference
[bar]     Capture bar by making a copy; don't copy anything else
[this]    Capture the this pointer of the enclosing class

```

根据文法，对代码加以修改，使其能够成功运行：



```
bash-3.2# vi testLambda.cpp
bash-3.2# g++-4.7 -std=c++11 testLambda.cpp -o testLambda
bash-3.2# ./testLambda
1024
bash-3.2# cat testLambda.cpp
#include <iostream>

using  namespace std;

int main()
{
     int i = 1024;
     auto func = [=] { printf("%d\n", i); };
     func();

     return 0;
}
bash-3.2#

```

#### 2. 从语法上看如何修改外部变量


上面代码中使用了符号=，通过拷贝方式捕获了外部变量i。  

但是如果尝试在Lambda表达式中修改变量i：



```
auto func = [=] { i = 0; printf("%d\n", i); };

```

会得到错误：



```
testLambda.cpp: 在 lambda 函数中:
testLambda.cpp:9:24: 错误：向只读变量‘i’赋值

```

可见*通过拷贝方式捕获的外部变量是只读的*。Python中也有一个类似的经典case，个人觉得有相通之处：



```
x = 10
def foo():
    print(x)
    x += 1
foo()

```

这段代码会抛出UnboundLocalError错误，原因可以参见[FAQ](https://docs.python.org/faq/programming.html#why-am-i-getting-an-unboundlocalerror-when-the-variable-has-a-value)。


在C++的闭包语法中，如果需要对外部变量的写权限，可以使用符号&，通过*引用方式*捕获：



```
int i = 1024;
auto func = [&] { i = 0; printf("%d\n", i); };
func();

```

反过来，将修改外部变量的逻辑放到Objective-C代码中：



```
int i = 1024;
void (^blk)(void) = ^{ i = 0; printf("%d\n", i); };
blk();

```

会得到如下错误：



```
main.m:14:29: error: variable is not assignable (missing __block type specifier)
    void (^blk)(void) = ^{ i++; printf("%d\n", i); };
                           ~^
1 error generated.

```

可见在block的语法中，默认捕获的外部变量也是只读的，如果要修改外部变量，需要使用\_\_block类型指示符进行修饰。  

为什么呢？请继续往下看 ：）


#### 3. 从实现上看如何捕获外部变量


闭包对于编程语言来说是一种语法糖，包括Block和Lambda，是为了方便程序员开发而引入的。因此，对Block特性的支持会落地在*编译器前端*，中间代码将会是C语言。


先看如下代码会产生怎样的中间代码。



```
int main(int argc, const char * argv[])
{
    int i = 1024;
    void (^blk)(void) = ^{ printf("%d\n", i); };
    blk();

    return 0;
}

```

首先是block结构体的实现：



```
#ifndef BLOCK_IMPL
#define BLOCK_IMPL
struct __block_impl {
    void *isa;
    int Flags;
    int Reserved;
    void *FuncPtr;
};
// 省略部分代码

#endif

```

第一个成员isa指针用来表示该结构体的类型，使其仍然处于Cocoa的对象体系中，类似Python对象系统中的PyObject。


第二、三个成员是标志位和保留位。


第四个成员是对应的“匿名函数”，在这个例子中对应函数：



```
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
    int i = __cself->i; // bound by copy
    printf("%d\n", i);
}

```

函数\_\_main\_block\_func\_0引入了参数\_\_cself，为struct \_\_main\_block\_impl\_0 \*类型，从参数名称就可以看出它的功能类似于C++中的this指针或者Objective-C的self。  

而struct \_\_main\_block\_impl\_0的结构如下：



```
struct __main_block_impl_0 {
    struct __block_impl impl;
    struct __main_block_desc_0* Desc;
    int i;
    __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int _i, int flags=0) : i(_i) {
        impl.isa = &_NSConcreteStackBlock;
        impl.Flags = flags;
        impl.FuncPtr = fp;
        Desc = desc;
    }
};

```

从\_\_main\_block\_impl\_0这个名称可以看出该结构体是为main函数中第零个block服务的，即示例代码中的blk；也可以猜到不同场景下的block对应的结构体不同，但本质上第一个成员一定是struct \_\_block\_impl impl，因为这个成员是block实现的基石。


结构体\_\_main\_block\_impl\_0又引入了一个新的结构体，也是中间代码里最后一个结构体：



```
static struct __main_block_desc_0 {
    unsigned long reserved;
    unsigned long Block_size;
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)};

```

可以看出，这个描述性质的结构体包含的价值信息就是struct \_\_main\_block\_impl\_0的大小。


最后剩下main函数对应的中间代码：



```
int main(int argc, const char * argv[])
{
    int i = 1024;
    void (*blk)(void) = (void (*)(void))&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, i);
    ((void (*)(struct __block_impl *))((struct __block_impl *)blk)->FuncPtr)((struct __block_impl *)blk);

    return 0;
}

```

从main函数对应的中间代码可以看出执行block的本质就是以block结构体自身作为\_\_cself参数，这里对应\_\_main\_block\_impl\_0，通过结构体成员FuncPtr函数指针调用对应的函数，这里对应\_\_main\_block\_func\_0。


其中，局部变量i是以值传递的方式拷贝一份，作为\_\_main\_block\_impl\_0的构造函数的参数，并以初始化列表的形式赋值给其成员变量i。所以，基于这样的实现，不允许直接修改外部变量是合理的——因为按值传递根本改不到外部变量。


#### 4. 从实现上看如何修改外部变量（\_\_block类型指示符）


如果想要修改外部变量，则需要用\_\_block来修饰：



```
int main(int argc, const char * argv[])
{
    __block int i = 1024;
    void (^blk)(void) = ^{ i = 0; printf("%d\n", i); };
    blk();

    return 0;
}

```

此时再看中间代码，发现多了一个结构体：



```
struct __Block_byref_i_0 {
    void *__isa;
    __Block_byref_i_0 *__forwarding;
    int __flags;
    int __size;
    int i;
};

```

于是，用\_\_block修饰的int变量i化身为\_\_Block\_byref\_i\_0结构体的最后一个成员变量。


代码中blk对应的结构体也发生了变化：



```
struct __main_block_impl_0 {
    struct __block_impl impl;
    struct __main_block_desc_0* Desc;
    __Block_byref_i_0 *i; // by ref
    __main_block_impl_0(void *fp, struct__main_block_desc_0 *desc, __Block_byref_i_0 *_i, int flags=0) : i(_i->__forwarding) {
        impl.isa = &_NSConcreteStackBlock;
        impl.Flags = flags;
        impl.FuncPtr = fp;
        Desc = desc;
    }
};

```

\_\_main\_block\_impl\_0发生的变化就是int类型的成员变量i换成了\_\_Block\_byref\_i\_0 \*类型，从名称可以看出现在要通过引用方式来捕获了。


对应的函数也不同了：



```
static void __main_block_func_0(struct  __main_block_impl_0 *__cself) {
    __Block_byref_i_0 *i = __cself->i; // bound by ref
    (i->__forwarding->i) = 0; // 看起来很厉害的样子
    printf("%d\n", (i->__forwarding->i));
}

```

main函数也有了变动：



```
int main(int argc, const char * argv[])
{
    __block __Block_byref_i_0 i = {(void*)0,(__Block_byref_i_0 *)&i, 0, sizeof(__Block_byref_i_0), 1024};
    void (*blk)(void) = (void (*)(void))&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, (struct __Block_byref_i_0 *)&i, 570425344);
    ((void (*)(struct __block_impl *))((struct __block_impl *)blk)->FuncPtr)((struct __block_impl *)blk);

    return 0;
}

```

前两行代码创建了两个关键结构体，特地高亮显示。


这里没有看\_\_main\_block\_desc\_0发生的变化，*放到后面讨论*。


使用\_\_block类型指示符的本质就是引入了\_\_Block\_byref\_{$var\_name}\_{$index}结构体，而被\_\_block关键字修饰的变量就被放到这个结构体中。另外，block结构体通过引入\_\_Block\_byref\_{$var\_name}\_{$index}指针类型的成员，得以间接访问到外部变量。


通过这样的设计，我们就可以修改外部作用域的变量了，再一次应了那句话：



> There is no problem in computer science that can’t be solved by adding another level of indirection.
> 
> 


指针是我们最经常使用的间接手段，而这里的本质也是通过指针来间接访问，为什么要特地引入\_\_Block\_byref\_{$var\_name}\_{$index}结构体，而不是直接使用int \*来访问外部变量i呢？


另外，\_\_Block\_byref\_{$var\_name}\_{$index}结构体中的\_\_forwarding指针成员有何作用？


请继续往下看 ：）


#### 5. 背后的内存管理动作


在Objective-C中，block特性的引入是*为了让程序员可以更简洁优雅地编写并发代码*（配合看起来像敏感词的GCD）。比较常见的就是将block作为函数参数传递，以供后续回调执行。


先看一段完整的、可执行的代码：



```
#import <Foundation/Foundation.h>
#include <pthread.h>

typedef void (^DemoBlock)(void);

void test();
void *testBlock(void *blk);

int main(int argc, const char * argv[])
{
    printf("Before test()\n");
    test();
    printf("After test()\n");

    sleep(5);
    return 0;
}

void test()
{
    __block int i = 1024;
    void (^blk)(void) = ^{ i = 2048; printf("%d\n", i); };

    pthread_t thread;
    int ret = pthread_create(&thread, NULL, testBlock, (void *)blk);
    printf("thread returns : %d\n", ret);

    sleep(3); // 这里睡眠1s的话，程序会崩溃
}

void *testBlock(void *blk)
{
    sleep(2);

    printf("testBlock : Begin to exec blk.\n");
    DemoBlock demoBlk = (DemoBlock)blk;
    demoBlk();

    return NULL;
}

```

在这个示例中，位于test()函数的block类型的变量blk就作为函数参数传递给testBlock。


正常情况下，这段代码可以成功运行，输出：



```
Before test()
thread returns : 0
testBlock : Begin to exec blk.
2048
After test()

```

如果按照注释，将test()函数最后一行改为休眠1s的话，正常情况下程序会在输出如下结果后崩溃：



```
Before test()
thread returns : 0
After test()
testBlock : Begin to exec blk.

```

从输出可以看出，当要执行blk的时候，test()已经执行完毕回到main函数中，对应的函数栈也已经展开，此时栈上的变量已经不存在了，继续访问导致崩溃——这也是不用int \*直接访问外部变量i的原因。


##### 5.1 拷贝block结构体


上文提到block结构体\_\_block\_impl的第一个成员是isa指针，使其成为NSObject的子类，所以我们可以通过相应的内存管理机制将其拷贝到堆上：



```
void test()
{
    __block int i = 1024;
    void (^blk)(void) = ^{ i = 2048; printf("%d\n", i); };

    pthread_t thread;
    int ret = pthread_create(&thread, NULL, testBlock, (void *)[blk copy]);
    printf("thread returns : %d\n", ret);

    sleep(1);
}

void *testBlock(void *blk)
{
    sleep(2);

    printf("testBlock : Begin to exec blk.\n");
    DemoBlock demoBlk = (DemoBlock)blk;
    demoBlk();
    [demoBlk release];

    returnNULL;
}

```

再次执行，得到输出：



```
Before test()
thread returns : 0
After test()
testBlock : Begin to exec blk.
2048

```

可以看出，在test()函数栈展开后，demoBlk仍然可以成功执行，这是由于blk对应的block结构体\_\_main\_block\_impl\_0已经在堆上了。不过这还不够——


##### 5.2 拷贝捕获的变量（\_\_block变量）


在拷贝block结构体的同时，还会将捕获的\_\_block变量，即结构体\_\_Block\_byref\_i\_0，复制到堆上。这个任务落在前面没有讨论的\_\_main\_block\_desc\_0结构体身上：



```
static void __main_block_copy_0(struct __main_block_impl_0*dst, struct __main_block_impl_0*src) {_Block_object_assign((void*)&dst->i, (void*)src->i, 8/*BLOCK_FIELD_IS_BYREF*/);}

static void __main_block_dispose_0(struct __main_block_impl_0*src) {_Block_object_dispose((void*)src->i, 8/*BLOCK_FIELD_IS_BYREF*/);}

static struct __main_block_desc_0 {
    unsigned long reserved;
    unsigned long Block_size;
    void (*copy)(struct __main_block_impl_0*, struct __main_block_impl_0*);
    void (*dispose)(struct __main_block_impl_0*);
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0), __main_block_copy_0, __main_block_dispose_0};

```

栈上的\_\_main\_block\_impl\_0结构体为src，堆上的\_\_main\_block\_impl\_0结构体为dst，当发生复制动作时，\_\_main\_block\_copy\_0函数会得到调用，将src的成员变量i，即\_\_Block\_byref\_i\_0结构体，也复制到堆上。


##### 5.3 \_\_forwarding指针的作用


当复制动作完成后，栈上和堆上都存在着\_\_main\_block\_impl\_0结构体。如果栈上、堆上的block结构体都对捕获的外部变量进行操作，会如何？


下面是一段示例代码：



```
void test()
{
    __block int i = 1024;
    void (^blk)(void) = ^{ i++; printf("%d\n", i); };

    pthread_t thread;
    int ret = pthread_create(&thread, NULL, testBlock, (void *)[blk copy]);
    printf("thread returns : %d\n", ret);

    sleep(1);
    blk();
}

void *testBlock(void *blk)
{
    sleep(2);

    printf("testBlock : Begin to exec blk.\n");
    DemoBlock demoBlk = (DemoBlock)blk;
    demoBlk();
    [demoBlk release];

    returnNULL;
}

```

1. 在test()函数中调用pthread\_create创建线程时，blk被复制了一份到堆上作为testBlock函数的参数。
2. test()函数中的blk结构体位于栈中，在休眠1s后被执行，对i进行自增动作。
3. testBlock函数在休眠2s后，执行位于堆上的block结构体，这里为demoBlk。


上述代码执行后输出：



```
Before test()
thread returns : 0
1025
After test()
testBlock : Begin to exec blk.
1026

```

可见无论是栈上的还是堆上的block结构体，修改的都是同一个\_\_block变量。


这就是前面提到的\_\_forwarding指针成员的作用了：


起初，栈上的\_\_block变量的成员指针\_\_forwarding指向\_\_block变量本身，即栈上的\_\_Block\_byref\_i\_0结构体。


当\_\_block变量被复制到堆上后，栈上的\_\_block变量的\_\_forwarding成员会指向堆上的那一份拷贝，从而保持一致。


#### 参考资料：


* <http://msdn.microsoft.com/en-us/library/dd293603.aspx>
* <http://www.cprogramming.com/c++11/c++11-lambda-closures.html>
* <http://developer.apple.com/library/ios/#documentation/cocoa/Conceptual/Blocks/Articles/00_Introduction.html>
* [http://en.wikipedia.org/wiki/Closure\_(computer\_science)](https://en.wikipedia.org/wiki/Closure_(computer_science))



