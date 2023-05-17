# Android将允许纯C-C++开发应用
>date: 2011-01-24T08:39:50+08:00


对于Android，长期以来，我一直有两件事搞不懂，


* 一个是为什么Android要选用Java。对于嵌入式开发，CPU和内存都很宝贵，居然还使用Java。
* 一个是为什么Android的[开发站点](https://developer.android.com)要被墙。这只是一个技术网站啊。


最近，在一个Android[开发人员的Blog](https://android-developers.blogspot.com/2011/01/gingerbread-ndk-awesomeness.html)上证实了在NDK r5使用C/C++进行开发。（以前，Android 对C/C++开发的支持仅限于用C/C++开发动态链接库，然后在Java中以JNI的形式来调用）现在，你可以用纯C/C++开发了（参看下面的程序代码）。还有一段[完整的代码示例在这里](https://developer.android.com/reference/android/app/NativeActivity.html)（墙，还有XML的manifest，[又见XML](https://coolshell.cn/articles/3498.html)）。看来，Google终于明白为什么使用Android的手机（如：Moto, 三星、索爱和HTC）的触摸体验远远不及object C搞出来的iPhone。



```
void android_main(struct android_app* state) {
    // Make sure glue isn't stripped.
    app_dummy();

    // loop waiting for stuff to do.
    while (1) {
        // Read all pending events.
        int ident;
        int events;
        struct android_poll_source* source;

        // Read events and draw a frame of animation.
        if ((ident = ALooper_pollAll(0, NULL, &events,
                (void**)&source)) >= 0) {
            // Process this event.
            if (source != NULL) {
                source->process(state, source);
            }
        }
        // draw a frame of animation
        bringTheAwesome();
    }
}
```

我个人估计有两个原因为什么Google回头支持C/C++了，


1. Google开始觉得自己整的JVM在性能上可以全面超越传统JVM，并接近C/C++，现在发现搞不定了。
2. Google发现Java的程序员不像C/C++程序员那样注重程序的性能和效率，开发App太耗CPU和内存。


于是只好转回支持C/C++。**本来就是用C/C++写出来的Android嘛，居然不能用C/C++而只能用Java，真是太侮辱C/C++了**。最后，只希望Google并不是又整了一个C/C++版的Dalvik虚拟机，不然就真是侮辱到极点了。


*——— 更新 2011/01/24 ————*


谢谢大家对这篇文章的评论，挺有意思的，欢迎讨论，我把我的回复更新在下面。不一定对，仅供大家参考。



Java的学习成本低，开放性好，兼容性也高，我不否认（但请大家也别否认C/C++的效率要比Java要高。而C/C++的程序员在普遍上要比Java程序员更注意性能和效率）。这应该是Andorid的一开始的定位，可见，Google关注的是程序员，而不是用户。现在转回支持C/C++必然有他的原因，如果不是性能上的原因。那么就请大家分析一下别的原因。


Android本来就是用C/C++写的，要跨平台，首先是Android自己跨平台。就像Linux一样，跨平台的首先是Linux，应用开发人员只需要符合Linux的API就OK了。JVM带来的便利只是无需重新编译（就算是无需重新编译，对于开发人员来说也要去那个平台做测试的，因为不同的平台的JVM同样是不一样的）。在Native平台上编译的成本其实并不高，这个编译过程完全可以在部署的时候自动化。


有人说，Java的开发成本比C/C++低，但这和语言没有关系，这其实和封装程度有关系。C/C++同样可以封装得很好。而且，C/C++的程序员比JAVA程序来说，天生就对内存和性能要敏感的多。这更有利于在手机这样资源不足的平台上做开发。


尤其对于像手机这样的时尚终端来说，在用户体验上花的成本要比在开发人员上花成本要大得多的多。我以为，Google 的Android 更多的关注了程序员，而不是用户。而iPhone更多的关注了用户，也让程序员在开发过程上受到了一些牺牲（iPhone的做法是如果程序员的程序要上App Store，先交99美刀的代码审查费，就像申请美国签证一样），但是，iPhone的程序员虽然在开发的方便上有一些牺牲，但是从收入上却得到了保障。最新的消息是苹果已向开发者支付20亿美元 音乐供应商分成达120亿美元。在《[偷了世界的程序员](https://coolshell.cn/articles/3363.html)》中对此有充分的论述。


最后，请大家思考 几个问题——


* Android支持C/C++是为什么？如果是为了程序效率，那么这又是为什么？
* 是开发人员更重要，还是用户更重要？（注意：我说的是“更重要”）
* 在当今这种诸如iPhone或Andorid的开发模式下，是完全开放好，还是有适当的封闭好？
* 开发和封闭的背后的商业驱动是什么？如何在开放和封闭中权衡用户、开发者、公司和版权商的利益？


苹果公司给出了一个很不错的商业模式。


（完）


