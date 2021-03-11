##### 平均负载

```shell
定义：单位时间内，系统处于可运行状态和不可中断状态的平均进程数，即平均活跃进程数

- 可运行状态：指正在使用 CPU 或者正在等待 CPU，ps 命令看到处于 R 状态（Running 或 Runnable）的进程。
- 不可中断状态：正处于内核态关键流程且流程是不可打断的，如常见的是等待硬件设备 I/O 响应， ps 命令看到 D 状态（Uninterruptible Sleep， Disk Sleep）的进程。比如，当一个进程向磁盘读写数据时，为保证数据一致性，在得到磁盘回复前，它是不能被其他进程或中断打断的。如果此时的进程被打断了，就容易出现磁盘数据与进程数据不一致的问题

平均负载与cpu的关系
• 从平均负载的含义出发，单位时间内处于可运行状态以及不可中断状态的活跃进程数。
• 如果系统中有大量的cpu密集型进程，此时平均负载高与cpu使用率高有正相关的联系，如果系统中有大量IO密集型进程且这些进程处于不可中断的状态，此时平均负载高与cpu无关
```

##### 常见工具

```shell
uptime ：系统负载情况查看工具
$ watch -d uptime
当前时间   系统运行时间         正在登录用户数          过去1 、5 、15 min的平均负载（负载逐渐降低）
02:34:03  up 2 days, 20:14,  1 user,              load average: 0.63, 0.83, 0.88

mpstat ：Multiprocessor Statistics，cpu实时监控工具，统计信息这些信息都存在/proc/stat文件中
$ mpstat -P ALL 5  1              # -P ALL ：监控所有CPU，5秒刷新，仅监控1次
13:30:06   CPU   %usr   %nice  %sys  %iowait %irq  %soft  %steal  %guest  %gnice   %idle
13:30:11   all   50.05  0.00   0.00  0.00    0.00  0.00    0.00   0.00    0.00     49.95
13:30:11   0     0.00   0.00   0.00  0.00    0.00  0.00    0.00   0.00    0.00     100.00
13:30:11   1     100.00 0.00   0.00  0.00    0.00  0.00    0.00   0.00    0.00     0.00

• user/ us: 用户态 CPU 时间  ( 不包括下面的 nice 时间，但包括了 guest 时间)
• nice/ni: 低优先级用户态 CPU 时间  (nice值为 1-19间时的CPU时间。范围是-20到19，值越大，优先级越低)
• system/sys: 代表内核态 CPU 时间
• idle/ id: 空闲时间 (不包括等待 I/O 的时间)
• Iowait/ wa: 等待 I/O 的 CPU 时间
• Irq/hi：处理硬中断的 CPU 时间
• softirq/si: 处理软中断的 CPU 时间
• steal/ st: 当系统运行在虚拟机中的时候，被其他虚拟机占用的 CPU 时间
• guest，运行虚拟机的 CPU 时间
• guest_nice: 低优先级运行虚拟机的时间

stress ：压力测试工具
$ stress --cpu 1 --timeout 600    # 模拟CPU压力 
$ stress -i 1 --timeout 600       # 模拟io压力
$ stress -c 8 --timeout 600       # 模拟多进程压力

pidstat：进程性能分析工具
$ pidstat -u 5 1
13:42:08  UID  PID   %usr  %system  %guest  %wait  %CPU   CPU    Command
13:42:13  0    104   0.00  3.39     0.00    0.00   3.39   1      kworker/1:1H
13:42:13  0    109   0.00  0.40     0.00    0.00   0.40   0      kworker/0:1H
13:42:13  0    2997  2.00  35.53    0.00    3.99   37.52  1      stress
13:42:13  0    3057  0.00  0.40     0.00    0.00   0.40   0      pidstat
```

