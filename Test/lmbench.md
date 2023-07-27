# lmbench for Lpi4A

## 软件版本

> lmbench for revyOS


## 测试说明
| 带宽测评工具 | 反应时间测评工具 | 其他 |
| --- | --- | --- |
| 读取缓存文件 | 上下文切换 | \\ |
| 拷贝内存 | 网络：连接的建立，管道，TCP，UDP 和RPC hot potato | \\ |
| 读内存 | 文件系统的建立和删除 | \\ |
| 写内存 | 进程创建 | 处理器时钟比率计算 |
| 管道 | 信号处理 | \\ |
| TCP | 上层的系统调用 | \\ |
| \\ | 内存读入反应时间 | \\ |


## 测试准备

下载测试工具：

```
git clone https://github.com/revyos/lmbench3.git
```

> 该版本为已为 RevyOS 移植版本。


在开始测试前，需要先安装依赖：

```
sudo apt install gcc make libntirpc-dev -y
```

## 测试执行

执行命令进行编译，配置，并测试

```
cd lmbench3
cd src
make results
```

编译完成后会有以下选项提示需要设置：

以下不需要更改的项目直接回车，会自动设置默认值。

`MULTIPLE COPIES [default 1]`: 设置同时运行 lmbench 的份数，份数多会使 lmbench 运行缓慢，默认是 1，这里设置为默认值 1。

```
=====================================================================

If you are running on an MP machine and you want to try running
multiple copies of lmbench in parallel, you can specify how many here.

Using this option will make the benchmark run 100x slower (sorry).

NOTE:  WARNING! This feature is experimental and many results are 
	known to be incorrect or random!

MULTIPLE COPIES [default 1]: 
=====================================================================
```

`Job placement selection [default 1]`: 作业调度控制方法，默认值是 1，表示允许作业调度，这里设置为默认值。

```
=====================================================================

Options to control job placement
1) Allow scheduler to place jobs
2) Assign each benchmark process with any attendent child processes
   to its own processor
3) Assign each benchmark process with any attendent child processes
   to its own processor, except that it will be as far as possible
   from other processes
4) Assign each benchmark and attendent processes to their own
   processors
5) Assign each benchmark and attendent processes to their own
   processors, except that they will be as far as possible from
   each other and other processes
6) Custom placement: you assign each benchmark process with attendent
   child processes to processors
7) Custom placement: you assign each benchmark and attendent
   processes to processors

Note: some benchmarks, such as bw_pipe, create attendent child
processes for each benchmark process.  For example, bw_pipe
needs a second process to send data down the pipe to be read
by the benchmark process.  If you have three copies of the
benchmark process running, then you actually have six processes;
three attendent child processes sending data down the pipes and 
three benchmark processes reading data and doing the measurements.

Job placement selection [default 1]: 
=====================================================================
```

`Memory`: 设置测试内存大小，默认是 `$MB` , 即为程序计算出来的最大可测试内存，也可以手动定义测试值，这里设置为这里使用默认值。

```
=====================================================================

Several benchmarks operate on a range of memory.  This memory should be
sized such that it is at least 4 times as big as the external cache[s]
on your system.   It should be no more than 80% of your physical memory.

The bigger the range, the more accurate the results, but larger sizes
take somewhat longer to run the benchmark.

MB [default 686]: 
Checking to see if you have 686 MB; please wait for a moment...
686MB OK
686MB OK
686MB OK
Hang on, we are calculating your cache line size.
OK, it looks like your cache line is 64 bytes.

=====================================================================
```

`SUBSET (ALL|HARWARE|OS|DEVELOPMENT) [default all]`: 要运行的测试集，包含 `ALL/HARWARE/OS/DEVELOPMENT`，默认选 `all`，这里选 `all`。

```
=====================================================================

lmbench measures a wide variety of system performance, and the full suite
of benchmarks can take a long time on some platforms.  Consequently, we
offer the capability to run only predefined subsets of benchmarks, one
for operating system specific benchmarks and one for hardware specific
benchmarks.  We also offer the option of running only selected benchmarks
which is useful during operating system development.

Please remember that if you intend to publish the results you either need
to do a full run or one of the predefined OS or hardware subsets.

SUBSET (ALL|HARWARE|OS|DEVELOPMENT) [default all]: 
=====================================================================
```

`FASTMEM [default no]`: 内存 `latency` 测试，如果跳过该测试，则设置为 `yes`，如果不跳过则设置为 `no`，默认是 `no`，这里设置为默认值。

