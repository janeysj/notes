ubuntu系统

## bpftool
1. 查询当前系统支持的程序类型: bpftool feature probe | grep program_type  
2. 给libbpf生成skel头文件：bpftool gen skeleton $@.bpf.o > $@.skel.h
3. bpftool map show; bpftool map dump name syscall_data_ma; bpftool map dump id 123 
4. bpftool prog show 
5. 查看具体某一个eBPF 程序的指令： `bpftool prog dump xlated id <prog-id>`
6. 查看JIT镜像：`bpftool prog dump jited id <prog-id>`
7. 将常规的 BPF 指令关联到 opcodes: 
  ```
  bpftool prog dump xlated id <prog-id> opcodes
  bpftool prog dump jited id <prog-id> opcodes
  ```

## eBPF挂载方式
eBPF程序是通过ioctl挂载的. perf_event_open打开一种类型的挂载点然后把bpf程序用编号索引的方式挂载到这个点。
 1. tracepoints:	A Linux kernel technology for providing static tracing. Tracepoint 是在内核中固定的 hook 点，这些 hook 点需要在内核源代码中预先写好,并不是在所有的函数中都有 tracepoint.
 2. kprobes:	A Linux kernel technology for providing dynamic tracing of kernel functions. 内核函数中没有 hook 点要通过 Linux kprobe 机制来加载 probe 函数。
 3. uprobes:	A Linux kernel technology for providing dynamic tracing of user-level functions.
 4. USDT:	User Statically-Defined Tracing: static tracing points for user-level software. Some applications support USDT.
 ```
 # 查询uprobe用户进程的跟踪点
 # uprobe 是基于文件的。当文件中的某个函数被跟踪时，除非对进程 PID 进行了过滤，默认所有使用到这个文件的进程都会被插桩。
 bpftrace -l 'uprobe:/usr/lib/x86_64-linux-gnu/libc.so.6:*'

 # 查询USDT用户进程的跟踪点
 bpftrace -l 'usdt:/usr/lib/x86_64-linux-gnu/libc.so.6:*'   


 用户进程的跟踪，本质上是通过断点去执行 uprobe 处理程序。虽然内核社区已经对 BPF 做了很多的性能调优，跟踪用户态函数（特别是锁争用、内存分配之类的高频函数）还是有可能带来很大的性能开销。因此，我们在使用 uprobe 时，应该尽量避免跟踪高频函数。
 ```

## eBPF程序分类
### 跟踪类
跟踪即从内核和程序的运行状态中提取跟踪信息，来了解当前系统正在发生什么。  
- BPF_PROG_TYPE_KPROBE
- BPF_PROG_TYPE_TRACEPOINT
- BPF_PROG_TYEP_PERF_EVENT
- BPF_PROG_TYPE_RAW_TRACEPOINT
- BPF_PROG_TYEP_RAW_TRACEPOINT_WRITABLE
- BPF_PROG_TYPE_RAW_TRACEING

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
官方指导文档： https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md 这里对tracepoint,kprobe,uprobe,USDT等做了详细的阐述   
脚本化的eBPF工具，提供了上百个遍布Linux各个层面的工具。参考 https://github.com/iovisor/bcc/ 图示。  
https://github.com/iovisor/bcc/tree/master/libbpf-tools 提供了BCC脚本到libbpf二进制程序的转换方法。  
ubuntu安装: sudo apt-get install -y make clang llvm libelf-dev libbpf-dev bpfcc-tools libbpfcc-dev linux-tools-$(uname -r) linux-headers-$(uname -r)  
centos安装: sudo yum install libbpf-devel make clang llvm elfutils-libelf-devel bpftool bcc-tools bcc-devel  

### 参数获取
因为 BCC 把所有参数都放到了  args  中，你可以使用  args->argv  来访问参数列表：`const char **argv = (const char **)(args->argv);`

### 利用github上bcc的示例
- ucalls.py 查看系统调用，其他语言配置参数-l none.也可以猜测关键字到内核文件中去搜索
- stacksnoop.py 打印函数调用堆栈信息

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
虽然 BCC 竭尽全力地简化 BPF 程序开发人员的工作，但其“黑魔法” （使用 Clang 前端修改了用户编写的 BPF 程序）使得出现问题时，很难找到问题的所在以及解决方法。必须记住命名约定和自动生成的跟踪点结构 。且由于 libbcc 库内部集成了庞大的 LLVM/Clang 库，使其在使用过程中会遇到一些问题：
 1. 在每个工具启动时，都会占用较高的 CPU 和内存资源来编译 BPF 程序，在系统资源已经短缺的服务器上运行可能引起问题；
 2. 依赖于内核头文件包，必须将其安装在每个目标主机上。即便如此，如果需要内核中未 export 的内容，则需要手动将类型定义复制/粘贴到 BPF 代码中；
 3. 由于 BPF 程序是在运行时才编译，因此很多简单的编译错误只能在运行时检测到，影响开发体验。  
 利用libbpf编译出来的二进制程序可以跨内核运行，CO-RE 编译一次到处运行(不然就是每次运行都要编译一次)。（应该要求内核在4.1以上吧？ 且要求打开 CONFIG_DEBUG_INFO_BTF 这个编译选项）
 
