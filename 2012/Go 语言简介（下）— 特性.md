# Go 语言简介（下）— 特性
>date: 2012-11-07T08:17:20+08:00


希望你看到这篇文章的时候还是在公交车和地铁上正在上下班的时间，我希望我的这篇文章可以让你利用这段时间了解一门语言。当然，希望你不会因为看我的文章而错过站。呵呵。


如果你还不了解Go语言的语法，还请你移步先看一下上篇——《**[Go语言简介（上）：语法](https://coolshell.cn/articles/8460.html "Go语言简介（上）：语法")**》


![](/assets/images/google-go-language.jpg "google-go-language")




目录



* [goroutine](#goroutine "goroutine")
* [goroutine的并发安全性](#goroutine%E7%9A%84%E5%B9%B6%E5%8F%91%E5%AE%89%E5%85%A8%E6%80%A7 "goroutine的并发安全性")
* [原子操作](#%E5%8E%9F%E5%AD%90%E6%93%8D%E4%BD%9C "原子操作")
* [Channel 信道](#Channel_%E4%BF%A1%E9%81%93 "Channel 信道")
* [定时器](#%E5%AE%9A%E6%97%B6%E5%99%A8 "定时器")
* [Socket编程](#Socket%E7%BC%96%E7%A8%8B "Socket编程")
* [系统调用](#%E7%B3%BB%E7%BB%9F%E8%B0%83%E7%94%A8 "系统调用")
* [执行命令行](#%E6%89%A7%E8%A1%8C%E5%91%BD%E4%BB%A4%E8%A1%8C "执行命令行")
* [命令行参数](#%E5%91%BD%E4%BB%A4%E8%A1%8C%E5%8F%82%E6%95%B0 "命令行参数")
* [一个简单的HTTP Server](#%E4%B8%80%E4%B8%AA%E7%AE%80%E5%8D%95%E7%9A%84HTTP_Server "一个简单的HTTP Server")

#### goroutine


GoRoutine主要是使用go关键字来调用函数，你还可以使用匿名函数，如下所示：




```
package main
import "fmt"

func f(msg string) {
    fmt.Println(msg)
}

func main(){
    go f("goroutine")

    go func(msg string) {
        fmt.Println(msg)
    }("going")
}
```

我们再来看一个示例，下面的代码中包括很多内容，包括时间处理，随机数处理，还有goroutine的代码。如果你熟悉C语言，你应该会很容易理解下面的代码。


你可以简单的把go关键字调用的函数想像成pthread\_create。下面的代码使用for循环创建了3个线程，每个线程使用一个随机的Sleep时间，然后在routine()函数中会输出一些线程执行的时间信息。



```
package main

import "fmt"
import "time"
import "math/rand"

func routine(name string, delay time.Duration) {

    t0 := time.Now()
    fmt.Println(name, " start at ", t0)

    time.Sleep(delay)

    t1 := time.Now()
    fmt.Println(name, " end at ", t1)

    fmt.Println(name, " lasted ", t1.Sub(t0))
}

func main() {

    //生成随机种子
    rand.Seed(time.Now().Unix())

    var name string
    for i:=0; i<3; i++{
        name = fmt.Sprintf("go_%02d", i) //生成ID
        //生成随机等待时间，从0-4秒
        go routine(name, time.Duration(rand.Intn(5)) * time.Second)
    }

    //让主进程停住，不然主进程退了，goroutine也就退了
    var input string
    fmt.Scanln(&input)
    fmt.Println("done")
}

```

运行的结果可能是：



```
go_00  start at  2012-11-04 19:46:35.8974894 +0800 +0800
go_01  start at  2012-11-04 19:46:35.8974894 +0800 +0800
go_02  start at  2012-11-04 19:46:35.8974894 +0800 +0800
go_01  end at  2012-11-04 19:46:36.8975894 +0800 +0800
go_01  lasted  1.0001s
go_02  end at  2012-11-04 19:46:38.8987895 +0800 +0800
go_02  lasted  3.0013001s
go_00  end at  2012-11-04 19:46:39.8978894 +0800 +0800
go_00  lasted  4.0004s

```

#### goroutine的并发安全性


关于goroutine，我试了一下，无论是Windows还是Linux，基本上来说是用操作系统的线程来实现的。不过，goroutine有个特性，也就是说，**如果一个goroutine没有被阻塞，那么别的goroutine就不会得到执行**。这并不是真正的并发，如果你要真正的并发，你需要在你的main函数的第一行加上下面的这段代码：



```
import "runtime"
...
runtime.GOMAXPROCS(4)
```

还是让我们来看一个有并发安全性问题的示例（注意：我使用了C的方式来写这段Go的程序）


这是一个经常出现在教科书里卖票的例子，我启了5个goroutine来卖票，卖票的函数sell\_tickets很简单，就是随机的sleep一下，然后对全局变量total\_tickets作减一操作。



```
package main

import "fmt"
import "time"
import "math/rand"
import "runtime"

var total_tickets int32 = 10;

func sell_tickets(i int){
    for{
        if total_tickets > 0 { //如果有票就卖
            time.Sleep( time.Duration(rand.Intn(5)) * time.Millisecond)
            total_tickets-- //卖一张票
            fmt.Println("id:", i, "  ticket:", total_tickets)
        }else{
            break
        }
    }
}

func main() {
    runtime.GOMAXPROCS(4) //我的电脑是4核处理器，所以我设置了4
    rand.Seed(time.Now().Unix()) //生成随机种子

    for i := 0; i < 5; i++ { //并发5个goroutine来卖票
         go sell_tickets(i)
    }
    //等待线程执行完
    var input string
    fmt.Scanln(&input)
    fmt.Println(total_tickets, "done") //退出时打印还有多少票
}
```

这个程序毋庸置疑有并发安全性问题，所以执行起来你会看到下面的结果：



```
$go run sell_tickets.go
id: 0   ticket: 9  
id: 0   ticket: 8  
id: 4   ticket: 7  
id: 1   ticket: 6  
id: 3   ticket: 5  
id: 0   ticket: 4  
id: 3   ticket: 3  
id: 2   ticket: 2  
id: 0   ticket: 1  
id: 3   ticket: 0  
id: 1   ticket: -1  
id: 4   ticket: -2  
id: 2   ticket: -3  
id: 0   ticket: -4  
-4 done
```

可见，我们需要使用上锁，我们可以使用互斥量来解决这个问题。下面的代码，我只列出了修改过的内容：



```
 package main
import "fmt"
import "time"
import "math/rand"
import "sync"
import "runtime"

var total_tickets int32 = 10;
var mutex = &sync.Mutex{} //可简写成：var mutex sync.Mutex

func sell_tickets(i int){
    for total_tickets>0 {
        mutex.Lock()
        if total_tickets > 0 {
            time.Sleep( time.Duration(rand.Intn(5)) * time.Millisecond)
            total_tickets--
            fmt.Println(i, total_tickets)
        }
        mutex.Unlock()
    }
}
.......
......

```

#### 原子操作


说到并发就需要说说原子操作，相信大家还记得我写的那篇《[无锁队列的实现](https://coolshell.cn/articles/8239.html "无锁队列的实现")》一文，里面说到了一些CAS – CompareAndSwap的操作。Go语言也支持。你可以看一下相当的文档


我在这里就举一个很简单的示例：下面的程序有10个goroutine，每个会对cnt变量累加20次，所以，最后的cnt应该是200。如果没有atomic的原子操作，那么cnt将有可能得到一个小于200的数。


下面使用了atomic操作，所以是安全的。



```
package main

import "fmt"
import "time"
import "sync/atomic"

func main() {
    var cnt uint32 = 0
    for i := 0; i < 10; i++ {
        go func() {
            for i:=0; i<20; i++ {
                time.Sleep(time.Millisecond)
                atomic.AddUint32(&cnt, 1)
            }
        }()
    }
    time.Sleep(time.Second)//等一秒钟等goroutine完成
    cntFinal := atomic.LoadUint32(&cnt)//取数据
    fmt.Println("cnt:", cntFinal)
}
```

这样的函数还有很多，参看[go的atomic包文档](https://golang.org/pkg/sync/atomic/)（被墙）


#### Channel 信道


Channal是什么？Channal就是用来通信的，就像Unix下的管道一样，在Go中是这样使用Channel的。


下面的程序演示了一个goroutine和主程序通信的例程。这个程序足够简单了。



```
package main

import "fmt"

func main() {
    //创建一个string类型的channel
    channel := make(chan string)

    //创建一个goroutine向channel里发一个字符串
    go func() { channel <- "hello" }()

    msg := <- channel
    fmt.Println(msg)
}[
```

**指定channel的buffer**


指定buffer的大小很简单，看下面的程序：



```
package main
import "fmt"

func main() {
    channel := make(chan string, 2)

    go func() {
        channel <- "hello"
        channel <- "World"
    }()

    msg1 := <-channel
    msg2 := <-channel
    fmt.Println(msg1, msg2)
}
```

**Channel的阻塞**


注意，channel默认上是阻塞的，也就是说，如果Channel满了，就阻塞写，如果Channel空了，就阻塞读。于是，我们就可以使用这种特性来同步我们的发送和接收端。


下面这个例程说明了这一点，代码有点乱，不过我觉得不难理解。



```
package main

import "fmt"
import "time"

func main() {

    channel := make(chan string) //注意: buffer为1

    go func() {
        channel <- "hello"
        fmt.Println("write \"hello\" done!")

        channel <- "World" //Reader在Sleep，这里在阻塞
        fmt.Println("write \"World\" done!")

        fmt.Println("Write go sleep...")
        time.Sleep(3*time.Second)
        channel <- "channel"
        fmt.Println("write \"channel\" done!")
    }()

    time.Sleep(2*time.Second)
    fmt.Println("Reader Wake up...")

    msg := <-channel
    fmt.Println("Reader: ", msg)

    msg = <-channel
    fmt.Println("Reader: ", msg)

    msg = <-channel //Writer在Sleep，这里在阻塞
    fmt.Println("Reader: ", msg)
}
```

上面的代码输出的结果如下：



```
Reader Wake up...
Reader:  hello
write "hello" done!
write "World" done!
Write go sleep...
Reader:  World
write "channel" done!
Reader:  channel

```

**Channel阻塞的这个特性还有一个好处是，可以让我们的goroutine在运行的一开始就阻塞在从某个channel领任务，这样就可以作成一个类似于线程池一样的东西。关于这个程序我就不写了。我相信你可以自己实现的。**


**多个Channel的select**



```
package main
import "time"
import "fmt"

func main() {
    //创建两个channel - c1 c2
    c1 := make(chan string)
    c2 := make(chan string)

    //创建两个goruntine来分别向这两个channel发送数据
    go func() {
        time.Sleep(time.Second * 1)
        c1 <- "Hello"
    }()
    go func() {
        time.Sleep(time.Second * 1)
        c2 <- "World"
    }()

    //使用select来侦听两个channel
    for i := 0; i < 2; i++ {
        select {
        case msg1 := <-c1:
            fmt.Println("received", msg1)
        case msg2 := <-c2:
            fmt.Println("received", msg2)
        }
    }
}
```

注意：上面的select是阻塞的，所以，才搞出ugly的for i <2这种东西**。**


**Channel select阻塞的Timeout**


解决上述那个for循环的问题，一般有两种方法：一种是阻塞但有timeout，一种是无阻塞。我们来看看如果给select设置上timeout的。



```
    for {
        timeout_cnt := 0
        select {
        case msg1 := <-c1:
            fmt.Println("msg1 received", msg1)
        case msg2 := <-c2:
            fmt.Println("msg2 received", msg2)
        case  <-time.After(time.Second * 30)：
            fmt.Println("Time Out")
            timout_cnt++
        }
        if time_cnt > 3 {
            break
        }
    }

```

上面代码中高亮的代码主要是用来让select返回的，注意 case中的time.After事件。


**Channel的无阻塞**


好，我们再来看看无阻塞的channel，其实也很简单，就是在select中加入default，如下所示：



```
    for {
        select {
        case msg1 := <-c1:
            fmt.Println("received", msg1)
        case msg2 := <-c2:
            fmt.Println("received", msg2)
        default: //default会导致无阻塞
            fmt.Println("nothing received!")
            time.Sleep(time.Second)
        }
    }

```

**Channel的关闭**


关闭Channel可以通知对方内容发送完了，不用再等了。参看下面的例程：



```
package main

import "fmt"
import "time"
import "math/rand"

func main() {

    channel := make(chan string)
    rand.Seed(time.Now().Unix())

    //向channel发送随机个数的message
    go func () {
        cnt := rand.Intn(10)
        fmt.Println("message cnt :", cnt)
        for i:=0; i<cnt; i++{
            channel <- fmt.Sprintf("message-%2d", i)
        }
        close(channel) //关闭Channel
    }()

    var more bool = true
    var msg string
    for more {
        select{
        //channel会返回两个值，一个是内容，一个是还有没有内容
        case msg, more = <- channel:
            if more {
                fmt.Println(msg)
            }else{
                fmt.Println("channel closed!")
            }
        }
    }
}
```

#### 定时器


Go语言中可以使用time.NewTimer或time.NewTicker来设置一个定时器，这个定时器会绑定在你的当前channel中，通过channel的阻塞通知机器来通知你的程序。


下面是一个timer的示例。



```
package main

import "time"
import "fmt"

func main() {
    timer := time.NewTimer(2*time.Second)

    <- timer.C
    fmt.Println("timer expired!")
}
```

上面的例程看起来像一个Sleep，是的，不过Timer是可以Stop的。你需要注意Timer只通知一次。如果你要像C中的Timer能持续通知的话，你需要使用Ticker。下面是Ticker的例程：



```
package main

import "time"
import "fmt"

func main() {
    ticker := time.NewTicker(time.Second)

    for t := range ticker.C {
        fmt.Println("Tick at", t)
    }
}
```

上面的这个ticker会让你程序进入死循环，我们应该放其放在一个goroutine中。下面这个程序结合了timer和ticker



```
package main

import "time"
import "fmt"

func main() {

    ticker := time.NewTicker(time.Second)

    go func () {
        for t := range ticker.C {
            fmt.Println(t)
        }
    }()

    //设置一个timer，10钞后停掉ticker
    timer := time.NewTimer(10*time.Second)
    <- timer.C

    ticker.Stop()
    fmt.Println("timer expired!")
}
```

#### Socket编程


下面是我尝试的一个Echo Server的Socket代码，感觉还是挺简单的。


**Server端**



```
 
package main

import (
    "net"
    "fmt"
    "io"
)

const RECV_BUF_LEN = 1024

func main() {
    listener, err := net.Listen("tcp", "0.0.0.0:6666")//侦听在6666端口
    if err != nil {
        panic("error listening:"+err.Error())
    }
    fmt.Println("Starting the server")

    for {
        conn, err := listener.Accept() //接受连接
        if err != nil {
            panic("Error accept:"+err.Error())
        }
        fmt.Println("Accepted the Connection :", conn.RemoteAddr())
        go EchoServer(conn)
    }
}

func EchoServer(conn net.Conn) {
    buf := make([]byte, RECV_BUF_LEN)
    defer conn.Close()

    for {
        n, err := conn.Read(buf);
        switch err {
            case nil:
                conn.Write( buf[0:n] )
            case io.EOF:
                fmt.Printf("Warning: End of data: %s \n", err);
                return
            default:
                fmt.Printf("Error: Reading data : %s \n", err);
                return
        }
     }
}

```

**Client端**



```
package main

import (
    "fmt"
    "time"
    "net"
)

const RECV_BUF_LEN = 1024

func main() {
    conn,err := net.Dial("tcp", "127.0.0.1:6666")
    if err != nil {
        panic(err.Error())
    }
    defer conn.Close()

    buf := make([]byte, RECV_BUF_LEN)

    for i := 0; i < 5; i++ {
        //准备要发送的字符串
        msg := fmt.Sprintf("Hello World, %03d", i)
        n, err := conn.Write([]byte(msg))
        if err != nil {
            println("Write Buffer Error:", err.Error())
            break
        }
        fmt.Println(msg)

        //从服务器端收字符串
        n, err = conn.Read(buf)
        if err !=nil {
            println("Read Buffer Error:", err.Error())
            break
        }
        fmt.Println(string(buf[0:n]))

        //等一秒钟
        time.Sleep(time.Second)
    }
}

```

#### 系统调用


Go语言那么C，所以，一定会有一些系统调用。Go语言主要是通过两个包完成的。一个是[os包](https://golang.org/pkg/os/)，一个是[syscall包](https://golang.org/pkg/syscall/)。（注意，链接被墙）


这两个包里提供都是Unix-Like的系统调用，


* syscall里提供了什么Chroot/Chmod/Chmod/Chdir…，Getenv/Getgid/Getpid/Getgroups/Getpid/Getppid…，还有很多如Inotify/Ptrace/Epoll/Socket/…的系统调用。


* os包里提供的东西不多，主要是一个跨平台的调用。它有三个子包，Exec（运行别的命令）, Signal（捕捉信号）和User（通过uid查name之类的）


syscall包的东西我不举例了，大家可以看看《Unix高级环境编程》一书。


os里的取几个例：


**环境变量**



```
package main

import "os"
import "strings"


func main() {
    os.Setenv("WEB", "https://coolshell.cn") //设置环境变量
    println(os.Getenv("WEB")) //读出来

    for _, env := range os.Environ() { //穷举环境变量
        e := strings.Split(env, "=")
        println(e[0], "=", e[1])
    }
}

```

#### 执行命令行


下面是一个比较简单的示例



```
package main
import "os/exec"
import "fmt"
func main() {
    cmd := exec.Command("ping", "127.0.0.1")
    out, err := cmd.Output()
    if err!=nil {
        println("Command Error!", err.Error())
        return
    }
    fmt.Println(string(out))
}
```

正规一点的用来处理标准输入和输出的示例如下：



```
package main

import (
    "strings"
    "bytes"
    "fmt"
    "log"
    "os/exec"
)

func main() {
    cmd := exec.Command("tr", "a-z", "A-Z")
    cmd.Stdin = strings.NewReader("some input")
    var out bytes.Buffer
    cmd.Stdout = &out
    err := cmd.Run()
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("in all caps: %q\n", out.String())
}
```

#### 命令行参数


Go语言中处理命令行参数很简单：(使用os的Args就可以了)



```
func main() {
    args := os.Args
    fmt.Println(args) //带执行文件的
    fmt.Println(args[1:]) //不带执行文件的
}
```

在Windows下，如果运行结果如下：


`C:\Projects\Go>go run args.go aaa bbb ccc ddd  

[C:\Users\haoel\AppData\Local\Temp\go-build742679827\command-line-arguments_  

obj\a.out.exe aaa bbb ccc ddd]  

[aaa bbb ccc ddd]`


那么，如果我们要搞出一些像 mysql -uRoot -hLocalhost -pPwd 或是像 cc -O3 -Wall -o a a.c 这样的命令行参数我们怎么办？Go提供了一个package叫flag可以容易地做到这一点



```
package main
import "flag"
import "fmt"

func main() {

    //第一个参数是“参数名”，第二个是“默认值”，第三个是“说明”。返回的是指针
    host := flag.String("host", "coolshell.cn", "a host name ")
    port := flag.Int("port", 80, "a port number")
    debug := flag.Bool("d", false, "enable/disable debug mode")

    //正式开始Parse命令行参数
    flag.Parse()

    fmt.Println("host:", *host)
    fmt.Println("port:", *port)
    fmt.Println("debug:", *debug)
}
```

执行起来会是这个样子：



```
#如果没有指定参数名，则使用默认值
$ go run flagtest.go
host: coolshell.cn
port: 80
debug: false

#指定了参数名后的情况
$ go run flagtest.go -host=localhost -port=22 -d
host: localhost
port: 22
debug: true

#用法出错了（如：使用了不支持的参数，参数没有=）
$ go build flagtest.go
$ ./flagtest -debug -host localhost -port=22
flag provided but not defined: -debug
Usage of flagtest:
  -d=false: enable/disable debug mode
  -host="coolshell.cn": a host name
  -port=80: a port number
exit status 2

```

感觉还是挺不错的吧。


#### 一个简单的HTTP Server


代码胜过千言万语。呵呵。这个小程序让我又找回以前用C写CGI的时光了。（Go的官方文档是《**[Writing Web Applications](https://golang.org/doc/articles/wiki/)**》）



```
package main

import (
    "fmt"
    "net/http"
    "io/ioutil"
    "path/filepath"
)

const http_root = "/home/haoel/coolshell.cn/"

func main() {
    http.HandleFunc("/", rootHandler)
    http.HandleFunc("/view/", viewHandler)
    http.HandleFunc("/html/", htmlHandler)

    http.ListenAndServe(":8080", nil)
}

//读取一些HTTP的头
func rootHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "rootHandler: %s\n", r.URL.Path)
    fmt.Fprintf(w, "URL: %s\n", r.URL)
    fmt.Fprintf(w, "Method: %s\n", r.Method)
    fmt.Fprintf(w, "RequestURI: %s\n", r.RequestURI )
    fmt.Fprintf(w, "Proto: %s\n", r.Proto)
    fmt.Fprintf(w, "HOST: %s\n", r.Host) 
}

//特别的URL处理
func viewHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "viewHandler: %s", r.URL.Path)
}

//一个静态网页的服务示例。（在http_root的html目录下）
func htmlHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Printf("htmlHandler: %s\n", r.URL.Path)
    
    filename := http_root + r.URL.Path
    fileext := filepath.Ext(filename)

    content, err := ioutil.ReadFile(filename)
    if err != nil {
        fmt.Printf("   404 Not Found!\n")
        w.WriteHeader(http.StatusNotFound)
        return
    }
    
    var contype string
    switch fileext {
        case ".html", "htm":
            contype = "text/html"
        case ".css":
            contype = "text/css"
        case ".js":
            contype = "application/javascript"
        case ".png":
            contype = "image/png"
        case ".jpg", ".jpeg":
            contype = "image/jpeg"
        case ".gif":
            contype = "image/gif"
        default: 
            contype = "text/plain"
    }
    fmt.Printf("ext %s, ct = %s\n", fileext, contype)
    
    w.Header().Set("Content-Type", contype)
    fmt.Fprintf(w, "%s", content)
    
}
```

Go的功能库有很多，大家自己慢慢看吧。**我再吐个槽——Go的文档真不好读。例子太少了**。


先说这么多吧。这是我周末两天学Go语言学到的东西，写得太仓促了，而且还有一些东西理解不到位，还大家请指正！


（全文完）


