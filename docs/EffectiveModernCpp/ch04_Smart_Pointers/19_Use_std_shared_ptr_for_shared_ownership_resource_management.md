我们既要 GC 的便利性，又要释放资源的确定性。C++11 引入的 `std::shared_ptr` 是结合两者的一种方式。通过 `std::shared_ptr` 访问的对象的生命周期由共享所有权来控制。没有一个 `std::shared_ptr` 拥有这个对象，但是当没有 `std::shared_ptr` 指向这个对象时，能够确保其被释放。当最后一个指向它的 `std::shared_ptr` 不再指向它时（比如析构，被赋值等），负责析构这个对象。

`std::shared_ptr` 通过引用计数（`reference count`）知道有多少个 `std::shared_ptr` 指向某个对象。`std::shared_ptr` 的构造会自增引用计数（移动 `std::shared_ptr` 对象除外），`std::shared_ptr` 的析构会进行自减操作，拷贝 `std::shared_ptr` 会包含两者（比如 `sp1 = sp2`，`sp1` 会指向 `sp2` 只想的对象，同时，`sp1` 原来对象的引用计数会自减，而 `sp2` 对应的对象的引用计数自增）。`std::shared_ptr` 发现引用计数为零，没有 `std::shared_ptr` 再指向这个对象，会析构这个对象。

引用计数的存在会有一些性能问题：
* `std::shared_ptr` 大小是两倍于裸指针的大小。除了指向对象的指针外，还有一个指针指向引用计数。
* 引用计数占用的空间需要动态分配。引用计数的值和指向的对象绑定，但是原始对象并不知道引用计数的存在，所以没有地方存这个值。Item 21（TODO link）会解释通过 `std::make_shared` 构造 `std::shared_ptr` 可以避免动态分配的开销，但是 `std::make_shared` 并不适用于所有场景。
* 引用计数的自增和自减必须是原子操作。因为读写可能在不同的进程，如果不是原子操作，那么结果是错误的。原子操作往往比非原子操作耗时，哪怕只有一个字的长度，所以可以假设读写是相对耗时的。

从一个 `std::shared_ptr` 移动构造一个 `std::shared_ptr` 对象，原始的 `std::shared_ptr` 会被设置成 `nullptr`，不再指向原来的对象，而新的 `std::shared_ptr` 开始指向这个对象，那么该对象对应的引用计数既不需要自增也不需要自减。拷贝 `std::shared_ptr` 需要自增而移动 `std::shared_ptr` 不需要，因此，移动操作要快些。

和 `std::unique_ptr` 类似，`std::shared_ptr` 默认也使用 `delete` 析构对象，同时，也可以自定义删除器。不过和 `std::unique_ptr` 不同的是，删除器类型并不是 `std::shared_ptr` 的一部分。
```cpp
auto loggingDel = [](Widget *pw) // custom deleter
{                                // (as in Item 18)
    makeLogEntry(pw);
    delete pw;
};

std::unique_ptr<Widget, decltype(loggingDel)> // deleter type is
    upw(new Widget, loggingDel);              // part of ptr type
std::shared_ptr<Widget>                       // deleter type is not
    spw(new Widget, loggingDel);              // part of ptr type
```

`std::shared_ptr` 的设计更灵活。考虑两个 `std::shared_ptr<Widget>` 对象，删除器类型不同，但是是智能指针是同一个类型，所以能放到对应类型的容器中。
```cpp
auto customDeleter1 = [](Widget *pw) {}; // custom deleters,
auto customDeleter2 = [](Widget *pw) {}; // each with a different type
std::shared_ptr<Widget> pw1(new Widget, customDeleter1);
std::shared_ptr<Widget> pw2(new Widget, customDeleter2);

std::vector<std::shared_ptr<Widget>> vpw{pw1, pw2};
```
这两个对象可以互相赋值，也可以传递给接受 `std::shared_ptr<Widget>` 参数类型的函数中。这对于 `std::unique_ptr` 而言是行不通的。

另一个问题是，当自定义删除器后，`std::unique_ptr` 的大小可能会变大，甚至由于函数对象包含大量数据而特别大，但是 `std::shared_ptr` 的大小始终是两个指针的大小。

本质上，需要的内存还是会变大，但是这没有占用 `std::unique_ptr` 自身的空间。
