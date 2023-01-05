### 1
这个题目没有统一的答案。我心中的解是参考答案的第一个，这个式子本身的指标集合是空集，和式为零。第二种是说对 $k$ 递减求和，即 $q_4+q_3+q_2+q_1+q_0$，但是这与 $n=0$ 时 $\sum_{k=1}^n q_k=0$ 的约定矛盾。第三种含义是 $\sum_{k=m}^nq_k=\sum_{k\leq n}q_k-\sum_{k<m}q_k$，那么题目的和式就等于 $-q_1-q_2-q_3$。

### 2
当 $x>0$ 时，$x\times ([x>0]-[x<0])=x\times (1-0)=x$，当 $x<0$ 时，$x\times ([x>0]-[x<0])=x\times (0-1)=-x$，所以 $x\times ([x>0]-[x<0])=|x|$。

### 3
第一个和式 $k\in \{0,1,2,3,4,5\}$，所以
$$\sum_{0\leq k\leq 5}a_k=a_0+a_1+a_2+a_3+a_4+a_5$$
第二个和式 $k\in \{-2,-1,0,1,2\}$，所以
$$\sum_{0\leq k^2\leq 5}a_{k^2}=a_4+a_1+a_0+a_1+a_4$$

### 4
满足集合条件的 $ijk$ 总共只有四种情况，分别是 $123,124,134,234$。  
先对 $k$ 求和，再对 $j,i$ 求和。就是
$$((a_{123}+a_{124})+a_{134})+a_{234}$$
第二个是
$$a_{123}+(a_{124}+(a_{134}+a_{234}))$$

### 5
第二个等式左右两边不同的指标 $j,k$ 变成了一个指标 $k$。这一点只有当对任意 $0\leq j,k\leq n$ 时，有 $a_j=a_k$ 才成立。

### 6
当 $1\leq j\leq n$ 时，和式的含义是 $[j,n]$ 的个数，所以和式值是 $[1\leq j\leq n](n-j+1)$。

### 7
$$\begin{aligned}
\nabla(x^{\overline{m}})&=x^{\overline{m}}-(x-1)^{\overline{m}}\\
&=x(x+1)(x+2)\cdots(x+m-2)(x+m-1)-(x-1)x(x+1)\cdots(x+m-2)\\
&=x(x+1)\cdots(x+m-2)(x+m-1-x+1)\\
&=mx(x+1)\cdots(x+m-2)\\
&=mx^{\overline{m-1}}
\end{aligned}$$

### 8
当 $m\geq 1$ 时，包含因子 0，所以整个式子的值是 0。  
当 $m\leq 0$ 时，根据负指数的定义
$$0^{\underline{m}}=\frac{1}{1\cdot 2\cdot 3\cdots (|m|)}=\frac{1}{|m|!}$$

### 9
等式 $(2.52)$ 需要改写为
$$x^{\overline{m+n}}=x^{\overline{m}}(x+m)^{\overline{n}}$$
令 $m=-n$，那么上式可以写作
$$x^{\overline{0}}=1=x^{\overline{-n}}(x-n)^{\overline{n}}$$
那么
$$\begin{aligned}
x^{\overline{-n}}&=\frac{1}{(x-n)^{\overline{n}}}\\
&=\frac{1}{(x-n)(x-n+1)\cdots(x-n+(n-1))}\\
&=\frac{1}{(x-1)(x-2)\cdots(x-n)}\\
&=\frac{1}{(x-1)^{\underline{n}}}
\end{aligned}$$

### 10
推导公式 $(2.54)$ 稍微变换一下中间项的展开和组合方式可以得到
$$\begin{aligned}
\Delta(u(x)v(x))&=u(x+1)v(x+1)-u(x)v(x)\\
&=u(x+1)v(x+1)-u(x+1)v(x)+u(x+1)v(x)-u(x)v(x)\\
&=u(x+1)\Delta v(x)+v(x)\Delta u(x)\\
&=Eu\Delta v+v\Delta u
\end{aligned}$$
得到了另外一个和 $(2.54)$ 对称的结果。

