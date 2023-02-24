### Defining Performance
响应时间（`response time`），就是一个任务从开始到结束的时间，也称为执行时间（`execution time`）。

吞吐（`throughput`），指的是给定时间处理的任务数，也称为带宽（`bandwidth`）。

减少响应时间一定能提高吞吐，反之才不一定。

我们衡量 $X$ 的性能，使用如下公式
$$\text{Perf}_X=\frac{1}{\text{Eexe Time}}_X$$
定义 $X$ 比 $Y$ 快多少
$$\frac{\text{Perf}_X}{\text{Perf}_Y}=n$$
为了统一，一般都说性能提高多少倍，而不是混用性能提高和执行之间减少。

### Measuring Performance
时间本身有多种定义。

最直观的是挂钟时间（`wall clock time`），也称为运行时间或经过时间（`elapsed time`），就是程序执行所花的实际时间，包括 CPU 运算、访问内存、I/O 等等一切开销。

计算机上往往运行多个程序，操作系统往往是优化吞吐而不是一个程序的执行时间。CPU 执行时间（`CPU execution time`）单指消耗了多少 CPU 的时间，又可以分为用户 CPU 时间（`user CPU time`）和系统 CPU 时间（`system CPU time`）。

衡量 CPU 时间往往使用时钟周期（`clock cycles`, `clock ticks`, `clock period` 等）表示。还有一个概念是时钟频率（`clock rate`），是时钟周期的倒数。比如时钟周期是 250ps，时钟频率是 4GHz。
