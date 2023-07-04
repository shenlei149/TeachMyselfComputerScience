[Item 5](/EffectiveModernCpp/ch02_auto/05_Prefer_auto_to_explicit_type_declarations.md) 列出了很多使用 `auto` 的优势。不过，有的时候 `auto` 的推导的类型并不是我们想要的。比如有一个接受 `Widget` 返回 `std::vector<bool>` 的函数，每一个 `bool` 表示 `Widget` 是否提供特定的能力。
```cpp
std::vector<bool> features(const Widget& w);
```

假定 bit 5 表示传入的 `Widget` 是否是高优先级的。我们可能会写如下代码：
```cpp
Widget w;
bool highPriority = features(w)[5];     // is w high priority?
processWidget(w, highPriority);         // process w in accord with its priority
```

上述代码工作的很好，如果将 `highPriority` 显式声明的类型改为 `auto`：
```cpp
auto highPriority = features(w)[5];     // is w high priority?
```

代码能够通过编译，但是行为不可预期了。
```cpp
processWidget(w, highPriority);         // undefined behavior!
```

原因是当使用 `auto` 之后，`highPriority` 类型不再是 `bool` 而是 `std::vector<bool>::reference`。`std::vector::operator[]` 返回的类型一般情况是 `T&`，但是 `std::vector<bool>` 不是，因为 C++ 的规范使得无法引用一个具体的 bit。`std::vector<bool>::reference` 的行为很像 `bool&`，可以转化成 `bool`（不是 `bool&`），所以显式地写类型 `bool` 是没有问题的。

具体行为依赖于实现。`std::vector<bool>::reference` 往往包含一个指向某个包含该 bit 的字，外加一个偏移量。这是一个临时对象。这个临时对象拷贝给了 `highPriority`，等语句结束的时候，这个临时对象就销毁了。`highPriority` 包含了一个悬垂指针。

`std::vector<bool>::reference` 是代理类（`proxy class`）的例子。第四章（TODO link）介绍的智能指针也是代理类的例子。

`std::shared_ptr` 和 `std::unique_ptr` 对用户可见，而 `std::vector<bool>::reference` 某种程度上是不可见的，这就导致了问题。

另一个常见的情况是表达式模板。考虑类 `Matrix` 的四个对象 `m1, m2, m3, m4`
```cpp
Matrix sum = m1 + m2 + m3 + m4;
```

为了效率，`Matrix` 的 `operator+` 可能返回的类型是 `Sum<Matrix, Matrix>`，这是 `Matrix` 的代理类，能够隐式地转化为 `Matrix` 类型。上面代码右边的类型很可能是 `Sum<Sum<Sum<Matrix, Matrix>, Matrix>, Matrix>` 然后能够隐式地转化成 `Matrix` 类型。

看不见的代理类和 `auto` 配合会有问题。这些类型的对象的生命周期应该局限于一个语句，但是创建这些类型的对象会违反这一点。


