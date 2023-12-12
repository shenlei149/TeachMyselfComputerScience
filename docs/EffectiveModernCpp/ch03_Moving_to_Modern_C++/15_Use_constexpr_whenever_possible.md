`constexpr` 用于对象时，是 `const` 的加强版，当用于函数时，意义就大不相同了。这一节主要讲解其含义。

`constexpr` 表明一个对象是 `const` 的，且在编译期确定。当应用于函数时，不能想当然认为函数会返回 `const` 对象，也不能再编译期确定其返回值。注意这是一个特性。

我们先讨论 `constexpr` 对象，它的值是在编译期确定的。

编译期确定的值享有特权。可以放到只读存储空间，这对于嵌入式而言是非常有用的。更广泛的应用是编译期确定的常量可以应用于需要整型常量表达式（`integral constant expression`）的上下文中，比如数组的大小，整数模版参数（`std::array` 的长度），枚举，对齐修饰符等等。如果需要在这些上下文中使用，那么就需要将变量声明为 `constexpr`，编译器保证它们在编译期确定其值。
```cpp
int sz;                            // non-constexpr variable
constexpr auto arraySize1 = sz;    // error! sz's value not known at compilation
std::array<int, sz> data1;         // error! same problem
constexpr auto arraySize2 = 10;    // fine, 10 is a compile-time constant
std::array<int, arraySize2> data2; // fine, arraySize2 is constexpr
```
使用 `const` 不能保证同样的事情，因为 `const` 变量不要求在编译期初始化。
```cpp
int sz;                          // as before
const auto arraySize = sz;       // fine, arraySize is const copy of sz
std::array<int, arraySize> data; // error! arraySize's value not known at compilation
```
简而言之，`constexpr` 对象都是 `const` 的，但是反之则不一定。如果在编译期初始化一个值，且能放到需要编译期常量的上下文中，那么就需要使用 `constexpr`。

将 `constexpr` 作用于函数时，如果实参是编译器确定的值，那么返回值是编译期常量；实参是运行时才能知道，那么函数返回一个运行时的值。
* `constexpr` 函数可以用于需要编译期常量的地方。如果你传入的实参是编译期确定的常量，那么函数返回一个编译期常量，否则，代码无法通过编译。
* 当传入 `constexpr` 函数的参数中有一个或者多个无法在编译期确定，那么行为和调用普通函数一样。这意味着我们不需要两个函数，一个为了编译期求值，一个为了运行时求值。一个 `constexpr` 函数就够了。

假定有这样一个需求。一个实验，有 $n$ 个变量，每个变量有三种可能性，那么共有 $3^n$ 种结果。假定 $n$ 编译期可知，那么 $3^n$ 理论上也是编译期可知的，那么使用 `std::array` 是一个合理的选择。为此，我们需要一个可以在编译期计算 $3^n$ 的函数。C++ 标准库提供了 `std::pow` 可以用于计算指数，不过有两个问题，第一个是返回值是浮点数而我们需要整数，第二个是这个函数不是 `constexpr` 的，无法在编译期求值。

我们可以自己实现一个。在此之前，我们看一下它是如何声明和使用的。

## Things to Remember
* `constexpr` objects are `const` and are initialized with values known during compilation.
* `constexpr` functions can produce compile-time results when called with arguments whose values are known during compilation.
* `constexpr` objects and functions may be used in a wider range of contexts than non-`constexpr` objects and functions.
* `constexpr` is part of an object's or function's interface.