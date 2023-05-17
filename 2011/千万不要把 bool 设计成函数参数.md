# 千万不要把 bool 设计成函数参数
>date: 2011-09-08T15:35:18+08:00


我们有很多Coding Style 或 代码规范。但这一条可能会经常被我们所遗忘，就是我们经常会在函数的参数里使用bool参数，这会大大地降低代码的可读性。不信？我们先来看看下面的代码。


当你读到下面的代码，你会觉得这个代码是什么意思？


`widget->repaint(false);`


是不要repaint吗？还是别的什么意思？看了文档后，我们才知道这个参数是immediate， 也就是说，false代表不立即重画，true代码立即重画。


Windows API中也有这样一个函数：InvalidateRect，当你看到下面的代码，你会觉得是什么意思？


`InvalidateRect(hwnd, lpRect,  false);`


我们先不说InvalidateRect这个函数名取得有多糟糕，我们先说一下那个false参数？invalidate意为“让XXX无效”，false是什么意思？双重否定？是肯定的意思？如果你看到这样的代码，你会相当的费解的。于是，你要去看一下文档，或是InvalidateRect的函数定义，你会看到那个参数是 **BOOL** *bErase*，意思是，是否要重画背景。


这样的事情有很多，再看下面的代码，想把str中的”%USER%”替换成真实的用户名：


`str.replace("%USER%", user, false); // Qt 3`


TNND，那个false是什么意思？不替换吗？还是别的什么意思，看了文档才知道，false代码大小写不敏感的替换。


其实，如果你使用枚举变量/常量，而不是bool变量，你会让你的代码更易读，如：




```
widget->repaint(PAINT::immediate);
widget->repaint(PAINT::deffer);

InvalidateRect(hwnd, lpRect,  !RepantBackground);

str.replace("%USER%", user, Qt::CaseInsensitive); // Qt 4
```

如果对这个事不以为然的话，我们再来看一些别的示例，你不妨猜猜看看下面的代码：


`component.setCentered(true, false);`


这什么玩意儿啊？看了文档你才知道，这原来是 setCentered(centered, autoUpdate);


`new Textbox(300, 100, false, true);`


这又是什么啊？看了文档才知道，这是创建一个文本框，第三个参数是是否要滚动条，第四个是是否要自动换行。TNND。


上面的情况还不算最差，看看下面的双重否定。



```
component.setDisabled(false);
filter.setCaseInsensitive(false)
```

再来一个，如果你读到下面的代码，相信你会和我一样，要么石化了，要么凌乱了。



```
event.initKeyEvent("keypress", true, true, null, null,
                    false, false, false, false, 9, 0); 
```

看完这篇文章，我希望你再也不要把bool为作为函数参数了。除非两个原因：


1. 你100%确认不会带来阅读上的问题，比如Java的 setVisible (bool).
2. 你100%确认你想去[写出无法维护很难阅读的代码](https://coolshell.cn/articles/4758.html "如何写出无法维护的代码")。


【更新2011/9/8】当然，别的参数也会有一样的问题，比如：`new Textbox(300, 100, false, true);`中的300 和 100，不知道是坐标还是长宽，只不过，一般长度或坐标这样的参数都不会被hard code，都会有变量名，而bool这种参数经常性地被传成true 和 false。 bool参数表现得更为明显一些罢了。


所以，程序中不要出现magic number，true/false 也是一种 magic number。但是，我想告诉大家，从API设计的角度来说，你无法强制调用者用常量来取代true/false，定义成枚举类型是最好的选择。


最后，如果你想设计一个好的API，强烈推荐你读一下Nokia的Qt的《[API Design Principles](http://qt-project.org/wiki/API-Design-Principles)》，本文就是其中的“[Boolean Trap](http://developer.qt.nokia.com/wiki/API_Design_Principles#e7794937cba47d5e9c54d50a6a32328b)”。


（全文完）


