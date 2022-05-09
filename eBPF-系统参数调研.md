ubuntu系统

## bpftool
1. 查询当前系统支持的程序类型: bpftool feature probe | grep program_type  
2. 给libbpf生成skel头文件：bpftool gen skeleton $@.bpf.o > $@.skel.h
3. bpftool map show
4. bpftool prog show 
5. 查看具体某一个eBPF 程序的指令： `bpftool prog dump xlated id <prog-id>`
6. 查看JIT镜像：`bpftool prog dump jited id <prog-id>`
7. 将常规的 BPF 指令关联到 opcodes: 
  ```
  bpftool prog dump xlated id <prog-id> opcodes
  bpftool prog dump jited id <prog-id> opcodes
  ```

## eBPF挂载方式
 1. tracepoints:	A Linux kernel technology for providing static tracing.
 2. kprobes:	A Linux kernel technology for providing dynamic tracing of kernel functions.
 3. uprobes:	A Linux kernel technology for providing dynamic tracing of user-level functions.
 4. USDT:	User Statically-Defined Tracing: static tracing points for user-level software. Some applications support USDT.

## eBPF程序分类
### 跟踪类
跟踪即从内核和程序的运行状态中提取跟踪信息，来了解当前系统正在发生什么。  
BPF_PROG_TYPE_KPROBE
BPF_PROG_TYPE_TRACEPOINT
BPF_PROG_TYEP_PERF_EVENT
BPF_PROG_TYPE_RAW_TRACEPOINT
BPF_PROG_TYEP_RAW_TRACEPOINT_WRITABLE
BPF_PROG_TYPE_RAW_TRACEING

### 网络类
网络，即对网络数据包进行过滤和处理，以便了解和控制网络数据包的收发过程。  
1. XDP程序
BPF_PROG_TYPE_XDP
根据网卡和网卡驱动是否原生支持 XDP 程序，XDP 运行模式可以分为下面这三种：
 - 通用模式。它不需要网卡和网卡驱动的支持，XDP 程序像常规的网络协议栈一样运行在内核中，性能相对较差，一般用于测试；
 - 原生模式。它需要网卡驱动程序的支持，XDP 程序在网卡驱动程序的早期路径运行；
 - 卸载模式。它需要网卡固件支持 XDP 卸载，XDP 程序直接运行在网卡上，而不再需要消耗主机的 CPU 资源，具有最好的性能。
 
2. TC程序
TC 程序的类型定义为 BPF_PROG_TYPE_SCHED_CLS 和 BPF_PROG_TYPE_SCHED_ACT，分别作为 Linux 流量控制 的分类器和执行器。

3. 套接字程序
套接字程序用于过滤、观测或重定向套接字网络包，具体的种类也比较丰富。根据类型的不同，套接字 eBPF 程序可以挂载到套接字（socket）、控制组（cgroup ）以及网络命名空间（netns）等各个位置。

4. cgroup程序
cgroup 程序用于对 cgroup 内所有进程的网络过滤、套接字选项以及转发等进行动态控制，它最典型的应用场景是对容器中运行的多个进程进行网络控制。 

### 其他
除跟踪和网络之外的其他类型，包括安全控制、BPF 扩展等等。

## BCC
sudo apt-get install -y make clang llvm libelf-dev libbpf-dev bpfcc-tools libbpfcc-dev linux-tools-$(uname -r) linux-headers-$(uname -r)# For RHEL8.2+sudo yum install libbpf-devel make clang llvm elfutils-libelf-devel bpftool bcc-tools bcc-devel

### 参数获取
因为 BCC 把所有参数都放到了  args  中，你可以使用  args->argv  来访问参数列表：`const char **argv = (const char **)(args->argv);`

## bpftrace
sudo apt-get install -y bpftrace

1. 查询所有内核插桩和跟踪点: sudo bpftrace -l  

2. 使用通配符查询所有的系统调用跟踪点: sudo bpftrace -l 'tracepoint:syscalls:*'  

3. 使用通配符查询所有名字包含"execve"的跟踪点: sudo bpftrace -l '*execve*'  

4. 查询execve入口参数格式:
   ```
   $ sudo bpftrace -lv tracepoint:syscalls:sys_enter_execve
   ```
   
5. 查询execve返回值格式:  
   ```
   $ sudo bpftrace -lv tracepoint:syscalls:sys_exit_execve
   ```   
   
