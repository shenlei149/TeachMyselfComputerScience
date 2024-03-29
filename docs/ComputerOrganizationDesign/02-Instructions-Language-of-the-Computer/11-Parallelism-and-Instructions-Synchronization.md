当任务独立时，并行执行（`parallel execution`）比较容易，不过仍旧需要同步。协作意味着一处理器写数据一些处理器处理这些数据。只有写完数据之后才能读，所以这些任务之间需要同步。如果不同步，可能会出现竞争问题（`data race`），那么结果完全依赖于这些任何执行的顺序。

在计算机系统中，同步机制依赖于硬件指令的支持，在用户态层面实现。下面会集中在加锁和释放锁（`lock, unlock`）机制上。利用这些，很容易实现临界区，只能有一个处理器进行操作，这也成为互斥（`mutual exclusion`），也可以实现更复杂的同步机制。

硬件需要有能力原子地读并且修改一个内存地址的值，即这些操作之间不能有其他指令。如果没有硬件支持，实现同步机制是非常耗时的。

硬件往往实现了原子地读、修改某个内存的值，且通过某种方式返回原子操作是否成功。一般用户不应该直接使用这些硬件层面的原语，而应该使用系统程序员写好的类库。

下面看一个基本的同步原语，原子交换（`atomic exchange, atomic swap`），交换寄存器和某个内存地址的值。

假定使用某个内存地址来实现锁机制，0 表示锁可用，1 表示锁已经被某个处理器加锁，不可用。某个处理器在寄存器中写入 1，然后与锁对应的内存值进行交换。如果返回值（内存的值）是 1，说明锁不可用，已经被其他处理器持有。如果返回值是 0，锁可用，同时把寄存器的 1 写入到对应内存，表示自己持有了锁，防止其他处理器再持有锁。

使用交换原语来实现同步的关键在于整个交换过程不能被打断，两个同时进行的交换由硬件来保证按序操作，不能让两个不同处理器都觉得是自己成功的设置了 1。

硬件实现的一种方式是使用一对指令，第二个指令返回某个值，表示这两个指令是否原子地执行了。如果任意其他处理器的指令都发生在这对指令之前或者之后，那么这对指令就是原子执行的。如果原子地执行了指令，说明在此期间，没有其他指令修改值。

RISC-V 中提供了这么一对特殊的指令：`lr.w`（`load-reserved word`）和 `sc.w`（`store-conditional word`）。在 `lr.w` 指定的内存在执行 `sc.w` 之间被修改了，那么 `sc.w` 返回失败，并不会写值到对应内存。`sc.w` 需要三个参数，一个寄存器保存要写到内存的值，一个保存内存的地址，一个是返回值，表示是否成功。`sc.w` 返回 0 表示成功，`lr.w` 里面存储着原始的值，下面的汇编表示原子地交换 `x20` 指向的内存地址的值。
```
again:  lr.w x10, (x20)         // load-reserved
        sc.w x11, x23, (x20)    // store-conditional
        bne x11, x0, again      // branch if store fails (0)
        addi x23, x10, 0        // put loaded value in x23
```
如果任意时刻有处理器在 `lr.w` 和 `sc.w` 修改了 `x20` 指向的内存的值，`sc.w` 将非零值写入 `x11`。当这几个指令执行完，`x23` 中的内存和 `x20` 指向的内存的值交换完成，且是原子地执行成功了。

尽管上述同步机制是为了在多个处理器之间同步，但是对于单个处理器处理多个进程也非常有用。如果处理器在两个指令之间做了上下文切换，那么 `sc.w` 也返回失败。

通过在这两个指令之间插入很少的指令，就可以实现原子地比较、交换值、读取然后自增等有用的并行编程模式。如果有其他处理器修改了指定内存的值或者发生异常 `sc.w` 返回失败而回到循环开始，所以只允许有算术运算和跳出这两个指令区间的跳转指令，否则可能会死锁。同时，这两条指令之间的指令应该尽可能地说，否则 `sc.w` 会频繁失败而导致性能变差。

下面是使用 `lr.w, sc.w` 实现锁机制的汇编代码。
```
// acquire lock
        addi x12, x0, 1         // copy locked value
again:  lr.w x10, (x20)         // load-reserved to read lock
        bne x10, x0, again      // check if it is 0 yet
        sc.w x11, x12, (x20)    // attempt to store new value
        bne x11, x0, again      // branch if store fails

// release lock
sw 0(x20)                       // free lock by writing 0
```
