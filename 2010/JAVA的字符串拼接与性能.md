# JAVA的字符串拼接与性能
>date: 2010-03-28T09:42:14+08:00


**概述：**本文主要研究的是JAVA的字符串拼接的性能，原文中的测试代码在功能上并不等价，导致concat的测试意义不大。不过原作者在评论栏给了新的concat结果，如果有兴趣的同学建议自己修改代码测试。


原文出处:<http://www.venishjoe.net/2009/11/java-string-concatenation-and.html>


在JAVA中拼接两个字符串的最简便的方式就是使用操作符”+”了。如果你用”+”来连接固定长度的字符串，可能性能上会稍受影响，但是如果你是在循环中来”+”多个串的话，性能将指数倍的下降。假设有一个字符串，我们将对这个字符串做大量循环拼接操作，使用”+”的话将得到最低的性能。但是究竟这个性能有多差？如果我们同时也把StringBuffer,StringBuilder或String.concat()放入性能测试中，结果又会如何呢？本文将会就这些问题给出一个答案！  




我们将使用[Per4j](http://perf4j.codehaus.org/index.html)来计算性能，因为这个工具可以给我们一个完整的性能指标集合，比如最小，最大耗时，统计时间段的标准偏差等。在测试代码中，为了得到一个准确的标准偏差值，我们将执行20个拼接”\*”50,000次的测试。下面是我们将使用到的拼接字符串的方法：


* Concatenation Operator (+)
* String concat method – concat(String str)
* StringBuffer append method – append(String str)
* StringBuilder append method – append(String str)


最后，我们将看看字节码，来研究这些方法到底是如何执行的。现在，让我们先开始来创建我扪的类。注意为了计算每个循环的性能，代码中的每段测试代码都需要用Per4J库进行封装。首先我们先定义迭代次数



```
private static  final int  OUTER_ITERATION=20;
private static final int INNER_ITERATION=50000;

```

接下来，我们将使用上述4个方法来实现我们的测试代码。



```
  	String addTestStr = "";
  	String concatTestStr = "";
  	StringBuffer concatTestSb = null;
  	StringBuilder concatTestSbu = null;
  	 
  	for (int outerIndex=0;outerIndex<=OUTER_ITERATION;outerIndex++) {
  	    StopWatch stopWatch = new LoggingStopWatch("StringAddConcat");
  	    addTestStr = "";
  	    for (int innerIndex=0;innerIndex<=INNER_ITERATION;innerIndex++)
  	    addTestStr += "*";
  	    stopWatch.stop();
  	}       
  	 
  	for (int outerIndex=0;outerIndex<=OUTER_ITERATION;outerIndex++) {
  	    StopWatch stopWatch = new LoggingStopWatch("StringConcat");
  	    concatTestStr = "";
  	    for (int innerIndex=0;innerIndex<=INNER_ITERATION;innerIndex++)
  	    concatTestStr.concat("*");
  	    stopWatch.stop();
  	}
  	 
  	for (int outerIndex=0;outerIndex<=OUTER_ITERATION;outerIndex++) {
  	    StopWatch stopWatch = new LoggingStopWatch("StringBufferConcat");
  	    concatTestSb = new StringBuffer();
  	    for (int innerIndex=0;innerIndex<=INNER_ITERATION;innerIndex++)
  	    concatTestSb.append("*");
  	    stopWatch.stop();
  	}
  	 
  	for (int outerIndex=0;outerIndex<=OUTER_ITERATION;outerIndex++) {
  	    StopWatch stopWatch = new LoggingStopWatch("StringBuilderConcat");
  	    concatTestSbu = new StringBuilder();
  	    for (int innerIndex=0;innerIndex<=INNER_ITERATION;innerIndex++)
  	    concatTestSbu.append("*");
  	    stopWatch.stop();
  	}

```

接下来通过运行程序来生成性能指标。我的运行环境是64位的Windown7操作系统，32位的JVM(7-ea) 带4GB内存，双核Quad 2.00GHz的CPU的机器.


经过20次迭代后，我们得到如下的数据：  

![](/assets/images/coolshell.cn/wp-content/uploads/2010/03/String_Perf_Chart_217.png "结果")


结果非常完美如我们想象的那样。唯一比较有趣的事情是为什么String.concat也很不错，我们都知道，String是一个常类（初始化后就不会改变的类），那么为什么concat的性能会更好一些呢。(**译者注**：其实原文作者的测试代码有问题，对于concat()方法的测试代码应该写成concatTestStr=concatTestStr.concat(“\*”)才对。)为了回答这个问题，我们应该看看concat反编译出来的字节码。在本文的下载包里面包含了所有的字节码，但是现在我们先看一下concat的这个代码片段：



```
    46:  new #6; //class java/lang/StringBuilder
    49:  dup
    50:  invokespecial   #7; //Method java/lang/StringBuilder."<init>":()V
    53:  aload_1
    54:  invokevirtual   #8; //Method java/lang/StringBuilder.append:
             (Ljava/lang/String;)Ljava/lang/StringBuilder;
    57:  ldc #9; //String *
    59:  invokevirtual   #8; //Method java/lang/StringBuilder.append:
             (Ljava/lang/String;)Ljava/lang/StringBuilder;
    62:  invokevirtual   #10; //Method java/lang/StringBuilder.toString:()
             Ljava/lang/String;
    65:  astore_1
    66:  iinc    7, 1
    69:  goto    38

```

这段代码是String.concat()的字节码，从这段代码中，我们可以清楚的看到，concat()方法使用了StringBuilder，concat()的性能应该和StringBuilder的一样好，但是由于额外的创建StringBuilder和做.append(str).append(str).toString()的操作，使得concate的性能会受到一些影响，所以StringBuilder和String Cancate的时间是1.8和3.3。


因此，即时在做最简单的拼接时，如果我们不想创建StringBuffer或StringBuilder实例使，我们也因该使用concat。但是对于大量的字符串拼接操作，我们就不应该使用concat(**译者注：**因为测试代码功能上并不完全等价，更换后的测试代码concat的平均处理时间是1650.9毫秒。这个结果在原文的评论里面。)，因为concat会降低你程序的性能，消耗你的cpu。因此，在不考虑线程安全和同步的情况下，为了获得最高的性能，我们应尽量使用StringBuilder


本文的源代码，编译目标文件和字节码可以通过下面的这个链接获得：


下载源代码，类和字节码：String\_Concatenation \_Performance.zip


