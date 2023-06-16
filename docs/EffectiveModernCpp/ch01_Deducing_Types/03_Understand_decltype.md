给一个名字或者表达式，`decltype` 返回其类型。大部分时候，它按照预期工作，不过极个别场合能让你头疼。

我们先从不会让我们感到惊讶的例子开始。`decltype` 近乎于鹦鹉学舌。
```cpp
const int i = 0;            // decltype(i) is const int
bool f(const Widget& w);    // decltype(w) is const Widget&
                            // decltype(f) is bool(const Widget&)
struct Point {
 int x, y;                  // decltype(Point::x) is int
};                          // decltype(Point::y) is int
Widget w;                   // decltype(w) is Widget
if (f(w))                   // decltype(f(w)) is bool
template<typename T>        // simplified version of std::vector
class vector {
public:
 T& operator[](std::size_t index);
};
vector<int> v;              // decltype(v) is vector<int>

if (v[0] == 0)              // decltype(v[0]) is int&
```

C++11 中，`decltype` 的一个常用的地方是写模板函数的返回类型，其依赖于函数参数的类型。比如一个函数接受一个容器对象，其支持索引 `operator[]` 运算符，返回的就是某个索引对应的对象，那么返回类型应该是该容器索引 `operator[]` 运算符的返回类型。

通常情况下，类型为 `T` 的容器的 `operator[]` 运算符返回类型是 `T&`。比如 `std::deque`，和大部分的 `std::vector`。对于 `std::vector<bool>` 而言，`operator[]` 返回的不是 `bool&` 而是一个新的对象。Item 6（TODO link）会探究其原因，不过，这里的重点是容器的 `operator[]` 返回类型依赖于容器。

## 
