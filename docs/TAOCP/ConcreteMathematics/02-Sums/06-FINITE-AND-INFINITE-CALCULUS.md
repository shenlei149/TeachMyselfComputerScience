从更高的层次审视这个问题。这里会借助与传统无限积分的概念类似的有限积分的概念来求和。

无限微分
$$Df(x)=\lim_{h\to 0}\frac{f(x+h)-f(x)}{h}$$
$D$是微分算子（`derivative`）。有限积分
$$\Delta f(x)=f(x+1)-f(x)\tag{2.42}$$
$\Delta$是差分（`difference`）。差分算子是微分算子的有限模拟，$h$只能取正整数，当$h\to 0$时，$h=1$是能达到的极限了，$\Delta f(x)$是$(f(x+h)-f(x))/h$取$h=1$时的值。

符号$D$、$\Delta$是算子（`operator`），给定一个函数，会产生一个新的函数。如果$f$是实数到实数的较为光滑的函数，那么$Df$也是一个实数到实数的函数。如果$f$是任意一个实数到实数的函数，$\Delta f$也是。函数$Df$、$\Delta f$在$x$处的定义如上。

如果$f=x^m$，那么微积分中$Df=mx^{m-1}$，也就是
$$D(x^m)=mx^{m-1}$$
但是算子$\Delta$不能得到同样优美的结果。比如
$$\Delta(x^3)=(x+1)^3-x^3=3x^2+3x+1$$
但是有一类$m$的次幂在$\Delta$的作用下可以得到优美的结果，这就是有限微积分的意义所在。新型的$m$次幂有规则
$$x^{\underline{m}}=x(x-1)\cdots(x-m+1),m\ge 0\tag{2.43}$$
定义。注意$m$下面有个横线，表示从$x$开始阶梯般的一直向下。类似的，因子可以一直向上
$$x^{\overline{m}}=x(x+1)\cdots(x+m-1),m\ge 0\tag{2.44}$$
当$m=0$时，$x^{\underline{0}}=x^{\overline{0}}=1$。通常没有因子相乘时默认值是1。

这些函数称为下降阶乘幂（`falling factorial power`）和上升阶乘幂（`rising factorial power`），因为和阶乘$n!=n(n-1)\cdots 1$密切相关，有$n!=n^{\underline{n}}=1^{\overline{n}}$。

下降阶乘幂$x^{\underline{m}}$与$\Delta$运算得到
$$\begin{aligned}
\Delta(x^{\underline{m}})&=(x+1)^{\underline{m}}-x^{\underline{m}}\\
&=(x+1)(x)\cdots(x-m)-x(x-1)\cdots(x-m+1)\\
&=mx(x-1)\cdots(x-m)
\end{aligned}$$
类似于$D(x^m)=mx^{m-1}$
$$\Delta(x^{\underline{m}})=mx^{\underline{m-1}}\tag{2.45}$$

微分算子$D$的逆运算是积分算子$\int$，它们之间的关系是
$$g(x)=Df(x) \Leftrightarrow \int g(x)dx=f(x)+C$$
类似的，$\Delta$的逆运算是求和$\sum$，两个之间关系是
$$g(x)=\Delta f \Leftrightarrow \sum g(x)\delta x=f(x)+C\tag{2.46}$$
$\sum g(x)\delta x$是$g(x)$的不定和式（`the indefinite sum`）。注意，$\delta,\Delta$是成对的，和$d,D$一样。不定积分中的$C$是常数，而不定和式中的$C$是任意满足$p(x+1)=p(x)$的函数$p(x)$。比如$C$可以是周期函数$a+b\sin 2\pi x$，去查分的时候，这样的函数会被消去，就和求导是常数被消除一样。在$x$为整数时，函数$C$是常数。

无限积分的定积分：如果$g(x)=Df(x)$，那么
$$\int_a^b g(x)dx=f(x)\bigg|_a^b=f(b)-f(a)$$
类似的，如果$g(x)=\Delta f(x)$，那么
$$\sum_a^b g(x)\delta x=f(x)\bigg|_a^b=f(b)-f(a)\tag{2.47}$$
这个公式让$\sum_a^b g(x)\delta x$有了定义。但是实际意义是什么呢？我们只是类似微积分定义了这些公式，好处是容易记忆，但是如果我们不理解它的意义，那这些符号就没有用处了。先从特殊情形入手。假设$g(x)=\Delta f(x)=f(x+1)-f(x)$。如果$b=a$，有
$$\sum_a^a g(x)\delta x=f(a)-f(a)=0$$
如果$b=a+1$，那么有
$$\sum_a^{a+1} g(x)\delta x=f(a+1)-f(a)=g(a)$$
更一般地，如果$b$增加1就有
$$\begin{aligned}
\sum_a^{b+1} g(x)\delta x-\sum_a^b g(x)\delta x&=(f(b+1)-f(a))-(f(b)-f(a))\\
&=f(b+1)-f(b)\\
&=g(b)
\end{aligned}$$
根据观察和数学归纳法，可以推出当$a,b$都是正整数且$b\ge a$时，$\sum_a^b g(x)\delta x$的确切含义是
$$\sum_a^b g(x)\delta x=\sum_{k=a}^{b-1}g(k)=\sum_{a\le k<b}g(k),b\ge a\tag{2.48}$$
也就是说，确定的和式和一般带上下界限的和式是相同的，差了一个上限值。
