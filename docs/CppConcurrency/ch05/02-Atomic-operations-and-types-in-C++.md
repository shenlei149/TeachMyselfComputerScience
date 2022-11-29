原子操作（`atomic operation`）是不可再分的操作。任何线程都不会看到执行到一半的结果，要么没有执行，要么执行完了。如果读操作对象是原子的（`atomic`），那么所有的写也是原子的，所以读到的结果要么是初始值，要么是某次写操作的结果。

反面就是非原子操作会被某个线程看到执行了一半的结果。如果一个非原子操作包含原子操作，比如更新一个`struct`，其中包含`atomic`成员，也包含非原子成员，那么其他线程会看到原子成员更新要么是完成的，要么没有开始，但是其他成员就可能被看到更新到一半的结果。在这个层面上讲，这会导致数据竞争和未定义行为。

### The standard atomic types
`<atomic>`中定义了标准的原子类型。虽然可以使用互斥使得读写看起来是原子操作，但是当且仅当在这些类型上的操作是原子的这件事是被标准化定义的。绝大部分原子类型都有一个成员函数`is_lock_free()`告诉用户操作是一条指令实现的（`true`）还是内部使用了互斥（`false`）。

原子操作的一个关键用途是作为某个操作的替代或者是使用互斥来确保同步。如果原子操作内部使用了互斥，为了性能可能没有物化，那么最好用一个基于互斥的实现而不是这些原子操作。第七章会讨论这些。

有一系列宏来在编译期检查各种整数类型是否是`lock-free`的。C++17开始，每一个类型有一个`static constexpr`的变量`is_always_lock_free`表示在当前编译器支持的所有平台上都是`lock-free`的。比如`std::atomic<int>`总是`lock-free`的，那么`std::atomic<int>::is_always_lock_free`就是`true`，`std::atomic<uintmax_t>`可能需要某些平台的特定支持才能是`lock-free`的，那么`std::atomic<uintmax_t>::is_always_lock_free`就是`false`。

这些宏是 ATOMIC_BOOL_LOCK_FREE, ATOMIC_CHAR_LOCK_FREE, ATOMIC_
CHAR16_T_LOCK_FREE, ATOMIC_CHAR32_T_LOCK_FREE, ATOMIC_WCHAR_T_LOCK_FREE,
ATOMIC_SHORT_LOCK_FREE, ATOMIC_INT_LOCK_FREE, ATOMIC_LONG_LOCK_FREE, ATOMIC
_LLONG_LOCK_FREE, ATOMIC_POINTER_LOCK_FREE。它们指定了整数类型及相应`unsigned`类型是否是`lock-free`的。0表示永远不是，2表示总是`lock-free`的，1的话需要用上述运行时属性来判断。

唯一没有提供`is_lock_free()`的类型是`std::atomic_flag`。这是一个简单的布尔类型，必须是`lock-free`的。有了这个类型，可能简单地实现锁和其他原子类型。这里的简单指的是`std::atomic_flag`只能检查并设置值（`test_and_set()`）或者清除标记（`clear()`），没有赋值，没有复制构造，没有检查并清除的操作等等。

其余的原子类型有更全面的功能，不过可能不是`lock-free`的。在流行的平台上，内置类型基本都是`lock-free`的，比如`std::atomic<int>`和`std::atomic<void*>`，不过不是必须的。每一个特化版本都反映了类型本身的属性，比如指针不支持`&=`操作，那么原子指针也不支持。

除了直接使用`std::atomic<>`之外，还可以使用下表的类型名。由于历史原因，最好不要在一个程序里面混用以影响可以执行。

| Atomic type | Corresponding specialization |
|--|--|
| `atomic_bool` | `std::atomic<bool>` |
| `atomic_char` | `std::atomic<char>` |
| `atomic_schar` | `std::atomic<signed char>` |
| `atomic_uchar` | `std::atomic<unsigned char>` |
| `atomic_int` | `std::atomic<int>` |
| `atomic_uint` | `std::atomic<unsigned>` |
| `atomic_short` | `std::atomic<short>` |
| `atomic_ushort` | `std::atomic<unsigned short>` |
| `atomic_long` | `std::atomic<long>` |
| `atomic_ulong` | `std::atomic<unsigned long>` |
| `atomic_llong` | `std::atomic<long long>` |
| `atomic_ullong` | `std::atomic<unsigned long long>` |
| `atomic_char16_t` | `std::atomic<char16_t>` |
| `atomic_char32_t` | `std::atomic<char32_t>` |
| `atomic_wchar_t` | `std::atomic<wchar_t>` |

C++标准还提供一系列`typedef`的原子类型，和非原子类型相对应，见下表。

| Atomic `typedef` | Corresponding Standard Library `typedef` |
|--|--|
| `atomic_int_least8_t` | `int_least8_t` |
| `atomic_uint_least8_t` | `uint_least8_t` |
| `atomic_int_least16_t` | `int_least16_t` |
| `atomic_uint_least16_t` | `uint_least16_t` |
| `atomic_int_least32_t` | `int_least32_t` |
| `atomic_uint_least32_t` | `uint_least32_t` |
| `atomic_int_least64_t` | `int_least64_t` |
| `atomic_uint_least64_t` | `uint_least64_t` |
| `atomic_int_fast8_t` | `int_fast8_t` |
| `atomic_uint_fast8_t` | `uint_fast8_t` |
| `atomic_int_fast16_t` | `int_fast16_t` |
| `atomic_uint_fast16_t` | `uint_fast16_t` |
| `atomic_int_fast32_t` | `int_fast32_t` |
| `atomic_uint_fast32_t` | `uint_fast32_t` |
| `atomic_int_fast64_t` | `int_fast64_t` |
| `atomic_uint_fast64_t` | `uint_fast64_t` |
| `atomic_intptr_t` | `intptr_t` |
| `atomic_uintptr_t` | `uintptr_t` |
| `atomic_size_t` | `size_t` |
| `atomic_ptrdiff_t` | `ptrdiff_t` |
| `atomic_intmax_t` | `intmax_t` |
| `atomic_uintmax_t` | `uintmax_t` |