```
=====================================================================

This benchmark measures, by default, memory latency for a number of
different strides.  That can take a long time and is most useful if you
are trying to figure out your cache line size or if your cache line size
is greater than 128 bytes.

If you are planning on sending in these results, please don't do a fast
run.

Answering yes means that we measure memory latency with a 128 byte stride.  

FASTMEM [default no]: 
=====================================================================
```

`SLOWFS [default no]`: 文件系统 `latency` 测试，如果跳过值设置为 `yes`，不跳过设置为 `no`，默认 `no`，这里设置为默认值。

```
=====================================================================

This benchmark measures, by default, file system latency.  That can
take a long time on systems with old style file systems (i.e., UFS,
FFS, etc.).  Linux' ext2fs and Sun's tmpfs are fast enough that this
test is not painful.

If you are planning on sending in these results, please don't do a fast
run.

If you want to skip the file system latency tests, answer "yes" below.

SLOWFS [default no]: 
=====================================================================
```

`DISKS [default none]`: 硬盘带宽和 `seek time`，需要设置测试硬盘的盘符，例如 `/dev/sda`，默认不测试(默认 none )，这里设置为默认值。

```
=====================================================================

This benchmark can measure disk zone bandwidths and seek times.  These can
be turned into whizzy graphs that pretty much tell you everything you might
need to know about the performance of your disk.  

This takes a while and requires read access to a disk drive.  
Write is not measured, see disk.c to see how if you want to do so.

If you want to skip the disk tests, hit return below.

If you want to include disk tests, then specify the path to the disk
device, such as /dev/sda.  For each disk that is readable, you'll be
prompted for a one line description of the drive, i.e., 

	Iomega IDE ZIP
or
	HP C3725S 2GB on 10MB/sec NCR SCSI bus

DISKS [default none]: 
=====================================================================
```

`REMOTE [default none]`: 网络测试，需要 `2` 台机器并设置 `rsh`，是测试机器能 `rsh` 访问另一台，默认不测试(默认 none )，这里设置为默认值。

```
=====================================================================

If you are running on an idle network and there are other, identically
configured systems, on the same wire (no gateway between you and them),
and you have rsh access to them, then you should run the network part
of the benchmarks to them.  Please specify any such systems as a space
separated list such as: ether-host fddi-host hippi-host.

REMOTE [default none]: 
=====================================================================
```

`Processor mhz [default 999 MHz, 1.0010 nanosec clock]`: 测试 `cpu`，默认 `$MHZ`，即为程序判断出的频率，也可以根据情况自己设定，例如 3500，单位 `MHz`，这里设置为默认值。

```
=====================================================================

Calculating mhz, please wait for a moment...
I think your CPU mhz is 

	999 MHz, 1.0010 nanosec clock
	
but I am frequently wrong.  If that is the wrong Mhz, type in your
best guess as to your processor speed.  It doesn't have to be exact,
but if you know it is around 800, say 800.  

Please note that some processors, such as the P4, have a core which
is double-clocked, so on those processors the reported clock speed
will be roughly double the advertised clock rate.  For example, a
1.8GHz P4 may be reported as a 3592MHz processor.

Processor mhz [default 999 MHz, 1.0010 nanosec clock]: 
=====================================================================
```

`FSDIR [default /usr/tmp]`: 临时目录用来存放测试文件，可以自己设定，默认 `/usr/tmp`，这里设置为默认值。

```
=====================================================================

We need a place to store a 686 Mbyte file as well as create and delete a
large number of small files.  We default to /usr/tmp.  If /usr/tmp is a
memory resident file system (i.e., tmpfs), pick a different place.
Please specify a directory that has enough space and is a local file
system.

FSDIR [default /usr/tmp]: 
=====================================================================
```

`Status output file [default /dev/tty]`: 测试输出信息文件存放目录，可以自己设定，默认 `/dev/tty`。

```
=====================================================================

lmbench outputs status information as it runs various benchmarks.
By default this output is sent to /dev/tty, but you may redirect
it to any file you wish (such as /dev/null...).

Status output file [default /dev/tty]: 
=====================================================================
```

`Mail results [default yes]`: 是否将测试结果邮件发出来，默认是 `yes`，这里设置为 `no`。

```
=====================================================================

There is a database of benchmark results that is shipped with new
releases of lmbench.  Your results can be included in the database
if you wish.  The more results the better, especially if they include
remote networking.  If your results are interesting, i.e., for a new
fast box, they may be made available on the lmbench web page, which is

	http://www.bitmover.com/lmbench

Mail results [default yes]: no
OK, no results mailed.
=====================================================================
```

以上项目设置完成后，开始自动执行测试。

## 测试结果


## 测试说明