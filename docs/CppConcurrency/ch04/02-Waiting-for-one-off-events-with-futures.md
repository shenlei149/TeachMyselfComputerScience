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
默认情况下，`std::async`启动一个线程执行，或者等到调用`wait()`或者`get()`的时候同步计算，这取决于实现。我们可以传入`std::launch`来确定其行为，`std::launch::deferred`的行为是调用`wait()`或者`get()`的时候才调用函数，`std::launch::async`的行为是要求一定启动线程来执行函数，`std::launch::deferred
| std::launch::async`意味着实现决定行为。如果函数是延迟调用的话，可能从来不会执行。
```c++
auto f6 = std::async(std::launch::async, Y(), 1.2);
auto f7 = std::async(std::launch::deferred, baz, std::ref(x));
auto f8 = std::async(
    std::launch::deferred | std::launch::async,
    baz, std::ref(x));
auto f9 = std::async(baz, std::ref(x));
f7.wait();
```

### Associating a task with a future
`std::packaged_task<>`将`future`和可调用对象绑定在一起。当`std::packaged_task<>`被调用时，函数或者可调用对象使`future`状态`ready`，返回值存储在与`future`关联的数据上。这可以用于构建一个线程池或者任务管理系统。一个很大的操作，分割成若干小任务，封装到`std::packaged_task<>`实例中，交给线程池或者调度系统执行。从抽象角度看，调度器处理的是`std::packaged_task<>`实例而不是一个一个的函数。

`std::packaged_task<>`模板参数是一个函数签名，`void()`表示没有参数也没有返回值，`int(std::string&,double*)`表示参数是非`const`的`string`引用和一个`double`指针，返回类型是`int`。当构造`std::packaged_task`实例的时候，传入的函数或可调用对象不一定要和签名完全一致，类型能隐式转换即可。比如构造`std::packaged_task<double(double)>`时可以用一个接受`int`类型参数返回`float`类型的函数。

通过其成员函数`get_future()`得到具体的`std::future<>`，此`future`的模板类型就是指定的函数签名的返回值，类似的，指定的函数签名的参数列表是其`operator()`的参数列表。比如`std::packaged
_task <std::string(std::vector<char>*,int)>`的定义大致如下。
```c++
template <>
class packaged_task<std::string(std::vector<char> *, int)>
{
public:
    template <typename Callable>
    explicit packaged_task(Callable &&f);
    std::future<std::string> get_future();
    void operator()(std::vector<char> *, int);
};
```
`std::packaged_task`是可调用对象，可以封装成函数，或者传递给`std::thread`或者任何需要可调用对象的函数。当然，也可以直接执行。我们可以用`std::packaged_task`封装成一个任务，在其他地方获取`future`，当我们需要结果的时候，等待其`ready`。看下面的示例。

#### PASSING TASKS BETWEEN THREADS
许多 GUI 框架要求从特定线程（一般就是 UI 线程或者主线程）对 GUI 进行更新，因此如果另一个线程需要更新 GUI，它必须向正其发送消息才能达到目的。`std:packaged_task`提供了一种方法来执行此操作，不用为每个与 GUI 相关的活动提供自定义消息。如下所示。
```c++
#include <deque>
#include <mutex>
#include <future>
#include <thread>
#include <utility>
std::mutex m;
std::deque<std::packaged_task<void()>> tasks;
bool gui_shutdown_message_received();
void get_and_process_gui_message();
void gui_thread()
{
    while (!gui_shutdown_message_received())
    {
        get_and_process_gui_message();
        std::packaged_task<void()> task;
        {
            std::lock_guard<std::mutex> lk(m);
            if (tasks.empty())
                continue;
            task = std::move(tasks.front());
            tasks.pop_front();
        }

        task();
    }
}

std::thread gui_bg_thread(gui_thread);

template <typename Func>
std::future<void> post_task_for_gui_thread(Func f)
{
    std::packaged_task<void()> task(f);
    std::future<void> res = task.get_future();
    std::lock_guard<std::mutex> lk(m);
    tasks.push_back(std::move(task));
    return res;
}
```
GUI 线程轮询是否收到了关闭的信号，如果没有，就尝试从任务队列中获得一个任务。任务执行完的时候，与之关联的`future`是`ready`状态。

添加任务也是类似的。利用给定函数创建`std::packaged_task`对象，获取`future`，在将其返回给调用者之前把任务放入队列。调用`post_task_for_gui_thread`想更新 GUI 的线程可以等待返回的`future`以确定任何执行完了，或者如果不关心的话可以直接丢弃。

这里使用`std::packaged_task<void()>`表示无参也无返回值，但是模板可以接受任意函数签名，所以很容易把这个例子扩展成需要传参并且返回结果的例子。

###  Making (std::)promises
当一个服务需要处理很多连接时，一个做法是一个线程处理一个连接，好处是简单易懂。当连接很少的时候没有问题，但是连接很多的时候会产生大量的线程消耗资源，同时需要调度和上下文切换，甚至在达到网络上限之前就消耗光了 OS 的资源。所以需要很少的线程（可能只有一个）来处理连接，即一个线程需要处理很多连接。本质上就是要接受网络包或者发送网络包。

`std::promise<T>`

### Saving an exception for the future

### Waiting from multiple threads
