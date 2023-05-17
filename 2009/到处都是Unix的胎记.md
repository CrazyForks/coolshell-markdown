# 到处都是Unix的胎记
>date: 2009-10-11T18:01:06+08:00


一说起Unix编程，不必多说，最著名的系统调用就是fork，pipe，exec，kill或是socket了（[`fork(2)`](http://www.kernel.org/doc/man-pages/online/pages/man2/fork.2.html), [`execve(2)`](http://www.kernel.org/doc/man-pages/online/pages/man2/execve.2.html), [`pipe(2)`](http://www.kernel.org/doc/man-pages/online/pages/man2/pipe.2.html), [`socketpair(2)`](http://www.kernel.org/doc/man-pages/online/pages/man2/socketpair.2.html), [`select(2)`](http://www.kernel.org/doc/man-pages/online/pages/man2/select.2.html), [`kill(2)`](http://www.kernel.org/doc/man-pages/online/pages/man2/kill.2.html), [`sigaction(2)`](http://www.kernel.org/doc/man-pages/online/pages/man2/sigaction.2.html)）这些系统调用都像是Unix编程的胎记或签名一样，表明着它来自于Unix。


下面这篇文章，将向大家展示Unix下最经典的socket的编程例子——使用fork + socket来创建一个TCP/IP的服务程序。这个编程模式很简单，首先是创建Socket，然后把其绑定在某个IP和Port上上侦听连接，接下来的一般做法是使用一个fork创建一个client服务进程再加上一个死循环用于处理和client的交互。这个模式是Unix下最经典的Socket编程例子。


下面，让我们看看用C，Ruby，Python，Perl，PHP和Haskell来实现这一例子，你会发现这些例子中的Unix的胎记。如果你想知道这些例子中的技术细节，那么，向你推荐两本经典书——《Unix高级环境编程》和《Unix网络编程》。





目录



* [C语言](#C%E8%AF%AD%E8%A8%80 "C语言")
* [Ruby](#Ruby "Ruby")
* [Python](#Python "Python")
* [Perl](#Perl "Perl")
* [PHP](#PHP "PHP")
* [Haskell](#Haskell "Haskell")

#### C语言


我们先来看一下经典的C是怎么实现的。



```
/**
 * A simple preforking echo server in C.
 *
 * Building:
 *
 * $ gcc -Wall -o echo echo.c
 *
 * Usage:
 *
 * $ ./echo
 *
 *   ~ then in another terminal ... ~
 *
 * $ echo 'Hello, world!' | nc localhost 4242
 *
 */

#include <unistd.h> /* fork, close */
#include <stdlib.h> /* exit */
#include <string.h> /* strlen */
#include <stdio.h> /* perror, fdopen, fgets */
#include <sys/socket.h>
#include <sys/wait.h> /* waitpid */
#include <netdb.h> /* getaddrinfo */

#define die(msg) do { perror(msg); exit(EXIT_FAILURE); } while (0)

#define PORT "4242"
#define NUM_CHILDREN 3

#define MAXLEN 1024

int readline(int fd, char *buf, int maxlen); // forward declaration

int
main(int argc, char** argv)
{
    int i, n, sockfd, clientfd;
    int yes = 1; // used in setsockopt(2)
    struct addrinfo *ai;
    struct sockaddr_in *client;
    socklen_t client_t;
    pid_t cpid; // child pid
    char line[MAXLEN];
    char cpid_s[32];
    char welcome[32];

    /* Create a socket and get its file descriptor -- socket(2) */
    sockfd = socket(AF_INET, SOCK_STREAM, 0);
    if (sockfd == -1) {
    die("Couldn't create a socket");
    }

    /* Prevents those dreaded "Address already in use" errors */
    if (setsockopt(sockfd, SOL_SOCKET, SO_REUSEADDR, (const void *)&yes, sizeof(int)) == -1) {
    die("Couldn't setsockopt");
    }

    /* Fill the address info struct (host + port) -- getaddrinfo(3) */
    if (getaddrinfo(NULL, PORT, NULL, &ai) != 0) {
    die("Couldn't get address");
    }

    /* Assign address to this socket's fd */
    if (bind(sockfd, ai->ai_addr, ai->ai_addrlen) != 0) {
    die("Couldn't bind socket to address");
    }

    /* Free the memory used by our address info struct */
    freeaddrinfo(ai);

    /* Mark this socket as able to accept incoming connections */
    if (listen(sockfd, 10) == -1) {
    die("Couldn't make socket listen");
    }

    /* Fork you some child processes. */
    for (i = 0; i < NUM_CHILDREN; i++) {
    cpid = fork();
    if (cpid == -1) {
        die("Couldn't fork");
    }

    if (cpid == 0) { // We're in the child ...
        for (;;) { // Run forever ...
        /* Necessary initialization for accept(2) */
        client_t = sizeof client;

        /* Blocks! */
        clientfd = accept(sockfd, (struct sockaddr *)&client, &client_t);
        if (clientfd == -1) {
            die("Couldn't accept a connection");
        }

        /* Send a welcome message/prompt */
        bzero(cpid_s, 32);
        bzero(welcome, 32);
        sprintf(cpid_s, "%d", getpid());
        sprintf(welcome, "Child %s echo> ", cpid_s);
        send(clientfd, welcome, strlen(welcome), 0);

        /* Read a line from the client socket ... */
        n = readline(clientfd, line, MAXLEN);
        if (n == -1) {
            die("Couldn't read line from connection");
        }

        /* ... and echo it back */
        send(clientfd, line, n, 0);

        /* Clean up the client socket */
        close(clientfd);
        }
    }
    }

    /* Sit back and wait for all child processes to exit */
    while (waitpid(-1, NULL, 0) > 0);

    /* Close up our socket */
    close(sockfd);

    return 0;
}

/**
 * Simple utility function that reads a line from a file descriptor fd,
 * up to maxlen bytes -- ripped from Unix Network Programming, Stevens.
 */
int
readline(int fd, char *buf, int maxlen)
{
    int n, rc;
    char c;

    for (n = 1; n < maxlen; n++) {
    if ((rc = read(fd, &c, 1)) == 1) {
        *buf++ = c;
        if (c == '\n')
        break;
    } else if (rc == 0) {
        if (n == 1)
        return 0; // EOF, no data read
        else
        break; // EOF, read some data
    } else
        return -1; // error
    }

    *buf = '\0'; // null-terminate
    return n;
}

```

#### Ruby


下面是Ruby，你可以看到其中的fork



```
# simple preforking echo server in Ruby
require 'socket'

# Create a socket, bind it to localhost:4242, and start listening.
# Runs once in the parent; all forked children inherit the socket's
# file descriptor.
acceptor = Socket.new(Socket::AF_INET, Socket::SOCK_STREAM, 0)
address = Socket.pack_sockaddr_in(4242, 'localhost')
acceptor.bind(address)
acceptor.listen(10)

# Close the socket when we exit the parent or any child process. This
# only closes the file descriptor in the calling process, it does not
# take the socket out of the listening state (until the last fd is
# closed).
#
# The trap is guaranteed to happen, and guaranteed to happen only
# once, right before the process exits for any reason (unless
# it's terminated with a SIGKILL).
trap('EXIT') { acceptor.close }

# Fork you some child processes. In the parent, the call to fork
# returns immediately with the pid of the child process; fork never
# returns in the child because we exit at the end of the block.
3.times do
  fork do
    # now we're in the child process; trap (Ctrl-C) interrupts and
    # exit immediately instead of dumping stack to stderr.
    trap('INT') { exit }

    puts "child #$$ accepting on shared socket (localhost:4242)"
    loop {
      # This is where the magic happens. accept(2) blocks until a
      # new connection is ready to be dequeued.
      socket, addr = acceptor.accept
      socket.write "child #$$ echo> "
      socket.flush
      message = socket.gets
      socket.write message
      socket.close
      puts "child #$$ echo'd: '#{message.strip}'"
    }
    exit
  end
end

# Trap (Ctrl-C) interrupts, write a note, and exit immediately
# in parent. This trap is not inherited by the forks because it
# runs after forking has commenced.
trap('INT') { puts "\nbailing" ; exit }

# Sit back and wait for all child processes to exit.
Process.waitall


```

#### Python



```
"""
Simple preforking echo server in Python.
"""

import os
import sys
import socket

# Create a socket, bind it to localhost:4242, and start
# listening. Runs once in the parent; all forked children
# inherit the socket's file descriptor.
acceptor = socket.socket()
acceptor.bind(('localhost', 4242))
acceptor.listen(10)

# Ryan's Ruby code here traps EXIT and closes the socket. This
# isn't required in Python; the socket will be closed when the
# socket object gets garbage collected.

# Fork you some child processes. In the parent, the call to
# fork returns immediately with the pid of the child process;
# fork never returns in the child because we exit at the end
# of the block.
for i in range(3):
    pid = os.fork()

    # os.fork() returns 0 in the child process and the child's
    # process id in the parent. So if pid == 0 then we're in
    # the child process.
    if pid == 0:
        # now we're in the child process; trap (Ctrl-C)
        # interrupts by catching KeyboardInterrupt) and exit
        # immediately instead of dumping stack to stderr.
        childpid = os.getpid()
        print "Child %s listening on localhost:4242" % childpid
        try:
            while 1:
                # This is where the magic happens. accept(2)
                # blocks until a new connection is ready to be
                # dequeued.
                conn, addr = acceptor.accept()

                # For easier use, turn the socket connection
                # into a file-like object.
                flo = conn.makefile()
                flo.write('Child %s echo> ' % childpid)
                flo.flush()
                message = flo.readline()
                flo.write(message)
                flo.close()
                conn.close()
                print "Child %s echo'd: %r" % \
                          (childpid, message.strip())
        except KeyboardInterrupt:
            sys.exit()

# Sit back and wait for all child processes to exit.
#
# Trap interrupts, write a note, and exit immediately in
# parent. This trap is not inherited by the forks because it
# runs after forking has commenced.
try:
    os.waitpid(-1, 0)
except KeyboardInterrupt:
    print "\nbailing"
    sys.exit()

```

#### Perl



```
#!/usr/bin/perl
use 5.010;
use strict;

# simple preforking echo server in Perl
use Proc::Fork;
use IO::Socket::INET;

sub strip { s/\A\s+//, s/\s+\z// for my @r = @_; @r }

# Create a socket, bind it to localhost:4242, and start listening.
# Runs once in the parent; all forked children inherit the socket's
# file descriptor.
my $acceptor = IO::Socket::INET->new(
    LocalPort => 4242,
    Reuse     => 1,
    Listen    => 10,
) or die "Couln't start server: $!\n";

# Close the socket when we exit the parent or any child process. This
# only closes the file descriptor in the calling process, it does not
# take the socket out of the listening state (until the last fd is
# closed).
END { $acceptor->close }

# Fork you some child processes. The code after the run_fork block runs
# in all process, but because the child block ends in an exit call, only
# the parent executes the rest of the program. If a parent block were
# specified here, it would be invoked in the parent only, and passed the
# PID of the child process.
for ( 1 .. 3 ) {
    run_fork { child {
        while (1) {
            my $socket = $acceptor->accept;
            $socket->printflush( "child $$ echo> " );
            my $message = $socket->getline;
            $socket->print( $message );
            $socket->close;
            say "child $$ echo'd: '${\strip $message}'";
        }
        exit;
    } }
}

# Trap (Ctrl-C) interrupts, write a note, and exit immediately
# in parent. This trap is not inherited by the forks because it
# runs after forking has commenced.
$SIG{ 'INT' } = sub { print "bailing\n"; exit };

# Sit back and wait for all child processes to exit.
1 while 0 < waitpid -1, 0;

```

#### PHP



```
<?
/*
Simple preforking echo server in PHP.
Russell Beattie (russellbeattie.com)
*/

/* Allow the script to hang around waiting for connections. */
set_time_limit(0);

# Create a socket, bind it to localhost:4242, and start
# listening. Runs once in the parent; all forked children
# inherit the socket's file descriptor.
$socket = socket_create(AF_INET, SOCK_STREAM, SOL_TCP);
socket_bind($socket,'localhost', 4242);
socket_listen($socket, 10);

pcntl_signal(SIGTERM, 'shutdown');
pcntl_signal(SIGINT, 'shutdown');

function shutdown($signal){
    global $socket;
    socket_close($socket);
    exit();
}
# Fork you some child processes. In the parent, the call to
# fork returns immediately with the pid of the child process;
# fork never returns in the child because we exit at the end
# of the block.
for($x = 1; $x <= 3; $x++){
   
    $pid = pcntl_fork();
   
    # pcntl_fork() returns 0 in the child process and the child's
    # process id in the parent. So if $pid == 0 then we're in
    # the child process.
    if($pid == 0){

        $childpid = posix_getpid();
       
        echo "Child $childpid listening on localhost:4242 \n";

        while(true){
            # This is where the magic happens. accept(2)
            # blocks until a new connection is ready to be
            # dequeued.
            $conn = socket_accept($socket);

            $message = socket_read($conn,1000,PHP_NORMAL_READ);
           
            socket_write($conn, "Child $childpid echo> $message");
       
            socket_close($conn);
       
            echo "Child $childpid echo'd: $message \n";
       
        }

    }
}
#
# Trap interrupts, write a note, and exit immediately in
# parent. This trap is not inherited by the forks because it
# runs after forking has commenced.
try{

    pcntl_waitpid(-1, $status);

} catch (Exception $e) {

    echo "bailing \n";
    exit();

}
```

#### Haskell



```
import Network
import Prelude hiding ((-))
import Control.Monad
import System.IO
import Control.Applicative
import System.Posix
import System.Exit
import System.Posix.Signals

main :: IO ()
main = with =<< (listenOn - PortNumber 4242) where

  with socket = do
    replicateM 3 - forkProcess work
    wait

    where
    work = do
      installHandler sigINT (Catch trap_int) Nothing
      pid <- show <$> getProcessID
      puts - "child " ++ pid ++ " accepting on shared socket (localhost:4242)"
     
      forever - do
        (h, _, _) <- accept socket

        let write   = hPutStr h
            flush   = hFlush h
            getline = hGetLine h
            close   = hClose h

        write - "child " ++ pid ++ " echo> "
        flush
        message <- getline
        write - message ++ "\n"
        puts - "child " ++ pid ++ " echo'd: '" ++ message ++ "'"
        close

    wait = forever - do
      ( const () <$> getAnyProcessStatus True True  ) catch const trap_exit

    trap_int = exitImmediately ExitSuccess

    trap_exit = do
      puts "\nbailing"
      sClose socket
      exitSuccess

    puts = putStrLn

  (-) = ($)
  infixr 0 -


```

如果你知道更多的，请你告诉我们。（全文完）


