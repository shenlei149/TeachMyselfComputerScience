本章前面介绍了很多同步并行的方式，主要目的是可以简化代码，使得我们把焦点放到功能上。相比使用共享数据，我们更倾向于在不同任务之间传递数据，提供给任务必要的数据，从任务中获取需要的数据。

### Functional programming with futures
术语函数编程（`functional programming`, `FP`）是一种编程模式，函数的结果只依赖于输入的参数。这个概念借鉴了数学中函数的概念，只要输入一样，那么输出就一定一样。一个纯的函数不修改外部状态，其副作用仅限于其返回值。

这使得很多事情都更容易思考，尤其是在并发的情况下，因为第三章介绍的麻烦都是共享数据导致的。不修改数据，也就没有竞争，那么也不需要互斥保护数据。类似`Haskell`的语言纯函数式的，越来越流行。大部分函数都是纯函数，部分非纯函数需要修改状态，那么我们就能更加仔细推敲它们在整个应用中是否协调。

FP 并非仅限于函数式编程语言。C++是多范式，也可以支持函数式编程，C++11有了 lambda、类型推导、bind 等等，写函数式代码也更加容易。`future`是最后一个组件，为写并发的函数式代码提供了便利。`future`用于多线程之间传递数据，而不是修改共享数据。

#### FP-STYLE QUICKSORT
快排的原理就不在这里赘述了。下面是 FP 风格的实现，并不像`std::sort()`一样就地排序，而是通过复制链表实现的。
```c++
template <typename T>
std::list<T> sequential_quick_sort(std::list<T> input)
{
    if (input.empty())
    {
        return input;
    }
    std::list<T> result;

    // 取第一个元素作为 pivot
    result.splice(result.begin(), input, input.begin());
    T const &pivot = *result.begin();

    // 使用 lambda 表达式声明分区函数，只用一次。引用捕捉避免复制
    auto divide_point = std::partition(input.begin(), input.end(),
                                       [&](T const &t)
                                       { return t < pivot; });

    // 链表中剩余元素从头到 divide_point 移动到 lower_part
    // input 里面剩余的就是比 pivot 大的元素了
    std::list<T> lower_part;
    lower_part.splice(lower_part.end(), input, input.begin(),
                      divide_point);

    // 递归排序两边，move 避免复制，结果会隐式的 move 出来
    auto new_lower(
        sequential_quick_sort(std::move(lower_part)));
    auto new_higher(
        sequential_quick_sort(std::move(input)));

    // 拼接结果，大的元素在后面拼接，小的元素从头拼接，pivot 本身就处于链表中间了
    result.splice(result.end(), new_higher);
    result.splice(result.begin(), new_lower);
    return result;
}
```

### FP-STYLE PARALLEL QUICKSORT
已经有了函数式的快排实现，很容易利用`future`改写成并行版本。
```c++
template <typename T>
std::list<T> parallel_quick_sort(std::list<T> input)
{
    if (input.empty())
    {
        return input;
    }

    std::list<T> result;
    result.splice(result.begin(), input, input.begin());
    T const &pivot = *result.begin();
    auto divide_point = std::partition(input.begin(), input.end(),
                                       [&](T const &t)
                                       { return t < pivot; });

    std::list<T> lower_part;
    lower_part.splice(lower_part.end(), input, input.begin(),
                      divide_point);
    std::future<std::list<T>> new_lower(
        std::async(&parallel_quick_sort<T>, std::move(lower_part)));
    auto new_higher(
        parallel_quick_sort(std::move(input)));

    result.splice(result.end(), new_higher);
    result.splice(result.begin(), new_lower.get());

    return result;
}
```
