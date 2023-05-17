# 一次Ajax查错的经历
>date: 2012-08-27T14:56:59+08:00


先说故事，再说想法吧。


我有一朋友做网站，用jQuery的Ajax方法从后端载入一段HTML代码然后动态插入到网页的Div元件中。这个东西太普遍了。jQuery强大的load方法可以完成这个事情。朋友的代码是这么写的：


[javascript]var tab = jQuery("#dynamic\_tab");  

var url = "/list\_ajax/";  

tab.load(url);[/javascript]


简单到不能再简单了。在Chrome，Firefox，Safari下运行一点问题也没有，只有IE不行，不管是IE7，IE8，还是IE9。问题的症壮是，使用IE访问那个Ajax的链接，没有问题，但是在jQuery的Ajax方法返回了“undefined”的respons对象。没有任何报错！


怎么搞也搞不定，只好Google了一下——“[jQuery load IE](https://www.google.com/#hl=zh-CN&newwindow=1&site=&source=hp&q=jQuery+load+IE&btnK=Google+%E6%90%9C%E7%B4%A2&oq=jQuery+load+IE)”，一看，很多人都在问这个问题。于是开始了[散弹枪编程方式](https://coolshell.cn/articles/2058.html "各种流行的编程风格")。


排在第一的就是StackOverflow被浏览了33K次的这个问题：[jQuery’s .load() not working in IE – but fine in Firefox, Chrome and Safari](http://stackoverflow.com/questions/1061525/jquerys-load-not-working-in-ie-but-fine-in-firefox-chrome-and-safari)，答案没有被打勾（不靠谱），StackOverflow还有很多人问相似的问题，不过都没有答案。不管三七二十一，先试了一下，散弹枪嘛。试了半天都没有用。


然后上Google查，又看到有人说的IE缓存的问题，什么，要把cache设置成false，或是用下面的方法来解决：


[javascript]var tab = jQuery("#dynamic\_tab");  

var fuckie = Math.random();  

var url = "/list\_ajax/"+"?fuckie="+fuckie;  

tab.load(url);[/javascript]


反正还是一样，统统不Work，几乎所有的都试了，都不Work。搞了一天的朋友恼怒道：“Microsoft应该快点倒闭吧，产品太烂了”。IE的确是太烂了。



于是我用IE9的网页调试器可以看到点了Ajax的链接后，**IE对网站有http的Ajax请求，也可以看到请求返回了，但是就是不显示在我的页面上——jQuery的Ajax的responseText为undefined!**


对于我这个老家伙，对jQuery也不熟，我只得开始调试jQuery的代码，想看看里面干了什么，报了什么错？调了一个小时，基本上把jQuery的Ajax的封装看懂了七七八八了，但是还是没找到为什么有问题。


于是，我只得架起原生态的Ajax，看看IE的那个Ajax的ActiveX的对象干了什么事？写了下面的代码（当年写Ajax就是这么写的，所以也不费劲，况且网上还有例程可以抄）：


[javascript]  

function InitAjax()  

{  

var ajax=false;  

try {  

ajax = new ActiveXObject("Msxml2.XMLHTTP");  

} catch (e) {  

try {  

ajax = new ActiveXObject("Microsoft.XMLHTTP");  

} catch (E) {  

ajax = false;  

}  

}  

if (!ajax && typeof XMLHttpRequest!=’undefined’) {  

ajax = new XMLHttpRequest();  

}  

return ajax;  

}


var ajax = InitAjax();  

ajax.open("GET", url, true);  

ajax.onreadystatechange = function() {  

if (ajax.readyState == 4 && ajax.status == 200) {  

var show = document.getElementById("HaoChenDIV").value;  

show.innerHTML = ajax.responseText;  

}  

}  

ajax.send(null);  

[/javascript]


一运行，还是不行，没见IE报什么错，不过，可以确定这不是jQuery的问题了，估计还是我们自己程序的问题。不过此时的程序太好调试了，调试中，在IE9下调式发现原生的IE的Ajax对象在onreadystatechange函数里，其responseText是下面这个样子：


![](/assets/images/ajax_error.jpg "ajax error in ie")


什么是“**系统错误: -1072896658**”？上[google一查](https://www.google.com/#hl=zh-CN&newwindow=1&q=ajax+%22%E7%B3%BB%E7%BB%9F%E9%94%99%E8%AF%AF:+-1072896658%22&oq=ajax+%22%E7%B3%BB%E7%BB%9F%E9%94%99%E8%AF%AF:+-1072896658%22)，一堆页面，基本上是说乱码了，也就是ajax的后端程序返回的网页编码不认识吧。需要在返回的http header里加上 charset=utf-8。


于是，修改后端的Ajax的程序，明确指定了返回的HTTP Header中的charset，于是IE下就工作正常了，再切回jQuery的load代码，一切正常了（后端的程序本来是utf-8的编码格式，但是不骨明确在HTTP Header中指定，但是只有IE不会自动检测）。


这个问题的原因就是因为我们没有按照规范去写网页。所以，举一反三，HTML的规范还有哪些，太多了，记也记不住。但也许你会知道**有一个叫 <http://validator.w3.org> 的网站可以帮你校验你网页中的很多不规范的东西**。这个工具会报很多很多错，很多都有点吹毛求疵，不过，可以让你看看（注：今天的coolshell装了很多插件，也被我调过一些东西，所以出错很多，我还记得以前没有插件没有我定制化的样式的时候，Wordpress一个错都不报）。


#### 后记


我把这个问题和过程分享出来，主要有这么几个目的，并抛出几个问题，大家可以思考一下：


1）这个问题网上有很多人都在报，但是基本上找不到答案（包括StackOverflow），所以，我分享出来，填补一下空白。


2）我相信我们的程序员天天都在经历这样的事，我不知道大家在遇到这样的事情会怎么做？也许大多数人都在网上查各种解决方案，然后一个一个的试，直到试对了——散弹枪式的编程，呵呵。当然，大多数答案都是可能找到的。但**当我们找到答案了后，我们还会深入去了解这个问题的具体原因并举一反三地去思考一其周边的东西吗**？


3）另外，在今天这样N多框架，N多lib，N多开源的年代下，**不知道大家有没有失去了从零开始自己写代码的能力？**比如上面的这个问题，不知道有多少人还会自己写原生态的Ajax？不过，我还是建议大家能在使用各种框架的时候，明白那些最基础的知识，求甚解，知其然知其所以然，真的很重要。


我是从那个“吃糠的年代”过来的程序员，那时的程序员什么都要自己干，很辛苦，今天我和很多人说我以前的那些经历，会被笑话，但是我从这些什么都自己的干的年代过的经历，让我受益很多。我把我的想法分享给大家，希望对大家有用。


(全文完)


