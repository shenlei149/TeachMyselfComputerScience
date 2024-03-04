为了进行书中的实验，需要做一些前期准备工作。

### 实验环境
硬件：
```
Architecture:            x86_64
  CPU op-mode(s):        32-bit, 64-bit
  Address sizes:         46 bits physical, 57 bits virtual
  Byte Order:            Little Endian
CPU(s):                  2
  On-line CPU(s) list:   0,1
Vendor ID:               GenuineIntel
  Model name:            Intel(R) Xeon(R) Platinum 8370C CPU @ 2.80GHz
```

操作系统：
* OS: debian 12.5

相关软件：
* g++: 12.2
* perf: 6.1.76
* google-perftools: 2.10
* google benchmark: commit 1576991177ba97a4b2ff6c45950f1fa6e9aa678c

google benchmark 是在 [github](https://github.com/google/benchmark) 下载安装，其他软件均为 apt 自带。

随着操作系统的更新和相关软件的更新，可能后面的实验所用的系统或软件版本会高。

### Google benchmark
https://github.com/google/benchmark

安装：https://github.com/google/benchmark#installation

### g++
整个实验中常用以下几个参数：
* `-g` - 添加调试信息，方便 gdb 调试
* `-O` - 优化级别，实验中基本使用 `-O3`，最高级别的优化
* `-march` - 指定计算机架构，实验中基本使用 `-march=native`，编译器会使用当前主机的架构
* `-Wall` - 打开所有警告，帮助我们提升代码质量
* `-pedantic` - 严格限制仅使用 ISO C++ 标准
* `-o` - 指定输出的名字

### google-perftools
https://github.com/gperftools/gperftools

CPU 性能分析方法：
```
1) Link your executable with -lprofiler
2) Run your executable with the CPUPROFILE environment var set:
     $ CPUPROFILE=/tmp/prof.out <path/to/binary> [binary args]
3) Run pprof to analyze the CPU usage
     $ pprof <path/to/binary> /tmp/prof.out      # -pg-like text output
     $ pprof --gv <path/to/binary> /tmp/prof.out # really cool graphical output
```

根据更详细的文档，执行的时候可能需要添加环境变量 `LD_PRELOAD=/path/to/libprofiler.so` 在当前实验环境，具体变量是 `LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libprofiler.so`。

debian 系统安装完后并没有 `pprof` 的软链接，需要使用原始名字 `google-pprof`。一般情况下输出文本，所以具体命令是 `google-pprof --text /path/to/binary /path/to/perf/data`。
