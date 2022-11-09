C++标准库中把一次性的事件称为`future`。简单来说，它代表了一个事件。线程可以周期性的检查（期间可以做其他事情）或者一直做其他事情直到`future`已经准备好了（`ready`）。一旦事件已经发生，变成了`ready`状态，就不能重置了。

C++标准库提供了两种`future`：`unique futures` (`std::future<>`)和`shared futures`(`std::shared_future<>`)。和相应的智能指针类似，前者只能有一个实例与事件关联，或者就不限制了。对于后者而言，一旦事件`ready`，那么所有线程都能访问与之关联的数据。模板参数表示数据类型，`std:future<void>`和`std::shared_future<void>`表示没有与之关联的数据。尽管`future`是为了多线程通信，但是如果线程访问同一个`future`对象，需要保护。

还有两个实验性质的类：`std::experimental::future<>`和`std::experimental::shared_future<>`，与标准库一致，此外还提供了其他设施。不过代码质量不保证，而且等到了`std`标准库的时候，语法和语义可能会有变化。

第二章中讲解`std::thread`时没有提供方法从一个任务中返回数据。下面会介绍。

### Returning values from background tasks
如果需要一个耗时较长的计算，但是不需要立即得到结果，这是使用`std::async`的好机会。

使用`std::async`启动了一个异步任务，相较于启动线程得到`std::thread`对象等待结束，`std::async`返回`std::future`对象，其最终会保存函数的返回结果。当需要结果的时候，调用`get()`取结果，这个时候可能会阻塞住，等到`future`状态是`ready`。下面是一个简单的例子。
```c++
#include <future>
#include <iostream>
int find_the_answer_to_ltuae();
void do_other_stuff();
int main()
{
    std::future<int> the_answer = std::async(find_the_answer_to_ltuae);
    do_other_stuff();
    std::cout << "The answer is " << the_answer.get() << std::endl;
}
```
`std::async`允许传递多个参数，这个`std::thread`是一致的，对于第一个参数是类成员函数的情况也一样。参数如果是右值，通过移动操作创建了一个副本，这使得可以使用只允许移动的类型。看下面的例子，说明了传参的各种情况。
```c++
#include <string>
#include <future>
struct X
{
    void foo(int, std::string const &);
    std::string bar(std::string const &);
};

X x;
auto f1 = std::async(&X::foo, &x, 42, "hello");
auto f2 = std::async(&X::bar, x, "goodbye");

struct Y
{
    double operator()(double);
};

Y y;
auto f3 = std::async(Y(), 3.141);
auto f4 = std::async(std::ref(y), 2.718);

X baz(X &);
std::async(baz, std::ref(x));

class move_only
{
public:
    move_only();
    move_only(move_only &&)
        move_only(move_only const &) = delete;
    move_only &operator=(move_only &&);
    move_only &operator=(move_only const &) = delete;
    void operator()();
};
auto f5 = std::async(move_only());
```
