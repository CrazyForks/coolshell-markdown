# 9个强大免费的PHP库
>date: 2009-04-12T12:29:53+08:00



### 1. ReCAPTCHA


[reCAPTCHA](http://recaptcha.net/plugins/php/) 允许你的网站集成一个Advanced CAPTCHA 系统，这个系统可以帮助你阻止一些垃圾信息。可视化的CAPTCHA 同样也有一个有用的声音功能。另外，在reCAPTCHA 服务里，这个PHP库也包含了一个给 “Mailhide” 服务用的API，这个可以把你的邮件地址隐藏于一些抓邮件地址的程序。


这个API是免费并且非常容易使用的，你需要做的就是申请一个API的KEY。


![ReCAPTCHA](http://nettuts.s3.amazonaws.com/267_libraries/libs/recaptcha.png)
[下载 ReCAPTCHA](https://code.google.com/p/recaptcha/downloads/list?q=label:phplib-Latest) | [获取一个API Key](http://recaptcha.net/api/getkey?app=php) | [相关文档](http://recaptcha.net/plugins/php/)



### 2. Akismet


[Akismet](http://akismet.com/) 是一个免费的服务项目，对于一些小型的网站它是完全免费的，对于一些大型的网址，他是部分免费的。这个库也是提供了处理一些和垃圾信息相关的功能。它主要通过比对自己数据库中已存在的被认定为垃圾的信息，而做出决定的。当然，数据库中的垃圾信息可能通过各个网站举报，大家供享的。这是一个每天都在更新，每天都在改进的库。许多许多的WordPress都装有这个库。


![Akismet](http://nettuts.s3.amazonaws.com/267_libraries/libs/akismet.jpg)
[实施Akismet](http://net.tutsplus.com/tutorials/tools-and-tips/the-best-ways-to-fight-spam/)


### 3. Services\_JSON


JSON 是一个非常小巧敏捷的PHP库，它主要用于把一些数据格式转成易于人们阅读的格式。并不是所有的人都会喜欢PHP5 （因为自PHP5.20后其中已经集成了JSON），所以，这个小PHP库可以在低版本的PHP中让你得到 JSON 的功能。


![JSON](http://nettuts.s3.amazonaws.com/267_libraries/libs/json.png)
[查看 Services\_JSON](http://pear.php.net/package/Services_JSON)


### 4. Smarty


[Smarty](http://smarty.net/) 是一个网面模板引擎，它主要是把程序和界面分开。Smarty 提供了许多强大的功能，比如循环，变量，以及一个强大的缓存系统。这个库不是一个新库了，其已经发展了很多年了，虽然只有3个release版，但应该是比较成熟了。


![Smarty](http://nettuts.s3.amazonaws.com/267_libraries/libs/smarty.png)
[下载 Smarty](http://smarty.net/download.php) | [查看文档](http://smarty.net/docs.php)


### 5. pChart


这是一个强大的画统计图的PHP库，像一些饼图或是柱状图，[pChart](http://pchart.sourceforge.net/index.php) 还允许你通过SQL查询语句或是手动的输入数据来创建一个统计图。当然它需要GD库的支持以便创建图片。这个库一看就是有很多非常专业的美工设计过，因为它可以让你的统计图显示的相当漂亮。


![pChart](http://nettuts.s3.amazonaws.com/267_libraries/libs/pchart.png)
[下载 pChart](http://pchart.sourceforge.net/download.php) | [相关文档](http://pchart.sourceforge.net/documentation.php) | [查看演示](http://pchart.sourceforge.net/demo.php)


### 6. SimplePie


[SimplePie](http://simplepie.org/)  允许你可以容易地 pull 一些信息，比如RSS feeds。它同样可以被集成于不同的平台和语言。并且可以通过很多不同的方法来处理远端的feed。


![SimplePie](http://nettuts.s3.amazonaws.com/267_libraries/libs/simplepie.png)


[下载 SimplePie](http://simplepie.org/downloads/) | [相关文档](http://simplepie.org/wiki/) | [Extending SimplePie to Parse Unique RSS Feeds](http://net.tutsplus.com/videos/screencasts/extending-simplepie-to-parse-unique-rss-feeds/)


### 7. XML-RPC PHP


我们的应用程序有时需要一些类似于 “ping” 的功能去探测一下其它站点，如BLOG的 trackbacks。一般来说，这都是通过一个叫做XML-RPC的协议来完成的。[XML-RPC PHP](http://phpxmlrpc.sourceforge.net/) 库可以让你的站点集成这些功能。


![XML-RPC](http://nettuts.s3.amazonaws.com/267_libraries/libs/xmlrpc.png)


[下载 XML-RPC PHP](http://phpxmlrpc.sourceforge.net/#download) | [相关文档](http://phpxmlrpc.sourceforge.net/#interest)


### 8. Amazon S3


Amazon 提供了一个“云服务”叫”S3″. 这个PHP库可以让你不需要第三方的插件就可以上传大的文件。


![Amazon S3](http://nettuts.s3.amazonaws.com/267_libraries/libs/s3.png)
[下载 Amazon S3 PHP 类](https://amazon-s3-php-class.googlecode.com/files/s3-php5-curl_0.3.9.tar.gz)


### 9. PHPMailer


很多应用都需要对外发送邮件，但是PHP的mail() 函数并不是特别好用。于是 PHPMailer 应运而生，这是一个功能强大的类，其允许你发送不同格式的邮件，并支持附件和自定义邮件头。


![Sending Mail](http://nettuts.s3.amazonaws.com/267_libraries/libs/mail.png)
[下载 PHPMailer](http://phpmailer.codeworxtech.com/index.php?pg=sf&p=dl) | [相关文档](http://phpmailer.codeworxtech.com/index.php?pg=tutorial)


文章：[来源](http://net.tutsplus.com/articles/web-roundups/9-extremely-useful-and-free-php-libraries/)


