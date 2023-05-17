# Web程序的最佳测试数据
>date: 2009-12-15T15:50:35+08:00



这里有一篇Matthias写的[关于转义字符文章-“The art of escaping”](https://united-coders.com/matthias-reuter/the-art-of-escaping "The art of escaping")，这篇文章告诉你有一些比较特殊的字符需要你去认真的处理，不然，你的网站程序轻则出错，重则被人黑了。这些物殊的字符是[`<"@%'&_\?/:;,>কী €`] ，你可以使用这个字符串到任意一个可以输入的Web程序上去做测试。


下面这个表格告诉你为什么这些字符很特殊。这个列表不会是完整的，而且也永远不会完整。  






| 相关领域 | 转义字符 |
| --- | --- |
| [HTML](http://www.w3.org/ "W3C") | < , > , & |
| [JSON](http://json.org/ "JSON-Resource") | “ |
| [SQL](http://dev.mysql.com/doc/refman/5.0/en/string-syntax.html "mysql character") in mySql | 字符串 “, ‘, 通配符 %, \_ |
| [rfc 1738](http://www.faqs.org/rfcs/rfc1738.html "rfc 1738 for urls") for URL-parameter | ;, /, ?, :, “, @, =, & 空格 |


  

把这些转义字符放在一起，然后再整些 utf-8 的一些特殊字符。这些utf-8的字符你可以参看本站的[Unicode字符预览表](/2009/Unicode%E5%AD%97%E7%AC%A6%E9%A2%84%E8%A7%88%E8%A1%A8.md)一文，并从中获取。另外，你还可以使用下面的这些工具来对你的程序进行调试或检查：


* 一个高级Web调试插件： [firebug](https://addons.mozilla.org/de/firefox/addon/1843 "firebug plugin")
* 标准的请求/响应插件： [Live HTTP headers](https://addons.mozilla.org/de/firefox/addon/3829)
* 一些抓包程序： [HTTPfox](https://addons.mozilla.org/en-US/firefox/addon/6647) or [tamper data](https://addons.mozilla.org/en-US/firefox/addon/966)
* IE的开发者可以试试这个：[Fiddler.com](http://www.fiddler2.com/fiddler2/)


如果上面的工具都不能帮助你的话，你可能需要打调试日志，或是使用一个透明的代理服务器：如： [Charles Web Debugging Proxy](http://www.charlesproxy.com/) （Windows）


（全文完）


