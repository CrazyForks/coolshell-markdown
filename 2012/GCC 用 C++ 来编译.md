# GCC 用 C++ 来编译
>date: 2012-08-20T08:40:04+08:00


GCC在2012年8月15日的时候，merge了一个patch – [Merge from cxx-conversion branch](http://gcc.gnu.org/git/?p=gcc.git;a=commitdiff;h=2b15d2ba7eb3a25dfb15a7300f4ee7a141ee8539)，这意味着，以后在GCC的编译只能用C++的编译器了，也意味着，gcc的实现代码开始转向C++了。


你可能会有两个问题，


* 一个问题是为什么GCC要转成C++的实现？


* 没有C++的编译器，我怎么编译C++编译器的代码？这不是“鸡生蛋还是蛋生鸡”的问题么？


那，我们来看一看吧。


#### 为什么要用C++


在[GNU的C++ Conversion文档](http://gcc.gnu.org/wiki/cxx-conversion)中，我们可以在Background中看到这样的描述：



> Whether we use C or C++, we need to try to ensure that interfaces are easy to understand, that the code is reasonably modular, that the internal documentation corresponds to the code, that it is possible for new developers to write new passes and to fix bugs. Those are the important issues for us to consider. The C++ features which are not present in C — features which are well documented in many books and many web sites — are not an important issue.
> 
> 


这句话的意思可以理解为，今天GCC在用C语言的实现已经有点hold不住了，因为，开发人员觉得，不管我们用C或C++，都需要努力确保接口是容易理解的，这样我们的代码是想当理性地被模块化的，这样内部文档和代码一致，这样可以更好地组织代码，这样有利于新人了fix-bug。而C++正好可以让他们更好的完成这些东西。


GNU还给出了下面这些理由：



* C++ 是一种标准化的，大众的，流行的语言。
* C++ 是C90的超集。
* C++作为C的扩展和C在性能上一样好。
* C++ 在一些有意义的案例上支持更干净的代码。
* C++ 让你更容易去写一个更干净的接口。
* C++ 永远不会让你的代码变得更丑。
* C++ 不是万灵药，他是C的一个改进。


然后，给了一个PDF <http://airs.com/ian/cxx-slides.pdf>，这是Google 的[Ian Lance Taylor](http://airs.com/ian/)的的一个PPT，这个文档可以让大家更好地理解我在《[C++的坑多吗？](https://coolshell.cn/articles/7992.html "C++的坑真的多吗？")》一文中那些观点。**我都不知道我要说多少遍C++的封装，继承和多态比C语言在代码组织上要好得多得多**。大家还是自己看一下代码吧：


**数据结构的操作 ——** 你写的一定不会有STL好


**[![](/assets/images/VEC-vs-vector.jpg "VEC vs vector")](https://coolshell.cn/wp-content/uploads/2012/08/VEC-vs-vector.jpg)**


**结构套结构还是继承？**


[![](/assets/images/tree-structure.jpg "tree-structure")](https://coolshell.cn/wp-content/uploads/2012/08/tree-structure.jpg)


**函数指针还是多态？**


[![](/assets/images/TARGET-vs-Target.jpg "TARGET vs Target")](https://coolshell.cn/wp-content/uploads/2012/08/TARGET-vs-Target.jpg)
**垃圾回收 还是 智能指针？**


[![](/assets/images/GC-vs-Smart-Pointer.jpg "GC vs Smart Pointer")](https://coolshell.cn/wp-content/uploads/2012/08/GC-vs-Smart-Pointer.jpg)
**Why not C++?**


* **C++慢吗**？某些特性会慢，但是有时C++更快，你可以只用你喜欢的C++特性。
* **C++复杂吗？**它只不过是另一种编程语言，他可以让你对程序员维护更简单。
* **FSF不喜欢C++！**因为FSF（自由软件基金会）这些人不写代码。


[![](/assets/images/Why-not-C++.jpg "Why not C++")](https://coolshell.cn/wp-content/uploads/2012/08/Why-not-C++.jpg)



#### Bootstrapping


最后，我想来介绍一下[Bootstrapping](https://en.wikipedia.org/wiki/Bootstrapping_%28compilers%29)。 所谓Bootstrapping，就是用自己这个语言写编译器来编译自己，也就是说如果你要编译gcc，你需要用一个c的编译器来编译之，这个就是bootstrapped process，自举过程。包括 [BASIC](https://en.wikipedia.org/wiki/BASIC "BASIC"), [Algol](https://en.wikipedia.org/wiki/Algol "Algol"), [C](https://en.wikipedia.org/wiki/C_(programming_language) "C (programming language)"), [C++](https://en.wikipedia.org/wiki/C%2B%2B "C++"), [Pascal](https://en.wikipedia.org/wiki/Pascal_programming_language "Pascal programming language"), [PL/I](https://en.wikipedia.org/wiki/PL/I "PL/I"), [Factor](https://en.wikipedia.org/wiki/Factor_programming_language "Factor programming language"), [Haskell](https://en.wikipedia.org/wiki/Haskell_(programming_language) "Haskell (programming language)"), [Modula-2](https://en.wikipedia.org/wiki/Modula-2 "Modula-2"), [Oberon](https://en.wikipedia.org/wiki/Oberon_programming_language "Oberon programming language"), [OCaml](https://en.wikipedia.org/wiki/OCaml "OCaml"),[Common Lisp](https://en.wikipedia.org/wiki/Common_Lisp "Common Lisp"), [Scheme](https://en.wikipedia.org/wiki/Scheme_(programming_language) "Scheme (programming language)"), [Java](https://en.wikipedia.org/wiki/Java_(programming_language) "Java (programming language)"), [Python](https://en.wikipedia.org/wiki/Python_(programming_language) "Python (programming language)"), [Scala](https://en.wikipedia.org/wiki/Scala_(programming_language) "Scala (programming language)") 等语言都这么干。


这样干的好处主要是，自己可以测试自己，编译器的改善和语言的改善相辅相成。


但是，这是一个“鸡生蛋，还是蛋生鸡”的问题，如果你需要用X语言来写一个X语言编译器的语言，你可以这样干：


* 用Y语言来实现X的语言解释器或编译器。 [Niklaus Wirth](https://en.wikipedia.org/wiki/Niklaus_Wirth "Niklaus Wirth") 说 [Pascal](https://en.wikipedia.org/wiki/Pascal_programming_language "Pascal programming language") 的第一个编译器是由 [Fortran](https://en.wikipedia.org/wiki/Fortran "Fortran") 写的。
* 已存在用Y语言写的X语言的编译器或解释器。[Scheme](https://en.wikipedia.org/wiki/Scheme_(programming_language) "Scheme (programming language)") 就是这么干的。
* 已经有一个编译器来编译一个早期版本的X语言，然后就可以用早期版本的X语言来编译新版本的X语言了。[Java](https://en.wikipedia.org/wiki/Java_(programming_language) "Java (programming language)")，[Haskell](https://en.wikipedia.org/wiki/Haskell_(programming_language) "Haskell (programming language)"), 和最初版的 [Free Pascal](https://en.wikipedia.org/wiki/Free_Pascal "Free Pascal") 就是这么干的。
* X在某平台上的编译器已经存在，可以使用交叉编译技术来编译另一个平台上X语言，C语言就是这么干的。
* 用X语言写一个编译器，然后手动编译之（不需要特别优化），（注：手动编译估计就是手动翻译成机器汇编代码），然后再运行这个手动编译的编译器来编译这个编译器的源码，并优化之。[Donald Knuth](https://en.wikipedia.org/wiki/Donald_Knuth "Donald Knuth") 在他的 [WEB](https://en.wikipedia.org/wiki/WEB "WEB") [literate programming](https://en.wikipedia.org/wiki/Literate_programming "Literate programming") 系统里用到了这个方法。


（全文完）