### libbpf 开发 eBPF 程序的四个步骤
  1. 使用 bpftool 生成内核数据结构定义头文件vmlinux.h。BTF 开启后，你可以在系统中找到  /sys/kernel/btf/vmlinux  这个文件，bpftool 正是从它生成了内核数据结构头文件。`bpftool btf dump file /sys/kernel/btf/vmlinux format c > vmlinux.h`
  2. 开发 eBPF 程序部分。为了方便后续通过统一的 Makefile 编译，eBPF 程序的源码文件一般命名为  <程序名>.bpf.c。
  3. 编译 eBPF 程序(<程序名>.bpf.c)为字节码，然后再调用  bpftool gen skeleton  为 eBPF 字节码生成脚手架头文件（Skeleton Header）。这个头文件包含了 eBPF 字节码以及相关的加载、挂载和卸载函数，可在用户态程序中直接调用。
    ```
    clang -g -O2 -target bpf -D__TARGET_ARCH_x86_64 -I/usr/include/x86_64-linux-gnu -I. -c execsnoop.bpf.c -o execsnoop.bpf.o
    bpftool gen skeleton execsnoop.bpf.o > execsnoop.skel.h
    clang -g -O2 -Wall -I . -c execsnoop.c -o execsnoop.o
    clang -Wall -O2 -g execsnoop.o -static -lbpf -lelf -lz -o execsnoop
    ```
  4. 最后就是用户态程序引入上一步生成的头文件，开发用户态程序(<程序名>.c)，包括 eBPF 程序加载、挂载到内核函数和跟踪点，以及通过 BPF 映射获取和打印执行结果等。  

### 查询手册
  1. 用bpftool生成的vmlinux.h文件中有内核预定义的数据结构：bpftool gen skeleton $@.bpf.o > $@.skel.h
  2. 查询当前系统支持哪些映射类型：bpftool feature probe | grep map_type
  3. 查询当前系统支持的辅助函数列表(eg: bpf_probe_read)： bpftool feature probe , 命令行中执行 man bpf-helpers ，或者参考内核头文件 /usr/src/linux-headers-5.11.0-16/include/uapi/linux/bpf.h ，来查看它们的详细定义和使用说明。内核代码位置：/usr/src/linux-headers-5.11.0-49
  4. 另外还有两个头文件预定义了一些数据结构：/usr/include/bpf/libbpf.h
  5. https://man7.org/linux/man-pages/man2/bpf.2.html#EXAMPLES
  6. https://elixir.bootlin.com/linux/v5.13/source/include/linux/syscalls.h
  7. 直接在linux终端输入man xxx, 查询具体的系统调用, 例如： `man openat`

### perf 命令
  - <https://www.brendangregg.com/perf.html> 帮助理解 perf-event, perf_events is an event-oriented observability tool
  - 使用perf list获取性能事件: sudo perf list tracepoint  

### 常用参数查询
https://blogs.oracle.com/linux/post/bpf-a-tour-of-program-types
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
 5. https://blog.aquasec.com/libbpf-ebpf-programs

## ringbuf、perfbuf
https://nakryiko.com/posts/bpf-ringbuf/
   都是内核和用户空间之间有效地交换数据的方式
  1. perfbuf是每个cpu循环缓冲区的集合，ringbuf是多个CPU共享一个buf

 ## 开发一个系统调用
 TODO:

 ## 开源项目
 ### falco
  1. 官方文档：<https://falco.org/docs/rules/supported-events/> 这是支持的系统调用事件列表
  2. tracee
  3. tracee的libbpfgo
     https://pkg.go.dev/github.com/aquasecurity/tracee/libbpfgo
     https://github.com/aquasecurity/libbpfgo/tree/main/selftest/

## Linux 系统调用
 - https://www.tutorialspoint.com/unix_system_calls/
 - https://elixir.bootlin.com/
 - https://manpages.ubuntu.com/
 
### 非敏感函数
 - futex: 一个在Linux上实现锁定和构建高级抽象锁如信号量和POSIX互斥的基本工具
 - epoll_wait: system call waits for events on the epoll
 - io_getevents: 从AIO上下文中读取最后某段时间内的事件
 - prctl：给进程、线程命名

