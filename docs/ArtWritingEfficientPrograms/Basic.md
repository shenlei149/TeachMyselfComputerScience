## Learning about high performance
影响程序性能的因素有
* Choosing the right algorithm
* Using CPU resources effectively
* Using memory effectively
* Avoiding unnecessary computations
* Using concurrency and multi-threading effectively
* Using the programming language effectively, avoiding inefficiencies
* Measuring performance and interpreting results

这里不会涉及算法，但是会介绍后续几点，特别是最后一个，根据场景定义清楚性能指标是一件比较困难的问题，而测试之后，分析数据就更困难了，需要一个合理的解释。

这里涉及的话题有点类似 [Computer Organization and Design RISC-V Edition](/ComputerOrganizationDesign/)，不过两者的侧重不同，

## Performance measurements
性能分析调优的第一原则：不要瞎猜。Never guess about performance! 详情参考反直觉的[例子](/ArtWritingEfficientPrograms/FirstExample.md)。

在怎么优化之前，需要先问两个问题。
1. 需要优化吗？
2. 优化什么？

实践中，可能由于各种原因，比如客户可以接受当前的性能，当前程序耗时在整个业务中占比很低，人力资源问题等等，是不需要优化的，或者现在不需要优化。

在需要优化的前提下，需要知道优化什么？优化哪里？这就需要性能分析。通过一些手段，找出瓶颈点。

优化一个函数，基本思路就两条：运行的更快，或者调用次数更少（这一点往往容易被忽视）。
