先看一个非常简单的例子。
```cpp
int x;
```
忘记初始化了。

再看下面的例子，解引用一个迭代器来初始化一个局部变量。
```cpp
template<typename It>       // algorithm to dwim ("do what I mean")
void dwim(It b, It e)       // for all elements in range from
{ // b to e
    while (b != e) {
        typename std::iterator_traits<It>::value_type currValue = *b;
    }
}
```

类型 `typename std::iterator_traits<It>::value_type` 之长，让写代码的乐趣少了很多。

第三个例子是想声明一个类型为闭包的局部变量，但是闭包的类型只有编译器知道，写不出来。。。这也是为什么闭包和 `auto` 同时出现在 C++11 的原因，只有闭包而不引入 `auto` 就没法写代码了。

`auto` 能够很好的解决这三个问题。

首先 `auto` 从初始化语句推导类型，这样就不会再忘记初始化变量了。
```cpp
int x1;         // potentially uninitialized
auto x2;        // error! initializer required
auto x3 = 0;    // fine, x's value is well-defined
```

`auto` 可以替代长长的类型名。
```cpp
template<typename It>       // as before
void dwim(It b, It e)
{
    while (b != e) {
        auto currValue = *b;
    }
}
```

由于 `auto` 使用 [Item 2](/EffectiveModernCpp/ch01_Deducing_Types/02_Understand_auto_type_deduction.md) 中解释的推导原则，可以表示只有编译器知道的类型，所以可以写闭包了。
```cpp
auto derefUPLess =                          // comparison func.
    [](const std::unique_ptr<Widget>& p1,   // for Widgets
       const std::unique_ptr<Widget>& p2)   // pointed to by
    { return *p1 < *p2; };                  // std::unique_ptrs
```

C++14 中，允许闭包参数用 `auto`，那么可以进一步简化写法。
```cpp
auto derefLess =            // C++14 comparison
    [](const auto& p1,      // function for
       const auto& p2)      // values pointed
    { return *p1 < *p2; };  // to by anything
                            // pointer-like
```

有些人可能会有疑问，这里并不一定要使用闭包，可以用 `std::function ` 替代。我们下面就解释一下两者之间的区别和联系。

C++11 引入的 `std::function` 可以理解成函数指针，不过函数指针只能指向函数，但是 `std::function` 可以存一些可以调用的对象。就像使用函数指针必须指定类型一样，使用 `std::function` 也必须指定类型。比如为了定义一个 `std::function` 对象，其指向如下签名的可调用对象。
```cpp
bool(const std::unique_ptr<Widget>&,    // C++11 signature for std::unique_ptr<Widget>
     const std::unique_ptr<Widget>&)    // comparison function
```

其变量名是 `func`
```cpp
std::function<bool(const std::unique_ptr<Widget>&,
                   const std::unique_ptr<Widget>&)> func;
```

因为 lambda 返回的是可调用对象，所以闭包可以存到 `std::function` 对象中。如果不用 `auto`，就必须写成
```cpp
std::function<bool(const std::unique_ptr<Widget>&,
                   const std::unique_ptr<Widget>&)>
    derefUPLess = [](const std::unique_ptr<Widget>& p1,
                     const std::unique_ptr<Widget>& p2)
                    { return *p1 < *p2; };
```

除了语法上的冗余以外，`std::function` 和 `auto` 并不完全一样。`auto` 声明的闭包对象和闭包同类型，内存占用也和闭包所需内存一样。`std::function` 模板类实例化的过程，会占用固定大小的内存，如果无法放下闭包，那么会从堆上分配内存。所以通常 `std::function` 比 `auto` 占用内存要高。由于内联和间接调用函数返回等限制，``std::function` 比 `auto` 要慢。再加上 `auto` 写法简洁，`auto` 往往是更好的选择。（类似的，为了存储函数返回的结果，lambda 也比 `std::bind` 要好很多，详见 Item 34（TODO link）。

`auto` 还能避免类型错误的问题。比如
```cpp
std::vector<int> v;
unsigned sz = v.size();
```


