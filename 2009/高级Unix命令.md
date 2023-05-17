# 高级Unix命令
>date: 2009-06-16T10:42:39+08:00


在Unix操作中有太多太多的命令，这些命令的强大之处就是一个命令只干一件事，并把这件事干好。Do one thing， do it well。这是unix的哲学。而且Unix首创的管道可以把这些命令任意地组合，以完成一个更为强大功能。这些哲学到今天都在深深地影响着整个计算机产业。比如今天最流行的“云计算”——把一个软件以碎片方式部署，然后这些功能可以任意组合。


这篇文章罗列了很多Unix下比较高级的命令，当然，Unix/Linux下还有更多更多的命令，我们相信你可能见过其中的某些命令，也有可能有一些命令没有见过。不管怎么说，我们希望这些命令一方面可以让你知道怎么使用Unix/Linux操作系统，另一方面，我们也希望你能从中感到Unix的那种软件开发的哲学思想。





|  |  |
| --- | --- |
| ACCTCOM | :: |
| 查看所有用户执行过的进程（命令） | acctcom | tail -20 |
| 查看指定用户执行过的进程（命令） | acctcom -u <username> | tail -20 |
| 使用一个正则表达式查找相关进程 | acctcom -n <pattern> | tail -20 |
| 查找所有以l开头的被用户执行过的命令 | acctcom -n ‘^l’ | tail -30 |
| 以反向顺序显示 | acctom -b | more |
| AGREP | :: |
| 在文件中查找一个可能拼写错的单词 | agrep -2 ‘macropperswan’ <file> |
| AT | :: |
| 在未来某个时间执行某个命令 | at now + 5 days < scriptfile |
| AWK | :: |
| 显示文件的第一列 | awk ‘{print $1}’ <file> |
| 反序显示文件的前两列 | awk ‘{print $2,”\t”,$1}’ <file> |
| 输出前两列的总和 | awk ‘{print $1 + $2}’ <file> |
| 查找所有包括”money” 行并输出最后一列 | awk ‘/money/ {print $NF}’ <file> |
| 查找第二列中包含 “money” | awk ‘$2 ~ /money/ {print $0}’ <file> |
| 查找第三列中不包括”A” | awk ‘$3 !~ /A$/ {print $0}’ <file> |
| BC | :: |
| 计算sin(5)的值 | echo ‘s(5)’ | bc -l |
| CANCEL | :: |
| 取消一个刚开始启动的打印的作业 | cancel <jobid> ( jobid可以由lpstat -o输出) |
| CASE in ESAC  | :: |
| sh/bash/ksh中的case语句 |  |
| CC | :: |
| 编译一个C文件file.c | cc -o <outfile> <infile> |
| CHGRP | :: |
| 修改文件的组所属 | chgrp <newgroupname> <file> |
| CHOWN | :: |
| 修改文件的所属人 | chown <newowner> <file> |
| CMP | :: |
| 比较两个文件 | cmp <file1> <file2> || <command> |
| COL | :: |
| 打印man pages，去除其中 “^H” | man <command> | col -b | <printcommand> |
| CRONTAB | :: |
| 查看你的crontab 文件 | crontab -l |
| 编译 crontab 文件 | crontab -e |
| 第周一的05:10 执行/home/fred/foo.ksh | 10 5 \* \* 1 /home/fred/foo.ksh |
| CRYPT | :: |
| 使用一个口令加密一个文件 | crypt password < infile > cryptfile |
| 解密一个被上面命令加密了的文件 | crypt password < cryptfile > cleanfile |
| CSH  | :: |
| 最好的Berkley shell |  |
| CUT | :: |
| 从last 命令的输出中得到hostname字段 | last | cut -c11-40 |
| DATE | :: |
| 设置时间(只能由root 执行) | date <mmddhhmm> |
| 输出指定日期格式 (如：月份) | date +%m |
| DF | :: |
| 以kB单位查看磁盘空间 | df -k |
| DIRCMP | :: |
| 比较两个目录 | dircmp <dir1> <dir2> |
| DTKSH | :: |
| dtksh 是一个 X11 图形的ksh93 | dtksh |
| DU | :: |
| 磁盘使用情况 | du -ks |
| ED  | :: |
| 命令行编译器。 | ed <file> |
| EGREP | :: |
| 使用“或”条件Grep 文件 | egrep ‘(A|B)’ <file> |
| grep文件中即不包括A也不包括B | egrep -v ‘(A|B)’ <file> |
| EX | :: |
| 使用一个shell脚来来编辑一个文件 | ex -s file <<EOF
g/money/s//cash/
EOF |
| 以一个脚本文件来编辑一个文件 | ex -s file < scriptfile |
| EXPR | :: |
| 求模 | expr 10 % 7 |
| 查看字串是否在变量$var中 | expr $var : ‘string’ |
| 显示第一个数字组成的字串 | expr $var : ‘[^0-9]\*\([a-z]\*\)’ |
| FGREP | :: |
| 查找不匹配于某正规表达式的文件行 | fgrep ‘\*,/.()’ <file> |
| FILE | :: |
| 查看文件类型(如： ascii) | file <file> |
| FIND | :: |
| 在整个文件系统中查的一个文件 | find / -type f -name <file> -print |
| 查找所有匹配于模式的文件 | find . -type f -name “\*<foo>\*” -print |
| 删除系统中所有的core文件 | find / -type f -name core -exec /bin/rm -f {} \; |
| 查找所有包含某单词的文件 | find . -type f -exec grep -l <word> {} \; |
| 查找所有修改日期在30天以前的文件 | find . -type f -ctime +30 -print |
| 使用xargs来备份所有的.c文件（加上.bak后缀） | find . -name “\*.c” -print | xargs -i cp {} {}.bak |
| 只搜索本地文件系统（不搜索nfs文件系统） | find . -local … |
| 在搜索的过程中，跟随link文件的实际位置 | find . -follow … |
| 查找大于1M的文件 | find /path -size 1000000c -print |
| 运行find命令但忽略”permission denied” | find … 2>/dev/null ( 只能在sh/bash/ksh ) |
| 查找所有的man目录 | find / -type d -print | egrep ‘.\*/(catman|man)$’ |
| 查找所有有写权限的目录 | find / -type d -perm -002 -print |
| GAWK  | :: |
| GNU版本的nawk |  |
| GREP | :: |
| 以某个正规表达式查找包含其的文件行 | grep ‘[a-z][0-9]’ <file> |
| 查找不包含指定正则表达式的文件行 | grep -v ‘^From’ <file> |
| 查找一组文件 | grep -l ‘^[cC]’ \*.f |
| 计算包括某正则表达式文件行的数目 | grep -c ‘[Ss]uccess’ <file> |
| 不区分大小写的查找 | grep -i ‘lAbEgF’ <file> |
| 在匹配到的文件内容前输出文件的行号 | grep -n ‘mo.\*y’ <file> |
| HINV  | :: |
| 命令显示系统硬件的详细列表，包括：CPU类型、内存大小、所有的磁盘设备。 | hinv -v |
| IF then else ENDIF  | :: |
| csh/tcsh中的if 语句 |  |
| IF then else FI  | :: |
| sh/bash/ksh 中的if 语句 | if [[ condition ]];then commands;fi |
| KSH | :: |
| Korn shell. (ksh88) |  |
| LN | :: |
| 创建一个硬链接文件a链接到文件A | ln a B |
| 创建一个符号链接文件a链接到文件A | ln -s a B |
| 删除链接文件B | rm B |
| LP | :: |
| 在默认打印机上打印文件 | lp <file> |
| 在指定打印机上打印文件 | lp -d <destination> <file> |
| LPSTAT | :: |
| 显示所有的打印机 | lpstat -a |
| 查看打印机任务队列 | lpstat -o |
| 查看默认打印机 | lpstat -d |
| 查看打印机状态 | lpstat -p |
| 查看计划任何状态 | lpstat -r |
| MAKE | :: |
| 执行一个 makefile中的第一个目标 | make |
| 执行一个 makefile中的指点目标 | make <target> |
| 指定一个特定的makefile文件名 | make -f <mymakefile> |
| 显示要做什么，但其实什么也没做 | make -n <target> |
| MKDIR | :: |
| 一次创键目录和子目录 | mkdir -p <path>/<path>/<path> |
| MOUNT  | :: |
| 查看挂载的文件卷 | mount |
| 查看挂载的文件卷（有格式的） | mount -p |
| 挂载一个光驱到目录/cdrom | mount /dev/cdrom /cdrom |
| 挂载一个磁盘分区到目录 /usr | mount /dev/dsk/c0t3d0s5 /usr |
| NAWK | :: |
| 增强版的 awk |  |
| NL | :: |
| 以带行号的方式输出文件 | nl -bt -nln <file> |
| NOHUP | :: |
| 启动一个命令马上退出 | nohup <command> & |
| PACK | :: |
| 一个很老的文件打包程序，现在被gzip代替了。 | pack <file> |
| PASSWD | :: |
| 修改你的帐号口令 | passwd |
| 删除一个用户的口令(root使用) | passwd -d <username> |
| 改变一个用户的口令 (root使用) | passwd <username> |
| PASTE | :: |
| 以列的方式把多个文件组合起来 | paste <file1> <file2> > <newfile> |
| PERL | :: |
| Perl脚本语言的解释器 |
| PR | :: |
| 把一个文件做成可打印的格式（76行一页） | pr -l76 -h”title” <filename> |
|  |  |
| REGCMP | :: |
| 从一个文件中编译正则表达式 | regcmp <file> |
| 文件内容示例 | varname “^[a-z].\*[0-9.\*$” |
| RESET | :: |
| 重置终端设备 | reset |
| RPCINFO | :: |
| 取得某主机的TCP端口信息 | rpcinfo -p <host> |
| RSH | :: |
| 执行一个远程服务器上的命令 | rsh <host> <comand> |
| SCRIPT | :: |
| 用来捕捉当前的终端会话中的所有输入输出结果到一个指定的文件 | script <logfile> |
| SED | :: |
| 把某文件中的fred替换成john | sed -e ‘s/fred/john/g’ <file> |
| 替换文件中匹配正则表达式的字符串 | sed -e ‘s/[0-9]+/number/g’ <file> |
| 把HTML文件中的 “X” 变成红色 | sed -e ‘s!X!<font color=”#FF0000″>X</font>!g; |
| 把所有后缀为.suf1 改名成.suf2 | ls -1 | grep ‘\.suf1$’ | sed -e ‘s/\(.\*\.\)suf1/mv & \1suf2/’ | sh |
| 把文件中包含c的行中的a 替换成b | sed -e ‘/C/s/A/B/’ <infile> ><outfile> |
| 删除所有包含 “you owe me”的文件行 | sed -e ‘/you owe me/d’ <infile> > <outfile> |
| 使用commandfile中的命令来编译infile文件，并输出到outfile中。其中的commandfile中包含了一系列的vi命令 | sed -f <commandfile> <infile> > <outfile> |
| SH | :: |
| 最老的 AT&T shell程序，也是使用最广泛的标准确shell。 |
| SHUTDOWN | :: |
| 关机 | shutdown -h now |
| SLEEP | :: |
| sleep 10秒钟 | sleep 10 |
| SORT | :: |
| 以字符顺序把文件的每一行排序 | sort <file> |
| 以数字顺序把文件的每一行排序 | sort -n <file> |
| 反向排序 | sort -r <file> |
| 排序时对于重复项只保留一个 | sort -u <file> |
|  |  |
| SPELL | :: |
| 检查拼写错误 | spell <file> |
| 检查拼写错误，但是忽略okfile中包含的单词 | spell +<okfile> <file> |
| SPLIT | :: |
| 拆分一个大文件，每个文件1m | split -b1m <file> |
| 把拆分后的文件合并起来 | cat x\* > <newfile> |
| STRINGS | :: |
| 从二进制文件中读取ascii 字符串 | strings <file> |
| STTY | :: |
| 显示终端设置 | stty -a |
| 设置 Ctrl+”H”为删除键 | stty erase “^H” |
| 对于用户的输入不回显 | stty -echo |
| 回显用户的输入 | stty echo |
| SU | :: |
| 切换到root用户 | su |
| 切换到root用户并使用其环境 | su – |
| 切换到另一用户 | su <username> |
| TAIL | :: |
| 显示某文件中的文件尾中包含pattern的文件行 | tail -f <file> | grep <pattern> |
| TAR | :: |
| 把整个目录打包（没有压缩） | tar cvf <outfile>.tar <dir> |
| 解包某个tar文件 | tar xvf <file>.tar |
| 先解压缩再解包 | gzip -dc <file>.tar.gz | tar xvf – |
| 打包成一个压缩包 | tar xzvf <file>tar.gz |
| 在.cshrc中设置 tar命令的tape 变量 | tape=/dev/rmt/0mbn |
| 把一个目录打包到tape变量所指的目录中 | tar cv <dir> |
| 从tape中解包 | tar xv |
| 从tape中解出一个文件 | tar xv <file> |
| 从 tape中得到一个内容表 | tar t |
| 以合适的权限和链接拷贝一个目录 | (cd fromdir && tar -cBf – . ) | ( cd todir && tar -xBf – ) |
| TCSH | :: |
| Berkly的另一个非常不错的shell |
| TEE | :: |
| 把标准输入重定向到标准输出 | who | tee -a > <file> |
| TEST | :: |
| 检查是否是一个文件 | test -a <file> |
| 检查是否某文件是否是root属性 | test -O /usr/bin/su |
| 检查某变量是否为 null | test -n “$foo” |
| 以数字的方式比较两个数字字符串 | test $var1 -gt $var2 |
| 在ksh 脚本中间接地使用”test” | if [[ -a <file> ]];then …;fi |
| TIME | :: |
| 查看运行一个命令需要多少时间 | time <command> |
| TOUCH | :: |
| 更新文件的修改时间为当前时间，文件不存在则创建文件 | touch <file> |
| TR | :: |
| 使用x替换a，y替换b，c替换z | tr ‘[a-c]’ ‘[x-z]’ < infile > outfile |
| TRAP | :: |
| 捕捉”^C” 并执行子程序 | trap “mysub;exit” 0 1 2 15 |
| TRUE | :: |
| 让个不存在的命令返回0 | ln -s /usr/bin/true ranlib |
| TRUSS | :: |
| 查看一个命令运行时的系统调用 | truss <command> > /dev/null |
| TYPSET | :: |
| 查看被激活的功能 | typset |
| TTY | :: |
| 查看终端所在的设备文件 | tty |
| ULIMIT | :: |
| 查看系统所支持的最大文件长度 | ulimit |
| UMASK | :: |
| 查看目前的umask | umask |
| 设置一个umask | umask 077 |
| UNIQ | :: |
| 查看一个文件中有多少行是一样的 | sort <file> | uniq -c |
| 仅输出唯一的没有重复的行 | sort <file> | uniq -u |
| UPTIME | :: |
| 查看你的电脑开机多少时间了 | uptime |
| UUENCODE | :: |
| Encode一个文件以便发送电子邮件 | uuencode decodedname namenow > codedname |
| UUDECODE | :: |
| Decode 一个 uuencoded 文件 | uudecode <file> |
| WAIT | :: |
| 等一个后进和运行结束 | wait $jobid |
| VI | :: |
| 最主要的unix编译器 | vi <file> |
| WC | :: |
| 计算一个文件的行号 | wc -l <file> |
| XARGS | :: |
| 把标准输出作为参数来执行一条命令 | <command> | xargs -i grep ‘pattern’ {} |
| XON | :: |
| 从另一台电脑上得到一个xterm | xon <host> |
| 从另一台电脑上得到所有的东西 | xon <host> <X-client> |


(全文完)


