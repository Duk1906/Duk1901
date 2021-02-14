```shell

平均负载=====================================================================================================================================

stress/模拟cpu io、多进程负载 ----- update查看平均负载 ------ mpstat查看系统负载情况 ---- pidstat 查看进程负载情况

mpstat 
•  Multiprocessor Statistics，实时监控工具，报告与cpu的一些统计信息这些信息都存在/proc/stat文件中
• 在多CPU系统里，其不但能查看所有的CPU的平均状况的信息，而且能够有查看特定的

$ uptime
02:34:03  up 2 days, 20:14,  1 user,              load average: 0.63, 0.83, 0.88
当前时间   系统运行时间          正在登录用户数    过去1 、5 、15 min的平均负载

平均负载： 单位时间内，系统处于可运行状态和不可中断状态的平均进程数，即平均活跃进程数
• 可运行状态：指正在使用 CPU 或者正在等待 CPU，ps 命令看到处于 R 状态（Running 或 Runnable）的进程。
• 不可中断状态：正处于内核态关键流程且流程是不可打断的，如常见的是等待硬件设备 I/O 响应， ps 命令看到 D 状态（Uninterruptible Sleep， Disk Sleep）的进程。
	比如，当一个进程向磁盘读写数据时，为保证数据一致性，在得到磁盘回复前，它是不能被其他进程或中断打断的。如果此时的进程被打断了，就容易出现磁盘数据与进程数据不一致的问题

查看系统cpu个数
$ grep 'model name' /proc/cpuinfo | wc -l
2

平均负载与cpu的关系
• 从平均负载的含义出发，单位时间内处于可运行状态以及不可中断状态的活跃进程数。
• 如果系统中有大量的cpu密集型进程，此时平均负载高与cpu使用率高有正相关的联系
• 如果系统中有大量IO密集型进程且这些进程处于不可中断的状态，此时平均负载高与cpu无关

模拟CPU压力
$ stress --cpu 1 --timeout 600    # 1表示模拟cpu使用率100%
$ watch -d uptime
$ mpstat -P ALL 5  1                # -P ALL ：监控所有CPU，5秒刷新，仅监控1次
13:30:06     CPU    %usr   %nice    %sys  %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
13:30:11     all    50.05    0.00    0.00    0.00    0.00    0.00    0.00     0.00      0.00      49.95
13:30:11       0    0.00     0.00    0.00    0.00    0.00    0.00    0.00     0.00      0.00      100.00
13:30:11       1    100.00   0.00    0.00    0.00    0.00    0.00    0.00     0.00      0.00      0.00

	• user/ us: 用户态 CPU 时间  ( 不包括下面的 nice 时间，但包括了 guest 时间)
	• nice/ni:   低优先级用户态 CPU 时间  (即进程的 nice 值被调整为 1-19 之间时的 CPU 时间。范围是 -20 到 19，数值越大，优先级越低)
	• system/sys:  代表内核态 CPU 时间
	• idle/ id: 空闲时间 (不包括等待 I/O 的时间)
	• Iowait/ wa: 等待 I/O 的 CPU 时间
	• Irq/hi：处理硬中断的 CPU 时间
	• softirq/si: 处理软中断的 CPU 时间
	• steal/ st: 当系统运行在虚拟机中的时候，被其他虚拟机占用的 CPU 时间
	• guest，运行虚拟机的 CPU 时间
	• guest_nice: 低优先级运行虚拟机的时间
	
$ pidstat -u 5 1                     # 每5秒后输出一组数据
13:37:07      UID       PID    %usr %system  %guest   %wait    %CPU   CPU  Command
13:37:12        0      2962  100.00    0.00      0.00        0.00      100.00     1     stress

模拟mem压力
$ stress -i 1 --timeout 600      # 1表示模拟cpu使用率100%
$ watch -d uptime
$ mpstat -P ALL 5 1 
13:41:28     CPU    %usr   %nice    %sys   %iowait   %irq   %soft  %steal  %guest  %gnice   %idle
13:41:33     all      0.21     0.00     12.07   32.67      0.00    0.21     0.00     0.00      0.00       54.84
13:41:33       0      0.43     0.00     23.87   67.53      0.00    0.43     0.00     0.00      0.00        7.74
13:41:33       1      0.00     0.00     0.81     0.20         0.00    0.00     0.00      0.00      0.00       98.99
$ pidstat -u 5 1
13:42:08      UID       PID    %usr %system  %guest   %wait    %CPU   CPU  Command
13:42:13        0       104     0.00    3.39       0.00       0.00       3.39      1      kworker/1:1H
13:42:13        0       109     0.00    0.40       0.00       0.00        0.40      0      kworker/0:1H
13:42:13        0      2997    2.00    35.53     0.00       3.99        37.52     1      stress
13:42:13        0      3057    0.00    0.40       0.00       0.00        0.40       0      pidstat

模拟多进程压力
$ stress -c 8 --timeout 600    # 模拟8个进程
$ watch -d uptime
$ pidstat -u 5 1
14:23:25      UID       PID    %usr %system  %guest   %wait    %CPU   CPU  Command
14:23:30        0      3190   25.00    0.00      0.00      74.80     25.00     0       stress
14:23:30        0      3191   25.00    0.00      0.00      75.20     25.00     0      stress
14:23:30        0      3192   25.00    0.00      0.00      74.80     25.00     1       stress
14:23:30        0      3193   25.00    0.00      0.00      75.00     25.00     1       stress
14:23:30        0      3194   24.80    0.00      0.00      74.60     24.80     0       stress
14:23:30        0      3195   24.80    0.00      0.00      75.00      24.80     0       stress
14:23:30        0      3196   24.80    0.00      0.00      74.60      24.80     1       stress
14:23:30        0      3197   24.80    0.00      0.00       74.80      24.80     1       stress
14:23:30        0      3200    0.00     0.20      0.00        0.20        0.20       0       pidstat


CPU性能分析=========================================================================================================================

每个进程任务执行前都需要获取到cpu时间片，并且需要在cpu中暂存程序命令的信息，这个暂存的地方成为cpu上下文
CPU 上下文 = CPU 寄存器+ 程序计数器
	• 寄存器：是 CPU 内置的容量小、但速度极快的内存
	• 程序计数器：用来存储 CPU 正在执行的指令位置、或者即将执行的下一条指令位置

CPU 上下文切换，先把前一个任务的上下文保存至系统内核中，然后加载新任务的上下文到寄存器和程序计数器，跳转到程序计数器所指的新位置，运行新任务。
	• 系统调用发生的切换
	用户态进程通过系统调用进入内核态，寄存器里原用户态指令位置保存起来，接着更新为内核态指令的新位置，最后才跳转到内核态运行内核任务。而系统调用结束后恢复原来保存的用户态，然后再切换到用户空间，继续运行进程。
	所以，一次系统调用，发生了两次 CPU 上下文切换（系统调用过程通常称为特权模式切换，而不是上下文切换，这类切换是无法避免的）
	• 进程上下文切换
	*  进程是由内核来管理和调度的，进程的切换只能发生在内核态。
	*  进程的上下文不仅包括用户空间的虚拟内存、栈、全局变量等的资源，还包括了内核堆栈、寄存器等内核空间的状态。
	*  进程的上下文切换就比系统调用时多了一步：在保存当前进程的内核状态和 CPU 寄存器之前，需要先把该进程的虚拟内存、栈等保存下来；而加载了下一进程的内核态后，还需要刷新进程的虚拟内存和用户栈。
	*  保存上下文和恢复上下文的过程需要内核在 CPU 上运行才能完成。
	• 线程上下文切换
	* 前后两个线程属于不同进程：因资源不共享，所以切换过程就跟进程上下文切换是一样
	* 前后两个线程属同一个进程：因用户空间虚拟内存共享，所以切换时，虚拟内存这些资源就保持不动，只需切换线程的私有数据、寄存器等不共享的数据。
	* 因此，同进程内的线程切换，要比多进程间的切换消耗更少的资源，而这，也正是多线程代替多进程的一个优势。
	• 中断上下文切换
	* 为了快速响应硬件的事件，中断处理会打断进程的正常调度和执行，转而调用中断处理程序，响应设备事件。
	* 因为中断是内核态的，所以打断用户态的进程，无需保存进程的虚拟内存、全局变量等用户态资源。仅需要切换内核态中断服务程序执行所必需的状态，包括 CPU 寄存器、内核堆栈、硬件中断参数等。
	* 对同一个 CPU 来说，中断处理比进程拥有更高的优先级，所以中断上下文切换并不会与进程上下文切换同时发生。
	* 同样道理，由于中断会打断正常进程的调度和执行，所以大部分中断处理程序都短小精悍，以便尽可能快的执行结束

        综上，过多的上下文切换，会把 CPU 时间消耗在寄存器、内核栈以及虚拟内存等数据的保存和恢复上

中断： 异步事件处理机制，可提高系统并发处理能力，为减少对正常进程运行调度的影响，中断处理程序就需要尽可能快地运行
硬中断： 直接处理硬件请求，快速执行（/proc/interrupts）
软中断： 由内核触发，延迟执行（/proc/softirqs）

软硬中断事例：
	• 网卡接收到数据包后，硬中断要把网卡的数据读到内存中，更新硬件寄存器状态，再发送一个软中断信号；
	• 软中断则负责从内存中读取网络数据，再按网络协议栈，逐层解析处理数据，直到送至应用程序

$ sysbench --threads=10 --max-time=300 threads run    # 以10个线程运行5分钟的基准测试，模拟多线程切换的问题

$ vmstat 1   # 每隔1秒输出1组数据（需要Ctrl+C才结束）
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b      swpd   free       buff       cache         si   so    bi    bo   in         cs             us sy  id wa st
 6  0      0        6487428 118240 1292772    0    0     0     0    9019    1398830 16 84  0  0  0
 8  0      0          6487428 118240 1292772    0    0     0     0    10191  1392312 16 84  0  0  0

r:   Running/Runnable,正在运行和等待 CPU 的进程数
b:  Blocked, 处于不可中断睡眠状态的进程数
cs: context switch，每秒上下文切换的次数
In: interrupt，每秒中断的次数
us: user，用户态cpu使用率
sy: system，系统内核态cpu使用率

$ pidstat -w 5     ----   每个进程上下文切换的情况
$ pidstat -wt 1   # -t, thread, 表示输出线程的上下文切换指标                                 
08:14:05      UID   TGID       TID   cswch/s nvcswch/s  Command
...
08:14:05        0     10551         -      6.00      0.00  sysbench
08:14:05        0         -     10551      6.00      0.00  |__sysbench
08:14:05        0         -     10552  18911.00 103740.00  |__sysbench
08:14:05        0         -     10553  18915.00 100955.00  |__sysbench
08:14:05        0         -     10554  18827.00 103954.00  |__sysbench
...
cswch : 每秒自愿上下文切换（voluntary context switches）次数，进程无法获取所需资源（ I/O、内存等系统资源不足时），导致的上下文切换。
nvcswch : 每秒非自愿上下文切换（non voluntary context switches）的次数，指进程由于时间片已到等原因（大量进程都在争抢 CPU 时），被系统强制调度，进而发生的上下文切换
$ watch -d cat /proc/interrupts   # -d 参数表示高亮显示变化的区域
           CPU0       CPU1
...
RES:    2450431    5279697   Rescheduling interrupts       # 重调度中断（RES），表示唤醒空闲状态的 CPU 来调度新的任务运行, 中断升高还是因为过多任务的调度问题
...

CPU 使用率过高
通过 top、ps、pidstat , 找出 CPU 使用率较高（比如 100% ）的进程
$ top
...
%Cpu0  : 98.7 us,  1.3 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu1  : 99.3 us,  0.7 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
...
  PID    USER          PR  NI      VIRT  RES       SHR   S  %CPU %MEM    TIME+ COMMAND
21514 daemon    20   0  336696  16384   8712 R  41.9   0.2           0:06.00 php-fpm
21513 daemon    20   0  336696  13244   5572 R  40.2   0.2           0:06.08 php-fpm
21515 daemon    20   0  336696  16384   8712 R  40.2   0.2           0:05.67 php-fpm
21512 daemon    20   0  336696  13244   5572 R  39.9   0.2           0:05.87 php-fpm
21516 daemon    20   0  336696  16384   8712 R  35.9   0.2           0:05.61 php-fpm

定位应用内部函数
$ perf top -g -p 21515    # -g开启调用关系分析，-p指定php-fpm的进程号21515

以上，可以定位到 sqrt、add_function，接下来就需要分析对应的代码了

如果cpu使用率较高，但通过top无法捕捉到对应的进程id，极可能出现了短时进程
	• 应用里直接调用了其他二进制程序，这些程序通常运行时间比较短，通过 top 等工具也不容易发现。
	• 应用本身在不停地崩溃重启，而启动过程的资源初始化，很可能会占用相当多的 CPU。
$ perf record -g   # 记录性能事件，等待大约15秒后按 Ctrl+C 退出
$ perf report        # 查看报告
$ execsnoop   # 专为短时进程设计的工具。通过 ftrace 实时监控进程的 exec() 行为，并输出短时进程的基本信息如 PID、父进程 PID、命令行参数以及执行的结果等
PCOMM            PID    PPID   RET ARGS
sh                 30394  30393    0
stress           30396  30394    0 /usr/local/bin/stress -t 1 -d 1
sh                 30398  30393    0
stress           30399  30398    0 /usr/local/bin/stress -t 1 -d 1
...

进程状态
	• R： Running 、Runnable，表示进程在 CPU 的就绪队列中，正在运行或者正在等待运行
	• D：Disk Sleep，不可中断状态睡眠（Uninterruptible Sleep），一般表示进程正在跟硬件交互，并且交互过程不允许被其他进程或中断打断
	• S：Interruptible Sleep，可中断状态睡眠，表示进程因为等待某个事件而被系统挂起。当进程等待的事件发生时，它会被唤醒并进入 R 状态
	• Z ：Zombie，僵尸进程，即进程实际已结束，但父进程还未回收它的资源（比如进程的描述符、PID 等）
	• I ： Idle ，空闲状态，用在不可中断睡眠的内核线程上
	• T/ t： Stopped 或 Traced，表示进程处于暂停或者跟踪状态
	• X： Dead，表示进程已消亡，top或ps命令中看不到

	1. 不可中断进程增加，可能是系统或硬件发生了故障，导致出现了io性能问题
	2. 正常子进程在结束后，会向父进程发送 SIGCHLD 信号，父进程会进行资源回收；如果子进程结束太快，父进程还没及时处理，子进程就会变成僵尸进程。
	通常，僵尸进程持续时间较短，在父进程回收资源后就会消亡；或父进程退出后，由 init 进程回收后也会消亡。
	一旦父进程没处理子进程的终止，且父进程一直保持运行状态，那么子进程就会一直处于僵尸状态。大量的僵尸进程会用尽 PID 进程号，导致新进程不能创建
	
	
案例：---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
$ top
top - 05:56:23 up 17 days, 16:45,  2 users,  load average: 2.00, 1.68, 1.39
Tasks: 247 total,   1 running,  79 sleeping,   0 stopped, 115 zombie
%Cpu0  :  0.0 us,  0.7 sy,  0.0 ni, 38.9 id, 60.5 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu1  :  0.0 us,  0.7 sy,  0.0 ni,  4.7 id, 94.6 wa,  0.0 hi,  0.0 si,  0.0 st
...
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 4340 root      20   0   44676   4048   3432 R   0.3  0.0   0:00.05 top
 4345 root      20   0   37280  33624    860 D   0.3  0.0   0:00.01 app
 4344 root      20   0   37280  33624    860 D   0.3  0.4   0:00.01 app
    1    root      20   0  160072   9416   6752 S   0.0  0.1   0:38.59 systemd
...
	Ø iowait 太高了，导致系统平均负载升高，并且已经达到了系统 CPU 的个数
	Ø 僵尸进程在不断增多，看起来是应用程序没有正确清理子进程的资源


iowait升高问题：
$ dstat 1 10    # 间隔1秒输出10组数据（可同时观察系统的 CPU、内存、磁盘 I/O、网络使用情况）
--total-cpu-usage-- -dsk/total-- -net/total- ---paging-- ---system--
usr sys idl wai stl | read  writ   | recv  send|  in   out | int   csw
  0   0  96   4   0    |1219k  408k|   0     0       |   0     0  |  42   885
  0   0   2  98   0    |  34M    0     |198B  790B|   0     0  |  42   138
	Ø 说明 iowait 的升高跟磁盘的读请求有关，很可能就是磁盘读导致的

从 top 的输出找到 D 状态进程的 PID
$ pidstat -d -p 4344 1 3    # -d 展示 I/O 统计数据，-p 指定进程号，间隔 1 秒输出 3 组数据
	Ø 指定进程无输出，可能是导致io问题的子进程已经执行结束，已经转变为了僵尸进程，也对应上面的僵尸进程增加的猜想

$ pidstat -d 1 20                     # 间隔 1 秒输出20组数据
06:48:46      UID       PID  kB_rd/s   kB_wr/s  kB_ccwr/s  iodelay  Command
06:48:47        0      4615          0.00      0.00      0.00          1  kworker/u4:1
06:48:47        0      6080  32768.00      0.00      0.00         170  app
06:48:47        0      6081  32768.00      0.00      0.00         184  app


$ strace -p 6082     跟踪进程系统调用
strace: attach: ptrace(PTRACE_SEIZE, 6082): Operation not permitted

$ ps aux | grep 6082       僵尸进程，strace跟踪不到
root      6082  0.0  0.0      0     0 pts/0    Z+   13:43   0:00 [app] <defunct>

$ perf record -g
$ perf report

	Ø app通过系统调用 sys_read() 读取数据, 从 new_sync_read 和 blkdev_direct_IO 能看出，进程正在对磁盘进行直接读，绕过了系统缓存，每个读请求都会从磁盘直接读，导致 iowait 升高

open(disk, O_RDONLY|O_DIRECT|O_LARGEFILE, 0755)   # 删除 O_DIRECT 选项即可

僵尸进程问题  
$ pstree -aps 3084       # -a ：输出命令行选项,  p：PID,  s：指定进程的父进程
systemd,1
  └─dockerd,15006 -H fd://
      └─docker-containe,15024 --config xxx
          └─docker-containe,3991 -namespace …
              └─app,4009
                  └─(app,3084)
$ ps -ef | grep 4009

	Ø 检查 4009进程/app 应用程序的代码，看下子进程结束的处理是否正确，比如有没有调用 wait() 或 waitpid() ，抑或是有没有注册 SIGCHLD 信号的处理函数

结论：
	1. iowait升高问题，可能原因除了io性能外，还可以是程序进行了直接磁盘读写操作，绕过了系统缓存
	2. 僵尸进程的产生，应该是父进程没有正确处理子进程的资源回收，比如fork()子进程时，需要在正确位置调用wait()及时回收子进程资源
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------













CPU性能分析问题套路（上图）
=========================================================================================================================


MEM性能分析=============================================================================================================
内存：存储系统和应用程序的指令、数据、缓存等
	• 日常提到的电脑16G内存，指的是物理内存，仅内核才可以直接访问物理内存
	• 进程看到的是内核提供的虚拟内存，通过页表，由系统映射为物理内存
	• 进程申请内存后，内存并不会立即分配，而是在首次访问时，才通过缺页异常陷入内核中分配内存

应对内存不足的机制：
	• 缓存的回收
	• 交换分区 Swap
	• OOM

$ free
                  total             used          free              shared   buff/cache   available
Mem:        8169348      263524     6875352       668        1030472       7611064
Swap:             0              0                0
Swap机制：把不常访问内存先写到磁盘中，释放内存给更需要的进程使用，再次访问这些内存时，重新从磁盘读入内存中。
Buffers：磁盘读写的缓存
Cache： 文件读写的缓存

磁盘和文件的区分
	• 磁盘是存储设备，可以被划分为不同的磁盘分区。
	• 在磁盘或者磁盘分区上，可以创建文件系统，并挂载到系统的某个目录中。这样，系统就可以通过这个挂载目录，来读写文件
	• 在读写普通文件时，I/O 请求经过文件系统与磁盘进行交互
	• 在读写磁盘时，会跳过文件系统，直接与磁盘交互，也就是所谓的“裸 I/O”



$ cachestat 1 3      # 整个操作系统缓存的读写命中情况
   TOTAL   MISSES     HITS  DIRTIES   BUFFERS_MB  CACHED_MB             DIRTIES：新增脏页数
       2        0        2        1           17        279
       2        0        2        1           17        279
       2        0        2        1           17        279 

$ cachetop           # 每个进程的缓存命中情况
11:58:50 Buffers MB: 258 / Cached MB: 347 / Sort: HITS / Order: ascending
PID        UID      CMD        HITS     MISSES   DIRTIES  READ_HIT%  WRITE_HIT%
13029   root     python   1           0              0             100.0%           0.0%

$ strace -p $(pgrep app)     # pgrep 命令来查找案例进程的 PID 号


$ vmstat 1
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
r  b    swpd   free        buff    cache    si   so    bi    bo   in   cs    us sy id   wa st
0  0    0       7743608   1112  92168    0    0     0     0    52  152  0   1 100  0  0
0  0    0       7743608   1112  92168    0    0     0     0    36   92   0   0  100  0  0
bi & bo： 块设备读取和写入的大小，单位为块 / 秒（Linux 中块的大小是 1KB）

内存泄漏
	• 危害：需要内存的进程，无法分配新的内存；内存不足，又会触发系统的缓存回收以及 SWAP 机制，从而进一步导致 I/O 的性能问题
	• 标准库函数 malloc() 来动态分配内存，用完内存后，调用库函数 free() 来释放它们。



$ vmstat 3
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
0  0      0 6601824  97620 1098784    0    0     0     0   62  322  0  0 100  0  0
0  0      0 6601700  97620 1098788    0    0     0     0   57  251  0  0 100  0  0
0  0      0 6601320  97620 1098788    0    0     0     3   52  306  0  0 100  0  0

	• 未使用内存在逐渐减小，而 buffer 和 cache 基本不变。这说明，系统中使用的内存一直在升高。但这并不能说明有内存泄漏
	• 因为应用程序运行中需要的内存也可能会增大。比如说，程序中如果用了一个动态增长的数组来缓存计算结果，占用内存自然会增长

# 应用运行在容器中，memleak 在容器外，不能直接访问进程路径 /app，需在容器外部构建相同路径的文件以及依赖库
$ docker cp app:/app /app 

$ /usr/share/bcc/tools/memleak -p $(pidof app) -a
Attaching to pid 12512, Ctrl+C to quit.
[03:00:41] Top 10 stacks with outstanding allocations:
    addr = 7f8f70863220 size = 8192
    addr = 7f8f70861210 size = 8192
    addr = 7f8f7085b1e0 size = 8192
    addr = 7f8f7085f200 size = 8192
    addr = 7f8f7085d1f0 size = 8192
    40960 bytes in 5 allocations from stack
        fibonacci+0x1f [app]
        child+0x4f [app]
        start_thread+0xdb [libpthread-2.27.so] 

(代码展示略)….
child() 调用了 fibonacci() 函数，但并没有释放 fibonacci() 返回的内存
fibonacci() 调用malloc() 动态分配内存，用完内存后，还需调用库函数 free() 来释放它们





快准狠分析套路：



IO性能分析=========================================================================================================================

磁盘：为系统提供了最基本的持久化存储
文件系统： 在磁盘的基础上，提供了一个用来管理文件的树状结构

Linux文件系统工作原理
	• 本身是对存储设备上的文件，进行组织管理的机制
	• Linux 文件系统为每个文件都分配两个数据结构，索引节点（index node）和目录项（directory entry）
	• 索引节点：记录文件元数据，如 inode 编号、文件大小、访问权限、修改日期、数据的位置等。会被持久化存储到磁盘中
	• 目录项：记录文件名字、索引节点指针以及与其他目录项的关联关系。目录项由内核维护内存数据结构，叫做目录项缓存


软硬链接的区分：

文件系统 I/O
缓冲 I/O： 利用标准库缓存来加速文件的访问，而标准库内部再通过系统调度访问文件
非缓冲 I/O：直接通过系统调用来访问文件，不再经过标准库缓存
直接 I/O：跳过操作系统的页缓存，直接跟文件系统交互来访问文件
非直接 I/O ：文件读写时，先要经过系统的页缓存，然后再由内核或额外的系统调用，真正写入磁盘。
阻塞 I/O：应用程序执行 I/O 操作后，如果没有获得响应，就会阻塞当前线程，自然就不能执行其他任务
非阻塞 I/O：应用程序执行 I/O 操作后，不会阻塞当前的线程，可以继续执行其他的任务，随后再通过轮询或者事件通知的形式，获取调用的结果
同步 I/O：应用程序执行 I/O 操作后，要一直等到整个 I/O 完成后，才能获得 I/O 响应。
异步 I/O：应用程序执行 I/O 操作后，不用等待完成和完成后的响应，而是继续执行就可以。等到这次 I/O 完成后，响应会用事件通知的方式，告诉应用程序

$ df -h /dev/sda1    # 查看磁盘使用情况
$ df -i /dev/sda1     # 查看磁盘inode使用情况

io性能指标：

查看系统io使用情况
$ iostat -d -x 1    # -d -x表示显示所有磁盘I/O的指标
Device            r/s     w/s     rkB/s     wkB/s   rrqm/s   wrqm/s  %rrqm  %wrqm r_await w_await aqu-sz rareq-sz wareq-sz  svctm  %util 
loop0            0.00    0.00      0.00      0.00     0.00     0.00         0.00      0.00      0.00      0.00         0.00     0.00       0.00           0.00     0.00 

 I/O 使用率：%util 
IOPS： r/s+ w/s 
吞吐量：rkB/s + wkB/s
响应时间： r_await + w_await 




$ pidstat -d 1 
13:39:51      UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s iodelay  Command 
13:39:52      102       916   0.00        4.00         0.00           0             rsyslogd

	• UID:  用户 ID
	• PID:  进程 ID
	• kB_rd/s:  每秒读取数据大小 ，单位是 KB
	• kB_wr/s:  每秒发出的写请求数据大小，单位是 KB
	• kB_ccwr/s:  每秒取消的写请求数据大小，单位是 KB
	• Iodelay:  块 I/O 延迟，包括等待同步块 I/O 和换入块 I/O 结束的时间，单位是时钟周期


案列：   

$ top 
top - 14:43:43 up 1 day,  1:39,  2 users,  load average: 2.48, 1.09, 0.63 
Tasks: 130 total,   2 running,  74 sleeping,   0 stopped,   0 zombie 
%Cpu0  :  0.7 us,  6.0 sy,  0.0 ni,  0.7 id, 92.7 wa,  0.0 hi,  0.0 si,  0.0 st 
%Cpu1  :  0.0 us,  0.3 sy,  0.0 ni, 92.3 id,  7.3 wa,  0.0 hi,  0.0 si,  0.0 st 
KiB Mem :  8169308 total,   747684 free,   741336 used,  6680288 buff/cache 
KiB Swap:        0 total,        0 free,        0 used.  7113124 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND 
18940 root      20   0  656108 355740   5236 R   6.3  4.4   0:12.56 python 
1312 root      20   0  236532  24116   9648 S   0.3  0.3   9:29.80 python3 

$ iostat -x -d 1    # -d表示显示I/O性能指标，-x表示显示扩展统计（即所有I/O指标） 
Device            r/s     w/s     rkB/s     wkB/s   rrqm/s wrqm/s  %rrqm  %wrqm r_await w_await aqu-sz rareq-sz wareq-sz  svctm  %util 
loop0            0.00    0.00      0.00      0.00     0.00     0.00       0.00     0.00      0.00      0.00        0.00      0.00     0.00          0.00   0.00 
sdb                0.00    0.00      0.00      0.00     0.00     0.00       0.00     0.00      0.00      0.00        0.00      0.00     0.00          0.00   0.00 
sda                0.00   64.00     0.00  32768.00 0.00    0.00       0.00     0.00      0.00     7270.44   1102.18 0.00   512.00      15.50  99.20

$ pidstat -d 1     # -d表示显示I/O性能指标
15:08:35      UID       PID         kB_rd/s   kB_wr/s   kB_ccwr/s iodelay  Command 
15:08:36        0        18940      0.00       45816.00   0.00           96           python 

$ strace -p 18940 
strace: Process 18940 attached 
...
mmap(NULL, 314576896, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f0f7aee9000 
mmap(NULL, 314576896, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f0f682e8000 
write(3, "2018-12-05 15:23:01,709 - __main"..., 314572844 
) = 314572844 
munmap(0x7f0f682e8000, 314576896)       = 0 
write(3, "\n", 1)                       = 1 
munmap(0x7f0f7aee9000, 314576896)       = 0 
close(3)                                = 0 
stat("/tmp/logtest.txt.1", {st_mode=S_IFREG|0644, st_size=943718535, ...}) = 0 


$ lsof -p 18940 
COMMAND PID USER   FD    TYPE DEVICE  SIZE/OFF    NODE NAME 
python  18940   root    cwd   DIR   0,50       4096        1549389 / 
python  18940   root    rtd     DIR   0,50       4096        1549389 / 
… 
python  18940   root    2u     CHR  136,0       0t0       3 /dev/pts/0 
python  18940   root    3w    REG   8,1 117944320   303 /tmp/logtest.txt 

碰到“狂打日志”场景，可以用top、iostat、pidstat、strace、lsof 定位狂打日志的进程，找出相应的日志文件，通过调整日志级别来解决问题
如果应用程序不能动态调整日志级别，可能还需要修改应用的配置，并重启应用让配置生效


Case2：strace命令查看不到进程调用write命令时，可用opensnoop查看，发现这个根源是大量读写临时文件
$ opensnoop       # 属于 bcc 软件包，可以动态跟踪内核中的 open 系统调用
12280  python      6   0 /tmp/9046db9e-fe25-11e8-b13f-0242ac110002/650.txt 
12280  python      6   0 /tmp/9046db9e-fe25-11e8-b13f-0242ac110002/651.txt 
12280  python      6   0 /tmp/9046db9e-fe25-11e8-b13f-0242ac110002/652.txt 

Case3：SQL查询慢可能是查询列没有索引或者查询没有选对索引（没选对可以用force_index()选项指定）
$ top
$ iostat -d -x 1
$ pidstat -d 1
$ strace -f -p 27458        #  -f展示对应线程的数据读取情况
$ lsof -p 28014
$ pstree -t -a -p 27458    #  -t表示显示线程，-a表示显示命令行参数
$ lsof -p 27458
mysql> show full processlist;
mysql> use test;
mysql> explain select * from products where productName='geektime';

Case4：
$ strace -f -T -tt -p 9085 # -f表示跟踪子进程和子线程，-T表示显示系统调用的时长，-tt表示显示跟踪时间






网络性能分析=========================================================================================================================
网络，本质上是一种进程间通信方式

OSI七层模型：
	• 应用层，负责为应用程序提供统一的接口
	• 表示层，负责把数据转换成兼容接收系统的格式
	• 会话层，负责维护计算机之间的通信连接
	• 传输层，负责为数据加上传输表头，形成数据包
	• 网络层，负责数据的路由和转发
	• 数据链路层，负责 MAC 寻址、错误侦测和改错
	• 物理层，负责在物理网络中传输数据帧

Linux采用更简单的tcp/ip模型


Linux网络模型：应用程序-----系统调用-----套接字-----tcp-----ip-----链路层-----网卡

	• 网卡收发网络包的基本设备
	• 在系统启动过程中，网卡通过内核中的网卡驱动程序注册到系统中。
	• 网卡硬中断只处理最核心的网卡数据读取或发送，协议栈中的大部分逻辑，都会放到软中断中处理


查看套接字状态（netstat / ss）
$ netstat -nlp | head -n 3     # -p ：process，l：listened，n：number/显示数字地址和端口，t：tcp，u：udp      
Proto Recv-Q Send-Q Local Address           Foreign Address         State        PID/Program name
tcp        0         0           127.0.0.53:53           0.0.0.0:*                     LISTEN      840/systemd-resolve

$ ss -ltnp | head -n 3
State    Recv-Q    Send-Q        Local Address:Port        Peer Address:Port
LISTEN   0            128              127.0.0.53%lo:53           0.0.0.0:*        users:(("systemd-resolve",pid=840,fd=13))
LISTEN   0            128               0.0.0.0:22                        0.0.0.0:*        users:(("sshd",pid=1459,fd=3))

当套接字处于连接状态（Established）时
	• Recv-Q 表示套接字缓冲还没有被应用程序取走的字节数（即接收队列长度）
	• Send-Q 表示还没有被远端主机确认的字节数（即发送队列长度）
当套接字处于监听状态（Listening）时
	• Recv-Q 表示全连接队列的长度
	• Send-Q 表示全连接队列的最大长度

全连接：完成TCP三次握手后的连接，服务端会把这个连接挪到全连接队列中
半连接：还没有完成 TCP 三次握手的连接，连接只进行了一半
                 server收到client SYN 包后，会把这个连接放到半连接队列中，再向client发送 SYN+ACK 包（即，server收到SYN包则进入半连接）



性能指标：带宽、吞吐、pps、延迟等
网络吞吐和 PPS（sar -n DEV 1）  
$ sar -n DEV 1 #  -n DEV 表示显示网络收发的报告，数字1表示每隔1秒输出一组数据
13:21:40        IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
13:21:41         eth0     18.00     20.00      5.79      4.25        0.00        0.00         0.00      0.00
13:21:41      docker0   0.00      0.00        0.00      0.00        0.00        0.00         0.00      0.00
13:21:41           lo         0.00      0.00         0.00      0.00        0.00        0.00          0.00      0.00
	• rxpck/s 和 txpck/s 分别是接收和发送的 PPS，单位为包 / 秒
	• rxkB/s 和 txkB/s 分别是接收和发送的吞吐量，单位是 KB/ 秒
	• rxcmp/s 和 txcmp/s 分别是接收和发送的压缩数据包数，单位是包 / 秒
	• %ifutil 是网络接口的使用率

连通性和延时（ping）
ping -c3   $target_host_ip


C10k问题解决
I/O 多路复用
	1. 使用非阻塞 I/O 和水平触发通知，使用 select 或 poll
	- select： 使用固定长度位相量表示文件描述符集合，因此会有最大描述符数量的限制
	- poll：非固定长度的数组，没有最大描述符数量的限制（当然还会受到系统文件描述符限制）。但仍需对文件描述符列表进行轮询
	2. 使用非阻塞 I/O 和边缘触发通知，比如 epoll。
	- epoll 使用红黑树，在内核中管理文件描述符的集合，不需要应用程序在每次操作时都传入、传出这个集合。
	- epoll 使用事件驱动的机制，只关注有 I/O 事件发生的文件描述符，不需要轮询扫描整个集合。
工作模型优化
	1. 主进程 + 多个 worker 子进程

如 nginx 就采用了“主进程 + 多个 worker 子进程”+  epoll机制io多路复用技术
	• Nginx 在每个 worker 进程中，都增加一个了全局锁（accept_mutex）。只有竞争到锁的进程才会加入到 epoll 中处理io事件，避免惊群事件发生


$ tcpdump -i eth0 -n tcp port 80   # -n不解析协议名和主机名
15:11:32.678966 IP 192.168.0.2.18238 > 192.168.0.30.80: Flags [S], seq 458303614, win 512, length 0
…





$ tcpdump -nn udp port 53 or host 35.190.27.188 -w ping.pcap

用 Wireshark 打开  ping.pcap





DDOS：distributed denial of service，分布式拒绝服务攻击
SYN Flood 攻击
发送大量SYN包，但不完成三次握手最后一次ACK，导致server端半连接队列堆积，从而无法新建连接

$ tcpdump -i eth0 -n tcp port 80
09:15:48.287047 IP 192.168.0.2.27095 > 192.168.0.30: Flags [S], seq 1288268370, win 512, length 0
09:15:48.287050 IP 192.168.0.2.27131 > 192.168.0.30: Flags [S], seq 2084255254, win 512, length 0
...
Flags [S] 表示这是一个 SYN 包。大量的 SYN 包表明，这是一个 SYN Flood 攻击

$ netstat -n -p | grep SYN_REC   # -n表示不解析名字，-p表示显示连接所属进程
tcp        0      0 192.168.0.30:80          192.168.0.2:12503      SYN_RECV    -
tcp        0      0 192.168.0.30:80          192.168.0.2:13502      SYN_RECV    -
...

$ netstat -n -p | grep SYN_REC | wc -l
193

解决：
	1. 设置黑白名单 
	$ iptables -I INPUT -s 192.168.0.2 -p tcp -j REJECT
	2. 限制单个IP在60秒新建立的连接数为10
	$ iptables -I INPUT -p tcp --dport 80 --syn -m recent --name SYN_FLOOD --update --seconds 60 --hitcount 10 -j REJECT
	3. 调大连接表的大小
	$ sysctl -w net.ipv4.tcp_max_syn_backlog=1024
	net.ipv4.tcp_max_syn_backlog = 1024
	4. TCP SYN Cookies 也是一种专门防御 SYN Flood 攻击的方法。
	• SYN Cookies 基于连接信息（源地址、源端口、目的地址、目的端口等）及加密种子，计算 cookie
	• 以该cookie作序列号，来应答 SYN+ACK 包，并释放连接状态
	• 当客户端发送完三次握手的最后一次 ACK 后，服务器就会再次计算这个哈希值，确认是上次返回的 SYN+ACK 的返回包，才会进入 TCP 的连接状态。因而，开启 SYN Cookies 后，就不需要维护半开连接状态了，进而也就没有了半连接数的限制。
	• 开启 TCP SYN Cookies：
	$ sysctl -w net.ipv4.tcp_syncookies=1
	net.ipv4.tcp_syncookies = 1


nginx连接网络延迟变大案列
nagle算法：合并tcp小包，提高网络带宽利用率（规定一个tcp连接上仅能有一个未被确认的包）
nginx开启nagle+client开启延迟确认，导致了网络延迟，在nginx.conf中设置tcp_nodey on关闭nagle设置，问题解决














```