6. 使用 bpftrace 来跟踪短时进程
  ```
  $ sudo bpftrace -e 'tracepoint:syscalls:sys_enter_execve,tracepoint:syscalls:sys_enter_execveat { printf("%-6d %-8s", pid, comm); join(args->argv);}'   
  ```  
  这个命令中的具体内容含义如下：
   - bpftrace -e  表示直接从后面的字符串参数中读入 bpftrace 程序（除此之外，它还支持从文件中读入 bpftrace 程序）；
   - tracepoint:syscalls:sys_enter_execve,tracepoint:syscalls:sys_enter_execveat  表示用逗号分隔的多个跟踪点，其后的中括号表示跟踪点的处理函数；
   - printf()  表示向终端中打印字符串，其用法类似于 C 语言中的  printf()  函数；
   - pid  和  comm  是 bpftrace 内置的变量，分别表示进程 PID 和进程名称（你可以在其官方文档中找到其他的内置变量）；
   - join(args->argv)  表示把字符串数组格式的参数用空格拼接起来，再打印到终端中。对于跟踪点来说，你可以使用  args->参数名  的方式直接读取参数（比如这里的  args->argv  就是读取系统调用中的  argv  参数）。  
   在另一个终端中执行  ls  等命令，然后你会在第一个终端中看到输出



## linux基础知识   
1. fork 和 execve
  (1) fork系统调用创建一个几乎和父进程完全一样的子进程
      - fork函数其与其它系统调用不同之处在于其：一次调用，两次返回。在父进程中被调用一次。父进程中fork返回子进程的ID，而在子进程中，返回0 ，基于返回值即可确定是在父进程还是子进程中.
	  - fork创建完子进程后，两者并发执行。内核依据调度程序交替的执行父子进程中的逻辑流指令。
	  - 相同但独立的地址空间，fork调用创建子进程之初，子进程和父进程的地址空间是相同的。也就意味着两个进程可以访问相同的地址空间内容。相同的代码段，相同的全局变量等。但一旦其中一个进程需要修改地址空间的内容，将触发写时复制
	  - 共享打开的文件描述符。子进程共享父进程打开的文件描述符，其通过引用到父进程描述符task_struct中的打开文件描述符，从而父子进程可以共享文件。
  (2) execve系统调用在当前进程上下文中加载并运行一个新的程序. fork和execve都是创建新的逻辑执行流，它们的不同之处在于fork创建一个全新的进程，而execve只是在当前进程中加载新的程序。execve是用于运行用户程序(a.out)或shell脚本的函数,是linux编程中常用的一个系统调用类函数。在linux命令行下运行用户程序本质其实就是执行execve系统调用。 
  
## libbpf 

### libbpf 开发 eBPF 程序的四个步骤
  1. 使用 bpftool 生成内核数据结构定义头文件vmlinux.h。BTF 开启后，你可以在系统中找到  /sys/kernel/btf/vmlinux  这个文件，bpftool 正是从它生成了内核数据结构头文件。
  2. 开发 eBPF 程序部分。为了方便后续通过统一的 Makefile 编译，eBPF 程序的源码文件一般命名为  <程序名>.bpf.c。
  3. 编译 eBPF 程序(<程序名>.bpf.c)为字节码，然后再调用  bpftool gen skeleton  为 eBPF 字节码生成脚手架头文件（Skeleton Header）。这个头文件包含了 eBPF 字节码以及相关的加载、挂载和卸载函数，可在用户态程序中直接调用。
  4. 最后就是用户态程序引入上一步生成的头文件，开发用户态程序(<程序名>.c)，包括 eBPF 程序加载、挂载到内核函数和跟踪点，以及通过 BPF 映射获取和打印执行结果等。  

### 查询手册
  1. 用bpftool生成的vmlinux.h文件中有内核预定义的数据结构：bpftool gen skeleton $@.bpf.o > $@.skel.h
  2. 查询当前系统支持哪些映射类型：bpftool feature probe | grep map_type
  3. 查询当前系统支持的辅助函数列表： bpftool feature probe , 命令行中执行 man bpf-helpers ，或者参考内核头文件 /usr/src/linux-headers-5.11.0-16/include/uapi/linux/bpf.h ，来查看它们的详细定义和使用说明。内核代码位置：/usr/src/linux-headers-5.11.0-49
  4. 另外还有两个头文件预定义了一些数据结构：/usr/include/bpf/libbpf.h
  5. https://man7.org/linux/man-pages/man2/bpf.2.html#EXAMPLES
  6. https://elixir.bootlin.com/linux/v5.13/source/include/linux/syscalls.h
  7. 直接在linux终端输入man xxx, 查询具体的系统调用, 例如： `man openat`

### 常用参数查询
#### 通用参数  
  1. 获取当前进程名称： bpf_get_current_comm(&event->comm, sizeof(event->comm));
  2. 获取当前PID：u64 id = bpf_get_current_pid_tgid();	pid_t pid = (pid_t) id;
#### 系统调用函数的参数
   一般用bpf_probe_read_* 系列函数去读取

  
   
## 几个ebpf编程示例网站 
 1. https://github.com/iovisor/bcc/tree/master/libbpf-tools
 2. https://ebpf.io/
 3. https://github.com/libbpf/libbpf
 4. https://github.com/feiskyer/ebpf-apps


 ## 开发一个系统调用
 TODO:

 ## 开源项目
 ### falco
  1. 官方文档：<https://falco.org/docs/rules/supported-events/> 这是支持的系统调用事件列表
