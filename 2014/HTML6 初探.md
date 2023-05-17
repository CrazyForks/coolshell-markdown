# HTML6 初探
>date: 2014-12-06T12:41:34+08:00



### html6


### HTML5 概述


HTML5 是 HTML 语言最受欢迎的版本之一，它支持音频和视频、离线存储、移动端、和标签属性等等。还提供了<article>, <section>, <header>这样的标签来帮助开发者更好地组织页面内容。然而 HTML5 规范仍然没有最后定稿，并且它并不是一个真正意义上的语义标记语言。


### HTML6 展望


你有没有曾经希望能在 HTML 中使用自定义标签？比如：使用<logo>来显示你的网站logo，还有使用<toolbar>来显示工具栏等等。我们经常使用<div id=”container”>和<div id=”wrapper”>来组织页面，在 HTML6 里我们希望可以直接使用象<container>和<wrapper>这样的自定义标签。


和 XML 一样，HTML6 应该支持 namespace（命名空间），如：xmlns:xhtml=”http://www.w3.org/1999/xhtml”


HTML6 代码样例：




```
<!DOCTYPE html>
 <html:html>
 <html:head>
 <html:title>A Look Into HTML6</html:title>
 <html:meta type="title" value="Page Title">
 <html:meta type="description" value="HTML example with namespaces">
 <html:link src="css/mainfile.css" title="Styles" type="text/css">
 <html:link src="js/mainfile.js" title="Script" type="text/javascript">
 </html:head>
 <html:body>
 <header>
 <logo>
 <html:media type="image" src="images/xyz.png">
 </logo>
 <nav>
 <html:a href="/img1">a1</a>
 <html:a href="/img2">a2</a>
 </nav>
 </header>
 <content>
 <article>
 <h1>Heading of main article</h1>
 <h2>Sub-heading of main article</h2>
 <p>[...]</p>
 <p>[...]</p>
 </article>
 <article>
 <h1>The concept of HTML6</h1>
 <h2>Understanding the basics</h2>
 <p>[...]</p>
 </article>
 </content>
 <footer>
 <copyright>This site is © to Anonymous 2014</copyright>
 </footer>
 </html:body>
 </html:html>
```

在上面的代码中，你也许注意到了一些奇怪的<html:x>标签，它们是 W3C 和 HTML6 规范中在命名空间里定义的标签。例如：<html:title>负责设定你浏览器的标题栏文字，<html:media>负责显示图片等等。用户可以自己定义标签以便 JavaScript 和 CSS 识别和处理，这样页面代码会更易读，语义更清晰。


### HTML6 APIs


HTML6 的标签前带有命名空间，如：<html:html>, <html:head>等等。


1. <html:html>



```
<!DOCTYPE html>
 <html:html>// this is equivalent to <html> tag written in previous HTML versions
 <!-- sample of HTML document -->
 </html:html>
```

2. <html:head> 和 <head> 标签一样。



```
<!DOCTYPE html>
 <html:html>
 <html:head>
 <!-- Main content would come here, like the <html:title> tag -->
 </html:head>
 </html:html>
```

3. <html:title> 和 <title> 标签类似。



```
<!DOCTYPE html>
 <html:html>
 <html:head>
 <html:title>A Look Into HTML6</html:title>
 </html:head>
 </html:html>
```

4. <html:meta> 和 <meta> 标签类似，不同之处在于，在 HTML5 中你只能使用标准的元数据类型，如：”keywords”, “description”, “author”等，而在 HTML6 中你可以使用任何元数据类型。



```
<!DOCTYPE html>
 <html:html>
 <html:head>
 <html:title>A Look Into HTML6</html:title>
 <html:meta type="description" value="HTML example with namespaces">
 </html:head>
 </html:html>
```

5. <html:link> 和 HTML6 之前版本的 <link> 标签类似。



```
<!DOCTYPE html>
 <html:html>
 <html:head>
 <html:title>A Look Into HTML6</html:title>
 <html:link src="js/mainfile.js" title="Script" type="text/javascript">
 </html:head>
 </html:html>
```

6. <html:body> 和 <body> 标签一样。



```
<!DOCTYPE html>
 <html:html>
 <html:head>
 <html:title>A Look Into HTML6</html:title>
 </html:head>
 <html:body>
 <!-- This is where your website content is placed -->
 </html:body>
 </html:html>
```

7. <html:a> 和 <a> 标签类似，区别是 <html:a> 只有 “href” 一个属性。



```
<!DOCTYPE html>
 <html:html>
 <html:head>
 <html:title>A Look Into HTML6</html:title>
 </html:head>
 <html:body>
 <html:a href="http://siteurl">Go to siteurl.com!</html:a>
 </html:body>
 </html:html>
```

8. <html：button> 和 <button> 及 <input type=”button”> 一样。



```
<!DOCTYPE html>
 <html:html>
 <html:head>
 <html:title>A Look Into HTML6</html:title>
 </html:head>
 <html:body>
 <html:button>Click Here</html:button>
 </html:body>
 </html:html>
```

9. <html:media> 涵盖 <img>, <video>, <embed> 等标签的所有功能。<html:media> 的好处是你不用根据不同的媒体文件类型使用不同的标签，媒体的类型由浏览器从文件内容（类型属性，扩展名，和MIME type）中来判断。



```
<!DOCTYPE html>
 <html:html>
 <html:head>
 <html:title>A Look Into HTML6</html:title>
 </html:head>
 <html:body>
 <!-- Image would come here -->
 <html:media src="img1/logo.jpg" type="image">
 <!-- Video doesn't need a type -->
 <html:media src="videos/slide.mov">
 </html:body>
 </html:html>
```

### 标签类型(Tag types)概述


和 HTML5 一样， HTML6 也有两种标签类型：单标签（single tag) 和双标签（double tag）



```
<html:meta type="author" content="single tag">
 <html:meta type="author" content="double tag" />
```

单标签不需要结束符’/’


### 结语


HTML6 规范还未发布，本文原作者 [Oscar Godson](http://html6spec.com/) 只是为我们提供了一个对 HTML6 规范的展望，或者说他希望 HTML6 能够支持的一些新特性。


原文链接：[A Look Into HTML6 – What Is It, and What Does it Have to Offer?](http://java.dzone.com/articles/look-html6-what-it-and-what)


