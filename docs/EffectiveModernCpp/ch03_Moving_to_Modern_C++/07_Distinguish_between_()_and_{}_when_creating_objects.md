C++ 11 提供了丰富的初始化对象的方法，令人迷惑，不知道选哪个。初始化对象基本有三种方式，大括号 {} 小括号 () 等号 =。
```cpp
int x(0);   // initializer is in parentheses
int y = 0;  // initializer follows "="
int z{0};   // initializer is in braces
```

大括号和等号可以一起用。
```cpp
int z = {0}; // initializer uses "=" and braces
```
这种情况等价于只是用大括号，后续不再提了。

使用等号初始化往往会让新手感到迷惑，他们以为是赋值，但实际不是。对于像 `int` 这样的类型，其差异非常学术，不过对于用户自定义类型，区分初始化和赋值是很重要的，因为调用的是不同的函数。
```cpp
Widget w1;          // call default constructor
Widget w2 = w1;     // not an assignment; calls copy ctor
w1 = w2;            // an assignment; calls copy operator=
```

纵使有很多初始化语法，C++98 也不能表达一些想要的初始化效果。比如指定元素的方式创建一个容器。

为了解决上述问题，C++11 引入了统一初始化（`uniform initialization`），概念上只有一种初始化的方式，并且可以用在任何地点。由于语法是大括号 {}，所以作者也称为大括号初始化（`Braced initialization`），这个名字是从语法层面来谈的，而统一初始化是从概念角度来说的。

使用统一初始化，可以做到之前无法做到的事情，比如指定容器包含哪些元素。
```cpp
std::vector<int> v{ 1, 3, 5 };      // v's initial content is 1, 3, 5
```

C++11 允许在类内直接初始化非成员变量，可以使用统一初始化或者等号，但是不能使用小括号的方式。
```cpp
class Widget
{
private:
    int x{0};  // fine, x's default value is 0
    int y = 0; // also fine
    int z(0);  // error!
};
```

不能拷贝的对象（比如 `std::atomic`，参考 Item 40（TODO link）），可以使用大括号或者小括号初始化，但是不能使用等号。
```cpp
std::atomic<int> ai1{0};  // fine
std::atomic<int> ai2(0);  // fine
std::atomic<int> ai3 = 0; // error!
```

通过上面两个例子，能够更好的理解统一初始化的意义，其他初始化方式总有不可用的地方。

同一个初始化还有一个功能，就是禁止内置类型隐式地向更窄的类型转化。
```cpp
double x, y, z;
int sum1{x + y + z}; // error! sum of doubles may not be expressible as int
```

等号赋值不会检查这一点，否则会使得大量旧代码不能通过编译。小括号初始化也不会做类似检查。
```cpp
int sum2(x + y + z);    // okay (value of expression truncated to an int)
int sum3 = x + y + z;   // ditto
```

大括号初始化还能避免 C++ 中闹人的解析问题。一个常见情况是想调用默认构造函数创建对象，结果声明了一个函数。可以如下调用一个带参数的构造函数。
```cpp
Widget w1(10); // call Widget ctor with argument 10
```

使用类似的语法调用无参的构造函数的话，会声明一个函数。
```cpp
Widget w2(); // most vexing parse! declares a function named w2 that returns a Widget!
```

由于函数声明不能有大括号，所以用统一初始化就可以调用到无参的构造函数。
```cpp
Widget w3{}; // calls Widget ctor with no args
```

统一初始化到处可以用，能够阻止向窄类型转化，还能调用无参构造函数。好处很多，但是标题不是首选这个方式呢？

因为统一初始化有很多缺陷导致了非预期的行为。[Item 2](/EffectiveModernCpp/ch01_Deducing_Types/02_Understand_auto_type_deduction.md) 就描述了它和 `auto` 配合时可能会产生的问题。

当 `std::initializer_list` 没有掺和构造函数时，大括号和小括号两种方式调用构造函数的行为是一致的。
```cpp
class Widget
{
public:
    Widget(int i, bool b);   // ctors not declaring
    Widget(int i, double d); // std::initializer_list params
};

Widget w1(10, true); // calls first ctor
Widget w2{10, true}; // also calls first ctor
Widget w3(10, 5.0);  // calls second ctor
Widget w4{10, 5.0};  // also calls second ctor
```

然后，如果有一个或者多个以 `std::initializer_list` 为参数的构造函数，那么编译器会尽一切可能使用有 `std::initializer_list` 参数的构造函数。现在给上面的类增加一个接受 `std::initializer_list<long double>` 参数的构造函数。
```cpp
class Widget
{
public:
    Widget(int i, bool b);                         // as before
    Widget(int i, double d);                       // as before
    Widget(std::initializer_list<long double> il); // added
};
```

