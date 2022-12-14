使用并发的原因有两个：关注内容的分离和性能。

### Using concurrency for separation of concerns
写代码的时候经常需要隔离关注点，把相关的代码放一起，不相关的分开，这样程序容易理解、测试、可能 bug 更少。通过并发能把需要同时发生的事情分开，否则，需要写一个任务切换或者调用不相关的代码。

举个例子，DVD 播放器需要能够一遍加载数据播放一边响应用户的请求（比如暂停）。那么两个线程，各做其中一件事，分离。

上面的例子说明了多线程提高响应能力。有时一个后台线程需要一直做一件事，比如桌面搜索应用需要监控文件的变化。使用线程能够使得每个线程的逻辑都更简单，因为交互只在确定的几个点上，否则需要把不同任务代码散落在各处。

这种情况线程数不必须和 CPU 核数一样，因为这里的线程基于概念设计的，而不是为了最大化吞吐量。

### Using concurrency for performance: task and data parallelism
多处理器出现了很多年，不过更多的是在服务器领域。对于家用的 CPU，单核，开发者啥也不用做，等着 CPU 升级就能运行的更快。但是这样的日子一去不复返了，因为多核处理器的普及。

有两种并发提高性能的方式。第一个是把一个任务拆成多份同时运行，称为任务并行（`task parallelism`）。听起来很直接，但其实很复杂。第二种方式是数据并行（`data parallelism`），同样的操作，应用于不同的数据。

容易采用第一种方式并发的算法称为尴尬并行（`embarrassingly parallel`）。拆解任务和实现算法太容易了，使得我们尴尬。也称为自然并行（`naturally parallel`）或者方便并发（`conveniently concurrent`）。这些算法的扩展性往往很好。还有一类没有那么尴尬的算法，只能拆成固定几个部分。第八章和第十章会讨论这些。

第二种并发方式适合解决更大的问题。单个数据处理的速度是不变的。数据并行不能解决所有问题，但是吞吐量可以增加很大。比如视频处理，可以并行的处理图片的不同部分。

### When not to use concurrency
知道什么时候不用并发和知道什么时候用并发一样重要。不用的理由很简单，收益抵不上开销。并发程序需要更多时间开发和维护，复杂性可能会引入 bug，也更难理解。如果抵不上性能提升或者分离关注点带来的收益，那么就不应该用。

操作系统需要为新线程分配内核资源并且需要调度，那么如果一个线程的任务运行很快，甚至比启动线程还快，就没有性能提升，或者不及预期。

线程是一种有限的资源。太多的线程会使得系统变慢。额外的栈空间也是个问题。第九章介绍的线程池是可以限制线程数量，但也并不是银弹，也有自己的问题。

假设对于客户端/服务器架构，每个应用都使用独立的线程来处理请求。如果连接数很少那么一切正常。连接数很大的话很快会消耗完系统资源。谨慎使用线程池可以达到最佳性能。

运行的线程过多那么操作系统切换上下文的开销就越大。本质上能用在有用工作上的资源就会减少。所以线程数增加到一定程度之后就不会再增加性能了，反而会下降。追求最佳性能需要考虑可用的硬件来调整线程数量。

使用并发来提升性能和像其他优化手段一样，能潜在提升性能，但也会使代码变得复杂、难以理解、容易出 bug。确定性能收益大，或者单纯为了分离关注点，那么可以使用多线程来优化。
