`std::make_shared` 是 C++11 标准引入的，不过 `std::make_unique` 是 C++14 标准引入的。实现一个不支持数组和自定义删除器的 `std::make_unique` 是很容的。
```cpp
template <typename T, typename... Ts>
std::unique_ptr<T> make_unique(Ts &&...params)
{
    return std::unique_ptr<T>(new T(std::forward<Ts>(params)...));
}
```
`std::make_unique` `std::make_shared` 是三个 `make` 函数中的两个：接受任意参数，完美转发给构造函数动态的创建一个对象，然后返回指向这个对象的智能指针。和 `std::make_shared` 类似，`std::allocate_shared` 接受一个分配器。

下面通过使用和不使用 `make` 给出了使用 `make` 函数的第一个原因。
```cpp
auto upw1(std::make_unique<Widget>());    // with make func
std::unique_ptr<Widget> upw2(new Widget); // without make func
auto spw1(std::make_shared<Widget>());    // with make func
std::shared_ptr<Widget> spw2(new Widget); // without make func
```
原书高亮了 `Widget`。不使用 `make`，`Widget` 重复了两次，使用 `make` 就避免了重复。好处有避免了代码冗余。代码冗余会使得编译时间更长，目标代码冗余，并且使代码库使用更加困难。通常会演进成不一致的代码，而不一致的代码往往会出现 bug。

第二个使用 `make` 的理由是异常安全。考虑有如下函数，根据优先级来处理 `Widget`
```cpp
void processWidget(std::shared_ptr<Widget> spw, int priority);
```
按值传递 `std::shared_ptr` 看起来有点奇怪，不过 Item 41（TODO link）给出了合理的理由，如果 `processWidget` 总是复制 `std::shared_ptr`。

假定我们有一个计算优先级的函数
```cpp
int computePriority();
```
下面使用 `new` 而不是 `make` 来使用这个函数。
```cpp
processWidget(std::shared_ptr<Widget>(new Widget), // potential resource leak!
              computePriority());
```
注释中解释说，可能产生内存泄露，但是这又是如何发生的呢？

在运行时，参数必须要在调用函数之前完成求值，所以在调用 `processWidget` 之前，一定要完成三件事：
* `new Widget` 求值，在对上创建一个 `Widget` 对象
* 构造 `std::shared_ptr<Widget>` 来管理 `new` 出来的对象
* `computePriority` 必须被执行

编译器没有必要按照上述顺序进行操作。考虑下面一种情况：
1. 执行 `new Widget`
2. 执行 `computePriority`
3. 构造 `std::shared_ptr`

如果运行时 `computePriority` 抛出一个异常，而第一步 `new` 出来的对象还没有被 `std::shared_ptr` 管理，那么就会泄露内存。

如果使用 `make` 就不会有这个问题。
```cpp
processWidget(std::make_shared<Widget>(), // no potential resource leak
              computePriority());
```
运行时 `std::make_shared` 和 `computePriority` 总有一个先执行。`std::make_shared` 先执行，那么裸指针安全的存储在 `std::shared_ptr` 中，如果 `computePriority` 抛出异常，`std::shared_ptr` 的析构函数释放 `Widget` 的内存资源。如果 `computePriority` 先执行且抛出异常，`std::make_shared` 还没有调用，也就不用担心动态创建 `Widget` 而导致的内存泄露问题。

`std::shared_ptr` `std::make_shared` 替换成 `std::unique_ptr` `std::make_unique`，上述分析同样成立。

`std::make_shared` 的一个特性是相比原始的 `new` 效率提升。使用 `std::make_shared` 能够生成更小更快的代码，并使用更精简的数据结构。考虑如下直接使用 `new` 的代码
```cpp
std::shared_ptr<Widget> spw(new Widget);
```

##
