# Git显示漂亮日志的小技巧
>date: 2012-06-24T23:29:05+08:00


原文：[http://garmoncheg.blogspot.com/2012/06/pretty-git-log.html](https://garmoncheg.blogspot.com/2012/06/pretty-git-log.html) （墙）


Git的传统log如下所示，你喜欢吗？


![](/assets/images/coolshell.cn/wp-content/uploads/2012/06/git.log_.01.png "默认的Git的log")


看看下面这个你喜不喜欢？（点击图片看大图）



[![](/assets/images/coolshell.cn/wp-content/uploads/2012/06/git.log_.02.png "改进版的Git的日志")](/assets/images/coolshell.cn/wp-content/uploads/2012/06/git.log_.02.png)


要做到这样，命令行如下：


`git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --`


这样有点长了，我们可以这样：


`git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --"`


然后，我们就可以使用这样的短命令了：


`git lg`


如果你想看看git log –pretty=format的参数，你可以看看[这篇文章](http://git-scm.com/book/zh/Git-%E5%9F%BA%E7%A1%80-%E6%9F%A5%E7%9C%8B%E6%8F%90%E4%BA%A4%E5%8E%86%E5%8F%B2)。


（全文完）


