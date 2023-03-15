即使只有一个互斥，也需要确保所有操作都在互斥保护之内，同时还要考虑接口的设计不要引入固有的竞争。如果使用不同互斥保护不同的数据，那么问题会变得更复杂，如果需要多个互斥保护同一个数据，那么还要考虑死锁的问题。

下面从第三章介绍的最简单的线程安全的栈开始。

### A thread-safe stack using locks
我们把第三章的线程安全的栈的代码贴在下面，然后解释是否安全。
```cpp
#include <exception>

struct empty_stack : std::exception
{
    const char *what() const throw();
};

template <typename T>
class threadsafe_stack
{
private:
    std::stack<T> data;
    mutable std::mutex m;

public:
    threadsafe_stack() {}
    threadsafe_stack(const threadsafe_stack &other)
    {
        std::lock_guard<std::mutex> lock(other.m);
        data = other.data;
    }
    threadsafe_stack &operator=(const threadsafe_stack &) = delete;

    void push(T new_value)
    {
        std::lock_guard<std::mutex> lock(m);
        data.push(std::move(new_value));
    }
    std::shared_ptr<T> pop()
    {
        std::lock_guard<std::mutex> lock(m);
        if (data.empty())
            throw empty_stack();

        std::shared_ptr<T> const res(
            std::make_shared<T>(std::move(data.top())));
        data.pop();

        return res;
    }

    void pop(T &value)
    {
        std::lock_guard<std::mutex> lock(m);
        if (data.empty())
            throw empty_stack();

        value = std::move(data.top());
        data.pop();
    }

    bool empty() const
    {
        std::lock_guard<std::mutex> lock(m);
        return data.empty();
    }
};
```
首先，每一个成员函数都通过锁住互斥 `m` 保护数据，那么基本安全是有的。这样一次只能有一个线程访问数据，每一个成员函数维护不变量，那么没有线程能破坏不变量。

在调用 `empty()` 和 `pop()` 之间有潜在的竞争风险，不过在 `pop()` 拿到锁之后，又检查了一下是否为空，那么这个问题就不存在了。`pop()` 将 `std::stack<>` 的 `pop(), top()` 结合起来，那么由于接口带来的竞争也不存在了。

加锁本身可能会抛出异常，这非常罕见，往往是系统资源导致的，不过这时我们还没有修改数据，所以是安全的。解锁不会失败，没问题。使用 `std::lock_guard<>` 的好处是不会使锁始终处于加锁的状态。

`data.push()` 可能会由于拷贝、移动数据时抛出异常、没有足够内存而出错，不过 `std::stack<>` 会保证不会出错。

第一个重载版本的 `pop()` 可能会抛出一个空栈的异常，不过这时还没有修改数据，所以没有问题。创建 `res` 的时候也可能抛出异常，可能是构造 `std::make_shared` 时没有空间，也有可能是移动、构造数据对象的时候抛出异常。不过不管是什么情况，C++ 运行时和标准库都会确保没有内存泄露。而此时还没有修改栈，所以也没有问题。`data.pop()` 不会有异常，能够正常返回。所以这个版本的 `pop()` 是异常安全的。

第二个版本的 `pop()` 类似，移动或者构造数据对象可能会抛出异常，不过在调用 `data.pop()` 之前都不会修改数据结构，所以也是异常安全的。

最后，`empty()` 没有修改数据，所以也是异常安全的。

这里有死锁的风险。因为我们在移动、构造数据对象的时候，本质上是调用了用户的代码，而这时是持有锁的。如果用户的代码又调用了线程安全栈的成员函数，那么就会死锁。这是可以理解的，用户应该有责任避免这种情况。

所有的成员函数都使用了 `std::lock_guard<>`，所以任意线程访问都是安全的。不过构造函数和析构函数是例外。但这不是问题，因为对象只能构造和析构一次，不管是否是线程安全的对象，对一个没有完全构造或者开始部分析构的对象调用其他成员函数，都是不明智的行为。用户自己必须确保这一点。

