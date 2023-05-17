# Resin服务器getResource揭秘
>date: 2012-01-05T08:28:59+08:00


**（感谢网友 liuxiaori 继续分享其经历）这样的详细的图文并茂的文章让我很佩服！**




目录



* [前言](#%E5%89%8D%E8%A8%80 "前言")
* [调试环境](#%E8%B0%83%E8%AF%95%E7%8E%AF%E5%A2%83 "调试环境")
* [开始](#%E5%BC%80%E5%A7%8B "开始")
	+ [1) this.getClass().getResource(“/”).getPath()](#1_thisgetClassgetResource%E2%80%9C%E2%80%9DgetPath "1) this.getClass().getResource(“/”).getPath()")
	+ [1) this.getClass().getResourceAsStream(“/a.txt”)](#1_thisgetClassgetResourceAsStream%E2%80%9Catxt%E2%80%9D "1) this.getClass().getResourceAsStream(“/a.txt”)")
* [总结](#%E6%80%BB%E7%BB%93 "总结")

#### 前言


接上文“[由一个问题到Resin ClassLoader的学习](https://coolshell.cn/articles/6112.html "由一个问题到 Resin ClassLoader 的学习")”，本文将以this.getClass().getResource(“/”).getPath()和this.getClass().getResourceAsStream(“/a.txt”)为例，一步步解析加载的过程。


#### 调试环境


1. 下载resin3.0.23的源码(<http://www.caucho.com/download/resin-3.0.23-src.zip>)。
2. 部署到myeclipse中，有错误，本人忽略了。Resin可运行。
3. 将EhCacheTestAnnotation部署到resin3.0.23中。
4. 调试this.getClass().getResource(“/”).getPath()。


问题来了，无论如何也模拟不出来<compiling-loader>所造成的影响，一直输出：/D:/work\_other/project/resin-3.0.23/bin/ 。无奈之下，采用了这种方式：使用两个eclipse，一个使用发布版本的，部署EhCacheTestAnnotation进行调试；另外一个部署resin3.0.23源码，调试到哪里对照看源码。


#### 开始


##### 1) this.getClass().getResource(“/”).getPath()


本次调试涉及的所有类加载器为：



> EnvironmentClassLoader$24156236[web-app:http://localhost:8787/EhCacheTestAnnotation]
> 
> 
> EnvironmentClassLoader$7806641[host:http://localhost:8787]
> 
> 
> EnvironmentClassLoader$22459270[servlet-server:]
> 
> 
> [[email protected]](/cdn-cgi/l/email-protection)
> 
> 
> [[email protected]](/cdn-cgi/l/email-protection)
> 
> 


首先进入Class的getResource(String name)方法，如下图：



![图片1](/assets/images/coolshell.cn/wp-content/uploads/2012/01/图片1.png "图片1")图1
最后委托给ClassLoader的getResource方法。那么这个ClassLoader是哪个呢？一看下图便知：


![图片2](/assets/images/coolshell.cn/wp-content/uploads/2012/01/图片2.png "图片2")图2
是DynamicClassLoader的getResource方法，原理上文已述。


最终会委托给[[email protected]](/cdn-cgi/l/email-protection)类加载器的getResource方法，返回null，然后开始回溯。


还记得吗？当java.net.URLClassLoader分支的ClassLoader的getResource方法返回值为null后，就要遍历嵌入DynamicClassLoader中的Resin的Loader(即\_loaders集合)。


当然回溯到EnvironmentClassLoader$22459270[servlet-server:]中，那么它中\_loaders这个集合中的Loader又有哪些呢？


以图为证，当天确实回溯到该ClassLoader，而且开始准备遍历\_loaders集合。


![图3](/assets/images/coolshell.cn/wp-content/uploads/2012/01/图片3.png "图3")图3
DynamicClassLoader的1306行，没问题，resin3.0.23源码截图为证：


![图4](/assets/images/coolshell.cn/wp-content/uploads/2012/01/图片4.png "图4")图4
不做多余解释，那么“servlet-server”这个ClassLoader中的\_loaders集合中都放了一些什么呢？


![图5](/assets/images/coolshell.cn/wp-content/uploads/2012/01/图片5.png "图5")图5
存放了两个TreeLoader(Loader的子类)，然未找到结果，返回null。继续回溯。


这次轮到遍历EnvironmentClassLoader$7806641[host:http://localhost:8787]的\_loaders。下图为证：


![图6](/assets/images/coolshell.cn/wp-content/uploads/2012/01/图片6.png "图6")图6
\_loaders中的内容如下图：


![图7](/assets/images/coolshell.cn/wp-content/uploads/2012/01/图片7.png "图7")图7
比较长，我贴出来：



> [CompilingLoader[src:/D:/work/resin-3.0.23/webapps/WEB-INF/classes], LibraryLoader[[[email protected]](/cdn-cgi/l/email-protection)], CompilingLoader[src:/D:/work/resin-3.0.23/webapps/WEB-INF/classes], LibraryLoader[[[email protected]](/cdn-cgi/l/email-protection)], CompilingLoader[src:/D:/work/resin-3.0.23/webapps/WEB-INF/classes], LibraryLoader[[[email protected]](/cdn-cgi/l/email-protection)]]
> 
> 


注意到了吧，主角来了。那仔细调试下把。爆料一下：CompilingLoader[src:/D:/work/resin-3.0.23/webapps/WEB-INF/classes]就是主角。


![图8](/assets/images/coolshell.cn/wp-content/uploads/2012/01/图片8.png "图8")图8
看到了吧，遍历时，当前的Loader为CompilingLoader[src:/D:/work/resin-3.0.23/webapps/WEB-INF/classes]，而且url可是不为null了哦。再贴一张，看看url的值到底是什么！


![图9](/assets/images/coolshell.cn/wp-content/uploads/2012/01/图片9.png "图9")图9
嗯，不用多做解释了吧。


最后看看程序输出是否吻合，如下图：


![图10](/assets/images/coolshell.cn/wp-content/uploads/2012/01/图片10.png "图10")图10
然后修改resin.conf中的<compiling-loader>将其注释掉，看看程序结果会不会是我们期望的：/D:/work/resin-3.0.23/webapps/EhCacheTestAnnotation/WEB-INF/classes/。拭目以待。


![图11](/assets/images/coolshell.cn/wp-content/uploads/2012/01/图片11.png "图11")图11
为节省篇幅，一下只关注关键位置。


首先调试到EnvironmentClassLoader$7806641[host:http://localhost:8787]，我们需要停下来一下。


![图12](/assets/images/coolshell.cn/wp-content/uploads/2012/01/图片12.png "图12")图12
再看一下\_loaders的值。


![图13](/assets/images/coolshell.cn/wp-content/uploads/2012/01/图片13.png "图13")图13
贴一个详细的：



> [LibraryLoader[[[email protected]](/cdn-cgi/l/email-protection)], LibraryLoader[[[email protected]](/cdn-cgi/l/email-protection)], LibraryLoader[[[email protected]](/cdn-cgi/l/email-protection)]]
> 
> 


对比一下，在注释掉<compiling-loader>后，loaders中是没有CompilingClassLoader实例的。


继续，下面就轮到EnvironmentClassLoader$24156236[web-app:http://localhost:8787/EhCacheTestAnnotation]这个ClassLoader了，会是什么样子呢？


![图14](/assets/images/coolshell.cn/wp-content/uploads/2012/01/图片14.png "图14")图14
进入该ClassLoader时，url值依旧为null，那\_loaders会有变化吗？如下图：


![图15](/assets/images/coolshell.cn/wp-content/uploads/2012/01/图片15.png "图15")图15
继续遍历\_loaders。


![图16](/assets/images/coolshell.cn/wp-content/uploads/2012/01/图片16.png "图16")图16
到这里就结束了，url在EnvironmentClassLoader$24156236[web-app:http://localhost:8787/EhCacheTestAnnotation]中被加载。


##### 1) this.getClass().getResourceAsStream(“/a.txt”)


getResourceAsStream(String name)方法也是采用双亲委派的方式。在前一篇文章中提出“getResourceAsStream可是将获取路径委托给getResource，<compiling-loader>却没有对getResourceAsStream产生影响”


ClassLoader中getResourceAsStream源码也确实是委托为getResource了，可是为什么呢？


getResourceAsStream(String name)方法。



```
public InputStream getResourceAsStream(String name) {
    URL url = getResource(name);
    try {
        return url != null ? url.openStream() : null;
    } catch (IOException e) {
        return null;
    }
}

```

其实不难解释，JVM中ClassLoader的getResourceAsStream(“/a.txt”)返回了null，然后开始回溯，与getResource方法的原理一致，直到某个ClassLoader及其子类或者Loader及其子类找到了”/a.txt”，并以流的形式返回，当然谁都没找到就返回null。


捡重点的说。


调试到[[email protected]](/cdn-cgi/l/email-protection)，即ClassLoader的子类，情形如下图：


![图17](/assets/images/coolshell.cn/wp-content/uploads/2012/01/图片17.png "图17")图17
看见getResource(name)喽，按F5进去看个究竟。如下图，其parent为：[[email protected]](/cdn-cgi/l/email-protection)，其返回null。


![图18](/assets/images/coolshell.cn/wp-content/uploads/2012/01/图片18.png "图18")图18
开始回溯到：EnvironmentClassLoader$1497769[servlet-server:]，与getResource方法一致，开始遍历\_loaders集合。


这样就可以解释为何<compiling-loader>没有影响到getResourceAsStream了。因为资源(这里是/a.txt)，就不是由AppClassLoader和ExtClassLoader加载的，而是由DynamicClassLoader或者其内部的\_loaders集合完成的加载。或者更确切的说是由CompilingClassLoader获取到的URL，再转换成InputStream。


**<comiling-loader>其实对getResourceAsStream还是有点影响的，如果配置中配置了<comiling-loader>，并且<comiling-loader>配置的路径下，与实际项目的指定路径下，都放置了同名资源，则会先加载<comiling-loader>配置路径下的资源。**


比如，下图所示：


![图19](/assets/images/coolshell.cn/wp-content/uploads/2012/01/图片19.png "图19")图19
<compiling-loader>配置的路径为：<compiling-loader path=”webapps/WEB-INF/classes”/>


在加载”/a.txt”时，优先加载webapps/WEB-INF/classes/a.txt。


#### 总结


1. <compiling-loader>如被注释掉，则只会在EnvironmentClassLoader$24156236[web-app:http://localhost:8787/EhCacheTestAnnotation]中的\_loaders中被初始化，否则会在EnvironmentClassLoader$24156236[web-app:http://localhost:8787/EhCacheTestAnnotation]和EnvironmentClassLoader$7806641[host:http://localhost:8787两个类加载器各自的\_loaders集合中被初始化。(通过调试this.getClass().getResource(“/test”).getPath()验证)
2. <compiling-loader>未注释掉，”/”(根路径)由EnvironmentClassLoader$7806641[host:http://localhost:8787]加载，注释掉后由EnvironmentClassLoader$24156236[web-app:http://localhost:8787/EhCacheTestAnnotation]加载。
3. EnvironmentClassLoader$7806641[host:http://localhost:8787]为Resin server的类加载器实例，EnvironmentClassLoader$24156236[web-app:http://localhost:8787/EhCacheTestAnnotation]为Web应用程序的类加载器实例。他们都属于java.net.URLClassLoader的实例。
4. <compiling-loader>某种程度上对getResourceAsStream方法有影响。


现在<compiling-loader>如何影响getResource(“/”)，以及getResourceAsStream“不”被影响全部真相大白。


注：<compiling-loader>只对获取根路径产生影响，也就是参数为”/”。比如加载”/test/Path.class”不会产生影响。


（全文完）


