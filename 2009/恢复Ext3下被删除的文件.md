# 恢复Ext3下被删除的文件
>date: 2009-08-14T10:31:48+08:00


 下面是这个教程将教你如何在Ext3的文件系统中恢复被rm掉的文件。


假设我们有一个文件名叫 ‘test.txt’


 $ls -il test.txt  
 15 -rw-rw-r– 2 root root 20 Apr 17 12:08 test.txt  
 
注意：: “-il” 选项表示显示文件的i-node号（15），如果你不知道Unix/Linux文件系统的“I结点”的话，你有必要先补充一下相关的知识。简单说来，i结点就是操作管理文件的一个标识号。



我们再看一下其内容：


$ cat test.txt  
this is test file  

好，现在我们开始删除文件：.


$rm test.txt  
rm: remove write-protected regular file `test.txt'? y</span></span></div>
 
<h4>使用 Journal 和 Inode 号恢复</h4>
注意，如果你删除文件后重启了系统，那么，相关的文件 journal 会丢失，我们也就无法恢复文件了。所以，恢复文件的前提是，Journal不能丢失，即，系统不能重启。
因为我们已经知道 test.txt 文件的 inode 号是 15，所以我们可以使用 debugfs 命令来查看：
<div style="margin-left: 40px;"><span style="font-weight: bold;"><span style="font-style: italic;">debugfs: logdump -i <15></span></span>
<span style="font-weight: bold;"><span style="font-style: italic;">FS block 1006 logged at sequence 404351, journal block 7241</span></span><br style="font-style: italic;" /><span style="font-weight: bold;"><span style="font-style: italic;">(inode block for inode 15):</span></span><br style="font-style: italic;" /><span style="font-weight: bold;"><span style="font-style: italic;">Inode: 15 Type: regular Mode: 0664 Flags: 0x0 Generation: 0</span></span><br style="font-style: italic;" /><span style="font-weight: bold;"><span style="font-style: italic;">User: 0 Group: 0 Size: 20</span></span><br style="font-style: italic;" /><span style="font-weight: bold;"><span style="font-style: italic;">File ACL: 0 Directory ACL: 0</span></span><br style="font-style: italic;" /><span style="font-weight: bold;"><span style="font-style: italic;">Links: 1 Blockcount: 8</span></span><br style="font-style: italic;" /><span style="font-weight: bold;"><span style="font-style: italic;">Fragment: Address: 0 Number: 0 Size: 0</span></span><br style="font-style: italic;" /><span style="font-weight: bold;"><span style="font-style: italic;">ctime: 0x48159f2d -- Mon Apr 28 15:25:57 2008</span></span><br style="font-style: italic;" /><span style="font-weight: bold;"><span style="font-style: italic;">atime: 0x48159f27 -- Mon Apr 28 15:25:51 2008</span></span><br style="font-style: italic;" /><span style="font-weight: bold;"><span style="font-style: italic;">mtime: 0x4806f070 -- Thu Apr 17 12:08:40 2008</span></span><br style="font-style: italic;" /><span style="font-weight: bold;"><span style="font-style: italic;">Blocks: (0+1): 10234</span></span><br style="font-style: italic;" /><span style="font-weight: bold;"><span style="font-style: italic;">No magic number at block 7247: end of journal.</span></span><br style="font-style: italic;" /></div>
<span style="font-weight: bold;">
</span>请注意上面信息中的这一行：
<div style="margin-left: 40px;"><span style="font-weight: bold;"><span style="font-style: italic;">Blocks: (0+1): 10234</span></span></div>
这就是inode 15存放文件的地址（数据块）。然后，我们知道了这个地址，我们就可以使用 dd 命令，把这个地址上的数据给取出来。
<div style="margin-left: 40px;"><span style="font-weight: bold;"><span style="font-style: italic;">#dd if=/dev/sda5 of=/tmp/test.txt bs=4096 count=1 skip= 10234</span></span><br style="font-style: italic;" /><span style="font-weight: bold;"><span style="font-style: italic;">1+0 records in</span></span><br style="font-style: italic;" /><span style="font-weight: bold;"><span style="font-style: italic;">1+0 records out</span></span><br style="font-style: italic;" /></div>
<ul>
<li>if 是输入的设备</li>
<li>of 是输出的设备.</li>
<li>bs 指定一个block的大小</li>
<li>count 说明有多少个block需要dump</li>
<li>skip 说明从开始的地方跳过 10234 个block，并从取下一个block的数据</li>
</ul>
下面让我们看一下被恢复的文件：
<div style="margin-left: 40px;"><span style="font-weight: bold;"><span style="font-style: italic;">$cat /tmp/test.txt </span></span><br style="font-style: italic;" /><span style="font-weight: bold;"><span style="font-style: italic;">this is test file</span></span><br style="font-style: italic;" /></div>
<span style="font-weight: bold;">
</span>当然，上面的文件恢复是基于我们知道文件的inode，可在现实中，我们并不知道这个信息，如果我们不知道inode，我们还可能恢复吗？是的，这是可能的，让我们来看一下如何恢复。
<h4>使用 Journal 和 文件名恢复</h4>
如果我们不知道文件的inode我们可能恢复吗？我可以告诉你，这是不可能的事情。不过我们有办法知道文件的inode号。下面让我们来看看怎么做到：
<div style="margin-left: 40px;"><span style="font-weight: bold; font-style: italic;">$rm mytest.txt</span><br style="font-weight: bold; font-style: italic;" /><span style="font-weight: bold; font-style: italic;">rm: remove write-protected regular file` .txt’? y  

