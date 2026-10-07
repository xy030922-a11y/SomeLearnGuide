# 现代 C++ 标准特性指南

> 按 **C++11、C++14、C++17、C++20、C++23** 介绍语言扩展及相应标准库新增能力。所有版本都属于 ISO 标准 C++。本文侧重特性之间的关系与正确使用，标准库与并发的完整专题见对应分卷。

[返回 C++ Guide 总导航](C++Guide.md)

配套分卷：

- [核心语言指南](C++CoreLanguageGuide.md)：类型、表达式、函数、类、继承、模板和对象生命周期。
- [标准库指南](C++StandardLibraryGuide.md)：固定宽度整数完整专题、容器、算法、智能指针、字符串、视图与结果类型。
- [并发指南](C++ConcurrencyGuide.md)：线程、锁、条件变量、原子操作、异步结果和协作式停止。
- [系统编程指南](C++SystemProgrammingGuide.md)：系统资源、平台接口和底层边界。
- [构建与工程指南](C++BuildEngineeringGuide.md)：标准模式、编译链接、CMake、调试、测试和质量工具。

**示例约定：** 每个“完整程序”都包含所需头文件和 `main()`，可以单独保存为 `main.cpp`；最低标准写在代码前。不同完整程序分别编译。片段需放在注明的作用域内并补齐头文件。错误写法仅放在注释里。输入规模和算术范围仍是接口契约的一部分。

## 阅读导航

