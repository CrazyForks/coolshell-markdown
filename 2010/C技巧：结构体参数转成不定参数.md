# C技巧：结构体参数转成不定参数
>date: 2010-08-10T08:24:52+08:00


下面这段程序是一个C语言的小技巧，其展示了如何把一个参数为结构体的函数转成一个可变参数的函数，其中用到了宏和内建宏“`__VA_ARGS__`”，下面这段程序可以在GCC下正常编译通过：



```
#include <stdio.h>

#define func(...) myfunc((struct mystru){__VA_ARGS__})

struct mystru { const char *name; int number; };

void myfunc(struct mystru ms )
{
  printf("%s: %d\n", ms.name ?: "untitled", ms.number);
}

int main(int argc, char **argv)
{
  func("three", 3);
  func("hello");
  func(.name = "zero");
  func(.number = argc, .name = "argc",);
  func(.number = 42);
  return 0;
}

```

从上面这段程序，我们可以看到一个叫 myfunc的函数，被func的宏改变了，本来myfunc需要的是一个叫mystru的结构，然而通过宏，我们把struct mystru的这个参数，变成了不定参数列表的一个函数。上面这段程序输出入下，




> three: 3  
> 
> hello: 0  
> 
> zero: 0  
> 
> argc: 1  
> 
> untitled: 42
> 
> 


虽然，这样的用法并不好，但是你可以从另外一个方面了解一下这世上对C稀奇古怪的用法。 如果你把宏展开后，你就明的为什么了。下面是宏展开的样子：



```
  myfunc((struct mystru){"three", 3});
  myfunc((struct mystru){"hello"});
  myfunc((struct mystru){.name = "zero"});
  myfunc((struct mystru){.number = argc, .name = "argc",});
  myfunc((struct mystru){.number = 42});

```


