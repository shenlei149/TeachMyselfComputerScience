如 word 或数据库这样的应用软件，数百万行代码，但是硬件只能执行非常有限的、简单的指令。这中间有很多很多的层次，这就是抽象的力量。这些层次大致由上至下分成三层：
* 应用软件
* 系统软件
* 硬件

系统软件（`systems software`）鉴于两者之间，承上启下，其中最重要的系统软件有两个：

一是操作系统（`operating system`），主要作用有
* 处理输入输出
* 分配管理内存
* 隔离保护同时运行的多个应用

二是编译器（`compilers`），将高级语言转化成低级指令。下面简单描述一下。

### From a High-Level Language to the Language of Hardware

