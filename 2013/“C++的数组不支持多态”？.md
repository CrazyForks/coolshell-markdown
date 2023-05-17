# “C++的数组不支持多态”？
>date: 2013-04-29T16:17:40+08:00


先是在微博上看到了个[微博](http://weibo.com/1876004965/zueproucp)和云风的评论，然后我回了“楼主对C的内存管理不了解”。


[![](https://coolshell.cn/wp-content/uploads/2013/04/weibo.jpg)](https://coolshell.cn/wp-content/uploads/2013/04/weibo.jpg)


后来引发了很多人的讨论，大量的人又借机来黑C++，比如：



> //[@Baidu-ThursdayWang](http://weibo.com/n/Baidu-ThursdayWang):这不就c++弱爆了的地方吗，需要记忆太多东西
> 
> 
> //[@编程浪子张发财](http://weibo.com/n/%E7%BC%96%E7%A8%8B%E6%B5%AA%E5%AD%90%E5%BC%A0%E5%8F%91%E8%B4%A2):这个跟C关系真不大。不过我得验证一下，感觉真的不应该是这样的。如果基类的析构这种情况不能 调用，就太弱了。
> 
> 
> //[@程序元](http://weibo.com/1401324585)：现在看来，当初由于毅力不够而没有深入纠缠c++语言特性的各种犄角旮旯的坑爹细枝末节，实是幸事。为现在还沉浸于这些诡异特性并乐此不疲的同志们感到忧伤。
> 
> 


然后，也出现了一些乱七八糟的理解：




> //[@BA5BO](http://weibo.com/n/BA5BO): 数组是基于拷贝的，而多态是基于指针的，派生类赋值给基类数组只是拷贝复制了一个基类新对象，当然不需要派生类析构函数
> 
> 
> //[@编程浪子张发财](http://weibo.com/n/%E7%BC%96%E7%A8%8B%E6%B5%AA%E5%AD%90%E5%BC%A0%E5%8F%91%E8%B4%A2):我突然理解是怎么回事了，这种情况下数组中各元素都是等长结构体，类型必须一致，的确没法多态。这跟C#和java不同。后两者对于引用类型存放的是对象指针。
> 
> 


等等，看来我必需要写一篇博客以正视听了。


因为没有看到上下文，我就猜测讨论的可能会是下面这两种情况之一：


1) 一个Base\*[]的指针数组中，存放了一堆派生类的指针，这样，你delete [] pBase; 只是把指针数组给删除了，并没有删除指针所指向的对象。这个是最基础的C的问题。你先得for这个指针数组，把数据里的对象都delete掉，然后再删除数组。很明显，这和C++没有什么关系。


2）第二种可能是：Base \*pBase = new Derived[n] 这样的情况。这种情况下，delete[] pBase 明显不会调用虚析构函数（当然，这并不一定，我后面会说） ，这就是上面云风回的微博。对此，我觉得如果是这个样子，这个程序员**完全没有搞懂C语言中的指针和数组是怎么一回事**，也没有搞清楚， 什么是对象，什么是对象的指针和引用，这完全就是C语言没有学好。


后来，在看到了 [@GeniusVczh](http://weibo.com/n/GeniusVczh) 的原文 《[如何设计一门语言（一）——什么是坑(a)](http://www.cppblog.com/vczh/archive/2013/04/27/199765.html)》最后时，才知道了说的是第二种情况。也就是下面的这个示例（我加了虚的析构函数这样方便编译）：



```
class Base
{
  public:
    virtual ~B(){ cout <<"B::~B()"<<endl; }
};

class Derived : public Base
{
  public:
    virtual ~D() { cout <<"D::D~()"<<endl; }
};

Base* pBase = new Derived[10];
delete[] pBase;
```

#### C语言补课


我先不说这段C++的程序在什么情况下能正确调用派生类的析构函数，我还是先来说说C语言，这样我在后面说这段代码时你就明白了。


对于上面的：


`Base* pBase = new Derived[10];`


这个语言和下面的有什么不同吗？



```
Derived d[10];

Base* pBase = d;
```

一个是堆内存动态分配，一个是栈内存静态分配。只是内存的位置和类型不一样，在语法和使用上没有什么不一样的。（如果你把Base 和 Derived想成struct，把new想成malloc() ，你还觉得这和C++有什么关系吗？）


**那么，你觉得pBase这个指针是指向对象的，是对象的引用，还是指向一个数组的，是数组的引用？**


于是乎，你可以想像一下下面的场景：



```
int *pInt; char* pChar;

pInt = (int*)malloc(10*sizeof(int));

pChar = (char*)pInt;
```

**对上面的pInt和pChar指针来说，pInt[3]和pChar[3]所指向的内容是否一样呢？当然不一样，因为int是4个字节，char是1个字节，步长不一样，所以当然不一样。**


**那么再回到那个把Derived[]数组的指针转成Base类型的指针pBase，那么pBase[3]是否会指向正确的Derrived[3]呢？**


我们来看个纯C语言的例程，下面有两个结构体，就像继承一样，我还别有用心地加了一个void \*vptr，好像虚函数表一样：



```
    struct A {
        void *vptr;
        int i;
    };

    struct B{
        void *vptr;
        int i;
        char c;
        int j;
    }b[2] ={
        {(void*)0x01, 100, 'a', -1},
        {(void*)0x02, 200, 'A', -2}
    };

```

注意：我用的是G++编译的，在64bits平台上编译的，其中的sizeof(void\*)的值是8。


我们看一下栈上内存分配：



```
    struct A *pa1 = (struct A*)(b);

```

用gdb我们可以看到下面的情况：(pa1[1]的成员的值完全乱掉了)



```
(gdb) p b
$7 = {{vptr = 0x1, i = 100, c = 97 'a', j = -1}, {vptr = 0x2, i = 200, c = 65 'A', j = -2}}
(gdb) p pa1[0]
$8 = {vptr = 0x1, i = 100}
(gdb) p pa1[1]
$9 = {vptr = 0x7fffffffffff, i = 2}

```

我们再来看一下堆上的情况：（我们动态了struct B [2]，然后转成struct A \*，然后对其成员操作）



```
    struct A *pa = (struct A*)malloc(2*sizeof(struct B));
    struct B *pb = (struct B*)pa；

    pa[0].vptr = (void*) 0x01;
    pa[1].vptr = (void*) 0x02;

    pa[0].i = 100;
    pa[1].i = 200;

```

用gdb来查看一下变量，我们可以看到下面的情况：（pa没问题，但是pb[1]的内存乱掉了）



```
(gdb) p pa[0]
$1 = {vptr = 0x1, i = 100}
(gdb) p pa[1]
$2 = {vptr = 0x2, i = 200}
(gdb) p pb[0]
$3 = {vptr = 0x1, i = 100, c = 0 '\000', j = 2}
(gdb) p pb[1]
$4 = {vptr = 0xc8, i = 0, c = 0 '\000', j = 0}

```

可见，这完全就是C语言里乱转型造成了内存的混乱，这和C++一点关系都没有。而且，C++的任何一本书都说过，父类对象和子类对象的转型会带来严重的内存问题。


但是，如果在64bits平台下，如果把我们的structB改一下，改成如下（把struct B中的int j给注释掉）：



```
    struct A {
        void *vptr;
        int i;
    };

    struct B{
        void *vptr;
        int i;
        char c;
        //int j; <---注释掉int j
    }b[2] ={
        {(void*)0x01, 100, 'a'},
        {(void*)0x02, 200, 'A'}
    };

```

你就会发现，上面的内存混乱的问题都没有了，因为struct A和struct B的size是一样的：



```
(gdb) p sizeof(struct A)
$6 = 16
(gdb) p sizeof(struct B)
$7 = 16
```

注：如果不注释int j，那么sizeof(struct B)的值是24。


这就是C语言中的内存对齐，内存对齐的原因就是为了更快的存取内存（详见《[深入理解C语言](https://coolshell.cn/articles/5761.html "深入理解C语言")》）


如果内存对齐了，而且struct A中的成员的顺序在struct B中是一样的而且在最前面话，那么就没有问题。


#### 再来看C++的程序


如果你看过我5年前写的《**[C++虚函数表解析](http://blog.csdn.net/haoel/article/details/1948051)**》以及《**C++内存对象布局 [上篇](http://blog.csdn.net/haoel/article/details/3081328)、[下篇](http://blog.csdn.net/haoel/article/details/3081385)**》，你就知道C++的标准会把虚函数表的指针放在类实例的最前面，你也就知道为什么我别有用心地在struct A和struct B前加了一个 void \*vptr。C++之所以要加在最前面就是为了转型后，不会找不到虚表了。


好了，到这里，我们再来看C++，看下面的代码：



```
#include
using namespace std;

class B
{
  int b;
  public:
    virtual ~B(){ cout <<"B::~B()"<<endl; }
};

class D: public B
{
  int i;
  public:
    virtual ~D() { cout <<"D::~D()"<<endl; }
};

int main(void)
{
    cout << "sizeB:" << sizeof(B) << " sizeD:"<< sizeof(D) <<endl;
    B *pb = new D[2];

    delete [] pb;

    return 0;
}

```

**上面的代码可以正确执行，包括调用子类的虚函数！因为内存对齐了**。在我的64bits的CentOS上——sizeof(B):16 ，sizeof(D):16


**但是，如果你在class D中再加一个int成员的问题，这个程序就Segmentation fault了**。因为—— sizeof(B):16 ，sizeof(D):24。pb[1]的虚表找到了一个错误的内存上，内存乱掉了。


再注：我在Visual Studio 2010上做了一下测试，对于 struct 来说，其表现和gcc的是一样的，但对于class的代码来说，其可以“正确调用到虚函数”无论父类和子类有没有一样的size。


然而，在C++的标准中，下面这样的用法是undefined! 你可以看看StackOverflow上的相关问题讨论：《[Why is it undefined behavior to delete[] an array of derived objects via a base pointer?](http://stackoverflow.com/questions/6171814/why-is-it-undefined-behavior-to-delete-an-array-of-derived-objects-via-a-base "Why is it undefined behavior to delete[] an array of derived objects via a base pointer?")》（同样，你也可以看看《More Effective C++》中的条款三）



```
Base* pBase = new Derived[10];

delete[] pBase;
```

所以，微软C++编程译器define这个事让我非常不解，对微软的C++编译器再度失望，看似默默地把其编译对了很漂亮，实则误导了好多人把这种undefined的东西当成defined来用，还赞扬做得好，真是令人无语。**（**[就像微博上的这个贴一样](http://weibo.com/2087077260/zup0V7LLM)，说VC多么牛，还说这是OO的特性。我勒个去！**）**


[![](https://coolshell.cn/wp-content/uploads/2013/04/hehe.png)](https://coolshell.cn/wp-content/uploads/2013/04/hehe.png)


现在，你终于知道Base\* pBase = new Derived[10];这个问题是C语言的转型的问题，你也应该知道用于数组的指针是怎么回事了吧？**这是一个很奇葩的代码！请你不要像那些人一样在微博上和这里的评论里高呼并和我理论到：“微软的C++编译器支持这个事！”。**


最后，我越来越发现，**很多说C++难用的人，其实是不懂C语言**。


（全文完）


