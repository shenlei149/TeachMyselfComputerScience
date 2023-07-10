0 是 `int` 类型，但是 C++ 在检查指针是否有效时，只能和 0 进行比较。但是 0 是 `int` 不是指针。

同样地，`NULL` 也是一个整数，甚至可能是 `long` 或者其他整数类型，这里重点不是具体类型，而是整数类型不是指针类型。0 和 `NULL` 都不是。

在 C++98 中，一个主要的影响是一个函数如果重载了整型和指针类型会导致奇怪的结果。传递 0 或者 `NULL` 并不会调用到接受指针作为参数类型的函数。
```cpp
void f(int); // three overloads of f
void f(bool);
void f(void *);

f(0);    // calls f(int), not f(void*)
f(NULL); // might not compile, but typically calls
         // f(int). Never calls f(void*)
```

当 `NULL` 是 `long` 时，`f(NULL)` 会编译失败，因为从 `long` 转化为 `int` `bool` `void*` 完全没有区别。但是这个语句实际想做的事情是调用 `f(void*)`。所以对于 C++98 程序员而言，推荐的做法是不要这样写重载函数，这对于 C++11 仍旧有效，因为这里推荐使用 `nullptr`，但是并不能阻碍程序员使用 `NULL` 或者 0。