尽管这个实现是安全的，但是每一次只能有一个线程修改数据。线程的串行化会导致性能比较差，特别是多个线程竞争的时候，等待的线程什么也做不了。另外对于要消费数据的线程而言，没有很好的等待机制，等待的线程只能频繁调用 `empty()` 或者调用 `pop()` 并捕获 `empty_stack`。下面线程安全的队列很好地解决了这个问题。

### A thread-safe queue using locks and condition variables
下面分析第四章介绍的线程安全的队里。和线程安全的栈类似，我们的实现封装了 `std::queue<>`，修改接口以避免竞争。
```cpp
template <typename T>
class threadsafe_queue
{
private:
    mutable std::mutex mut;
    std::queue<T> data_queue;
    std::condition_variable data_cond;

public:
    threadsafe_queue() {}

    void push(T new_value)
    {
        std::lock_guard<std::mutex> lk(mut);
        data_queue.push(std::move(new_value));
        data_cond.notify_one();
    }

    void wait_and_pop(T &value)
    {
        std::unique_lock<std::mutex> lk(mut);
        data_cond.wait(lk, [this]
                       { return !data_queue.empty(); });
        value = std::move(data_queue.front());
        data_queue.pop();
    }

    std::shared_ptr<T> wait_and_pop()
    {
        std::unique_lock<std::mutex> lk(mut);
        data_cond.wait(lk, [this]
                       { return !data_queue.empty(); });
        std::shared_ptr<T> res(
            std::make_shared<T>(std::move(data_queue.front())));
        data_queue.pop();
        return res;
    }

    bool try_pop(T &value)
    {
        std::lock_guard<std::mutex> lk(mut);
        if (data_queue.empty())
            return false;

        value = std::move(data_queue.front());
        data_queue.pop();
        return true;
    }

    std::shared_ptr<T> try_pop()
    {
        std::lock_guard<std::mutex> lk(mut);
        if (data_queue.empty())
            return std::shared_ptr<T>();

        std::shared_ptr<T> res(
            std::make_shared<T>(std::move(data_queue.front())));
        data_queue.pop();
        return res;
    }

    bool empty() const
    {
        std::lock_guard<std::mutex> lk(mut);
        return data_queue.empty();
    }
};
```
除了 `push()` 中调用了 `data_cond.notify_one()` 和 `wait_and_pop()` 函数之外，和上一个小节栈都一样。`try_pop()` 函数也基本类似，不过不会抛出空容器异常，而是返回 `bool` 表示是否成功了，或者是是否是空指针。所以除了 `wait_and_pop()` 之外，上一节中其他函数的分析也适用于这里。

`wait_and_pop()` 解决了上一节描述的忙等待的问题，`data_cond.wait()` 直到有数据了才会返回，所以不用担心队列为空，同时数据仍旧处于互斥的保护之下。所以这个函数不会增加新的条件竞争、死锁等问题，不变量也不会被破坏。

不过有一个问题值得关注，当添加了一个新的数据，`data_cond.notify_one()` 被调用，一个线程被唤醒，但是构造 `std::shared_ptr<>` 的时候抛出了一场，那么其他线程也不会被唤醒。如果这不可以接受的话，可以调用 `data_cond.notify_all()` 唤醒所有的进行，不过也只有一个会处理数据，其他会再次睡眠，浪费资源。第二个可选的方法是如果 `wait_and_pop()` 中有异常，继续调用`notify_one()` 唤醒其他线程处理。第三种方式是把构造 `std::shared_ptr<>` 放到 `push()` 中，也就是保存的是持有数据的智能指针而不是数据本身。从 `std::queue<>` 复制 `std::shared_ptr<>` 不会有异常，所以 `wait_and_pop()` 是安全的。下面是基于这种思想改写的线程安全队列。
```cpp
```

