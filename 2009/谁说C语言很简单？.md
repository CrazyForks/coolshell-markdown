# 谁说C语言很简单？
>date: 2009-05-19T22:10:25+08:00



前两天，Neo写了一篇《[语言的歧义](/2009/%E8%AF%AD%E8%A8%80%E7%9A%84%E6%AD%A7%E4%B9%89.md)》其使用C语言讨论了一些语言的歧义。大家应该也顺便了解了一下C语言中的很多不可思异的东西，可能也是你从未注意到的东西。


是的，C语言并不简单，让我们来看看下面这些示例：


1. **为什么下面的代码会返回0？(这题应该很简单吧)**

```
 int x;
 return x == (1 && x);

```

本题主要是关于C/C++中变量初始化的问题。
3. **为什么下面的代码会返回0而不是-1？**

```
 return ((1 - sizeof(int)) >> 32);

```

答案：sizeof 是一个unsigned的类型，所以……
4. **代码作用域是一件很诡异的事，下面这个函数返回值是什么？**

```
int x = 5;
int f() {
 int x = 3;
 {
 extern int x;
 return x;
 }
}

```

答案：5
5. **函数和函数指针可以相互转换。下面的语句哪些是合法的？**

```
int (\*pf)(void);
int f(void)
{

 pf = &f; // 没问题
 pf = \*\*\*f; // 取址？
 pf(); // 函数指针可以调用？
 (\*\*\*\*pf)(); // 这又是什么？
 (\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*f)(); // 这个够变态了吧？
}

```

答案：全部合法。
6. **初始化可能是ISO C中最难的部分了。无论是MSVC 还是GCC 都没有完全实现。GCC 可能更接近标准。在下面的代码中，i.nested.y 和i.nested.z的最终值是什么？**

```
struct {
 int x;
 struct {
 int y, z;
 } nested;
} i = { .nested.y = 5, 6, .x = 1, 2 };

```

答案：2和6
7. **下面这个示例是C语言的痛，main函数返回值是什么？**

```
typedef struct
{
 char \*key;
 char \*value;
} T1;

typedef struct
{
 long type;
 char \*value;
} T3;

T1 a[] =
{
 {
 "",
 ((char \*)&((T3) {1, (char \*) 1}))
 }
};
int main() {
 T3 \*pt3 = (T3\*)a[0].value;
 return pt3->value;
}

```

答案：1（你知道为什么吗？）
8. **下面这个例就更变态了。在GCC的文档中，这个语法是合法的，但是不知道为什么GCC并没有实现。下面的代码返回 2.**

```
 return ((int []){1,2,3,4})[1];

```
9. **在下面的这个示例中，有一个“bar” 函数及其函数指针 “pbar” 的两个拷贝(static 类型一般作用于语句块或文件域).**

```
 int foo() {
 static bar();
 static (\*pbar)() = bar;

 }

 static bar() {
 return 1;
 }

 static (\*pbar)() = 0;

```
10. **下面的这个函数返回值是什么？取决于编译器是先处理unsigned long转型，还是负号。**

```
 unsigned long foo() {
 return (unsigned long) - 1 / 8;
 }

```

如果是： ((unsigned long) - 1) / 8，那将是一个很大的数。  

如果是： (unsigned long) (- 1 / 8 ), 那将是 0


是的，这样使用C语言可能很奇怪，不过我们可以从另一方面了解C语言的很多我们不常注意的特性。C语言其实并不容易。