### 11
$$\begin{aligned}
\sum_{0\leq k<n}(a_{k+1}-a_k)b_k&=\sum_{0\leq k<n}a_{k+1}b_k-\sum_{0\leq k<n}a_kb_k\\
&=\sum_{0\leq k<n}a_{k+1}b_k-(\sum_{-1\leq k<n-1}a_{k+1}b_{k+1})\\
&=\sum_{0\leq k<n}a_{k+1}b_k-(\sum_{0\leq k<n-1}a_{k+1}b_{k+1}+a_0b_0)\\
&=\sum_{0\leq k<n}a_{k+1}b_k-(\sum_{0\leq k<n}a_{k+1}b_{k+1}+a_0b_0-a_nb_n)\\
&=a_nb_n-a_0b_0+\sum_{0\leq k<n}a_{k+1}(b_k-b_{k+1})\\
&=a_nb_n-a_0b_0-\sum_{0\leq k<n}a_{k+1}(b_{k+1}-b_k)
\end{aligned}$$

### 12
令 $p(k)=n=k+(-1)^kc$，两边同时加上$c$得到
$$n+c=k+((-1)^k+1)c$$
那么
$$(-1)^{n+c}=(-1)^{k+((-1)^k+1)c}=(-1)^k\cdot(-1)^{((-1)^k+1)c}$$
由于 $((-1)^k+1)$ 是个偶数，即零或者二，所以 $(-1)^{((-1)^k+1)c}=1$，那么
$$(-1)^{n+c}=(-1)^k$$
代入到 $p(k)$ 的定义中得到
$$n=k+(-1)^{n+c}c$$
所以
$$k=n-(-1)^{n+c}c$$
那么对于任意 $n$，可以反推出来原始的 $k$。

### 13
令
$$\begin{aligned}
R_0&=\alpha\\
R_n&=R_{n-1}+(-1)^n(\beta+\gamma n+\delta n^2)
\end{aligned}$$
那么
$$R_n=A(n)\alpha+B(n)\beta+C(n)\gamma+D(n)\delta$$
要求的和式是
$$\sum_{k=0}^n(-1)^kk^2$$
对应递归式得到对应的 $\alpha=\beta=\gamma=0,\delta=1$，也就是 $D(n)$ 就是要求的和式。

令 $R_n=(-1)^nn$，那么 $R_0=0=\alpha$。
$$\begin{aligned}
(-1)^nn&=(-1)^{n-1}(n-1)+(-1)^n(\beta+\gamma n+\delta n^2)\\
(-1)n&=(n-1)+(-1)(\beta+\gamma n+\delta n^2)\\
-n&=n-1-\beta-\gamma n-\delta n^2\\
-2n+1&=-\beta-\gamma n-\delta n^2
\end{aligned}$$
所以
$$\beta=-1,\gamma=2,\delta=0$$
那么
$$(-1)^nn=-B(n)+2C(n)$$

令 $R_n=(-1)^nn^2$，那么 $R_0=0=\alpha$。
$$\begin{aligned}
(-1)^nn^2&=(-1)^{n-1}(n-1)^2+(-1)^n(\beta+\gamma n+\delta n^2)\\
(-1)n^2&=(n-1)^2+(-1)(\beta+\gamma n+\delta n^2)\\
-n^2&=n^2-2n+1-\beta-\gamma n-\delta n^2\\
-2n^2+2n-1&=-\beta-\gamma n-\delta n^2
\end{aligned}$$
所以
$$\beta=1,\gamma=-2,\delta=2$$
那么
$$(-1)^nn^2=B(n)-2C(n)+2D(n)$$
可以得到 $D(n)$
$$2D(n)=(-1)^nn^2+(-B(n)+2C(n))=(-1)^nn^2+(-1)^nn=(-1)^n(n^2+n)$$

### 14
由于 $k=\sum_{1\leq j\leq k}1$，所以 $\sum_{1\leq k\leq n}k2^k$ 可以改写成 $\sum_{1\leq j\leq k\leq n}2^k$。先对 $k$ 求和。
$$\begin{aligned}
\sum_{1\leq j\leq k\leq n}2^k&=\sum_{1\leq j\leq n}\sum_{j\leq k\leq n}2^k\\
&=\sum_{1\leq j\leq n}(2^j(2^{n-j+1-1}))\\
&=\sum_{1\leq j\leq n}(2^{n+1}-2^j)\\
&=\sum_{1\leq j\leq n}2^{n+1}-\sum_{1\leq j\leq n}2^j\\
&=n2^{n+1}-(2(2^n-1))\\
&=n2^{n+1}-(2^{n+1}-2)\\
\end{aligned}$$
