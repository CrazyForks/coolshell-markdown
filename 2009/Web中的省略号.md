# Web中的省略号
>date: 2009-12-15T08:43:59+08:00


在Web开发中，对于一种情况很常见。那就是，文本太长，而放置文本的容器不够长，而我们又不想让文本换行，所以，我们想使用省略号来解决这个问题。但是，在今天HTML的标准中并没有相关的标识或属性让你可以简单地完成这个事。但是我们可以使用CSS样式表来完成这个事，在IE，Safari，Chrome，Opera中都可以。但在Firefox中却不行，但我们可以使用jQuery来解决Firefox不兼容的问题。下面是相关的代码示例。


**使用CSS设置省略号**



```
overflow: hidden;
text-overflow: ellipsis;
-o-text-overflow: ellipsis;
white-space: nowrap;
width: 100%;

```


* **overflow** 属性是必需的，并且属性要是hidden。
* **white-space: nowrap** 也是必需的。如果文本可以自动换行，就算是不可见，也不会有省略号的。因为文本换行了，所以没有超过容器的尺寸，所以也就不会有省略号了。
* **width**属性仅在需要支持IE6时设置。 设置成100%仅是为了解决IE6的不兼容问题。（关于IE中的那些不兼容问题，你可以参看本站的《[9个最常见IE的Bug及其fix](https://coolshell.cn/articles/1817.html)》）
* **text-overflow: ellipsis** 就是设置省略号了。目前还不是HTML的标准规范。其是由IE引入的，可以在IE6+，Safari 3.2+和Chrome上工作。
* **-o-text-overflow: ellipsis** 是 Opera 支持的。可以在 Opera 9.0+使用。


**使用jQuery设置省略号**


正如前面所说过的，Firefox并不支持CSS中的那些省略号设置，因为那并不是标准的HTML规范。所以，我们需要使用jQuery的Javascript来干这个事。你可以下载由Devon Govett写的[jQuery 省略号插件](http://devongovett.wordpress.com/2009/04/06/text-overflow-ellipsis-for-firefox-via-jquery/)，于是，你可以简单的把”ellipsis”赋给某些元件或是CSS定义，如下所示：


[javascript]$(document).ready(function() {  

    $(‘.ellipsis’).ellipsis();  

}[/javascript]


或是


[javascript]$("span").ellipsis();[/javascript]