1. [标准版本总表](#versions)
2. [C++11：类型推导、列表初始化与范围 for](#cpp11-basics)
3. [C++11：constexpr、static_assert 与类型特征](#cpp11-constexpr)
4. [C++11：现代类接口](#cpp11-classes)
5. [C++11：右值引用、移动与完美转发](#cpp11-move)
6. [C++11：Lambda 与可调用对象](#cpp11-lambda)
7. [C++11：标准库新增能力与整数类型](#cpp11-library-summary)
8. [C++14：泛型 Lambda、初始化捕获与 make_unique](#cpp14-lambda)
9. [C++14：返回类型推导、decltype(auto) 与类型别名](#cpp14-deduction)
10. [C++14：constexpr 放宽、变量模板与字面量](#cpp14-constexpr)
11. [C++17：结构化绑定、if 初始化与 inline 变量](#cpp17-bindings)
12. [C++17：if constexpr、折叠表达式与 CTAD](#cpp17-templates)
13. [C++17：复制消除与标准库扩展](#cpp17-elision)
14. [C++20：Concepts、requires 与 span](#cpp20-concepts)
15. [C++20：consteval、constinit 与三路比较](#cpp20-constants)
16. [C++20：Ranges、协程、Modules 与其他补充](#cpp20-facilities)
17. [C++23：if consteval 与显式对象参数](#cpp23-language)
18. [C++23：expected、print 与标准库补充](#cpp23-library)
19. [编译配置与特性检测](#build)
20. [学习顺序与参考资料](#learning)

---

<a id="versions"></a>
## 1. 标准版本总表

| 最低标准 | 语言特性 | 标准库新增能力 |
| --- | --- | --- |
| C++11 | `auto` 推导、`decltype`、列表初始化、范围 `for`、`nullptr`、`enum class`、右值引用、Lambda、可变参数模板、别名模板、`constexpr`、`static_assert`、`noexcept`、`override`、`final`、`= default` / `= delete` | `<cstdint>`、`array`、无序容器、`tuple`、智能指针、类型特征、线程与内存模型、future、chrono、random |
| C++14 | 泛型 Lambda、初始化捕获、普通函数 `auto` 返回推导、`decltype(auto)`、变量模板、放宽 `constexpr`、二进制字面量、数字分隔符 | `make_unique`、类型转换别名 `_t`、`integer_sequence`、标准库字符串与时间字面量、`shared_timed_mutex` |
| C++17 | 结构化绑定、`if` / `switch` 初始化语句、`if constexpr`、折叠表达式、CTAD、inline 变量、特定场景保证复制消除 | `optional`、`variant`、`any`、`string_view`、`from_chars`、`filesystem`、`scoped_lock`、`shared_mutex`、`apply`、类型特征 `_v` |
| C++20 | Concepts、`requires`、协程、Modules、`consteval`、`constinit`、三路比较、指定成员初始化、Lambda 显式模板参数列表、`char8_t` | Ranges、`span`、`jthread`、停止令牌、`format`、`bit`、`source_location`、更多 `constexpr` 接口 |
| C++23 | `if consteval`、显式对象参数、多维下标运算符、Lambda 静态调用运算符等 | `expected`、`print` / `println`、`mdspan`、`generator`、更多 Ranges 适配器及容器范围接口 |

`auto` 在传统 C++ 中曾是存储类说明符，表中 C++11 指它用于类型推导的语义。`long long` 从 C++11 起纳入 ISO C++ 标准整数类型。

选择语言标准还要确认标准库实现和具体功能支持。MSVC 没有 `/std:c++11`，GCC / Clang 可用对应的 `-std=c++11`、`-std=c++14`、`-std=c++17`、`-std=c++20`、`-std=c++23` 模式；配置和命令见 [构建与工程指南](C++BuildEngineeringGuide.md)。工具链支持情况随版本变化，应查 [GCC 标准模式文档](https://gcc.gnu.org/onlinedocs/gcc/C_002b_002b-Dialect-Options.html) 与 [MSVC 标准版本选项](https://learn.microsoft.com/en-us/cpp/build/reference/std-specify-language-standard-version?view=msvc-170)。

<a id="cpp11-basics"></a>
## 2. C++11：类型推导、列表初始化与范围 for

### 2.1 `auto`、`decltype` 与引用

`auto` 根据初始化表达式推导静态类型。普通的 `auto` 声明通常去掉引用与顶层 `const`；需要保留借用关系时应明确写 `auto&` 或 `const auto&`。

**片段，最低 C++11；放在函数体内：**

```cpp
int original = 10;
const int& readonly = original;
auto copied = readonly;         // int：复制值
auto& alias = original;         // int&：引用原对象
const auto& view = original;    // const int&：只读借用
decltype(original) same = 20;   // int：未加括号的变量名取声明类型
decltype((original)) ref = original; // int&：括号后按表达式类别推导
```

`decltype` 不对表达式实际求值，适合描述相关类型。括号会影响推导结果，不应机械地给 `decltype` 的操作数添加括号。

### 2.2 列表初始化、`nullptr` 与作用域枚举

花括号可用于初始化标量、类对象和容器，并拒绝规则所定义的窄化转换。初始化与赋值仍是不同操作。

**完整程序，最低 C++11：**

```cpp
#include <iostream>
#include <string>
#include <vector>

enum class Status { idle, running, finished };

int main() {
    int count{};                       // 0
    const std::string name{"Alice"};
    const std::vector<int> values{3, -1, 4, 0, 5};
    int* pointer = nullptr;
    const Status state = Status::running;

    for (int value : values) {
        if (value > 0) {
            count += value;
        }
    }
    if (pointer == nullptr && state == Status::running) {
        std::cout << name << ": " << count << '\n'; // Alice: 12
    }
    // int invalid{3.14}; // 编译错误：窄化
}
```

`nullptr` 是空指针字面量，具有 `std::nullptr_t` 类型，减少用整数 `0` 表示空指针时的重载歧义。`enum class` 的枚举名受作用域限制，不会隐式转换为整数；也可以明确指定底层整数类型。

`std::vector<int>(3, 7)` 创建三个 `7`，`std::vector<int>{3, 7}` 创建两个元素 `3`、`7`。带有 `initializer_list` 构造函数的类型，需要特别留意这类差异。

### 2.3 范围 `for`

| 写法 | 行为 |
| --- | --- |
| `for (auto item : items)` | 复制元素，修改副本不影响容器 |
| `for (auto& item : items)` | 引用元素，允许修改 |
| `for (const auto& item : items)` | 只读引用，避免复制较大对象 |

范围循环要求被遍历的序列在访问期间有效；在循环中插入或删除元素仍要考虑迭代器失效。反向遍历可以用反向迭代器，不要写无符号的 `i >= 0` 倒数循环。


<a id="cpp11-constexpr"></a>
## 3. C++11：constexpr、static_assert 与类型特征

`const` 限制通过该对象进行修改，不要求初始化在编译期完成。`constexpr` 变量需要常量表达式初始化；`constexpr` 函数既可参与常量表达式，也可用运行期输入调用。

C++11 的 `constexpr` 函数体限制较严格，常见写法是单个返回表达式；循环和普通局部变量等写法留到 C++14。`static_assert` 在编译期间检查条件，C++11 / C++14 需要消息参数。

**完整程序，最低 C++11：**

```cpp
#include <iostream>
#include <type_traits>

constexpr int square(int value) {
    return value * value;
}

template <typename T>
struct PlainType {
    using type = typename std::remove_cv<
        typename std::remove_reference<T>::type>::type;
};

int main() {
    constexpr int area = square(6);
    static_assert(area == 36, "unexpected square");
    static_assert(std::is_integral<int>::value, "int must be integral");
    static_assert(std::is_same<PlainType<const int&>::type, int>::value,
                  "remove reference and const");
    int runtimeValue = 7;
    std::cout << square(runtimeValue) << '\n'; // 49
}
```

`<type_traits>` 提供类型属性与转换工具，如 `is_integral`、`is_same`、`remove_reference`、`enable_if`。C++11 通过 `::value` 访问布尔结果，通过 `typename ...::type` 访问转换后的类型；更短的 `_t` 别名从 C++14 起，`_v` 变量模板从 C++17 起。

别名声明 `using Count = unsigned int;` 与别名模板也是 C++11 特性。它们为已有类型起名字，不会制造新的强类型。可变参数模板在 C++11 中允许接受参数包，C++17 的折叠表达式进一步简化参数包操作。


<a id="cpp11-classes"></a>
## 4. C++11：现代类接口

### 4.1 默认成员初始化、委托构造与特殊成员函数

**片段，最低 C++11；类型定义放在命名空间作用域：**

```cpp
class Counter {
public:
    Counter() = default;
    explicit Counter(int initial) : value_(initial) {}
    Counter(int initial, bool) : Counter(initial) {} // 委托构造

    Counter(const Counter&) = delete;
    Counter& operator=(const Counter&) = delete;
    Counter(Counter&&) = default;
    Counter& operator=(Counter&&) = default;

    int value() const noexcept { return value_; }

private:
    int value_ = 0; // 默认成员初始化；若初始化列表指定此成员，则采用列表
};
```

`= default` 请求按规则生成默认实现，`= delete` 禁止相应调用。默认实现也可能因为成员类型不满足要求而被定义为删除。委托构造把初始化工作交给同类另一个构造函数；委托初始化列表不能同时再初始化其他成员。

`noexcept` 表达不抛异常的契约，若异常逃出该函数会调用 `std::terminate`。只有能够保证时才添加。对资源类型声明移动操作为 `noexcept`，也会影响容器扩容时选择移动还是复制。

### 4.2 `override` 与 `final`

继承与虚函数本身属于传统 C++；C++11 的 `override` 要求确实覆盖基类虚函数，`final` 禁止继续派生或覆盖。

**完整程序，最低 C++11：**

```cpp
#include <iostream>
#include <memory>

class Shape {
public:
    virtual ~Shape() = default;
    virtual double area() const = 0;
};

class Rectangle final : public Shape {
public:
    Rectangle(double width, double height) : width_(width), height_(height) {}
    double area() const override { return width_ * height_; }

private:
    double width_;
    double height_;
};

int main() {
    std::unique_ptr<Shape> shape(new Rectangle(3.0, 4.0));
    std::cout << shape->area() << '\n'; // 12
}
```

示例约定宽高是有效的非负有限值，实际接口应按需求校验。这里使用 C++11 的 `unique_ptr` 构造方式；C++14 可写 `std::make_unique<Rectangle>(3.0, 4.0)`。通过基类指针删除派生对象时，基类应提供合适的虚析构函数。不要按基类值传递派生对象，以免切片。


<a id="cpp11-move"></a>
## 5. C++11：右值引用、移动与完美转发

### 5.1 初始化、赋值与值类别

**片段，最低 C++11；需要 `<string>` 与 `<utility>`，放在函数体内：**

```cpp
std::string first = "hello";
std::string second = first;           // 复制构造
second = first;                       // 复制赋值
std::string third = std::move(first); // 可选择移动构造
```

左值通常表示可重复访问、有身份的对象；右值包括纯右值和将亡值。`T&&` 可表达右值引用，但**有名字的变量表达式仍是左值**，即使该变量声明类型是 `T&&`。

`std::move` 本身只转换值类别，并不转移资源；具体操作由后续构造、赋值或重载决定。对 `const` 对象使用 `std::move` 通常仍然发生复制，因为常见移动操作需要非 `const` 的右值引用。

除非具体接口另有保证，标准库对象移动后处于**有效但状态未指定**的状态。可以销毁、重新赋值，或执行前置条件得到满足的操作；不能假设移动后的 `std::string` 必为空。

### 5.2 Rule of Zero / Three / Five

| 原则 | 需要审视的设计 |
| --- | --- |
| Rule of Zero | 用 `string`、`vector`、智能指针等成员管理资源，尽量不手写特殊成员函数 |
| Rule of Three | 手动管理资源时一起考虑析构、复制构造、复制赋值 |
| Rule of Five | 现代 C++ 还应一起考虑移动构造和移动赋值 |

这些是设计原则，不要求把所有函数都写出来。用户声明析构函数会阻止隐式移动操作的生成；成员类型和其他声明也会影响某项操作是否存在、是否被删除。

### 5.3 转发引用与 `std::forward`

函数模板中的 `T&&` 在 `T` 从实参推导等适用条件下可以是转发引用；`const T&&` 或类模板中已经确定的 `T&&` 不能一概称为转发引用。推导与引用折叠让它能够接收左值和右值，`std::forward<T>` 用于保留原实参的值类别。

**完整程序，最低 C++11：**

```cpp
#include <iostream>
#include <string>
#include <utility>

void consume(const std::string& text) {
    std::cout << "borrow: " << text << '\n';
}

void consume(std::string&& text) {
    std::string owned = std::move(text);
    std::cout << "take: " << owned << '\n';
}

template <typename T>
void relay(T&& value) {
    consume(std::forward<T>(value));
}

int main() {
    std::string text = "hello";
    relay(text);              // 左值：borrow
    relay(std::string("hi")); // 右值：take
    relay(std::move(text));   // 允许移动：take
    text = "reused";          // 移动后可重新赋值
    std::cout << text << '\n';
}
```

转发用于确实需要转交实参的泛型接口；普通业务函数优先选择清楚的传值、`const T&` 或 `T&`。同一个转发实参若被多次转发为右值，第一次操作可能已经消耗其状态。

返回局部值对象通常写 `return result;`，编译器可进行 NRVO 或在规则允许时移动。随手写 `return std::move(result);` 可能妨碍 NRVO；C++17 的保证复制消除见 [第 13 节](#cpp17-elision)。


<a id="cpp11-lambda"></a>
## 6. C++11：Lambda 与可调用对象

Lambda 在使用处创建函数对象，适合算法、短小局部操作和回调。C++11 Lambda 的普通参数要写明确类型；使用 `auto` 参数的泛型 Lambda 是 C++14。

**完整程序，最低 C++11：**

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
    const std::vector<int> values{1, 4, 7, 10};
    const int threshold = 5;
    const auto count = std::count_if(values.begin(), values.end(),
        [threshold](int value) { return value > threshold; });

    int original = 0;
    auto localCounter = [original]() mutable { return ++original; };
    std::cout << count << '\n';          // 2
    std::cout << localCounter() << '\n'; // 1
    std::cout << original << '\n';       // 0：修改的是捕获副本
}
```

| 捕获 | 最低标准 | 含义 |
| --- | --- | --- |
| `[]` | C++11 | 不捕获局部变量 |
| `[value]` / `[&value]` | C++11 | 捕获值副本 / 借用原对象 |
| `[=]` / `[&]` | C++11 | 默认按值 / 按引用捕获需要捕获的局部实体 |
| `[this]` | C++11 | 捕获当前对象指针，不复制对象 |
| `[owned = std::move(value)]` | C++14 | 初始化捕获，可接管可移动对象 |
| `[*this]` | C++17 | 按值捕获当前对象的副本 |

按值捕获默认不能在调用运算符内修改捕获成员，`mutable` 可改变这一点，但不会因此修改原变量。按引用捕获与 `[this]` 必须保证相关对象存活到调用结束，尤其是异步回调。`[=]` 在成员函数中涉及 `this` 时也不意味着复制整个对象。

能直接使用模板参数或 `auto` 保存 Lambda 时，不一定要包装为 `<functional>` 的 `std::function`。本文标准中的 `std::function` 要求目标可复制；捕获 `unique_ptr` 的闭包通常不满足该要求。


<a id="cpp11-library-summary"></a>
## 7. C++11：标准库新增能力与整数类型

`std::array` 提供固定大小连续数组，`unordered_map` / `unordered_set` 提供哈希容器，`forward_list` 提供单向链表，`tuple` 组合不同类型的值。容器的列表初始化、范围遍历、移动与 `emplace` 接口也让已有容器更方便使用；具体选择、算法前置条件和句柄失效见 [标准库指南](C++StandardLibraryGuide.md)。

### 7.1 固定宽度与相关整数

`<cstdint>` 从 C++11 起提供 `std::int8_t`、`std::int16_t`、`std::int32_t`、`std::int64_t`，以及对应的 `std::uint8_t`、`std::uint16_t`、`std::uint32_t`、`std::uint64_t`。数字表示位数；精确宽度类型是可选别名，不能假定所有平台均提供。

| 类型族 | 含义 |
| --- | --- |
| `std::intN_t` / `std::uintN_t` | 精确 N 位，无填充位；满足实现条件时提供 |
| `std::int_leastN_t` / `std::uint_leastN_t` | 至少 N 位，满足要求且存储大小最小 |
| `std::int_fastN_t` / `std::uint_fastN_t` | 至少 N 位，由实现选择通常运算较快的类型 |
| `std::intmax_t` / `std::uintmax_t` | 可表示对应其他整数类型全部值 |
| `std::intptr_t` / `std::uintptr_t` | 可表示 `void*` 转换结果并支持转回；本文涉及标准中是可选类型 |

完整的取值范围、可选性、`size_t` / `ptrdiff_t` 比较、`numeric_limits`、常量与格式宏、8 位输出、整数提升、符号混用和转换陷阱均保留在 [标准库指南的整数专题](C++StandardLibraryGuide.md)。其中 `least` 的“最小”指存储大小，不是仅比较有效位数；整数转无符号的模规则也不能推广到浮点转整数。相关别名声明见 [C++17 `<cstdint>` 草案](https://timsong-cpp.github.io/cppwp/n4659/cstdint.syn)。

**片段，最低 C++11；需要 `<cstdint>`、`<limits>`，放在函数体内；要求实现提供所用精确类型：**

```cpp
const std::uint32_t packetLength{1024};
const std::int32_t offset{-12};
const auto upper = std::numeric_limits<std::uint32_t>::max();
```

### 7.2 智能指针、时间、随机与并发

`unique_ptr`、`shared_ptr`、`weak_ptr` 和 `make_shared` 都属于 C++11，`make_unique` 是 C++14。RAII 是传统 C++ 已有的资源管理原则；智能指针将所有权关系放进标准接口，使用详解见 [标准库指南](C++StandardLibraryGuide.md)。

`chrono` 表达时间点和时长，`random` 将随机引擎与分布分开。标准线程、互斥锁、条件变量、原子操作、future 和内存模型也从 C++11 起提供，详见 [并发指南](C++ConcurrencyGuide.md)。`volatile` 不提供线程同步，原子变量也不自动组成多字段事务。

<a id="cpp14-lambda"></a>
## 8. C++14：泛型 Lambda、初始化捕获与 make_unique

泛型 Lambda 的参数可以使用 `auto`，相当于闭包拥有模板调用运算符。初始化捕获允许在捕获位置构造一个新状态，也能将独占资源移动进闭包。规范见 [C++14 草案的 Lambda 条款](https://timsong-cpp.github.io/cppwp/n4140/expr.prim.lambda)。

**完整程序，最低 C++14：**

```cpp
#include <iostream>
#include <memory>
#include <string>
#include <utility>

int main() {
    auto larger = [](const auto& left, const auto& right) {
        return left < right ? right : left;
    };
    std::cout << larger(3, 7) << '\n';       // 7
    std::cout << larger(2.5, 1.2) << '\n';   // 2.5

    auto resource = std::make_unique<std::string>("owned text");
    auto show = [owned = std::move(resource)] {
        std::cout << *owned << '\n';
    };
    show();
    std::cout << std::boolalpha << (resource == nullptr) << '\n'; // true
}
```

`std::make_unique<T>(args...)` 从 C++14 起提供，转发参数并返回独占智能指针；`std::make_unique<T[]>(n)` 创建并值初始化 n 个数组元素。`make_shared` 早在 C++11 就已存在。

上例闭包拥有 `unique_ptr`，因此不可复制；可以直接保存或移动，不能装进要求目标可复制的 `std::function`。初始化捕获也可写 `[copy = expression]` 或 `[&alias = object]`，按引用形式依然要求对象有效。

泛型 Lambda 可以使用 `auto&&` 参数转发实参，但应配合 `std::forward<decltype(value)>(value)` 表达真实转发需求。不要因为参数是 `auto&&` 就在每次使用时无条件移动。


<a id="cpp14-deduction"></a>
## 9. C++14：返回类型推导、decltype(auto) 与类型别名

### 9.1 普通函数返回类型推导

C++11 已支持 `auto f(...) -> ReturnType` 的尾置返回类型；C++14 进一步允许普通函数写 `auto f(...) { return expression; }` 并由定义推导返回类型。多个返回语句需要符合一致的推导要求，调用点需要能够看到完成推导的定义。

`decltype(auto)` 使用 `decltype` 规则，能保留返回表达式的引用关系；普通 `auto` 返回值通常复制。它也会让括号差异变得关键，应谨慎使用。

**完整程序，最低 C++14：**

```cpp
#include <iostream>
#include <type_traits>
#include <vector>

auto add(int left, int right) {
    return left + right;
}

template <typename Container>
decltype(auto) first(Container& values) {
    return values.front(); // 前置条件：容器非空；保留返回的引用
}

int main() {
    std::vector<int> values{1, 2, 3};
    first(values) = 9;
    std::cout << values.front() << ' ' << add(2, 3) << '\n'; // 9 5

    int original = 10;
    decltype(auto) copy = original;   // int
    decltype(auto) ref = (original);  // int&
    static_assert(std::is_same<decltype(copy), int>::value, "value");
    static_assert(std::is_same<decltype(ref), int&>::value, "reference");
    ref = 20;
    std::cout << original << '\n'; // 20
}
```

不能用 `decltype(auto)` 返回对局部对象的引用，例如 `return (local);` 会产生悬空引用。返回一个表达式时，应先明确调用者应该获得独立值还是借用对象。

### 9.2 `_t` 别名与 `integer_sequence`

C++14 在 `<type_traits>` 中增加 `remove_reference_t<T>`、`enable_if_t<...>` 等别名，简化 C++11 的 `typename Trait<T>::type` 写法。

**片段，最低 C++14；需要 `<type_traits>`，放在命名空间作用域：**

```cpp
template <typename T>
using Plain = std::remove_cv_t<std::remove_reference_t<T>>;
```

`<utility>` 的 `integer_sequence`、`index_sequence` 与 `make_index_sequence` 可生成编译期索引包，用于展开 tuple、固定数组等。普通应用代码可以先使用清楚的循环与库接口；需要泛型展开时再引入这些工具。


<a id="cpp14-constexpr"></a>
## 10. C++14：constexpr 放宽、变量模板与字面量

### 10.1 放宽的 `constexpr`

C++14 允许 `constexpr` 函数使用初始化的局部变量、分支和循环等，使编译期计算更接近普通函数写法；仍有常量表达式和函数体限制。规范见 [C++14 constexpr 条款](https://timsong-cpp.github.io/cppwp/n4140/dcl.constexpr)。

**完整程序，最低 C++14：**

```cpp
#include <iostream>

constexpr int sumTo(int limit) {
    int result = 0;
    for (int value = 1; value <= limit; ++value) {
        result += value;
    }
    return result;
}

template <typename T>
constexpr T pi = static_cast<T>(3.14159265358979323846L);

int main() {
    static_assert(sumTo(10) == 55, "unexpected sum");
    constexpr int mask = 0b1010;     // 二进制字面量
    constexpr int count = 1'000'000; // 数字分隔符
    std::cout << sumTo(10) << ' ' << mask << ' ' << count << '\n';
    std::cout << pi<double> << '\n';
}
```

`pi<T>` 是 C++14 变量模板，每种 T 对应一个变量实例；它与 C++17 的 inline 变量是不同特性。这个 `sumTo` 示例只用于给定小范围输入，不处理大输入导致的整数溢出。

### 10.2 标准库字面量与其他补充

**片段，最低 C++14；需要 `<chrono>` 与 `<string>`，放在函数体内：**

```cpp
using namespace std::chrono_literals;
using namespace std::string_literals;
const auto timeout = 250ms;
const auto text = "hello"s; // std::string，而不是字符指针
```

把字面量命名空间的 `using` 限制在合适作用域中，不在公共头文件全局导入。C++14 还有 `std::exchange`、`std::shared_timed_mutex` 等标准库补充；`exchange` 用新值替换对象并返回旧值，常用于实现明确的状态转移。


<a id="cpp17-bindings"></a>
## 11. C++17：结构化绑定、if 初始化与 inline 变量

### 11.1 结构化绑定与局部初始化

结构化绑定可将数组、满足规则的 tuple 类对象或成员可访问的对象分解为多个名字。`auto [a, b]` 通常从一个隐藏的值对象分解，`auto&` / `const auto&` 表达对原对象的借用。

**完整程序，最低 C++17：**

```cpp
#include <iostream>
#include <map>
#include <string>

int main() {
    std::map<std::string, int> scores{{"Alice", 90}, {"Bob", 80}};
    for (auto& [name, score] : scores) {
        score += 1; // 修改原字典中的值；map 的键仍是 const
        std::cout << name << ' ' << score << '\n';
    }

    if (const auto found = scores.find("Alice"); found != scores.end()) {
        const auto& [name, score] = *found;
        std::cout << "found: " << name << ' ' << score << '\n';
    }
}
```

`if (init; condition)` 将临时变量的作用域限制在整个 `if` / `else` 中；`switch` 也支持初始化语句。结构化绑定不会自动延长被借用容器元素的有效期。

### 11.2 Inline 变量、属性与嵌套命名空间

`inline` 变量可以在头文件中定义并被多个翻译单元包含，满足 ODR 要求时表示同一个实体；类中可用 `inline static` 数据成员。C++17 的 `static constexpr` 数据成员也隐式为 inline。

**片段，最低 C++17；放在头文件的命名空间作用域：**

```cpp
namespace guide::config {
inline constexpr int defaultLimit = 100;

struct Settings {
    inline static int activeCount = 0;
};
}
```

| 写法 | 用途 |
| --- | --- |
| `[[nodiscard]]` | 提示不要忽略结果，适合有检查意义的函数或类型 |
| `[[maybe_unused]]` | 表达变量等实体可能有意不使用 |
| `[[fallthrough]];` | 在 `switch` 中表达有意贯穿到下一分支 |
| `static_assert(condition);` | 允许省略诊断消息 |

属性不等于运行期检查；例如 `nodiscard` 不自动处理错误。`[[deprecated]]` 已在 C++14 引入，不应把所有属性都归为 C++17。


<a id="cpp17-templates"></a>
## 12. C++17：if constexpr、折叠表达式与 CTAD

### 12.1 编译期分支与参数包

`if constexpr` 在模板实例化中按常量条件选择分支，未选分支在适用规则下不实例化。它不是简单的运行期 `if`；但也不是“任何错误代码都能藏进去”，语法仍需正确，非模板上下文的检查等规则仍然存在。

折叠表达式把参数包与运算符组合，取代许多递归模板写法。空参数包的行为取决于折叠形式和运算符，下例提供初始值 `0`。

**完整程序，最低 C++17：**

```cpp
#include <iostream>
#include <string>
#include <type_traits>

template <typename T>
void describe(const T& value) {
    if constexpr (std::is_integral_v<T>) {
        std::cout << "integer: " << value << '\n';
    } else {
        std::cout << "other: " << value << '\n';
    }
}

template <typename... Ts>
auto sum(Ts... values) {
    return (0 + ... + values);
}

int main() {
    describe(42);
    describe(std::string("hello"));
    std::cout << sum(1, 2, 3) << ' ' << sum() << '\n'; // 6 0
}
```

`std::is_integral_v<T>` 是 `std::is_integral<T>::value` 的 C++17 简写。`void_t` 可用于检测类型表达式是否有效；实际需求较简单时，优先选择直接的模板接口与清楚的诊断。

### 12.2 类模板实参推导（CTAD）

**片段，最低 C++17；需要 `<array>`、`<utility>` 与 `<vector>`，放在函数体内：**

```cpp
std::pair record{1, 2.5};       // std::pair<int, double>
std::vector values{1, 2, 3};    // std::vector<int>
std::array fixed{1, 2, 3};      // std::array<int, 3>
```

CTAD 利用构造函数与推导指引推导类模板实参，并不保证任何表达式都能推导。`auto` 推导变量的类型，与 CTAD 推导类模板实参是两个相关但不同的机制。接口处明确写类型通常更便于表达稳定意图。

<a id="cpp17-elision"></a>
## 13. C++17：复制消除与标准库扩展

### 13.1 保证复制消除与 NRVO 的区别

C++17 在特定同类型纯右值初始化场景下直接构造最终对象，不要求中间复制或移动；析构函数仍需满足可访问等要求。返回具名局部变量的 NRVO 则一般仍是可选优化。

**完整程序，最低 C++17：**

```cpp
#include <iostream>

struct Token {
    Token() { std::cout << "constructed\n"; }
    ~Token() = default;
    Token(const Token&) = delete;
    Token(Token&&) = delete;
};

Token makeToken() {
    return Token{}; // 同类型纯右值：直接构造结果对象
}

int main() {
    [[maybe_unused]] Token token = makeToken();
}
```

若将函数改为 `Token local; return local;`，因为 NRVO 不保证发生而且复制 / 移动都删除，该写法不成立。复制消除是语言规则与允许的优化，不能只根据一次日志观察推广所有返回场景。

### 13.2 标准库扩展

| C++17 设施 | 主要作用 | 使用细节 |
| --- | --- | --- |
| `optional`、`variant`、`any` | 组织缺失值、固定候选类型或开放类型集合 | 见标准库指南 |
| `string_view` | 借用字符序列 | 不拥有数据，不延长生命周期 |
| `filesystem` | 路径与目录操作 | 文件系统权限、编码和失败仍要处理 |
| `from_chars` / `to_chars` | 字符序列与数值转换 | 检查错误码与实际消费范围，具体实现支持需确认 |
| `scoped_lock`、`shared_mutex` | 锁管理和读写同步 | 见并发指南 |
| `invoke`、`apply`、`as_const` | 通用调用、展开 tuple、只读访问 | 不替代接口生命周期分析 |

固定宽度整数、字符串、容器和成绩统计完整项目见 [标准库指南](C++StandardLibraryGuide.md)，线程与同步见 [并发指南](C++ConcurrencyGuide.md)。

<a id="cpp20-concepts"></a>
## 14. C++20：Concepts、requires 与 span

Concepts 为模板参数表达编译期约束，`requires` 表达式可以检查所需操作。约束类型能力不等于检查运行期数值范围，也不自动证明业务语义正确。

`std::span<T>` 借用连续元素，携带长度；`span<const T>` 提供只读元素访问。它不拥有元素，源容器销毁或重新分配后，旧 span 可能失效。

**完整程序，最低 C++20：**

```cpp
#include <concepts>
#include <iostream>
#include <span>
#include <vector>

template <std::integral T>
T square(T value) {
    return value * value;
}

int sum(std::span<const int> values) {
    int result = 0;
    for (int value : values) { result += value; }
    return result;
}

int main() {
    const std::vector<int> values{1, 2, 3};
    std::cout << square(4) << '\n'; // 16
    std::cout << sum(values) << '\n'; // 6
}
```

本例约定输入范围和总和不会溢出，调用期间 `values` 保持有效。Concepts 与 span 的标准定义见 [C++20 约束条款](https://timsong-cpp.github.io/cppwp/n4861/temp.constr) 和 [span 概览](https://timsong-cpp.github.io/cppwp/n4861/views.span)。

<a id="cpp20-constants"></a>
## 15. C++20：consteval、constinit 与三路比较

- `consteval` 声明立即函数，适用的调用必须在编译期求值。
- `constinit` 要求静态或线程存储期变量进行静态初始化；不意味着变量不可修改。
- `<=>` 和比较类别类型支持三路比较，用户可声明默认比较操作。

**完整程序，最低 C++20：**

```cpp
#include <compare>
#include <iostream>

consteval int immediateSquare(int value) { return value * value; }
constinit int globalCount = 0;

struct Point {
    int x;
    int y;
    auto operator<=>(const Point&) const = default;
};

int main() {
    constexpr int result = immediateSquare(4);
    const Point first{1, 2};
    const Point second{1, 3};
    ++globalCount;
    std::cout << result << '\n'; // 16
    std::cout << std::boolalpha << (first < second) << '\n'; // true
    std::cout << globalCount << '\n'; // 1
}
```

`Point` 按成员声明顺序比较，因此是字典序，未表达几何距离。浮点成员可能产生偏序比较，不能把所有 `<=>` 结果都当作全序。编译期算术同样必须遵守数值范围。

<a id="cpp20-facilities"></a>
## 16. C++20：Ranges、协程、Modules 与其他补充

### 16.1 Ranges 与惰性视图

Ranges 提供范围算法和可组合视图。视图经常采用惰性求值，保存它不意味着已经生成独立结果容器；要检查底层对象的生命周期和句柄失效规则。

**完整程序，最低 C++20：**

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> values{1, 2, 3, 4, 5, 6};
    auto selected = values
        | std::views::filter([](int value) { return value % 2 == 0; })
        | std::views::transform([](int value) { return value * 2; });
    for (int value : selected) { std::cout << value << ' '; }
    std::cout << '\n'; // 4 8 12，带一个末尾空格
}
```

遍历期间不修改或销毁源容器。并非所有视图都能通过 `const` 对象遍历，也并非所有范围都支持同一种迭代器操作，见 [C++20 范围库条款](https://timsong-cpp.github.io/cppwp/n4861/ranges)。

### 16.2 协程与 Modules

协程通过 `co_await`、`co_yield`、`co_return` 支持可暂停恢复的函数机制。使用它需要符合协程协议的返回类型和相关对象；语言本身不自动提供线程、事件循环、调度器或异步 I/O。协程帧、悬挂点和捕获对象也需要生命周期管理。

Modules 使用模块接口与导入组织代码，不能仅把 `#include` 替换成 `import` 就假定项目可构建。构建顺序、依赖扫描、编译产物和第三方库支持均取决于工具链，详见工程化指南。

### 16.3 其他常用补充

| 设施 | 作用或边界 |
| --- | --- |
| 指定成员初始化 | 按允许的聚合成员规则初始化，不能任意乱序指定 |
| Lambda 显式模板参数列表 | 直接表达泛型 Lambda 模板参数 |
| `char8_t` / `u8` 字面量 | UTF-8 字符单元类型变化，旧 `std::string` 接口要注意兼容 |
| `std::format` | 类型安全的格式化，需相应标准库支持 |
| `std::bit_cast` / `std::endian` | 合适类型的表示转换与端序查询，不替代外部格式定义 |
| `std::source_location` | 记录调用位置等信息 |
| `jthread` / `stop_token` | 协作式停止和线程回收，详见并发指南 |

C++20 的部分标准容器支持更多常量求值操作，不意味着任意动态分配结果都能长期保存为编译期对象。

<a id="cpp23-language"></a>
## 17. C++23：if consteval 与显式对象参数

`if consteval` 根据调用所处的求值上下文选择分支，与 `if constexpr` 根据常量条件选择编译期分支的用途不同。

**完整程序，最低 C++23；要求编译器支持该特性：**

```cpp
#include <iostream>

constexpr int choosePath() {
    if consteval {
        return 1;
    } else {
        return 2;
    }
}

int main() {
    constexpr int atCompileTime = choosePath();
    int atRunTime = choosePath();
    std::cout << atCompileTime << ' ' << atRunTime << '\n'; // 1 2
}
```

运行期调用即使被编译器优化，也不因此变成语言意义上的常量求值。相关背景见 [WG21 `if consteval` 提案](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p1938r3.html)。

显式对象参数允许在成员函数参数列表中显式表达对象参数，为减少不同引用限定重载的重复等需求提供工具。其他语言扩展还包括多维下标运算符和 Lambda 的静态调用运算符；应查工具链是否实现，不只看标准选项名称。

<a id="cpp23-library"></a>
## 18. C++23：expected、print 与标准库补充

| 设施 | 用途 | 注意事项 |
| --- | --- | --- |
| `std::expected<T, E>` | 值或错误 | 错误类型由接口定义，不能无条件访问值 |
| `std::print` / `println` | 格式化输出 | 需要 `<print>` 和相应实现支持 |
| `std::mdspan` | 多维数组借用访问 | 不拥有元素，布局和访问器是契约的一部分 |
| `std::generator` | 同步协程生成序列 | 不自动提供异步调度或线程 |
| `std::move_only_function` | 支持仅可移动目标的可调用包装 | 与需要可复制目标的 `std::function` 区分 |
| `std::byteswap` | 支持范围内整数的字节交换 | 不自动完成完整协议的编码校验 |
| 新 Ranges 适配器与容器范围接口 | 更丰富的数据处理 | 逐项确认最低版本和实现支持 |

这些组件不能因为开启标准模式就被假定可用，完整使用契约仍要查对应实现文档。优先将最小示例纳入构建，必要时结合特性测试宏检测。

<a id="build"></a>
## 19. 编译配置与特性检测

GCC / Clang 的语言模式使用 `-std=c++11`、`-std=c++14`、`-std=c++17`、`-std=c++20` 或 `-std=c++23`，配合 `-Wall -Wextra -Wpedantic`。较旧工具链可能没有全部模式或功能。

现代 MSVC 最早的可选 C++ 模式为 C++14，没有独立 C++11 模式。C++14 / 17 / 20 分别使用相应 `/std` 参数；更晚特性按安装版本选择受支持的标准或预览模式。`/std:c++latest` 可能包含多个后续版本特性，不能用它证明某示例的最低版本。见 [MSVC 官方文档](https://learn.microsoft.com/en-us/cpp/build/reference/std-specify-language-standard-version?view=msvc-170)。

语言特性可检查相应 `__cpp_*` 宏，标准库设施可检查 `__cpp_lib_*` 宏。库宏需要包含适当头文件，较新实现可通过 `<version>` 汇总；宏应按具体特性及要求值使用，不只检查是否定义。

完整 CMake 项目、标准传播与构建配置见 [工程化指南](C++BuildEngineeringGuide.md)。不要把编辑器语法设置当作真实编译参数。

<a id="learning"></a>
## 20. 学习顺序与参考资料

学习顺序：C++11 的初始化与所有权 → 移动与引用 → Lambda 与泛型 → C++14 的表达简化 → C++17 的分支和结果组织 → 根据项目需要学习 C++20/23。

练习时，对熟悉的传统例子做局部改写，并说明新增语法改变了什么契约。优先验证生命周期、移动后状态、错误访问和输入边界，而不是仅比较代码长度。

参考 [GCC C++ 特性实现表](https://gcc.gnu.org/projects/cxx-status.html)、[Microsoft C++ 文档](https://learn.microsoft.com/en-us/cpp/)、[WG21 标准资料](https://www.open-std.org/jtc1/sc22/wg21/)。标准库详解、并发和系统接口各有独立分卷，继续从总导航选择需要的主题。

