# 如何知道某网站运行在GAE上
>date: 2009-05-06T16:33:52+08:00


GAE就是[Google Application Engine](https://code.google.com/appengine/)，通过Alexa的网站排名系统，我们可以知道Top 10的使用GAE的网站，他们是：（截止至今天）


- 1. [www.robtex.com](https://www.robtex.com/) (Alexa rank: 1691)

- 2. [www.twibes.com](http://www.twibes.com/) (Alexa rank: 13143)

- 3. [www.acid-play.com](http://www.acid-play.com/) (Alexa rank: 25884)

- 4. [www.jaiku.com](http://www.jaiku.com/) (Alexa rank: 29061)

- 5. [www.wordle.net](http://www.wordle.net/) (Alexa rank: 34022)

- 6. [www.twazzup.com](http://www.twazzup.com/) (Alexa rank: 40910)

- 7. [www.twollo.com](http://www.twollo.com/) (Alexa rank: 41414)

- 8. [www.downforeveryoneorjustme.com](http://www.downforeveryoneorjustme.com/) (Alexa rank: 41718)

- 9. [www.chromeexperiments.com](http://www.chromeexperiments.com/) (Alexa rank: 49899)

- 10. [www.desktop-reporting.com](http://www.desktop-reporting.com/) (Alexa rank: 51447)

那么，我们如何才能知道一个网站是运行在GAE上的呢？



如果一个网站运行在GAE上，那么其会有如下三个事情会为真：


* 网站的 别名记录（CNAME）会 指向ghs.google.com， ghs.l.google.com 或者appspot.l.google.com 。
* 访问网站的/form 路径会返回Google风格的404 错误页。
* 网站的”Server”标题会是 “Google Frontend”


测试这三个条件并不难，在Linux下，我们可以用这样的命令行检查：


`**有google.com字样的CNAME**  

  dig www.example.com cname | egrep -i 'cname.*google.com'`


`**Google 404 错误for /form:**  

  curl -s -D - http://www.example.com/form | egrep 'G.*o.*o.*g.*l.*e'`


`**"Google Frontend" 字符串**``curl -s -D - http://www.example.com/ | egrep '^Server:'`


请注意，头两个条件在一些时候对于运行在Blogspot 的主机也是成立的，估计Blogspot就是运行在GAE上的一个站点。但第三个条件就不一样了。GAE上的是”Google Frontend”，而 Blogspot上的则是 “GFE/2.0″。


