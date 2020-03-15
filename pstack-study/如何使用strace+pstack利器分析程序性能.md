如何使用strace+pstack利器分析程序性能
原创tycoon1988 最后发布于2014-09-03 18:25:29 阅读数 1872  收藏
展开
如何使用strace+pstack利器分析程序性能

http://www.cnblogs.com/bangerlee/archive/2012/04/30/2476190.html

http://www.cnblogs.com/bangerlee/archive/2012/02/20/2356818.html 

引言

有时我们需要对程序进行优化、减少程序响应时间。除了一段段地对代码进行时间复杂度分析，我们还有更便捷的方法吗？

若能直接找到影响程序运行时间的函数调用，再有针对地对相关函数进行代码分析和优化，那相比漫无目的地看代码，效率就高多了。

将strace和pstack工具结合起来使用，就可以达到以上目的。strace跟踪程序使用的底层系统调用，可输出系统调用被执行的时间点以及各个调用耗时；pstack工具对指定PID的进程输出函数调用栈。

下面我们通过一个简单的消息收发程序，说明使用strace、pstack进行程序分析的具体方法。

程序说明
该程序是一个简单的socket程序，由server/client组成。server端监听某端口，等待client的连接，client连接server后定时向server发送消息，server每接收一条消息后向client发送响应消息。程序server与client交互如下图示：



在程序运行起来之后，发现server接收到client的submit消息之后，需要较长时间才发出resp响应。通过tcpdump抓包发现，time2与time1的时间间隔在1s左右：




由上初步分析可知，消息响应慢是server端程序问题。下面我们来看如何使用strace和pstack分析server端程序响应慢的原因。

 

strace查看系统调用
首先我们拉起server/client程序，并使用strace对server进程进行跟踪：

# ps -elf | grep server | grep -v grep
0 S root 16739 22642 0 76 0 - 634 1024 14:26 pts/2 00:00:00 ./server
# strace -o server.strace -Ttt -p 16739
Process 16739 attached - interrupt to quit
 

稍等一段时间之后，我们将strace停掉， server.strace文件中有以下输出：

14:46:39.741366 select(8, [3 4], NULL, NULL, {1, 0}) = 1 (in [4], left {0, 1648}) <0.998415>
14:46:40.739965 recvfrom(4, "hello", 6, 0, NULL, NULL) = 5 <0.000068>
14:46:40.740241 write(1, "hello\n", 6)  = 6 <0.000066>
14:46:40.740414 rt_sigprocmask(SIG_BLOCK, [CHLD], [], 8) = 0 <0.000046>
14:46:40.740565 rt_sigaction(SIGCHLD, NULL, {SIG_DFL, [], 0}, 8) = 0 <0.000048>
14:46:40.740715 rt_sigprocmask(SIG_SETMASK, [], NULL, 8) = 0 <0.000046>
14:46:40.740853 nanosleep({1, 0}, {1, 0}) = 0 <1.000276>
14:46:41.741284 sendto(4, "hello\0", 6, 0, NULL, 0) = 6 <0.000111>
可以看到server接收数据之后(对应recvfrom调用)，经过1s左右时间将消息发出(对应sendto调用)，从响应时间看，与抓包的结果吻合。又可以看出nanosleep系统调用耗费了1s时间。

因而可以断定响应延时由nanosleep对应的函数调用造成。

那具体是哪一个函数调用呢？在strace输出结果中并不能找到答案，因其输出显示都是系统调用，要显示程序中函数调用栈信息，就轮到pstack上场了。

 

pstack查看函数堆栈
pstack是一个脚本工具，其核心实现就是使用了gdb以及thread apply all bt命令，下面我们使用pstack查看server进程函数堆栈：

# sh pstack.sh 16739
#0 0x00002ba1f8152650 in __nanosleep_nocancel () from /lib64/libc.so.6
#1 0x00002ba1f8152489 in sleep () from /lib64/libc.so.6
#2 0x00000000004007bb in ha_ha ()
#3 0x0000000000400a53 in main ()

从以上信息可以看出，函数调用关系为：main->ha_ha->sleep，因而我们可以找到ha_ha函数进行分析和优化修改。

小结
本文通过一个server/client程序事例，说明了使用strace和pstack分析响应延时的方法。

由最初server端响应慢现象，到使用strace跟踪出具体耗时的系统调用，再到使用pstack查到程序中具体的耗时函数，一步步找到了影响程序运行时间的程序代码。

更多地了解底层，从操作系统层面着手，更有助于程序性能分析与优化。

 

本文中使用的server/client程序和pstack脚本可从这里下载。

strace 通用的完整用法    ： 

strace -o output.txt -T -tt -e trace=all -p 10423
上面的含义是 跟踪28979进程的所有系统调用（-e trace=all），并统计系统调用的花费时间，以及开始时间（并以可视化的时分秒格式显示），最后将记录结果存在 

output.txt文件里面。

限制strace只跟踪特定的系统调用     : 

如果你已经知道你要找什么，你可以让strace只跟踪一些类型的系统调用。例如，在nginx执行程序时，你需要监视的系统调用epoll_wait。

让strace只记录epoll_wait的调用用这个命令：

strace -f -o epoll-strace.txt -e epoll_wait -p 10423
命令strace跟踪的是系统调用，对于nginx本身的函数调用关系无法给出更为明朗的信息，如果我们发现nginx当前运行不正常，想知道nginx当前内部到底在执行什么函数，

那么命令pstack就是一个非常方便实用的工具。pstack的使用也非常简单，后面跟进程id即可，比如在无客户端请求的情况下，nginx阻塞在epoll_wait系统调用处，此时

利用pstack查看到的nginx函数调用堆栈关系如下：




从main()函数到epoll_wait()函数的调用关系一目了然，和在gdb内看到的堆栈信息一样。我们可以利用此进行分析优化等。

 
————————————————
版权声明：本文为CSDN博主「tycoon1988」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/tycoon1988/article/details/39030985