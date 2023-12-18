C++ 特殊成员函数是指可以自动生成的成员函数。C++98 中有四个：默认构造函数，析构函数，拷贝构造函数和拷贝赋值运算符。只有需要时，这些函数才会被自动生成。一个类没有任何构造函数，才会生成默认构造函数。自动生成的特殊成员函数是 `public` 且是 `inline` 的。继承自有虚析构函数的基类，自动生成的析构函数是虚函数，初次智蛙，自动生成的函数默认都是非虚函数。

随着时间流逝，C++ 中生成特殊成员函数的规则也发生了变化。

C++11 中，有两个新的特殊成员函数：移动构造函数和移动赋值运算符。签名如下：
```cpp
class Widget
{
public:
    Widget(Widget &&rhs);            // move constructor
    Widget &operator=(Widget &&rhs); // move assignment operator
};
```

移动操作相关函数的生成规则和拷贝操作类似，只在必要时生成，如果生成了，那么行为是对所有非 `static` 成员变量进行逐字段的移动。移动构造函数对参数 `rhs` 中每一个非 `static` 成员进行移动构造，移动赋值运算符类似。如果存在基类部分，会对基类部分做同样的处理。

假定现在我更倾向于使用移动操作，比如移动构造或者移动赋值运算法，但是移动操作本身并不一定发生了。成员变量的移动操作更像是一个移动操作的请求，如果类型是不可移动的，比如 C++98 的对象，那么会使用拷贝替代移动。移动成员变量的核心是使用 `std::move`，运行时是会决定使用移动操作还是拷贝操作。详见 Item 23 TODO add link。能移动才会移动，不支持的话使用拷贝。

与拷贝类似，一旦我们自己定义了移动操作，那么编译器则不会生成。不过细节略有不同。

两个拷贝函数是独立的。如果只声明其中一个，不会影响编译器自动生成另外一个。比如声明了拷贝构造函数，但是又用到了拷贝赋值运算符，那么编译器会自动生成后者。反之亦然。

两个移动函数不是独立的。如果声明了其中一个，那么编译器不会自动生成另外一个。如果我们声明了其中一个，比如移动构造函数，这说明默认的行为和我们实现有不同，那么编译器就推断说无法生成一个合适的移动赋值运算符。即声明移动构造函数会抑制编译器自动合成移动赋值运算符。反之亦然。

类似的，如果一个类声明了拷贝函数，那么也不会生成移动函数。因为声明拷贝函数，那么意味着自动生成的行为——逐字段复制——不适合当前类，那么编译器会推断说既然逐字段复制不适合这个类，那么逐字段移动也可能不适合这个类，因此不会再自动生成了。

反方向也成立。声明了移动函数，那么编译器会使用 `delete` 禁止生成拷贝函数。理由也是类似的。这个新增的限制并不会破坏 C++98 的代码，因为旧代码没有移动函数。不过当你处理这些遗留代码，新增了移动函数以提升性能，那么自动生成特殊成员函数的规则就变了。

三原则（`Rule of Three`）是说一旦你声明了析构函数、拷贝构造函数和拷贝赋值函数中的一个，那么也应该声明另外两个。

## Things to Remember
* The special member functions are those compilers may generate on their own: default constructor, destructor, copy operations, and move operations.
* Move operations are generated only for classes lacking explicitly declared move operations, copy operations, and a destructor.
* The copy constructor is generated only for classes lacking an explicitly declared copy constructor, and it’s deleted if a move operation is declared. The copy assignment operator is generated only for classes lacking an explicitly declared copy assignment operator, and it’s deleted if a move operation is declared. Generation of the copy operations in classes with an explicitly declared destructor is deprecated.
* Member function templates never suppress generation of special member functions.