注意，我们并不知道其inode号，但我们可以使用 debugfs 命令来查看（使用其 ls -d 选项）。


debugfs:  ls -d  
 2  (12) .    2  (12) ..    11  (20) lost+found    2347777  (20) oss  
<2121567> (20) mytest.txt  

你看文件名了吧，它的inode号是 <2121567> ，注意，被删除了的文件的inode都是用尖括号包起来的。


即然知道了inode号，那么我们就很容易恢复了（使用 logdump选项）：


debugfs:  logdump -i <2121567>  
Inode 2121567 is at group 65, block 2129985, offset 3840  
Journal starts at block 1, transaction 405642  
  FS block 2129985 logged at sequence 405644, journal block 9  
    (inode block for inode 2121567):  
    Inode: 2121567   Type: bad type        Mode:  0000   Flags: 0x0   Generation: 0  
    User:     0   Group:     0   Size: 0  
    File ACL: 0    Directory ACL: 0  
    Links: 0   Blockcount: 0  
    Fragment:  Address: 0    Number: 0    Size: 0  
    ctime: 0x00000000 — Thu Jan  1 05:30:00 1970  
    atime: 0x00000000 — Thu Jan  1 05:30:00 1970  
    mtime: 0x00000000 — Thu Jan  1 05:30:00 1970  
    Blocks:  
  FS block 2129985 logged at sequence 405648, journal block 64  
    (inode block for inode 2121567):  
    Inode: 2121567   Type: regular        Mode:  0664   Flags: 0x0   Generation: 913772093  
    User:   100   Group:     0   Size: 31  
    File ACL: 2130943    Directory ACL: 0  
    Links: 1   Blockcount: 16  
    Fragment:  Address: 0    Number: 0    Size: 0  
    ctime: 0x4821d5d0 — Wed May  7 21:46:16 2008  
    atime: 0x4821d8be — Wed May  7 21:58:46 2008  
    mtime: 0x4821d5d0 — Wed May  7 21:46:16 2008  
    Blocks:  (0+1): 2142216  

上面有很多信息，让我们仔细地查看，你可以看到下面一行信息：


 FS block 2129985 logged at sequence 405644, journal block 9  

并且，其类型是：


 Type: bad type   

再仔细看一下文件的时间戳下面的Blocks: 什么也没有。那么，让我们看一下下一个block：


FS block 2129985 logged at sequence 405648, journal block 64  
    (inode block for inode 2121567):  

这一条Journal就有block信息了：


Blocks:  (0+1): 2142216
这就是被删除文件的地址，让我们再次运行恢复命令：


$sudo dd if=/dev/sda5 of=/home/hchen/mytest\_recovered.txt bs=4096 skip=2142216 count=1  

再让我们来检查一下文件内容：


$ cat mytest\_recovered.txt  
this is my test file   

#### 小结


好了，下面是我们的一些总结：  

1)使用 debugfs: ls -d 找到被删除文件的inode号。  
2)使用 debugfs:logdump找到文件的数据块地址。  
3)使用dd 命令把数据取出来存成文件。


网上有很其它不同的方法来恢复文件，基本上也是使用debugfs这个命令，有的还使用到了lsdel，其实大同小异，这个教程是我在网上看到的，虽然他说只是针对Ext3文件系统的，但我总感觉应该可以用于Ext2文件系统，不过我没有试过。也许Ext2和Ext3被debugfs输出的信息不一样吧。大家可以去试试。


（全文完）


