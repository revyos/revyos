# CoreMark

## Summary

CoreMark is a comprehensive benchmark designed to measure the performance of central processing units (CPUs) used in embedded systems. Developed in 2009 by Shay Gal-On from EEMBC, it was intended to be an industry standard, replacing the outdated Dhrystone benchmark. The code is written in C and incorporates the following algorithms: list operations (add, delete, modify, search, and sort), matrix operations (common matrix tasks), state machine (determining if an input stream contains valid numbers), and CRC. Users can freely download CoreMark, port it to their platform, and run it to see their scores.

## Test File Introduction

Directory structure:
```
├── barebones          -- Directory for modifications when porting to a bare-metal environment
│   ├── core_portme.c       -- Platform configuration for porting
│   ├── core_portme.h       -- Timer and board-level initialization
│   ├── core_portme.mak     -- Makefile for this sub-directory
│   ├── cvt.c
│   └── ee_printf.c         -- Print function for serial port transmission
├── core_list_join.c   -- List operation program
├── core_main.c        -- Main program
├── coremark.h         -- Header file for project configuration and data structure definitions
├── coremark.md5       
├── core_matrix.c      -- Matrix operation program
├── core_state.c       -- State machine program
├── core_util.c        -- CRC computation program
├── cygwin             -- Test code for x86 cygwin and GCC 3.4 (quad-core, dual-core, and single-core systems)
│   ├── core_portme.c
│   ├── core_portme.h
│   └── core_portme.mak
├── freebsd            -- Test code for different operating systems (similar structure follows)
│   ├── ...
├── LICENSE.md
├── linux
│   ├── ...
├── linux64
│   ├── ...
├── macos
│   ├── ...
├── Makefile           
├── README.md          -- Introductory file for the CoreMark project
├── rtems
│   ├── ...
└── simple
    ├── ...
```

## Build Targets

- **run** - Default target, creates run1.log and run2.log
- **run1.log** - Run benchmark test with performance parameters and output to run1.log
- **run2.log** - Run benchmark test with verification parameters and output to run2.log
- **run3.log** - Run benchmark test using configuration file parameters and output to run3.log
- **compile** - Compile the benchmark executable
- **link** - Link the benchmark executable
- **check** - Check the MD5 of source files that may not have been modified
- **clean** - Remove temporary files

## Compilation/Usage

According to the README, one only needs to execute `make` in the `coremark` directory to compile and test. Test results will be found in the `Results` folder, where `run1.log` contains the test outcomes.

## Test Plan

The overall testing time for CoreMark is relatively short. The plan is to run tests 10 times and take the average.

## Steps to Reproduce

```
$ git clone https://github.com/microseyuyu/coremark.git
$ cd coremark
$ sudo make
```

## Analysis Report

```
2K performance run parameters for coremark.
CoreMark Size    : 666
Total ticks      : 12915
Total time (secs): 12.915000
Iterations/Sec   : 8517.228029
Iterations       : 110000
Compiler version : GCC13.1.0
...
CoreMark 1.0 : 8517.228029 / GCC13.1.0 -O2 -DPERFORMANCE_RUN=1  -lrt / Heap
```