### 敏感函数
 - accept: 和建立连接相关，它抽取
 - getsockopt：get options on sockets
 - setsockopt：set options on sockets
 - stat: stats the file pointed to by path and fills in buf
 - lstat: 和stat类似，只不过用是stat到一个symbolic link
 - fstat: 和stat类似，只不过用是stat到fd指定的文件
 - unlinkat,unlink: unlink 只可以删除文件。 unlinkat 可以删除文件（默认）或文件夹（需要设置flags为AT_REMOVEDIR）  

 以下是不会单独出现的事件
 - close： 关闭fd.
 - open:
 - openat: 

 # 开发过程中的奇闻
 ## 奇奇怪怪的系统程序
 - dirname: dirname is a built-in command on Linux and Unix-like OSes; it is used to identify paths in shell scripts.
 - id: Print user and group information for each specified USER

 # eBPF对各类编程语言的用户空间挂载点
  - 第一类是编译型语言。 C、C++、Golang 等编译为机器码后再执行的编译型语言。这类编程语言开发的程序，通常会编译成 ELF 格式的二进制文件，包含了保存在寄存器或栈中的函数参数和返回值，因而可以直接通过二进制文件中的符号进行跟踪。  
  bpftrace -l 'uprobe:/usr/local/go/bin/go:*'   10470个uprobe挂载点; usdt 类型为0  
 
  - 第二类是解释型语言 Python、Bash、Ruby 等通过解释器语法分析之后再执行的解释型语言。这类编程语言开发的程序，无法直接从语言运行时的二进制文件中获取应用程序的调试信息，通常需要跟踪解释器的函数，再从其参数中获取应用程序的运行细节。  
  bpftrace -l 'uprobe:/usr/bin/python3:*' 1447个uprobe挂载点；usdt 类型为8个  
 
  - 第三类是即时编译型语言。 Java、.Net、JavaScript 等先编译为字节码，再由即时编译器（JIT）编译为机器码执行的即时编译型语言。同解释型语言类似，这类编程语言无法直接从语言运行时的二进制文件中获取应用程序的调试信息。跟踪 JIT 编程语言开发的程序是最困难的，因为 JIT 编译的状态只存在于内存中。	  
  bpftrace -l 'uprobe:/usr/bin/java:*' 输出为0；usdt 输出也为0  

# 问题：
1. ebpf maps是放在ebpf虚拟机的512M栈上的吗？
Ebpf源代码里通过 kernel/bpf/syscall.c kmalloc_node(array_map_alloc->bpf_map_area_alloc->__bpf_map_area_alloc) 到numa节点上申请空间，所以应该不是ebpf虚拟机上的栈空间吧？应该是内核空间的  
在eBPF中，maps是存储在内核空间中的数据结构，而不是存储在虚拟机中。eBPF程序运行在内核空间，可以直接访问内核的内存和数据结构。因此，eBPF maps被存储在内核空间的特定区域中，而不是用户空间或虚拟机中.当创建一个eBPF程序时，eBPF maps被分配在内核内存中的特定区域。这个区域被称为“eBPF对象区”（BPF object area），它是eBPF程序和maps的共享内存区域。eBPF程序和maps都可以在对象区域中分配和释放内存，但是它们必须遵循特定的规则，以确保它们不会访问未分配的内存或释放已经被其他对象使用的内存。

2. verifier,JIT都是在eBPF虚拟机内部的吗？  
BPF源码：https://git.kernel.org/pub/scm/linux/kernel/git/bpf/bpf.git  
verifier内核中有，eBPF虚拟机也有。如果内核开启就不需要要到eBPF中检查。是这样吗？那JIT呢  
bpf源码里有verifier部分代码
```
./include/linux/verification.h
./kernel/bpf/verifier.c

bpf源码里也有JIT部分的代码
./arch/arm/net/bpf_jit_32.c
./arch/arm/net/bpf_jit_32.h
./arch/x86/net/bpf_jit_comp32.c
./arch/x86/net/bpf_jit_comp.c

还有llvm相关代码，但是就只是测试的
./Documentation/bpf/llvm_reloc.rst
./Documentation/kbuild/llvm.rst
./tools/perf/util/llvm-utils.c
./tools/perf/util/llvm-utils.h
./tools/perf/tests/llvm.h
./tools/perf/tests/llvm.c
```

3. eBPF程序每次运行都需要verifier检查安全性，JIT再编译成字节码，然后再挂载到内核钩子函数上吗？如果不是每次都编译并挂载，那么是如何存储的呢？那又是在什么时机清除这些指令或者代码的呢？
是的每次都这样。使用引用计数法，一般用户态进程退出引用计数为0则挂载在内核的函数就退出。但有些global的函数不退出，例如XDP,cgroup,tc, lwt等类型。
