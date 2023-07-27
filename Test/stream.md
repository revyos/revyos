# Stream for Lpi4A

## 使用说明

STREAM 基准测试是一个简单的综合基准测试程序，它测量可持续内存带宽（以 MB/s 为单位）和简单向量内核的相应计算速率。

stream 仅有单个文件，在进行测试时只需要对 `stream.c` 进行编译即可：

```
git clone <https://github.com/microseyuyu/STREAM.git>
cd STREAM
gcc -O3 -fopenmp -DN=2000000 -DNTIMES=10 stream.c -o stream
export OMP_NUM_THREADS=8
./stream
```

参数说明：

- -O3：指定最高编译优化级别，即 3。
- fopenmp：启用OpenMP，适应多处理器环境，更能得到内存带宽实际最大值。开启后，程序默认运行线程为CPU线程数

DN=2000000：指定测试数组a[]、b[]、c[]的大小（Array size）。该值对测试结果影响较大（5.9版本默认值2000000,。若stream.c为5.10版本，参数名变为-DSTREAM_ARRAY_SIZE，默认值10000000）。注意：必须设置测试数组大小远大于CPU 最高级缓存（一般为L3 Cache）的大小，否则就是测试CPU缓存的吞吐性能，而非内存吞吐性能。
- -DNTIMES=10：执行的次数，并从这些结果中选最优值。
- OMP_NUM_THREADS=8 线程数量。

参考结果：

```
debian@lpi4a:~/Desktop/STREAM$ ./stream
-------------------------------------------------------------
STREAM version $Revision: 5.10 $
-------------------------------------------------------------
This system uses 8 bytes per array element.
-------------------------------------------------------------
*****  WARNING: ******
      It appears that you set the preprocessor variable N when compiling this code.
      This version of the code uses the preprocesor variable STREAM_ARRAY_SIZE to control the array size
      Reverting to default value of STREAM_ARRAY_SIZE=10000000
*****  WARNING: ******
Array size = 10000000 (elements), Offset = 0 (elements)
Memory per array = 76.3 MiB (= 0.1 GiB).
Total memory required = 228.9 MiB (= 0.2 GiB).
Each kernel will be executed 10 times.
 The *best* time for each kernel (excluding the first iteration)
 will be used to compute the reported bandwidth.
-------------------------------------------------------------
Number of Threads requested = 8
Number of Threads counted = 8
-------------------------------------------------------------
Your clock granularity/precision appears to be 1 microseconds.
Each test below will take on the order of 21622 microseconds.
   (= 21622 clock ticks)
Increase the size of the arrays if this shows that
you are not getting at least 20 clock ticks per test.
-------------------------------------------------------------
WARNING -- The above is only a rough guideline.
For best results, please be sure you know the
precision of your system timer.
-------------------------------------------------------------
Function    Best Rate MB/s  Avg time     Min time     Max time
Copy:            8364.2     0.019258     0.019129     0.019508
Scale:           8291.0     0.019572     0.019298     0.020162
Add:             6223.6     0.038835     0.038563     0.040011
Triad:           6222.5     0.038776     0.038570     0.039470
-------------------------------------------------------------
Solution Validates: avg error less than 1.000000e-13 on all three arrays
-------------------------------------------------------------
```