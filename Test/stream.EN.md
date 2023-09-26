# STREAM for Lpi4A

## Usage Instructions

The STREAM benchmark test is a simple synthetic benchmark program that measures sustainable memory bandwidth (in MB/s) and the computational rate of simple vector kernels.

STREAM consists of a single file. To conduct the test, simply compile `stream.c`:

```
git clone <https://github.com/microseyuyu/STREAM.git>
cd STREAM
gcc -O3 -fopenmp -DN=2000000 -DNTIMES=10 stream.c -o stream
export OMP_NUM_THREADS=8
./stream
```

Parameter Explanation

-O3: Specifies the highest compilation optimization level, which is level 3.
-fopenmp: Enables OpenMP for multi-processor environments, which provides a better representation of the maximum real-world memory bandwidth. Once enabled, the program runs as many threads as there are CPU threads by default.
-DN=2000000: Specifies the size of the test arrays a[], b[], and c[]. This value significantly impacts the test results. In version 5.9, the default value is 2000000. If stream.c is version 5.10, the parameter name changes to -DSTREAM_ARRAY_SIZE with a default value of 10000000. Note: It's crucial to set the test array size significantly larger than the highest-level CPU cache (typically L3 Cache); otherwise, the test measures the throughput performance of the CPU cache and not memory throughput.
-DNTIMES=10: The number of executions, with the best result selected from these runs.
OMP_NUM_THREADS=8: The number of threads.

Reference Results (Tested on RevyOS 0810 version):

```
debian@lpi4a:~/STREAM$ ./stream
-------------------------------------------------------------
STREAM version $Revision: 5.10 $
-------------------------------------------------------------
...
-------------------------------------------------------------
Number of Threads requested = 8
Number of Threads counted = 8
-------------------------------------------------------------
...
-------------------------------------------------------------
Function    Best Rate MB/s  Avg time     Min time     Max time
Copy:            8333.3     0.019253     0.019200     0.019286
Scale:           8273.8     0.019399     0.019338     0.019489
Add:             6203.5     0.038816     0.038688     0.038931
Triad:           6237.7     0.038771     0.038476     0.039272
-------------------------------------------------------------
Solution Validates: avg error less than 1.000000e-13 on all three arrays
-------------------------------------------------------------

```