这时，虽然相比于其他构造函数，这个构造函数的模板类型是 `long double`，是更差的选择，但是上面的四个对象中 `w2,w4` 会使用接受 `std::initializer_list<long double>` 参数的构造函数。
```cpp
Widget w1(10, true); // uses parens and, as before, calls first ctor
Widget w2{10, true}; // uses braces, but now calls std::initializer_list ctor
                     // (10 and true convert to long double)
Widget w3(10, 5.0);  // uses parens and, as before, calls second ctor
Widget w4{10, 5.0};  // uses braces, but now calls std::initializer_list ctor
                     // (10 and 5.0 convert to long double)
```

拷贝和移动构造函数也会被参数为 `std::initializer_list` 的构造函数劫持。
```cpp
class Widget
{
public:
    Widget(int i, bool b);                         // as before
    Widget(int i, double d);                       // as before
    Widget(std::initializer_list<long double> il); // as before
    operator float() const;                        // convert to float
};
Widget w5(w4); // uses parens, calls copy ctor
Widget w6{w4}; // uses braces, calls std::initializer_list ctor
               // (w4 converts to float, and float converts to long double)
Widget w7(std::move(w4)); // uses parens, calls move ctor
Widget w8{std::move(w4)}; // uses braces, calls std::initializer_list ctor
                          // (for same reason as w6)
```

编译器调用接受 `std::initializer_list<long double>` 参数的构造函数的意愿十分强烈，以至于不能调用接受 `std::initializer_list<long double>` 参数的构造函数也不尝试其他构造函数。
```cpp
class Widget
{
public:
    Widget(int i, bool b);                  // as before
    Widget(int i, double d);                // as before
    Widget(std::initializer_list<bool> il); // element type is now bool
    // no implicit conversion funcs
};

Widget w{10, 5.0}; // error! requires narrowing conversions
```

只有当参数不能转化成 `std::initializer_list` 的模板类型时，才会考虑其他构造函数。比如上面的例子，把 `std::initializer_list<bool>` 换成 `std::initializer_list<std::string>`，这时由于不能把 `int` 和 `bool` 类型转化成 `std::string`，编译器会重新考虑不带有 `std::initializer_list` 的构造函数。
```cpp
class Widget
{
public:
    Widget(int i, bool b);   // as before
    Widget(int i, double d); // as before

    // std::initializer_list element type is now std::string
    Widget(std::initializer_list<std::string> il); // no implicit conversion funcs
};

Widget w1(10, true); // uses parens, still calls first ctor
Widget w2{10, true}; // uses braces, now calls first ctor
Widget w3(10, 5.0);  // uses parens, still calls second ctor
Widget w4{10, 5.0};  // uses braces, now calls second ctor
```

最后再来讨论一种情况。我们有一个无参的构造函数，也有一个接受 `std::initializer_list` 参数的构造函数，这时写一个空的大括号是什么意思呢？调用无参构造函数？还是调用接受 `std::initializer_list` 参数的构造函数但是初始化列表为空？

答案是前者。空大括号的意思是没有参数，而不是一个空的初始化列表。
```cpp
class Widget
{
public:
    Widget();                              // default ctor
    Widget(std::initializer_list<int> il); // std::initializer_list ctor
                                           // no implicit conversion funcs
};

Widget w1;   // calls default ctor
Widget w2{}; // also calls default ctor
Widget w3(); // most vexing parse! declares a function!
```

如果想调用接受 `std::initializer_list` 参数的构造函数，且初始化列表为空，写两层大括号。
```cpp
Widget w4({}); // calls std::initializer_list ctor with empty list
Widget w5{{}}; // ditto
```

可能已经被统一初始化的晦涩规则、`std::initializer_list` 和构造函数重载给搞晕了。不禁要问，这和日常开发有什么关系呢？超出想象，一个直接影响就是使用 `std::vector`。`std::vector` 有一个不接受 `std::initializer_list` 的两个参数的构造函数，允许给定元素的个数和每个元素的默认值，也有一个接受 `std::initializer_list` 为参数的构造函数，允许我们指定容器包含的元素。如果定义一个 `std::vector<int>`，然后传递两个参数，那么小括号和大括号的意义完全不同。
```cpp
std::vector<int> v1(10, 20); // use non-std::initializer_list ctor: create 10-element
                             // std::vector, all elements have value of 20
std::vector<int> v2{10, 20}; // use std::initializer_list ctor: create 2-element
                             // std::vector, element values are 10 and 20
```
