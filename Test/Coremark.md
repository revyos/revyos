## 摘要
CoreMark是一个综合基准，用于测量嵌入式系统中使用的中央处理器(CPU)的性能。它是在2009由eembc的shay gal-on开发的，旨在成为一个行业标准，取代过时的dehrystone基准。代码用C编写，包含以下算法：列表处理(增删改查和排序)、矩阵操作(公共矩阵操作)、状态机(确定输入流是否包含有效数字)和CRC。用户可以自由的下载Coremark，并移植到自己的平台上运行，随后就可以看到分数。
## 测试项介绍
### 文件介绍

```
├── barebones	 --移植到裸机环境下需要修改的目录
│   ├── core_portme.c		--移植的目标平台配置信息
│   ├── core_portme.h		--计时以及板级初始化实现
│   ├── core_portme.mak		--该子目录的makefile
│   ├── cvt.c
│   └── ee_printf.c			--打印函数串口发送实现
├── core_list_join.c	--列表操作程序
├── core_main.c			--主程序
├── coremark.h			--项目配置与数据结构的定义头文件
├── coremark.md5		
├── core_matrix.c		--矩阵运算程序
├── core_state.c		--状态机控制程序
├── core_util.c			--CRC计算程序
├── cygwin				--x86 cygwin和gcc 3.4（四核，双核和单核系统）的测试代码
│   ├── core_portme.c
│   ├── core_portme.h
│   └── core_portme.mak
├── freebsd				--以下同理，是在不同操作系统下的测试代码
│   ├── ...
├── LICENSE.md
├── linux
│   ├── ...
├── linux64
│   ├── ...
├── macos
│   ├── ...
├── Makefile			
├── README.md			--自述文件，CoreMark项目的基本介绍
├── rtems
│   ├── ...
└── simple
    ├── ...
    └──
```

### 制作目标

- run - 默认目标，创建 run1.log 和 run2.log
- run1.log - 用性能参数运行基准测试，并输出到 run1.log
- run2.log - 用验证参数运行基准测试，并输出到 run2.log
- run3.log - 用配置文件生成参数运行基准测试，并输出到 run3.log
- complie - 编译基准测试可执行文件
- link - 链接基准测试可执行文件
- check - 测试可能未修改的源文件的 MD5
- clean - 清理临时文件
### 编译/使用
根据 README 说法，只需要在 coremark 文件夹下执行 make 即可进行编译与测试。测试结果会出现在 Resluts 文件夹中，其中 run1.log 是测试结果。
## 测试计划
coremark 总体测试时间较短。
计划同时测试 10 次取平均数值。
## 操作步骤

```
$ git clone https://github.com/microseyuyu/coremark.git
$ cd coremark
$ sudo make
```

## 分析报告

```
2K performance run parameters for coremark.	(Run type)
CoreMark Size    	: 666					(Buffer size)
Total ticks			: 25875					(platform dependent value)
Total time (secs) 	: 25.875000				(actual time in seconds)
Iterations/Sec 		: 3864.734300			(Performance value to report)
Iterations			: 100000				(number of iterations used)
Compiler version	: GCC3.4.4				(Compiler and version)	
Compiler flags		: -O2					(Compiler and linker flags)
Memory location		: Code in flash, data in on chip RAM
seedcrc				: 0xe9f5				(identifier for the input seeds)
[0]crclist			: 0xe714				(validation for list part)
[0]crcmatrix		: 0x1fd7				(validation for matrix part)
[0]crcstate			: 0x8e3a				(validation for state part)
[0]crcfinal			: 0x33ff				(iteration dependent output)
Correct operation validated. See README.md for run and reporting rules.  (*Only when run is successful*)
CoreMark 1.0 : 6508.490622 / GCC3.4.4 -O2 / Heap 						  (*Only on a successful performance run*)
```
