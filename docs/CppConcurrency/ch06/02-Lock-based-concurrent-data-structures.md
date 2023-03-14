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

