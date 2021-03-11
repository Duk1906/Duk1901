##### 简介

```shell
内存：存储系统和应用程序的指令、数据、缓存等
	• 日常提到的电脑16G内存，指的是物理内存，仅内核才可以直接访问物理内存
	• 进程看到的是内核提供的虚拟内存，通过页表，由系统映射为物理内存
	• 进程申请内存后，内存并不会立即分配，而是在首次访问时，才通过缺页异常陷入内核中分配内存

应对内存不足的机制：
	• 缓存的回收
	• 交换分区 Swap
	• OOM

$ free
       total        used       free          shared     buff/cache   available
Mem:   8169348      263524     6875352       668        1030472       7611064
Swap:  0              0                0
Swap机制：把不常访问内存先写到磁盘中，释放内存给更需要的进程使用，再次访问这些内存时，重新从磁盘读入内存中。
Buffers：磁盘读写的缓存
Cache： 文件读写的缓存

磁盘和文件的区分
• 磁盘是存储设备，可以被划分为不同的磁盘分区。
• 在磁盘或者磁盘分区上，可以创建文件系统，并挂载到系统的某个目录中。系统可以通过这个挂载目录，来读写文件
• 在读写普通文件时，I/O 请求经过文件系统与磁盘进行交互
• 在读写磁盘时，会跳过文件系统，直接与磁盘交互，也就是所谓的“裸 I/O”
```

##### 常用定位命令

```shell
$ cachestat 1 3      # 整个操作系统缓存的读写命中情况
   TOTAL   MISSES     HITS  DIRTIES   BUFFERS_MB  CACHED_MB             DIRTIES：新增脏页数
       2        0        2        1           17        279
       2        0        2        1           17        279
       2        0        2        1           17        279 

$ cachetop           # 每个进程的缓存命中情况
11:58:50 Buffers MB: 258 / Cached MB: 347 / Sort: HITS / Order: ascending
PID     UID      CMD      HITS     MISSES   DIRTIES  READ_HIT%  WRITE_HIT%
13029   root     python   1        0        0        100.0%     0.0%

$ strace -p $(pgrep app) # pgrep 命令来查找案例进程的 PID 号

$ vmstat 1
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
r  b    swpd   free        buff    cache    si   so    bi    bo   in   cs    us sy id   wa st
0  0    0       7743608   1112  92168    0    0     0     0    52  152  0   1 100  0  0
0  0    0       7743608   1112  92168    0    0     0     0    36   92   0   0  100  0  0
bi & bo： 块设备读取和写入的大小，单位为块 / 秒（Linux 中块的大小是 1KB）

```

##### 内存泄漏

```shell
- 危害：需要内存的进程，无法分配新内存；内存不足又会触发缓存回收以及SWAP机制，进而导致 I/O 的性能问题
- 标准库函数malloc() 来动态分配内存，用完内存后，调用库函数 free() 来释放它们。
```

