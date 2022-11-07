有几种方式实现一个线程等待另一个线程完成工作这种场景。第一种是等待线程通过不停的轮询共享变量（通过互斥保护）来得知这个事情，工作线程一旦完成了，就设置这个变量。这有两个方面的浪费：第一个等待线程不停的轮询，浪费资源，可能使得工作线程工作的更慢；如果正在轮询（加了锁），那么工作线程将不得不等待等待线程释放之后再获得锁修改变量。

第二种方式是利用`std::this_thread::sleep_for()`等待一段时间再轮询。
```c++
bool flag;
std::mutex m;
void wait_for_flag()
{
    std::unique_lock<std::mutex> lk(m);
    while (!flag)
    {
        lk.unlock();
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
        lk.lock();
    }
}
```
睡眠之前释放锁，线程唤醒之后再加锁轮询。

这会有很大的性能提升，因为睡眠的时候不会有资源浪费。睡眠时长是一个问题，如果太短，还是会浪费资源，如果太长，工作线程完成之后会有延迟才检查变量。大部分场景延迟问题不大，不过实时性要求高的场景就会是一个弊端。

更好的做法是使用c++标准库提供的条件变量（`condition variable`）让工作线程通知等待线程。一个条件变量可以看做是事件或者条件（`condition`），一个或多个线程可以等待这个条件被满足，一个线程一旦使之成立之后就可以通知（`notify`）等待在这个条件变量上的线程，这些等待线程继续它们的工作。

### Waiting for a condition with condition variables
C++标准库提供的条件变量有两个：`std::condition_variable`和`std::condition_variable_any`。后者更通用，需要的仅是`metux-like`对象即可，有潜在的附加开销，如果没有额外需要，使用`std::condition_variable`即可。

下面的示例展示了如何使用条件变量。
```c++
std::mutex mut;
std::queue<data_chunk> data_queue;
std::condition_variable data_cond;
void data_preparation_thread()
{
    while (more_data_to_prepare())
    {
        data_chunk const data = prepare_data();
        {
            std::lock_guard<std::mutex> lk(mut);
            data_queue.push(data);
        }

        data_cond.notify_one();
    }
}

void data_processing_thread()
{
    while (true)
    {
        std::unique_lock<std::mutex> lk(mut);
        data_cond.wait(
            lk, []
            { return !data_queue.empty(); });

        data_chunk data = data_queue.front();
        data_queue.pop();
        lk.unlock();
        process(data);

        if (is_last_chunk(data))
            break;
    }
}
```
有一个队列用于在两个线程之间传递数据。生产者使用`std::lock_guard`加锁然后添加数据，解锁后调用`notify_one()`通知等待在条件变量`std::condition_variable`实例上的线程。我们是在解锁后再通知，好处是一旦唤醒了某个线程，不需要再次被`metux`阻塞住。

另外一侧，使用`std::unique_lock`加锁。然后调用`std::condition_variable`实例的`wait()`方法等待第二个参数（lambda 表达式）成立，即队列中有数据。

`wait()`的实现是说检查条件返回值。如果返回`false`，那么释放锁，进入睡眠等待通知。当生产者`notify_one()`会唤醒该线程，首先加锁，然后检查条件，如果`false`，释放锁继续等待，如果是`true`，那么已经获得了锁，执行后续的代码。因为`wait()`需要加锁和解锁，所以这里需要使用`std::unique_lock`而不是`std::lock_guard`，后者没有提供这种灵活性。如果`wait()`进入睡眠时不释放锁，那么生产者也无法获取锁把数据放入队列，进一步也就无法调用`notify_one()`来唤醒等待线程。


