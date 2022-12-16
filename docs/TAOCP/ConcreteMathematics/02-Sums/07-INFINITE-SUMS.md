当前处理的问题都是假设有有限多个非零项求和。我们还是不得不面对无限的情况。

坏消息是事实表明当涉及无限和式的时候，处理$\sum$时所用的方法并不总是有效。好消息是存在一大类容易理解的无限和式，对它们做所有运算都是合法的。在我们完全审视这些和式之和，会对这两个消息理解的更透彻。

对于有限和式，一项一项相加，直到都加起来。对于无限和式，需要仔细的定义，否则就会出现荒谬的结果。

例如，可以如下定义一个无限和式
$$S=1+\frac{1}{2}+\frac{1}{4}+\frac{1}{8}+\cdots$$
和是2。因为两边加倍就得到
$$2S=2+1+\frac{1}{2}+\frac{1}{4}+\cdots=2+S$$
按照同样的方法我们定义
$$T=1+2+4+8+\cdots$$
两边加倍得到
$$2T=2+4+8+16+\cdots=T-1$$
那么和是-1，这明显不对，无限正数相加得到一个负数。应该说，$T=\infty$，相加的项会大于任意指定的数。$\infty$是$2T-1$的解，也是$2S=2+S$的“解”。

现在尝试对$\sum_{k\in K}a_k$构思一个好的定义，其中$K$可以是无限的。首先假设$a_k$是非负数，这样就不难找到一个定义：如果有一个常数$A$为界，使得对所有有限子集$F\subset K$都有
$$\sum_{k\in F}a_k\leq A$$
就定义了$\sum_{k\in K}a_k$为满足条件的最小的$A$。如果常数$A$没有界，那么就说$\sum_{k\in K}a_k=\infty$，这意味给定任意$A$，都有有限项$a_k$之和大于$A$。