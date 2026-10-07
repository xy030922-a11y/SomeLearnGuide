# C++ 核心语言指南

> 本文讲语言本身的基础：名字与类型、表达式、函数、对象生命周期、类、继承、模板与异常。沿用本系列的 **C++98/03 基础范围**，涉及两版差异时单独说明。新增版本语法见 [现代 C++ 标准特性指南](C++ModernFeaturesGuide.md)，标准库组件详解见 [C++ 标准库指南](C++StandardLibraryGuide.md)。

[返回总导航](C++Guide.md) · [并发](C++ConcurrencyGuide.md) · [底层与系统](C++SystemProgrammingGuide.md) · [编译与工程化](C++BuildEngineeringGuide.md)

**示例约定：** “完整程序”包含头文件和 `main()`，分别保存为 `main.cpp`；“片段”需要补齐上下文。注释中的错误写法仅用于说明。例子使用少量标准库设施展示结果，其接口细节放在标准库专题。

**阅读方法：** 每个主题先理解规则，再观察示例，最后检查前置条件和失败路径。不要把“能编译”“运行一次正常”和“符合语言规则”混为一谈。首次阅读可依次读 1–6 章，再学习复制、多态和模板。

## 阅读导航

1. [程序与作用域](#program)
2. [类型与初始化](#types)
3. [表达式与流程控制](#control)
4. [函数与参数](#functions)
5. [指针、引用与生命周期](#lifetime)
6. [类与对象](#classes)
7. [复制、赋值与资源](#copy)
8. [继承与多态](#polymorphism)
9. [模板与泛型](#templates)
10. [异常与接口契约](#exceptions)
11. [常见错误与练习](#practice)

## 重点速查

| 要查的规则 | 位置 |
| --- | --- |
| 作用域、链接与存储期 | [1.3](#scope-linkage) |
| 初始化与函数声明歧义 | [2.3](#initialization) |
| 聚合初始化与 sizeof | [2.7](#aggregate-initialization)、[2.8](#sizeof) |
| 优先级与求值顺序 | [3.2](#evaluation-order) |
| 重载与 const 限定 | [4.3](#overloads) |
| ADL 与默认表达式 | [4.9](#adl-defaults) |
| 数组退化和指针范围 | [5.2](#array-decay) |
| 指向成员的指针 | [6.7](#member-pointers) |
| 隐藏、重载和覆盖 | [8.3](#virtual-dispatch) |
| 模板全特化、偏特化与重载 | [9.5](#specialization) |
| 传统类型 SFINAE 示例 | [9.9](#type-sfinae) |
| 构造失败与异常安全 | [10.3](#construction-failure) |

<a id="program"></a>
## 1. 程序与作用域

### 1.1 程序入口与编译模式

**完整程序：**

```cpp
#include <iostream>

int main() {
    std::cout << "Hello, C++!\n";
    return 0;
}
```

`main()` 是普通宿主环境的入口，返回 `0` 表示成功。`#include` 引入声明，`std::` 指定标准库命名空间。

使用 GCC / Clang 检查传统标准语法：

```bash
g++ -std=c++98 -Wall -Wextra -pedantic-errors main.cpp -o demo
```

也可用 `-std=c++03`。现代 MSVC 没有严格的 C++98/03 模式，因此在 MSVC 中编译成功不能单独证明旧标准兼容性，见 [MSVC 标准模式说明](https://learn.microsoft.com/en-us/cpp/build/reference/std-specify-language-standard-version?view=msvc-170)。更完整的工具链配置见工程化指南。

变量和函数名在对应作用域中可见。块作用域、类作用域、命名空间作用域决定名字的查找和访问；**名字仍可见**与**对象仍存活**是不同问题。

`namespace` 用于组织名字。声明描述接口，定义创建实体或提供实现。跨源文件的定义还要遵守单一定义规则，见工程化指南。

### 1.2 声明、定义与名字查找

| 代码 | 含义 |
| --- | --- |
| `int calculate(int);` | 函数声明，尚未提供实现 |
| `int count = 0;` | 定义变量并初始化 |
| `extern int total;` | 声明外部变量，通常不提供定义 |
| `class Widget;` | 类的前置声明，类型仍不完整 |
| `typedef unsigned long Counter;` | 声明类型别名 |

使用名字前需要相应声明可见。嵌套作用域中的同名声明可能隐藏外层名字，名字查找之后才进行重载匹配或访问检查。

**完整程序：限定名与局部遮蔽。**

```cpp
#include <iostream>

int value = 1;
namespace settings { int value = 2; }

int main() {
    int value = 3;
    std::cout << value << ' ' << ::value << ' ' << settings::value << '\n';
    return 0;
}
```

输出 `3 1 2`。`::value` 指全局命名空间中的名字，`settings::value` 指指定命名空间中的名字。实际代码应避免为了方便而大量使用容易混淆的同名变量。

本例有意展示名字遮蔽，开启相应诊断时会收到警告；这是可被语言规则解释的写法，不应与未定义行为混淆。

`using std::cout;` 引入特定名字，`using namespace std;` 影响未限定名查找的候选集合。头文件中的全局 using 指令会影响所有包含者；命名空间别名则可以给长名字取一个局部的短名称。

<a id="scope-linkage"></a>
### 1.3 作用域、存储期与链接

| 概念 | 回答的问题 | 例子 |
| --- | --- | --- |
| 作用域 | 名字在哪里可见？ | 局部变量名只在对应块中使用 |
| 存储期 | 存储持续多久？ | 函数内 `static` 对象的存储持续到程序结束 |
| 对象生命周期 | 对象何时已构造、何时被销毁？ | 动态存储中可以先取得存储，再创建对象 |
| 链接 | 不同作用域或翻译单元的声明能否指同一实体？ | 外部链接函数可由另一个源文件调用 |

命名空间作用域的普通变量通常有外部链接；使用 `static` 可为相应实体安排内部链接。命名空间作用域的非 `extern`、非 `volatile` 普通 `const` 变量在此基础版本中通常有内部链接。具体声明仍要结合先前声明等规则判断。

未命名命名空间也可将辅助名字限制在当前翻译单元的访问范围内，但在 C++98/03 中，其成员可以有外部链接，由翻译单元独有的命名空间名隔离；不要把这一旧版链接规则直接等同于 `static`。后续标准对此作了调整。

函数内 `static` 变量的名字仍是局部名字，不因此获得外部链接。多文件接口和 ODR 详见工程化指南。

### 1.4 正确区分几种结果

| 分类 | 含义 | 示例 |
| --- | --- | --- |
| 良构程序 | 满足相应语法和语义要求 | 用合法类型和有效参数调用函数 |
| 非良构程序 | 违反语言要求，常见情况需要诊断 | `int& r = 1;`：非 const 左值引用不能这样绑定 |
| 未定义行为 | 标准不规定该执行的结果 | 有符号溢出、越界解引用 |
| 未指定行为 | 存在允许的选择，不要求实现固定并说明选择 | 普通函数实参的求值顺序 |
| 实现定义行为 | 实现需说明选择 | 普通 `char` 的符号性 |

并非所有违反规则的程序都保证收到编译错误，例如某些跨翻译单元 ODR 问题不要求诊断。未定义行为也不等于“一定崩溃”：优化后的结果可能难以用逐行执行直觉解释。

良构程序仍可能在执行时触发未定义行为，这些分类不能理解为互斥的“安全等级”。

### 1.5 预处理与保留名字

预处理在正常的 C++ 类型分析之前执行。`#include` 引入头文件内容，`#define` 定义宏，`#if` / `#ifdef` 用于条件包含。宏没有普通函数的类型和作用域语义。

**片段，仅展示预处理规则：**

```cpp
#ifndef GUIDE_EXAMPLE_HPP
#define GUIDE_EXAMPLE_HPP

// 对调用者可见的声明

#endif

// #define SQUARE(x) ((x) * (x))
// SQUARE(++i) 会把参数展开两次，不能用它猜测合法执行结果。
```

宏中加括号能修正某些分组问题，但不能避免实参重复求值。简单计算优先用函数或模板。预处理条件不能直接检查 `sizeof` 或 C++ 类成员；需要运行期判断时使用普通语言表达式。

含双下划线、或以下划线加大写字母开头的名字保留给实现；以下划线开头的名字在全局命名空间还有保留规则。自己的头文件保护宏和公开接口使用普通、独特的名称，不模仿编译器内部名字。多文件包含与条件编译的工程安排见工程化指南。

<a id="types"></a>
## 2. 类型与初始化

### 2.1 基本类型与大小

| 类型 | 用途 | 注意事项 |
| --- | --- | --- |
| `bool` | 逻辑值 | `true` / `false` |
| `char`、`signed char`、`unsigned char` | 字符单元或小整数 | 三者是不同类型；普通 `char` 的符号性由实现决定 |
| `short`、`int`、`long` | 有符号整数 | 不假定具体位宽；`int` 至少 16 位，`long` 至少 32 位 |
| 对应的 `unsigned` 类型 | 无符号整数 | 不能表示负数，运算按其范围取模 |
| `float`、`double`、`long double` | 浮点数 | 存在舍入误差 |
| `wchar_t` | 宽字符类型 | 位宽和编码由实现决定 |
| 指针、引用、数组、函数类型 | 组织访问与调用关系 | 各自有独立规则 |

`sizeof(T)` 返回字节数，且 `sizeof(char) == 1`。字节的位数由 `<climits>` 的 `CHAR_BIT` 给出，不能无条件假定为 8。精确宽度整数、大小和指针差值类型见 [标准库整数类型](C++StandardLibraryGuide.md#integers)。

### 2.2 初始化与赋值

**片段，放在函数体内：**

```cpp
int count = 0;
double price = 12.5;
bool ready = false;
int scores[3] = {80, 90, 100}; // 传统数组聚合初始化
count = 3;                    // 修改已有对象

typedef unsigned long Counter;
Counter requests = 0;
```

初始化建立对象的初始状态；赋值修改已存在的对象。函数内的 `int value;` 没有初始化数值，读取该未初始化整数会造成未定义行为。不要以为局部标量会自动为零。

`typedef` 为已有类型起别名，不产生独立的新类型。运行期长度的局部数组不属于传统标准 C++，动态序列使用容器等方式管理。

<a id="initialization"></a>
### 2.3 初始化形式与声明歧义

| 写法 | 含义 | 关键区别 |
| --- | --- | --- |
| `int n;` | 自动存储期局部整数没有初始数值 | 不得直接读取 |
| `int n = 5;` | 复制初始化 | 语法中的 `=` 不等于赋值操作 |
| `int n(5);` | 直接初始化 | 在创建对象时初始化，不是对已有对象赋值 |
| `int n = int();` | 标量空初始化表达式得到 0 | 不将局部无初始化与零初始化混淆 |
| `int a[3] = {1};` | 数组聚合初始化 | 其余两个元素为 0 |
| `Widget object;` | 定义对象并进行相应初始化 | 类需要可用的默认构造 |
| `Widget object();` | 声明返回 Widget 的无参数函数 | 不是创建一个对象 |

最后一种常被称为声明歧义或 most vexing parse 的相关情形。遇到可被解释为声明的写法，应先检查它究竟声明了什么。

C++03 对值初始化作了调整，历史缺陷修正也会影响旧工具链的解释。不要以为任意类对象的空初始化都会把其所有成员清零；用户编写的构造函数仍应明确初始化基本类型成员。本文的资源类都用成员初始化列表或明确赋值建立状态。相关差异可查 [CWG 178：值初始化](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2012/n3368.html#178)。

### 2.4 不完整类型、数组与字符串字面量

类前置声明后可以声明指向该类型的指针或引用；定义该类型的值对象、访问成员、计算 `sizeof` 等需要完整类型的相应操作，必须等到完整定义可见。

**片段，放在命名空间作用域：**

```cpp
class Widget;
Widget* createWidget(); // 此处不需要 Widget 的完整定义

// Widget object;      // 错误：类型尚不完整
// int size = sizeof(Widget);
```

内建数组的元素数量属于类型的一部分，数组本身不能用普通赋值整体复制。字符串字面量具有静态存储期，末尾含空字符。

**片段，放在函数体内：**

```cpp
const char* literal = "abc"; // 借用字面量，不能修改字面量
char editable[] = "abc";    // 独立数组，含 4 个 char
editable[0] = 'A';
```

旧版本虽允许某些已弃用的字面量到 `char*` 转换，修改字符串字面量仍是未定义行为。接口应使用正确的只读类型。

### 2.5 `const`、枚举与转换

**片段：**

```cpp
const int bufferSize = 64;
int buffer[bufferSize]; // 满足整型常量表达式要求

enum Status { statusIdle, statusRunning, statusFinished };
Status state = statusRunning;

int truncated = static_cast<int>(3.14); // 得到 3
```

`const` 限制通过相应访问路径修改对象，不保证其他别名不能改变底层对象。运行期输入初始化的 `const int` 不一定能充当编译期数组长度。

普通枚举的枚举项进入外围作用域，枚举值可以隐式转换为整数。

| 转换运算符 | 主要用途 | 注意事项 |
| --- | --- | --- |
| `static_cast` | 数值和符合规则的类型转换 | 不自动校验范围 |
| `dynamic_cast` | 多态类型的运行期检查 | 转换失败有明确结果，见第 8 节 |
| `const_cast` | 改变类型中的限定 | 修改原本就是 `const` 的对象会造成未定义行为 |
| `reinterpret_cast` | 底层重新解释 | 不自动满足对齐、别名和生命周期规则 |

数值转换还需区分：整数转无符号按模规则得到可表示值；不能表示的整数转有符号类型，在此旧版本范围中结果由实现定义；浮点转整数会截断小数部分，截断后超出目标范围是未定义行为。强制转换不等于检查成功。

### 2.6 字面量、字符与联合体

| 写法 | 解释 | 注意事项 |
| --- | --- | --- |
| `42`、`42u`、`42L` | 十进制整数、无符号后缀、long 后缀 | 无后缀类型还取决于值能否表示，不假定所有字面量都为 int |
| `010`、`0x10` | 八进制 8、十六进制 16 | 前导 0 不是普通十进制装饰 |
| `1.5`、`1.5f`、`1.5L` | double、float、long double 字面量 | 不保证十进制小数精确表示 |
| `'A'`、`'\n'`、`'\0'` | 单字符或转义字符 | `'\0'` 是空字符，和文本字符 `'0'` 不同 |
| `"abc"` | 含终止空字符的字符串字面量 | 与字符字面量不是同一类型 |

传统版本没有二进制字面量或数字分隔符等新增语法。源字符集和执行字符集的对应由工具链规则决定，不能把字符编码假设写进通用语言规则。

`union` 让多个成员共享存储，通常在某一时刻只能按规则使用当前有效的成员。它不自动记录“现在是哪一种类型”，需要接口另行维护状态。

**片段，放在命名空间作用域：**

```cpp
union Number {
    int integer;
    double real;
};
// 在函数体内：
// Number number;
// number.integer = 7;
// 不据此把 number.real 当作一个可移植的类型转换结果读取。
```

旧版 union 的成员还有构造、析构、引用等限制，不能直接把有非平凡构造或析构的标准字符串塞进这个示例。不要用读取另一个成员替代数值转换；对象表示、别名和外部格式见系统编程指南。

<a id="aggregate-initialization"></a>
### 2.7 聚合类初始化

在本文件的传统范围中，聚合类需满足相应条件：没有用户声明的构造函数，没有私有或保护的非静态数据成员，没有基类，也没有虚函数。普通成员函数和静态成员不单独排除聚合资格。

**片段，类型定义放在命名空间作用域，初始化放在函数体内：**

```cpp
struct Dimensions {
    int width;
    int height;
};

// 在函数体内：
// Dimensions square = {3, 3};
// Dimensions partial = {3}; // width 为 3，height 为 0
// Dimensions empty = {};    // 两个 int 成员都为 0
```

聚合初始化按成员声明顺序提供初值，未列出的成员按相应规则初始化；这里的整数成员得到 0。对非聚合类，构造函数建立状态，不能把这种简单的 `{...}` 写法无条件推广。

这属于传统聚合初始化，与现代版本对一般类的列表初始化、窄化检查及聚合条件变化要分别学习。也不要把 `Dimensions object;` 误认为上面的空初始化列表：函数内未初始化的整数成员仍没有可读初值。

<a id="sizeof"></a>
### 2.8 `sizeof` 与不求值的操作数

`sizeof` 查询对象表示所需的字节数，不执行其表达式操作数。对数组表达式使用它时，不先退化为首元素指针；对引用表达式使用它时，查询被引用类型的大小，不是某种“引用存储大小”。

**片段，需要 `<cstddef>`，放在函数体内：**

```cpp
int values[3] = {1, 2, 3};
int (&arrayReference)[3] = values;
int& firstReference = values[0];
std::size_t count = sizeof(values) / sizeof(values[0]); // 3
std::size_t arrayBytes = sizeof(arrayReference);       // 同 sizeof(values)
std::size_t elementBytes = sizeof(firstReference);     // 同 sizeof(int)

int number = 0;
std::size_t integerBytes = sizeof(++number); // number 仍为 0
```

不执行操作数不意味着忽略语法、名字或类型错误。标准 `sizeof` 不能用于 `void`、函数类型或需要完整定义却尚未完整的类型。

内建固定数组的界必须是合法的正整型常量表达式，`int values[0];` 不是标准写法；第 5.4 节允许的 `new T[0]` 是不同规则。结构体大小还可能包含填充，不能只把成员大小相加。

<a id="control"></a>
## 3. 表达式与流程控制

### 3.1 运算与数值边界

- `7 / 2` 得到整数 `3`，`7 / 2.0` 得到浮点结果 `3.5`。
- 整数除零、有符号算术溢出属于未定义行为。
- 小整数可能先提升为 `int`；表达式结果类型不一定等于变量类型。
- 有符号与无符号混用可能产生意外转换，应在比较和运算前确认范围。
- 内建 `&&`、`||` 有短路行为，可先检查条件再访问对象。
- `=` 是赋值，`==` 是比较；复杂表达式应拆开，避免依赖未规定的求值顺序。
- 浮点近似比较应按量纲和误差要求选择阈值。

**片段：比较之前发生类型转换。**

```cpp
int negative = -1;
unsigned int positive = 1u;
bool result = negative < positive; // false：negative 先转换为 unsigned int
```

这里不是“负数在数学上大于正数”，而是参与比较的值已改变。先确认输入非负且能由目标类型表示，再转成大小或无符号字段；仅添加强制转换会隐藏诊断，不等于修正范围问题。

在 C++98/03 中，负整数参与除法时不能把商的舍入方向无条件假定为向零截断；后续版本的规则有变化。需要跨旧工具链一致的行为时，明确定义运算契约并验证输入和边界。

<a id="evaluation-order"></a>
### 3.2 优先级不决定求值顺序

优先级和结合性决定表达式如何分组；求值顺序决定各部分何时执行。括号明确分组，不自动把独立子表达式变成按从左到右执行。

**片段，仅说明错误模式：**

```cpp
// int i = 0;
// consume(i++, i++); // 此基础版本中存在未定义行为
// 两次修改之间没有所需的顺序点；拆成独立语句
```

普通调用 `consume(first(), second())` 的实参先后通常未指定，但每个函数内部状态访问仍须合法。逗号运算符与参数列表中的分隔逗号不是同一回事。

内建逻辑运算的短路可以写成 `pointer != 0 && pointer->valid()`；若对应运算符被用户重载，不应继续假定同样的短路语义。

### 3.3 分支和循环

**完整程序：**

```cpp
#include <cstddef>
#include <iostream>

int main() {
    const int values[] = {3, -1, 4, 0, 5};
    const std::size_t count = sizeof(values) / sizeof(values[0]);
    int sum = 0;
    for (std::size_t i = 0; i < count; ++i) {
        if (values[i] <= 0) {
            continue;
        }
        sum += values[i];
    }
    if (sum > 10) {
        std::cout << "large sum: " << sum << '\n';
    } else {
        std::cout << "sum: " << sum << '\n';
    }
    return 0;
}
```

输出 `large sum: 12`。`continue` 跳过本次迭代剩余部分；`break` 退出最近一层循环或 `switch`。`switch` 的分支通常需要 `break`，有意贯穿时应说明。

无符号倒序下标尤其要注意：`i >= 0` 永远成立，空序列中 `n - 1` 转成无符号下标后可能得到很大的值。较窄整数还可能先提升再转换，不能只按变量表面的类型推断。遍历容器通常用迭代器更清楚。

`while` 先判断条件，`do ... while` 至少执行一次。条件中涉及读取时，应使用操作本身的成功结果推进循环，避免先判断旧状态再使用失败结果。

### 3.4 左值、右值与赋值对象

在传统分类中，左值表示可用于标识对象或函数的表达式；右值常来自计算结果或临时值。左值不等于“总能写入”：`const int` 对象表达式是左值，但不能赋值。

**片段：**

```cpp
int number = 1;
const int fixed = 2;
number = fixed + 3;
// fixed = 4;        // 错误：不是可修改左值
// (number + 1) = 5; // 错误：不能给这个内建计算结果赋值
```

后续版本细分值类别并引入右值引用，详见现代特性指南；不要把这些新增规则直接套到本文件的传统范围。

### 3.5 位运算与条件表达式

`&`、`|`、`^`、`~` 是按位与、或、异或、取反；`&&`、`||`、`!` 则处理逻辑条件。不能因为符号相似就替换它们。

**片段，放在函数体内：**

```cpp
unsigned int flags = 0;
const unsigned int readable = 1u << 0;
const unsigned int writable = 1u << 1;
flags |= readable | writable;
bool canRead = (flags & readable) != 0;
flags &= ~readable;
```

掩码优先使用合适的无符号类型，并确认位移次数在提升后操作数的合法位宽内。对有符号位移，尤其负数，还要遵守相应旧版规则，不能当成通用乘除法。

条件运算符 `condition ? first : second` 只求值被选择的分支，但两个分支的类型共同决定表达式的类型及可能的转换。不要以为结果类型永远与运行时选中分支原来的类型相同。

<a id="functions"></a>
## 4. 函数与参数

函数把输入、运算和结果组成一个可重复调用的接口。先说明参数允许的值、是否修改调用者对象、返回值的意义和失败方式，再选择传递形式。

### 4.1 声明、定义与调用

**片段，放在命名空间作用域：**

```cpp
int add(int left, int right); // 声明，供调用者了解接口
int add(int left, int right) { // 定义，提供函数体
    return left + right;
}
```

声明可以省略参数名，如 `int add(int, int);`，但有意义的名字能帮助读者理解接口。重复声明必须一致；普通非 `inline` 函数在整个程序中只能有一个定义，跨文件规则见工程化指南。

调用前需要可见的声明。形参与实参的区别是：形参属于函数接口，实参是一次调用提供的表达式。形参是函数内的局部对象或引用，名字只在对应作用域有效。

`void` 表示函数不返回值，提前结束可写 `return;`。C++ 中的 `void f();` 声明无参数函数。有返回值的函数应保证每条正常结束路径都返回有效结果；`main()` 到达结尾等效于返回 `0`，不要将此特例套到其他函数。

`inline` 的重要语言作用是允许符合规则的相同定义出现在多个翻译单元中，并不保证编译器把调用展开。不要为了“更快”给所有函数加 `inline`。

### 4.2 按值、按引用和按指针传递

| 形式 | 常见用途 | 对调用者的影响 |
| --- | --- | --- |
| `int value` | 读取小型值类型 | 修改形参不修改实参 |
| `const T& value` | 借用较大对象供读取 | 限制通过该引用直接修改对象，不提供深度只读保证 |
| `T& value` | 修改已有对象 | 修改反映到调用者，不能合法绑定到不存在的对象 |
| `T* value` | 可为空的借用或连续数据接口 | 指针本身按值复制，所指对象仍可能被修改 |
| `const T* value` | 可为空的只读借用 | 需要约定空值、长度和有效期 |

按值传类对象通常需要复制；按值传指针只复制地址，不复制所指对象。`const T&` 可以绑定临时对象，但函数不能据此把该引用保存到任意更长的生命周期中。

**片段，放在命名空间作用域；递增操作要求输入值小于 `int` 的最大值：**

```cpp
void incrementCopy(int value) { ++value; }
void incrementOriginal(int& value) { ++value; }
void incrementPointed(int* value) {
    if (value != 0) { ++*value; }
}
```

`incrementCopy(x)` 保持 `x` 不变；后两种形式可以修改 `x`。需要改变调用者的指针指向时，可使用 `T*&`，同时说明新地址的所有权；普通 `T*` 参数的赋值只改变局部指针。

<a id="overloads"></a>
### 4.3 重载解析与 `const`

同名函数可以用不同参数类型或参数个数区分。普通函数**不能仅凭返回类型不同重载**；默认参数也不会产生新的函数类型。

编译器先找到候选函数，再检查参数数量和转换是否可行，最后比较可行函数。对常见内建转换，可先理解“精确匹配优于提升，提升优于一般转换”；涉及多个参数、引用或用户自定义转换时，还要按完整规则比较。

内层同名声明可以隐藏外层整组重载，不能以为所有同名函数都会自动参加选择。未限定函数调用还可能通过实参依赖查找（ADL）找到实参类型的关联命名空间中的函数，完整例子见第 4.9 节。

**片段，放在命名空间作用域：**

```cpp
void choose(int);
void choose(double);

void modify(int&);       // 可修改对象
void modify(const int&); // 只读访问，也能接收常量和临时值

void inspect(int);
void inspect(const int); // 同一个函数的再次声明，不是新重载

void read(int*);
void read(const int*);   // 不同重载：所指类型的 const 保留
```

`choose(1)` 精确匹配 `int`；`choose(1.0)` 精确匹配 `double`；`choose(1.0f)` 优先采用 `float` 到 `double` 的提升。`choose(1L)` 在这组声明下有歧义，因为两条路径都是一般转换。

对普通非 `const` 的 `int` 左值，`modify` 优先选择 `int&`；对 `const int` 对象或整数临时值，只能选择 `const int&`。不要混合提供 `f(int)` 与 `f(const int&)` 来表达读写差异，常见调用会发生歧义。

按值形参的**顶层 `const`** 不参与区分函数类型，但在函数定义内部仍限制修改形参。指针所指对象的 `const`、引用所引用对象的 `const` 是另一层限定，不能一概忽略。

### 4.4 默认参数

默认参数通常写在调用者可见的声明中，定义不重复指定。同一作用域中，即使写相同的默认值也不能重复提供它。

**片段，放在命名空间作用域：**

```cpp
int multiply(int value, int factor = 2);
int multiply(int value, int factor) { return value * factor; }

// int multiply(int value, int factor = 2); // 错误：重复默认参数
// void send(int retry = 3, int timeout);   // 错误：后续参数没有默认值
```

`multiply(4)` 等效于使用默认实参调用 `multiply(4, 2)`。一旦某形参有默认值，后续形参也必须有默认值，或已在同一作用域可见的先前声明中获得默认值。

默认表达式在每次缺省该实参的调用时求值。默认值属于调用处可见的声明，不是函数指针类型的一部分；通过函数指针调用时必须提供它的全部参数。

默认表达式中的名字在声明点查找并绑定，表达式的值在相应调用时计算。调用处出现同名局部变量不会重新绑定其中的名字；已经绑定的对象若被修改，则下一次调用可能获得不同的默认值。虚函数的默认实参与动态分派也要分别判断，见第 8.3 节。

默认参数与重载组合时要防止歧义，例如同时声明 `f(int)` 和 `f(int, int = 0)`，调用 `f(1)` 无法确定唯一目标。

### 4.5 数组形参与长度

**片段，需要 `<cstddef>`；以下三个声明表示同一函数：**

```cpp
int sum(const int values[], std::size_t count);
int sum(const int values[10], std::size_t count);
int sum(const int* values, std::size_t count);

void processRows(int values[][3], std::size_t rows);
// 等效形参类型为 int (*)[3]，不是 int**
```

数组形参会调整为指针形参，`[10]` 不验证实际数组长度。在函数内对该形参使用 `sizeof`，得到指针大小，无法恢复原数组元素个数。多维数组只调整最外层，后续维度仍属于所指类型。

指针与长度接口应规定：`count == 0` 时可以允许空指针；`count > 0` 时必须指向至少 `count` 个有效元素；若进行求和，还要保证结果可表示。更好的长度保留方式包括数组引用，例如 `const int (&values)[3]`，或第 9 章的数组引用模板。

### 4.6 函数指针与回调

**片段，放在命名空间作用域：**

```cpp
typedef int (*BinaryOperation)(int, int);

int subtract(int left, int right) { return left - right; }
int invoke(BinaryOperation operation, int left, int right) {
    return operation(left, right); // 契约：operation 不为空
}
```

`int (*p)(int, int)` 是指向函数的指针；括号不可省略，`int* p(int, int)` 声明的是返回 `int*` 的函数。`typedef` 能减少这种声明的阅读成本。

初始化可写 `BinaryOperation p = &subtract;` 或 `BinaryOperation p = subtract;`。调用前确保指针有效；如果名字被重载，目标指针类型用于选择匹配的函数。

普通非静态成员函数需要对象，其指针类型和调用形式不同，不能直接当作上述普通函数指针。带状态的可调用对象见第 9 章。

### 4.7 返回值、生命周期与递归

返回值适合传递独立结果；返回引用或指针表示借用，应明确对象由谁拥有、何时失效。允许调用者修改返回对象时，接口还需要说明可修改的范围。

**片段，放在命名空间作用域：**

```cpp
struct Dimensions { int width; int height; };
Dimensions makeDimensions(int width, int height) {
    Dimensions result = {width, height};
    return result; // 返回值对象，安全
}
// int& broken() { int local = 1; return local; } // 错误：返回悬空引用
```

返回局部值对象是正常用法。C++98/03 允许符合条件的复制消除，但不保证发生；可访问的复制构造函数等要求仍须满足。禁止返回指向已销毁局部对象的指针或引用，也不能返回绑定局部临时对象的引用。

递归要同时说明**基例**、输入如何向基例推进、可接受的最大深度和数值范围。语言不保证尾调用优化；即使数学上有终点，大规模输入也可能耗尽调用栈。

### 4.8 完整程序：传参、回调与有界递归

本例只处理固定的小整数。`sumTo` 的契约是 `n <= 100`，`apply` 要求函数指针不为空；所有算术结果均在传统 `int` 的最小保证范围内。

```cpp
#include <iostream>

typedef int (*BinaryOperation)(int, int);

int add(int left, int right) { return left + right; }
int multiply(int value, int factor = 2) { return value * factor; }

void incrementCopy(int value) { ++value; }
void incrementOriginal(int& value) { ++value; }

int apply(BinaryOperation operation, int left, int right) {
    return operation(left, right);
}

int sumTo(unsigned int n) {
    if (n == 0) { return 0; } // 基例
    return static_cast<int>(n) + sumTo(n - 1);
}

int main() {
    int number = 4;
    incrementCopy(number);
    std::cout << number << '\n';
    incrementOriginal(number);
    std::cout << number << '\n';
    std::cout << multiply(number) << '\n';
    BinaryOperation operation = &add;
    std::cout << apply(operation, 3, 7) << '\n';
    std::cout << sumTo(5) << '\n';
    return 0;
}
```

输出依次为 `4`、`5`、`10`、`10`、`15`，每项占一行。函数声明与默认实参的传统规则可查 [WG21 1997 公开审阅稿声明章节](https://www.open-std.org/jtc1/sc22/open/n2356/decl.html)，重载比较见 [同稿重载章节](https://www.open-std.org/jtc1/sc22/open/n2356/over.html)。

<a id="adl-defaults"></a>
### 4.9 完整程序：ADL 与默认表达式求值

**完整程序，C++98/03：**

```cpp
#include <iostream>

namespace geometry {
struct Point {
    int x;
    int y;
    Point(int first, int second) : x(first), y(second) {}
};
void show(const Point& point) {
    std::cout << "point: " << point.x << ' ' << point.y << '\n';
}
}

namespace configuration { int factor = 2; }

int scale(int value, int factor = configuration::factor) {
    return value * factor;
}

int main() {
    geometry::Point point(3, 4);
    show(point); // ADL 找到 geometry::show，不需要 using 指令
    std::cout << scale(3) << '\n';
    configuration::factor = 4;
    std::cout << scale(3) << '\n';
    std::cout << scale(3, 5) << '\n';
    return 0;
}
```

输出依次是 `point: 3 4`、`6`、`12`、`15`。`scale` 的默认表达式已绑定到 `configuration::factor`，每次省略实参时读取其当时的值。本例只使用小整数；接口仍要求乘法结果可表示。

ADL 不是在全程序中任意搜索同名函数，关联类型、普通名字查找的结果及调用形式都有规则。相关机制见 [WG21 实参依赖查找条款](https://www.open-std.org/jtc1/sc22/open/n2356/basic.html#basic.lookup.koenig)。

<a id="lifetime"></a>
## 5. 指针、引用与生命周期

### 5.1 指针和引用的基本关系

**完整程序：**

```cpp
#include <iostream>

int main() {
    int number = 10;
    int& alias = number;
    int* pointer = &number;
    int* empty = 0;
    ++alias;
    if (pointer != 0) {
        std::cout << *pointer << '\n'; // 11
    }
    if (empty == 0) {
        std::cout << "no object\n";
    }
    return 0;
}
```

`&number` 取地址，`*pointer` 解引用，`pointer->member` 访问成员。指针可以改指向；整数常量 `0` 可作为空指针常量。

引用必须初始化，之后不能重新绑定。给引用赋值是在修改被引用对象；引用和非空指针都可能悬空。

**片段：**

```cpp
int value = 1;
const int* readOnly = &value; // 指向可以改变，不能通过它修改 value
int* const fixed = &value;   // 指向不可改变，可以修改 value
const int* const both = &value;
```

自动存储期对象通常随局部作用域结束而销毁；静态存储期对象具有不同的创建和销毁规则；动态存储期对象由对应所有者释放。口语中的“栈”和“堆”不能替代生命周期分析。

临时对象直接绑定到局部 `const` 引用时，在适用规则下可延长其生命周期；任意函数返回引用不会自动产生这种保证。

保存句柄时检查三个问题：对象由谁拥有、何时销毁、哪些操作使访问失效。

<a id="array-decay"></a>
### 5.2 数组退化与指针运算

数组表达式在许多场合会转换为指向首元素的指针，但数组和指针不是同一种类型。在 `sizeof` 或取数组地址等情形中，不能照搬退化后的解释。

**完整程序：首元素指针、数组指针与长度。**

```cpp
#include <cstddef>
#include <iostream>

int main() {
    int values[3] = {10, 20, 30};
    int* begin = values;
    int* end = values + 3;
    int (*wholeArray)[3] = &values;

    for (const int* current = begin; current != end; ++current) {
        std::cout << *current << ' ';
    }
    const std::ptrdiff_t length = end - begin;
    std::cout << "\nlength=" << length << '\n';
    std::cout << "middle=" << (*wholeArray)[1] << '\n';
    return 0;
}
```

输出元素 `10 20 30`，随后输出 `length=3` 和 `middle=20`。`int*` 指向一个整数，`int (*)[3]` 指向一个含三个整数的数组。相应指针加一时，移动的单位由所指类型决定。

指针算术必须在同一数组允许的范围内进行；可以形成尾后指针，不能解引用它。两个指针相减还要求结果可由 `std::ptrdiff_t` 表示。把指针先移到数组范围之外，即使暂时不解引用，也可能违反规则。

无关对象的指针不能用相减来测量“距离”。内建关系比较也不能随意用来定义这些地址的可移植排序；需要对应功能时查标准库指针比较器的契约。

### 5.3 指针声明、间接层级与 `void*`

**片段：**

```cpp
int number = 7;
int* pointer = &number;
int** pointerToPointer = &pointer;
void* opaque = pointer;
int* restored = static_cast<int*>(opaque);

int* firstPointer, ordinaryInteger; // 只有 firstPointer 是指针
```

`void*` 可用于保存对象指针，不能直接解引用，也没有标准的 `void*` 加减运算。转回类型需要保证原对象、对齐和生命周期满足要求；它不是自动的类型安全容器。

函数指针与对象指针有不同规则，不假定它们可以通过 `void*` 通用转换。指向成员的指针还需要配合具体对象使用，见类与对象章节。

多层指针的限定转换不能照搬单层规则。例如 `int*` 可以转换为 `const int*`，但 `int**` 不能隐式转换为 `const int**`，否则可能间接破坏只读对象的保护。

为避免 `int* p, q;` 造成误读，复杂声明优先拆开，或使用清楚的类型别名。

### 5.4 动态对象与配对释放

| 操作 | 作用 | 对应释放 |
| --- | --- | --- |
| `new T(arguments)` | 分配存储并构造单个对象 | `delete pointer` |
| `new T[count]` | 分配存储并构造数组元素 | `delete[] pointer` |
| 普通局部对象 | 由作用域管理 | 自动析构，不手动 `delete` |

普通 `new` 取得存储失败会抛出 `std::bad_alloc`；`new (std::nothrow)` 的分配失败返回空指针，但构造函数自身仍可能抛异常。使用后者需要 `<new>`。

对标量，`new int` 不提供初始数值，`new int()` 初始化为 0。`new T[0]` 可以得到随后交给匹配 `delete[]` 的指针，但数组没有元素，不能据此解引用。

删除空指针是允许的；删除已经释放的地址、错误来源的地址或配对错误的数组指针则不合法。单纯把某个别名设成 `0` 不会修复其他别名的悬空问题。

手动内存管理应集中在拥有者内部，正常返回、提前返回与异常路径都由同一所有权协议负责。placement new、原始存储和别名规则详见底层/系统编程指南。

### 5.5 临时对象与借用接口

**片段，需要 `<string>`，函数放在命名空间作用域：**

```cpp
const std::string& echo(const std::string& text) {
    return text;
}

// 在函数体内：
// const std::string& bad = echo(std::string("temporary"));
// 参数临时对象在该完整表达式结束时销毁，返回引用不能重新延长它
```

`const T&` 参数可以接收临时对象，因此函数若保存或返回参数引用，要明确对象的来源与使用期限。长期保存数据通常需要自己的值对象，而不是依赖调用者的临时值。

### 5.6 静态对象的初始化和销毁

需要运行期构造的函数局部 `static` 对象，通常在控制流首次经过相应定义时初始化，名字仍局限在局部作用域。若初始化抛异常，下次进入时再次尝试；递归进入正在初始化的同一声明是未定义行为。只有已经完成构造的对象最终执行相应析构。

传统标准范围不提供后续版本的线程安全初始化保证，线程相关设计移至并发指南。

不同翻译单元中的动态初始化依赖可能造成初始化顺序问题，销毁时也要检查相反方向的依赖。避免让全局对象构造或析构隐式依赖另一个尚未初始化或已经销毁的对象。

<a id="classes"></a>
## 6. 类与对象

### 6.1 接口、成员与不变式

`struct` 和 `class` 都能有数据成员、成员函数、构造函数和继承关系。两者的主要语言差别是默认访问权限：`struct` 的成员和继承默认 `public`，`class` 默认 `private`。

| 访问级别 | 谁能直接访问成员？ |
| --- | --- |
| `public` | 有相应对象或限定名的调用者 |
| `protected` | 本类、友元，以及满足访问规则的派生类代码 |
| `private` | 本类和友元；派生类不能直接访问 |

封装把有效状态的维护集中到接口中。**不变式**是对象完成构造后、每次公开操作完成时都应满足的条件；访问权限本身不会自动证明这些条件。

**完整程序：账户使用整数单位，余额保持在 `0` 到 `int` 最大值之间。**

```cpp
#include <iostream>
#include <limits>
#include <stdexcept>

class Account {
public:
    explicit Account(int initialBalance = 0) : balance_(initialBalance) {
        if (initialBalance < 0) {
            throw std::invalid_argument("negative initial balance");
        }
    }
    void deposit(int amount) {
        if (amount <= 0) {
            throw std::invalid_argument("deposit must be positive");
        }
        if (amount > std::numeric_limits<int>::max() - balance_) {
            throw std::overflow_error("balance overflow");
        }
        balance_ += amount;
    }
    int balance() const { return balance_; }

private:
    int balance_;
};

int main() {
    try {
        Account account(100);
        account.deposit(50);
        std::cout << "balance: " << account.balance() << '\n';
        try {
            account.deposit(-1);
        } catch (const std::invalid_argument& error) {
            std::cout << "rejected: " << error.what() << '\n';
        }
        std::cout << "balance after rejection: " << account.balance() << '\n';
    } catch (const std::exception& error) {
        std::cerr << "failure: " << error.what() << '\n';
        return 1;
    }
    return 0;
}
```

正常输出依次是 `balance: 150`、`rejected: deposit must be positive`、`balance after rejection: 150`。数值校验在修改前完成，参数错误或溢出异常都保持余额不变。先做减法检查避免了“先溢出、再检测”的未定义行为；它成立的前提由余额不变式保证。

### 6.2 构造、析构与初始化顺序

构造函数建立初始状态，析构函数结束对象生命周期并清理其资源；它们都没有返回类型。类成员若需要初始化，不应先让其处于无效状态，再在构造函数体里补救。

成员初始化列表在函数体之前执行。初始化次序固定为：

1. 最终派生类负责的虚基类，按继承图的深度优先、从左到右次序。
2. 直接基类，按继承列表中的声明顺序。
3. 非静态数据成员，按类中的声明顺序。
4. 构造函数体。

析构时先执行析构函数体，再按构造完成的逆序销毁成员和基类。初始化列表的书写顺序不能改变这些规则；应按实际次序书写，减少误读。

**片段，放在命名空间作用域：**

```cpp
class Limits {
public:
    explicit Limits(int first) : first_(first), next_(first_ + 1) {}
private:
    int first_; // 先初始化
    int next_;  // 再读取已经初始化的 first_
};
// 调用前置条件：first < int 的最大值，避免 first_ + 1 溢出。
```

`const` 标量成员（包括整数、指针等）、引用成员，以及不能合法默认构造的类类型成员，需要通过相应初始化列表建立状态。具有合适默认构造函数的 `const` 类类型成员可以省略相应初始化项，例如 `const std::string` 可默认构造。引用成员还要求被引用对象活得足够久。不要认为省略基本类型成员的初始化就会自动归零：常见的局部对象中，没有初始化也没有赋值的整数成员不能直接读取。静态存储期等情形还涉及先前的零初始化规则。

没有用户声明的构造函数时，编译器隐式声明默认构造函数；一旦自行声明任意构造函数，就不再自动补这个无参构造。隐式声明也不保证可用：基类或成员无法合法默认构造时，使用它会导致非良构程序。默认构造函数是**可以无实参调用**的构造函数，`Account(int = 0)` 也属于这一类。[规则核对：原始工作稿的构造与初始化章节](https://www.open-std.org/jtc1/sc22/open/n2356/special.html#class.ctor)。

### 6.3 `explicit`、`this` 与只读接口

`explicit` 阻止可用一个实参调用的构造函数被用于相应的隐式转换；仍允许直接初始化。注意有多个参数、但后续参数有默认值的构造函数也可能只需一个实参。

**片段，使用上面的 `Account`，放在函数体内：**

```cpp
Account empty;      // 默认实参使初始余额为 0
Account funded(5);  // 直接初始化
// Account wrong = 5; // 编译错误：explicit 阻止这种隐式转换
```

非静态成员函数中，`this` 是指向当前对象的指针；`return *this;` 可返回当前对象的引用。静态成员函数没有 `this`，也不能声明为 `const` 成员函数。

`int balance() const` 中末尾的 `const` 限制通过当前对象修改普通成员，并允许在 `const Account` 上调用该接口。它不会自动使指针类型数据成员所指向的独立对象只读。成员函数可以按这个 `const` 限定重载，常用于提供读写两种访问接口。

`mutable` 允许特定非静态成员在只读成员函数中修改，适合不改变逻辑值的辅助状态；不要用它绕过接口的状态承诺。

**片段，放在命名空间作用域：读取计数按无符号规则回绕。**

```cpp
class ObservedValue {
public:
    explicit ObservedValue(int value) : value_(value), reads_(0) {}
    int value() const { ++reads_; return value_; }
private:
    int value_;
    mutable unsigned long reads_;
};
```

### 6.4 静态成员与类外定义

非静态数据成员各对象各有一份；静态数据成员属于类，所有对象共享同一实体，也可以在没有该类对象时使用。访问权限同样适用，公共静态成员通常写成 `ClassName::member`。

**片段，放在命名空间作用域；类外定义只放在一个实现文件：**

```cpp
class Config {
public:
    static int count;
    static const int capacity = 16;
    static void reset() { count = 0; }
};

int Config::count = 0;      // 定义共享存储，此处不再写 static
const int Config::capacity; // 不重复类内初始化式

int slots[Config::capacity];
const int& capacityRef = Config::capacity; // 绑定到实际对象
```

类内的 `static int count;` 只是声明。传统版本允许 `static const` 整型或枚举成员在类内用整型常量表达式初始化；需要实体存储的使用，例如取地址或绑定引用，仍须类外定义。这常称为 ODR 使用。示例始终提供定义，避免依赖是否只作为常量表达式使用的细节。漏掉所需定义通常表现为链接错误，也可能属于不要求诊断的 ODR 违规。[规则核对：静态数据成员](https://www.open-std.org/jtc1/sc22/open/n2356/class.html#class.static.data)。

### 6.5 友元与组合

`friend` 把访问本类非公开成员的权限授予指定函数或类。在本类中直接声明的普通友元函数不是本类的成员；也可以将另一个类的已有成员函数授为友元。友元关系不自动互相授予、不传递，也不由派生类继承。仅在确实需要访问内部表示的协作接口上授权，详见 [传统友元规则](https://www.open-std.org/jtc1/sc22/open/n2356/access.html#class.friend)。

组合是把另一个类对象作为成员，例如 `class Service { Account account_; };`。成员对象的生命周期跟随所属对象，构造和销毁由语言规则安排。拥有某种组件并不意味着可以替代该组件；为了组织职责和复用实现，组合往往比继承更直接。

### 6.6 运算符重载

普通重载运算符是函数调用的另一种写法，至少一个操作数应是类或枚举类型；分配和释放函数 `operator new`、`operator delete` 另有规则。不能创造新运算符，不能改变优先级、结合性和所需操作数数量；也不能改变纯内建类型之间的运算含义。

| 形式 | 常见选择 | 原因 |
| --- | --- | --- |
| 成员 `operator=`、`operator[]`、`operator()`、`operator->` | 必须作为非静态成员实现 | 语言要求 |
| 成员前置 `operator++()` | 修改自身并返回自身引用 | 对应 `++object` 的惯常语义 |
| 成员后置 `operator++(int)` | 返回修改前的值 | 虚设的 `int` 参数区分 `object++` |
| 非成员 `operator<<` | 输出流在左侧 | 不需要修改标准库的流类 |
| 对称二元运算符 | 常用非成员 | 两侧可按规则参与隐式转换 |

**片段，需要 `<ostream>`，放在命名空间作用域：计数器按模规则递增。**

```cpp
class Counter {
public:
    explicit Counter(unsigned long value = 0) : value_(value) {}
    Counter& operator++() { ++value_; return *this; }
    Counter operator++(int) {
        Counter old(*this);
        ++(*this);
        return old;
    }
    friend std::ostream& operator<<(std::ostream& out, const Counter& value);
private:
    unsigned long value_;
};

std::ostream& operator<<(std::ostream& out, const Counter& value) {
    return out << value.value_;
}
```

返回流引用可继续连接输出。重载 `&&`、`||` 在本文件的传统版本中不会保留内建运算符的短路行为；复杂副作用应拆开。类的复制、赋值和资源关系需要一起设计，见下一章。

<a id="member-pointers"></a>
### 6.7 指向成员的指针

成员指针描述“某类的哪个非静态成员”，使用时还需要对象。它与普通对象指针或函数指针是不同种类，不能假定表示或大小相同。`.*` 配合对象，`->*` 配合对象指针；成员函数指针的调用需要相应括号。

**片段，放在命名空间作用域；`memberExample()` 内展示访问方式：**

```cpp
struct Point {
    int x;
    explicit Point(int initial) : x(initial) {}
    int get() const { return x; }
};
void memberExample() {
    int Point::* data = &Point::x;
    int (Point::* method)() const = &Point::get;
    Point object(3);
    object.*data = 4;
    int value = (object.*method)();       // 4
    Point* pointer = &object;
    int same = (pointer->*method)();      // 4
    (void)value; (void)same;
}
```

这里的 `const` 是成员函数签名的一部分。静态成员不需要对象，其地址按对应的普通指针规则处理。调用非静态成员函数时，对象必须有效且匹配类型；空成员指针也不能直接应用或调用。

### 6.8 转换构造函数与转换函数

构造函数可以把其他类型转换为本类；成员转换函数 `operator T()` 可以把本类对象转换为另一类型。这两个方向独立，构造函数上的 `explicit` 不会自动禁止反向转换。

**片段，类放在命名空间作用域：**

```cpp
class SmallValue {
public:
    explicit SmallValue(int value) : value_(value) {}
    operator int() const { return value_; }
private:
    int value_;
};
// 在函数体内：
// SmallValue value(3);
// int number = value; // 使用 operator int()
// SmallValue other = 3; // 错误：explicit 限制这个方向的隐式转换
```

转换函数不写单独的返回类型，目标类型在 `operator` 后说明，也没有普通函数参数。C++98/03 的转换函数不能使用现代的 `explicit operator ...` 写法。

隐式转换会影响重载选择；多个转换路径可能产生歧义，一条普通隐式转换序列也不能随意串联多个用户定义转换。需要精确控制接口时，命名访问函数往往更清楚。

<a id="copy"></a>
## 7. 复制、赋值与资源

### 7.1 复制构造与复制赋值

复制构造创建对象，复制赋值修改已有对象。默认成员复制会复制裸指针的地址，不自动深复制它所指的资源。

手动拥有资源的类，应一起审视析构函数、复制构造函数和复制赋值运算符，即**三法则**。用标准容器成员表达存储所有权，通常可以减少手写这些操作。

**片段，假定 `Widget` 是可复制的类：**

```cpp
// Widget first;
// Widget second(first); // 复制构造
// Widget third = first; // 仍是初始化，不是赋值
// second = first;       // 复制赋值
```

按值传参和返回值也可能涉及复制，不能只检查显式出现的 `=`。传统版本允许在适用情形下消除复制，但所需复制操作的可访问性等规则仍必须满足。

自行声明析构函数不等于禁止复制：在 C++98/03 中，若没有声明相应复制操作，编译器仍可能隐式声明它们。隐式复制构造或赋值的形参也不一定总是 `const T&`；基类或成员只能从非 `const` 对象复制时，相应操作可能采用 `T&` 形式。隐式声明与实际能合法使用是两件事，仍要检查成员、访问权限和资源语义。

### 7.2 深复制与先复制后交换

**完整程序：深复制与复制后交换。**

```cpp
#include <algorithm>
#include <cstddef>
#include <iostream>
#include <stdexcept>

class Buffer {
public:
    explicit Buffer(std::size_t size) : size_(size), data_(new int[size]) {
        for (std::size_t i = 0; i < size_; ++i) { data_[i] = 0; }
    }
    Buffer(const Buffer& other)
        : size_(other.size_), data_(new int[other.size_]) {
        for (std::size_t i = 0; i < size_; ++i) { data_[i] = other.data_[i]; }
    }
    Buffer& operator=(const Buffer& other) {
        Buffer temporary(other);
        swap(temporary);
        return *this;
    }
    ~Buffer() { delete[] data_; }
    void swap(Buffer& other) {
        std::swap(size_, other.size_);
        std::swap(data_, other.data_);
    }
    int& at(std::size_t index) {
        if (index >= size_) { throw std::out_of_range("buffer index"); }
        return data_[index];
    }
    const int& at(std::size_t index) const {
        if (index >= size_) { throw std::out_of_range("buffer index"); }
        return data_[index];
    }
    std::size_t size() const { return size_; }

private:
    std::size_t size_;
    int* data_;
};

int main() {
    Buffer first(2);
    first.at(0) = 7;
    Buffer second(first);
    second.at(0) = 9;
    std::cout << first.at(0) << ' ' << second.at(0) << '\n'; // 7 9
    first = second;
    std::cout << first.at(0) << '\n'; // 9
    first = first; // 验证自赋值仍保持内容
    const Buffer& readonly = first;
    std::cout << "size=" << readonly.size()
              << ", first=" << readonly.at(0) << '\n';
    try {
        first.at(first.size()); // 尾后位置不是有效元素
    } catch (const std::out_of_range&) {
        std::cout << "out of range rejected\n";
    }
    return 0;
}
```

本例的 `int` 元素复制和指针交换不会抛异常；若临时分配失败，原对象状态保持不变。推广到复杂元素类型时，必须重新分析构造失败时的释放路径。

同一对象赋给自己也可通过该方式保持正确：先创建独立副本，再交换拥有的存储。`swap` 是否可靠取决于成员类型；不能把这个 `int` 数组示例的强保证无条件推广到任意类型。

完整输出依次为 `7 9`、`9`、`size=2, first=9`、`out of range rejected`。只读对象选择返回 `const int&` 的接口；`at()` 返回的引用借用内部数组。对象销毁或赋值替换存储后，旧数组的引用失效，即使赋的是相同对象也不能继续使用旧引用。

### 7.3 成员复制究竟复制了什么

| 成员 | 默认复制的常见结果 | 设计问题 |
| --- | --- | --- |
| 数值类型 | 复制值 | 检查业务范围和不变式 |
| 值语义的字符串、容器 | 调用成员的复制操作 | 成员内部的共享关系仍由其类型决定 |
| 裸指针 | 复制地址 | 是借用、深复制还是共享？ |
| 引用成员 | 复制构造时仍绑定同一被引用对象 | 有效期由外部对象决定 |
| `const` 或引用成员 | 使用隐式生成的复制赋值会成为非良构 | 禁止赋值，或手写赋值并明确其语义 |

拥有资源与观察资源是不同关系。不要仅因成员是指针，就一律深复制或一律删除；由接口决定责任。

### 7.4 不可复制对象与资源释放

禁止复制的传统做法是在 `private` 中声明复制构造和复制赋值，不提供定义。RAII 将资源交给对象，析构时释放；它早已属于核心设计方法。现代所有权工具另见版本和标准库指南。

**片段，类定义放在命名空间作用域：**

```cpp
class NonCopyable {
public:
    NonCopyable() {}
    ~NonCopyable() {}

private:
    NonCopyable(const NonCopyable&);
    NonCopyable& operator=(const NonCopyable&);
};
```

类内或友元仍可能误用私有声明而产生链接错误，所以此做法也依赖审查。不要把“只声明未定义”当成所有场合都能给出同样编译诊断的机制。

优先让成员对象自行管理资源。裸 `new` 后若其他操作失败，在交给所有者之前就可能泄漏；析构时要只释放自己拥有的资源。

<a id="polymorphism"></a>
## 8. 继承与多态

### 8.1 继承关系与访问权限

公开继承通常表达“派生对象可以作为基类接口使用”的可替代关系；设计时还要满足基类的行为契约。仅为复用实现时，可先考虑组合。

下表描述基类成员在派生类中的可访问级别。基类自己的私有成员始终仍存在于基类子对象中，派生类代码不能因此直接访问它们。

| 基类成员原权限 | `public` 继承 | `protected` 继承 | `private` 继承 |
| --- | --- | --- | --- |
| `public` | `public` | `protected` | `private` |
| `protected` | `protected` | `protected` | `private` |
| `private` | 不能直接访问 | 不能直接访问 | 不能直接访问 |

公开且无歧义的继承允许外部调用者把派生类指针或引用隐式转换为基类指针或引用。保护或私有继承对这种转换施加访问限制。成员本身的访问控制与继承方式要分别判断。

`protected` 也不表示派生类可以通过任意基类对象修改成员：对非静态成员，派生类中的访问还限制对象表达式的类型。不要把保护成员当成对所有继承相关对象开放的全局权限。

### 8.2 虚函数与抽象接口

包含至少一个虚函数的类是多态类。通过基类指针或引用调用虚函数时，根据被引用对象及当前生命周期阶段选择最终覆盖函数。非虚函数则不进行这种动态分派。

纯虚函数用 `= 0` 声明。仍有纯虚最终覆盖函数的类是抽象类，不能创建该类的完整对象，但可以声明其指针和引用。

**完整程序：这个示例的矩形接口约定每一维处于 `0` 到 `1000000`，超出范围即拒绝。**

```cpp
#include <iostream>
#include <stdexcept>
#include <typeinfo>

class Shape {
public:
    virtual ~Shape() {}
    virtual double area() const = 0;
};

class Rectangle : public Shape {
public:
    Rectangle(double width, double height) : width_(width), height_(height) {
        const double maximum = 1000000.0;
        if (!(width >= 0.0 && width <= maximum &&
              height >= 0.0 && height <= maximum)) {
            throw std::invalid_argument("invalid rectangle dimensions");
        }
    }
    virtual double area() const { return width_ * height_; }

private:
    double width_;
    double height_;
};

class EmptyShape : public Shape {
public:
    virtual double area() const { return 0.0; }
};

void printArea(const Shape& shape) { std::cout << shape.area() << '\n'; }

int main() {
    try {
        Rectangle rectangle(3.0, 4.0);
        printArea(rectangle);
        const Shape& base = rectangle;
        std::cout << "is Rectangle: " << (typeid(base) == typeid(Rectangle)) << '\n';
        EmptyShape empty;
        Shape* pointer = &empty;
        std::cout << "cast failed: " << (dynamic_cast<Rectangle*>(pointer) == 0) << '\n';
        try {
            dynamic_cast<Rectangle&>(*pointer);
        } catch (const std::bad_cast&) {
            std::cout << "reference cast rejected\n";
        }
        try {
            Rectangle invalid(-1.0, 2.0);
        } catch (const std::invalid_argument& error) {
            std::cout << "rejected: " << error.what() << '\n';
        }
    } catch (const std::exception& error) {
        std::cerr << "failure: " << error.what() << '\n';
        return 1;
    }
    return 0;
}
```

正常输出依次是 `12`、`is Rectangle: 1`、`cast failed: 1`、`reference cast rejected`、`rejected: invalid rectangle dimensions`。尺寸检查拒绝负数、超出接口上限的值，以及实现具有无穷值或 NaN 时的这些值。尺寸上限使面积处于 `double` 保证支持的数值范围内；乘法仍有浮点舍入，极小结果也可能下溢，这里不承诺精确的实数计算。

<a id="virtual-dispatch"></a>
### 8.3 隐藏、重载与覆盖

| 概念 | 判断方式 | 结果 |
| --- | --- | --- |
| 重载 | 同一可参与查找的名字具有不同参数列表等合法差别 | 编译期选择匹配函数 |
| 隐藏 | 内层或派生类中查找到同名声明 | 外层或基类的同名候选可能不再参与查找 |
| 覆盖 | 派生函数与基类虚函数满足覆盖要求 | 参与虚调用的动态分派 |

覆盖时参数列表和成员函数的 cv 限定需要匹配，返回类型相同或符合协变返回类型的严格规则。普通的返回值类型差别不能随意形成覆盖。基类函数一旦是虚函数，派生类的合法覆盖仍是虚函数；重复写 `virtual` 只是便于阅读。

**片段，放在命名空间作用域：**

```cpp
class Base {
public:
    virtual ~Base() {}
    virtual void show(int) const {}
    void show(const char*) const {}
};
class Derived : public Base {
public:
    using Base::show; // 使基类同名重载参与派生类的名字查找
    virtual void show(int) const {} // 覆盖
    void show(double) const {}     // 新重载，不覆盖 show(int)
};
```

去掉 `using Base::show;` 后，`Derived` 的同名声明会隐藏基类的同名重载集合。`using` 恢复查找候选，不负责把签名不匹配的函数变成覆盖。`void show(int)` 漏掉末尾 `const` 时也不会覆盖 `void show(int) const`；传统标准没有 `override` 关键字，需仔细检查签名。

默认实参取决于调用处可见的声明和静态类型，虚函数实现则按动态分派选择。例如基类虚接口默认值为 1、派生覆盖接口默认值为 2 时，经基类引用省略实参调用，会把 1 传给派生实现；直接经派生接口调用才使用其默认值 2。为避免同一实现接到不同隐含输入，默认值应在接口层清楚统一。

### 8.4 虚析构与对象切片

若接口允许通过基类指针执行普通 `delete` 销毁派生对象，基类应提供可访问的虚析构函数。此基础范围中，对这种派生对象通过无虚析构的基类指针删除会造成未定义行为，不能只解释为“少调用一个析构函数”。

**片段，使用完整程序中的类型，放在函数体内：**

```cpp
Shape* owned = new Rectangle(3.0, 4.0);
delete owned; // 虚析构保证按相应派生类型销毁
owned = 0;
```

这是展示删除规则的最小片段。较复杂代码还必须让所有者负责处理分配后的异常路径，见资源章节。若基类设计为禁止外部通过它删除对象，可用受保护的非虚析构，并在接口中明确销毁方式。

纯虚析构函数也必须提供定义，例如在类外写 `Base::~Base() {}`；派生对象销毁时仍要执行基类析构过程。

**片段，放在命名空间作用域：**

```cpp
struct PlainBase {
    int value;
    PlainBase() : value(0) {}
};
struct PlainDerived : PlainBase {
    int extra;
    PlainDerived() : extra(7) {}
};
void consumeBase(PlainBase value) { /* 只取得独立的基类值 */ }
// 在函数体内调用：PlainDerived object; consumeBase(object);
```

按基类值复制会产生对象切片：新对象只有基类部分，派生类状态不会保留下来。对于具体的多态基类，切片后的动态类型也变成基类；函数参数使用基类引用或指针才能保留原对象。上面的 `Shape` 是抽象类，因此尝试按值创建或接收 `Shape` 对象是编译错误，而不是运行一个切片示例。

### 8.5 构造和析构期间的虚调用

构造期间，派生类部分尚未全部建立；析构期间，其部分已经结束。因此在基类构造函数或析构函数中对当前对象的虚调用，使用该阶段类的最终覆盖函数，不会分派到更派生类的覆盖函数。

不要从构造或析构过程对当前正在构造或析构的对象进行纯虚函数的虚调用，这会造成未定义行为。某些明显写法可能被编译器警告或导致链接失败，但语言结论不能只由是否收到诊断决定。显式限定名调用具有另一套非虚调用规则，不能据此把上述虚调用当成合法。

如果初始化工作必须用到完整派生类行为，可在对象完成构造后通过明确的操作执行，并重新考虑是否可以用组合减少这种阶段依赖。[规则核对：原始工作稿的构造和析构期间规则](https://www.open-std.org/jtc1/sc22/open/n2356/special.html#class.cdtor)。

### 8.6 RTTI：`dynamic_cast` 与 `typeid`

RTTI 指运行期类型信息，这两种运算符已属于传统 C++。需要运行期检查的向下转换或交叉转换，源指针或引用对应的类型必须是多态类；合法的向上转换等情形不要求如此。完整程序中的源基类 `Shape` 因虚函数而满足要求。

| 写法或情形 | 结果或要求 |
| --- | --- |
| `dynamic_cast<Derived*>(base)` | 检查失败返回目标类型的空指针 |
| `dynamic_cast<Derived&>(baseRef)` | 检查失败抛 `std::bad_cast` |
| 合法的指针形式以空指针为源 | 返回目标类型的空指针 |
| `typeid(baseRef)`，引用到多态对象 | 给出被引用对象的动态类型 |
| `typeid(pointer)` | 给出指针自身的静态类型，不检查其目标 |
| `typeid(*nullPointer)`，目标是多态类 | 抛 `std::bad_typeid` |

参与转换的类型完整性、继承的可访问性和歧义也要满足相应规则。源对象必须处于允许的生命周期内；把悬空指针交给 `dynamic_cast` 不会变成安全检查。`static_cast` 的向下转换不进行同样的运行期验证。

使用 `typeid` 及标准异常类型时包含 `<typeinfo>`。用 `typeid(x) == typeid(T)` 比较类型；`type_info::name()` 的字符串由实现决定，不适合作为可移植的类型标识或序列化格式。优先由虚接口表达行为，只在需要识别具体类型的接口边界使用转换。[规则核对：动态转换与类型识别](https://www.open-std.org/jtc1/sc22/open/n2356/expr.html#expr.dynamic.cast)。

### 8.7 多重继承、虚继承与接口组合

多重继承允许一个类有多个直接基类，适合一个对象同时实现几个相互独立的抽象接口。出现同名成员时，要判断名字查找和调用是否有歧义；不能假定编译器会自动选某条继承路径。

菱形继承中，若两条路径都非虚继承同一基类，最终对象通常具有两个该基类子对象。两条路径都使用虚继承时，可以共享一个虚基类子对象，由最终派生类构造函数负责初始化它。

**片段，放在命名空间作用域：只展示共享基类关系。**

```cpp
struct Root { int tag; Root() : tag(0) {} };
struct Left : virtual public Root {};
struct Right : virtual public Root {};
struct Joined : public Left, public Right {};
```

`Joined` 中只有一个共享的 `Root` 子对象。虚继承解决子对象重复，并不自动解决所有成员查找或虚函数最终覆盖的歧义。接口组合通常应让每个抽象接口职责小、析构契约清晰，避免用复杂继承图承载大量共享可变状态。

<a id="templates"></a>
## 9. 模板与泛型

模板描述一族函数或类，让类型和某些编译期值成为参数。实例化会形成具体的函数或类；模板本身不是一个“能自动处理任何类型”的运行期对象。

### 9.1 类型参数与模板契约

**片段，放在命名空间作用域：**

```cpp
template <typename T>
T larger(T left, T right) {
    return left < right ? right : left;
}
```

在类型模板形参位置，`typename T` 与 `class T` 含义相同；`class` 不要求实参必须是类类型，`int` 同样可以。不要与后面的依赖名 `typename` 用法混淆。

此函数要求 `T` 能按值传递和返回，`left < right` 的结果能按条件判断规则使用；当比较结果为假时返回 `left`。不假定所有自定义类型的 `<` 和 `==` 必然一致。泛型接口也要写清比较语义、复制成本和可能抛出的异常。

模板只能在适当的命名空间或类作用域声明，不能在普通函数体中声明；局部类也不能声明成员模板。C++98/03 不能把函数内部定义的局部类作为模板类型实参；需要此类类型时，把定义移到合适的命名空间或类作用域。

### 9.2 函数模板实参推导

调用函数模板时，编译器通常根据函数实参类型推导模板实参；也可以显式给出模板实参。

**片段，放在函数体内，使用上面的 `larger`：**

```cpp
int first = larger(3, 7);            // 推导 T 为 int
double second = larger(2.5, 1.2);    // 推导 T 为 double
double third = larger<double>(1, 2.5); // 先固定 T，再转换函数实参
// double fourth = larger(1, 2.5);   // 错误：同一个 T 推导出冲突类型
```

推导不会为了让两个冲突的 `T` 统一，自动寻找一个“最合适的公共类型”。`larger<double>` 先确定 `T` 为 `double`，此时整数实参才可以按普通调用规则转换。

按值形参推导通常忽略实参类型的顶层 `const`，数组和函数实参通常按相应指针类型参与推导；引用形参可保留数组类型。不要把“函数实参推导”和“类模板实参”混为一谈：传统版本必须写 `Box<int>`，不会根据构造参数自动写成 `Box box(7)`。

只有返回类型出现的参数，通常不能从普通调用的赋值目标推导，例如 `template<class T> T make();` 需要 `make<int>()`。C++98/03 不允许给函数模板形参指定默认模板实参；函数普通形参的默认实参是另一套规则。

### 9.3 完整程序：函数模板和类模板

`Box<T>` 拥有一个 `T` 值；构造时复制，访问时借用。`value()` 返回的引用只在对应 `Box` 对象仍存活且内部值未失效时可用。

```cpp
#include <iostream>

template <typename T>
T larger(T left, T right) {
    return left < right ? right : left;
}

template <class T = int>
class Box {
public:
    explicit Box(const T& value) : value_(value) {}
    const T& value() const { return value_; }

private:
    T value_;
};

int main() {
    Box<int> first(larger(3, 7));
    Box<> second(9); // 使用类模板默认实参 int
    std::cout << first.value() << '\n';
    std::cout << second.value() << '\n';
    std::cout << larger(2.5, 1.2) << '\n';
    std::cout << larger<double>(1, 2.5) << '\n';
    return 0;
}
```

输出依次为 `7`、`9`、`2.5`、`2.5`。即使类模板的全部参数都有默认值，使用模板时仍要写 `<>`。嵌套模板结尾在传统语法中应写 `> >`，例如 `Box<Box<int> >`。

### 9.4 非类型参数与数组引用

非类型参数把合适的编译期值写进类型或算法，例如固定容量。传统版本允许整型、枚举和符合规则的指针、引用、成员指针参数；不允许 `double` 或普通类对象直接作为这种形参的类型。

**片段，放在命名空间作用域，需要 `<cstddef>`：**

```cpp
template <class T, std::size_t N>
std::size_t arrayCount(const T (&values)[N]) {
    (void)values;
    return N;
}

template <class T, int N>
class FixedStorage {
    typedef char PositiveCapacity[(N > 0) ? 1 : -1];
    T elements_[N];
};
```

`arrayCount` 通过引用保留数组类型，推导出元素类型和长度；传入普通指针不能推导出 `N`。`FixedStorage<int, 3>` 和 `FixedStorage<int, 4>` 是不同类型，容量不是运行期变量。

`PositiveCapacity` 利用负数组长度在 `N <= 0` 时产生编译错误，是传统编译期约束的示意；实际容器还需要补齐访问、初始化和范围约定。整型模板实参需要整型常量表达式；不要把运行期输入误当作模板参数。

指针、引用实参在旧标准中还有链接属性和表达形式限制；字符串字面量、局部对象地址不能直接替代合格的非类型实参。这类底层用法应逐条查规则，不应从“它在编译时已知”推断一定合法。

<a id="specialization"></a>
### 9.5 全特化、偏特化与重载

全特化针对完整的一组模板实参提供专门实现；类模板还允许偏特化，针对某类实参模式提供实现。函数模板**不允许偏特化**，常见替代方式是重载函数模板。

**片段，放在命名空间作用域：**

```cpp
template <class T> struct Kind {
    static const int value = 0;
};
template <> struct Kind<bool> { // 类模板全特化
    static const int value = 1;
};
template <class T> struct Kind<T*> { // 类模板偏特化
    static const int value = 2;
};

template <class T> int category(const T&) { return 0; }
template <class T> int category(T*) { return 2; } // 重载，不是偏特化
```

特化声明必须在可能触发相应隐式实例化的使用之前可见；跨翻译单元也要保持一致。类模板特化是独立的类定义，不自动继承主模板的全部成员。

函数模板可以全特化，但它与重载的交互容易误解：先进行重载选择，再使用被选中模板对应的特化。为一类参数改变行为时，优先考虑清楚的重载；不要把函数全特化当成一条独立参与常规重载排名的候选函数。

### 9.6 依赖名、`typename` 与 `template`

当名字的含义依赖模板实参时，编译器在模板定义阶段可能无法确定它是类型、值还是模板。必要时用关键字消除语法歧义。

**片段，放在命名空间作用域：**

```cpp
template <class Container>
typename Container::value_type firstCopy(const Container& values) {
    // 契约：values 非空，且提供 value_type 和 front()
    return values.front();
}

template <class Reader>
int readInteger(Reader& reader) {
    return reader.template read<int>();
}
```

`typename Container::value_type` 告诉编译器这个依赖的限定名表示类型；`reader.template read<int>()` 指出依赖成员名 `read` 是模板，让 `<` 按模板实参列表解析。

这些关键字只解决解析问题，不保证实参类型提供相应成员。依赖基类的成员查找也需要注意，常见写法是 `this->member` 或合适的基类限定名，不能假设所有未限定名字都在实例化时才查找。

### 9.7 定义可见、显式实例化与声明

常用的非导出模板采用包含模型：把模板定义放在头文件，使需要隐式实例化的翻译单元看到定义。只有声明而把通用定义放在 `.cpp` 中，通常不能支持调用方任意类型的实例化。

如果只支持一组固定类型，可以在实现文件中提供定义和**显式实例化定义**，调用方只见到模板声明。

**多文件片段：**

```cpp
// identity.hpp：对调用者可见的模板声明
template <class T> T identity(T value);

// identity.cpp：包含头文件，提供定义及指定实例
#include "identity.hpp"
template <class T> T identity(T value) { return value; }
template int identity<int>(int); // 显式实例化定义

// user.cpp：包含头文件后可调用 identity<int>(3)
// identity<double>(3.0) 未获支持，不能指望自动生成其定义
```

此处模板声明不等于“显式实例化声明”。`extern template` 形式是 C++11 的显式实例化声明，放在现代特性指南讲解。显式实例化只能覆盖已安排的类型；类模板实例化还有成员定义可见性等规则，不能当作自动生成所有成员的万能开关。

### 9.8 传统约束方式与函数对象

旧标准没有 `concepts`。最基础的约束是记录所需操作，让不满足条件的实例化产生诊断；对整型条件可使用上面的负数组长度技巧。需要按类型选择实现时，可以使用特征类、偏特化、重载或标签分派。

SFINAE 表示模板实参替换在适用的函数声明上下文中失败时，可使该候选退出，而非立即报整个程序错误。**函数体内部的任意错误不都属于 SFINAE**；也不能把现代标准扩展后的表达式检测机制直接套到 C++98/03。

**片段，放在命名空间作用域：**

```cpp
class AddOffset {
public:
    explicit AddOffset(int offset) : offset_(offset) {}
    int operator()(int value) const { return value + offset_; }
private:
    int offset_;
};
```

函数对象通过 `operator()` 支持 `AddOffset plusTwo(2); plusTwo(3);` 这样的调用，并能保存状态；本例要求加法结果能由 `int` 表示。模板算法可以接受函数对象或普通函数指针，标准库应用见另卷。

`typename`、传统非类型参数和特化规则可核对 [WG21 1997 公开审阅稿模板章节](https://www.open-std.org/jtc1/sc22/open/n2356/template.html)。该稿存在后来修正的条款；本文仅使用此处说明的传统基础规则。

<a id="type-sfinae"></a>
### 9.9 完整程序：传统类型 SFINAE

**完整程序，C++98/03：** 检查类型是否提供可形成相应指针的 `value_type`，用候选退出演示声明上下文的替换失败。

```cpp
#include <iostream>

struct Record { typedef int value_type; };

template <class T>
int hasValueType(const T&, typename T::value_type* = 0) {
    return 1;
}

int hasValueType(...) { return 0; }

int main() {
    Record record;
    std::cout << hasValueType(record) << ' '
              << hasValueType(3) << '\n';
    return 0;
}
```

输出 `1 0`。`T = Record` 时模板参数类型合法，普通匹配优于省略号；`T = int` 时不存在 `int::value_type`，相应模板候选退出，选择省略号重载。

这里检测的是特定声明是否可形成，不是任意“类型能力判断器”。函数体内的一般错误不会因此消失；传统规则也不能直接替代现代的表达式 SFINAE 或 Concepts。上述机制的边界见 [传统模板实参推导条款](https://www.open-std.org/jtc1/sc22/open/n2356/template.html#temp.deduct)。

<a id="exceptions"></a>
## 10. 异常与接口契约

### 10.1 抛出、匹配与重抛

异常表示不能正常完成操作。通常按 `const` 引用捕获，避免对象切片。

**片段，需要 `<stdexcept>` 与 `<iostream>`，放在函数体内：**

```cpp
try {
    throw std::runtime_error("operation failed");
} catch (const std::exception& error) {
    std::cerr << error.what() << '\n';
}
```

异常展开期间，已完整构造的局部对象按规则析构。RAII 释放资源，但不会自动回滚外部副作用。

若没有匹配的处理器，程序会调用 `std::terminate`，终止前是否进行栈展开在此传统规则中由实现定义。因此资源清理保证应放在正常返回或实际异常展开的条件下理解，不能把进程终止也当作正常清理路径。

异常处理器按书写顺序尝试匹配，所以派生异常的处理器放在相应基类处理器之前。`catch (...)` 可作为末尾兜底，但处理后仍需明确传播、退出或恢复策略。

`throw;` 在正在处理异常的上下文中重新抛出当前异常，保留原异常类型。`throw error;` 则根据表达式创建新的异常对象，经基类引用重新抛出时可能切片。没有正在处理的异常时执行无操作数的 `throw;` 会终止程序。

### 10.2 栈展开与局部对象清理

**完整程序：观察析构顺序。**

```cpp
#include <exception>
#include <iostream>
#include <stdexcept>

class Trace {
public:
    explicit Trace(const char* name) : name_(name) {
        std::cout << "enter " << name_ << '\n';
    }
    ~Trace() {
        std::cout << "leave " << name_ << '\n';
    }

private:
    Trace(const Trace&);
    Trace& operator=(const Trace&);
    const char* name_;
};

int main() {
    try {
        Trace outer("outer");
        {
            Trace inner("inner");
            throw std::runtime_error("operation failed");
        }
    } catch (const std::exception& error) {
        std::cout << "caught " << error.what() << '\n';
    }
    return 0;
}
```

输出：

```text
enter outer
enter inner
leave inner
leave outer
caught operation failed
```

这里的名称指针借用静态存储期字面量。跟踪类只演示作用域清理；真实资源包装应释放资源，并避免析构传播异常。

<a id="construction-failure"></a>
### 10.3 构造失败与异常安全

| 保证 | 含义 |
| --- | --- |
| 基本保证 | 不泄漏资源，对象仍满足不变式，内容可能变化 |
| 强保证 | 失败后可观察状态保持为操作前的状态 |
| 不抛保证 | 操作不向调用者抛出异常 |

构造失败时，对象本身的析构函数不会执行，但已构造的基类和成员会按规则清理。析构函数应避免让异常向外传播，尤其在已有异常展开时。

外部输入使用实际校验分支；`assert` 可能因 `NDEBUG` 被禁用，只适合检查内部假设。

构造函数体中取得的裸资源如果尚未交给成员所有者，后续失败时可能泄漏。用已构造的成员对象或局部所有者管理，通常比在每个失败分支手写清理更可靠。

普通 `new T(...)` 在取得存储后若构造失败，会按规则调用匹配的释放函数，避免这次分配本身泄漏。数组构造中某个元素失败时，已经完成构造的元素会按逆序析构，再按相应规则处理分配的存储。这不自动释放构造函数另行取得却无人拥有的资源；自定义分配与 placement new 还需要核对匹配释放规则。

### 10.4 异常说明与边界

传统动态异常说明属于本版本的语言设施，例如 `void close() throw();` 表达不允许异常逃出该函数。它不替代资源清理和真实失败分析；某些旧工具链对其支持方式也不同，不能仅凭签名推断运行行为。

违反动态异常说明会进入 `std::unexpected` 机制，默认处理导致终止；这不是编译器证明“函数体中没有任何抛出操作”。自定义处理器还受到该版本异常说明规则限制。不要将旧版 `throw()` 的机制与现代 `noexcept` 的规则混为一谈。

异常应在能够恢复、补充信息或转换为接口规定错误表示的层面处理。调用 C 接口或跨不支持同一异常机制的边界时，先在适当层捕获并转换错误；二进制兼容和系统接口分别见工程化与系统编程指南。

<a id="practice"></a>
## 11. 常见错误与练习

### 11.1 常见错误速查

| 错误 | 原因 | 改进 |
| --- | --- | --- |
| 返回局部对象引用 | 引用悬空 | 返回值对象 |
| 未初始化变量参与计算 | 未定义行为 | 声明时初始化 |
| 指针非空就访问 | 对象可能已销毁 | 明确所有权与有效期 |
| 有符号溢出依赖回绕 | 未定义行为 | 运算前校验范围 |
| 资源类只复制裸指针 | 可能重复释放 | 深复制或禁止复制 |
| `new[]` 配对 `delete` | 释放机制不匹配 | 使用 `delete[]` |
| 按基类值传派生对象 | 对象切片 | 使用引用或指针 |
| 用指针的 `sizeof` 推算数组长度 | 得到的是指针大小 | 保留长度或使用容器 |

| 其他错误 | 原因 | 改进 |
| --- | --- | --- |
| `Widget object();` 以为定义对象 | 实际声明了函数 | 检查声明解析，使用适当初始化 |
| 对修改同一变量的复杂表达式猜执行顺序 | 可能违反顺序点规则 | 拆成清楚的独立语句 |
| 保存绑定临时参数的返回引用 | 临时对象已经销毁 | 返回值或明确要求持久对象 |
| 私有继承被当作公开可替代接口 | 访问与转换受限 | 按接口意图选择继承或组合 |
| 默认参数与虚函数混用时预期错误 | 默认参数与动态分派规则不同 | 将接口默认值安排清楚 |
| 认为非抛分配形式屏蔽构造异常 | 只影响对应分配失败 | 分别处理分配与构造 |

### 11.2 分阶段练习

练习顺序：单位转换器 → 有输入校验的账户类 → 深复制缓冲区 → 多态图形接口 → 泛型容器访问函数。每次说明参数是否借用、是否修改状态、失败后状态怎样、资源何时释放。

| 练习 | 主要知识 | 验收点 |
| --- | --- | --- |
| 单位转换函数 | 类型、转换、函数接口 | 正常值、边界值和不合法输入均有定义行为 |
| 固定长度数组访问器 | 数组类型、引用、指针范围 | 空数据接口和越界访问处理清楚 |
| 账户模型 | 构造、不变式、封装 | 负金额、余额溢出和无效初值被拒绝 |
| 可复制缓冲区 | 三法则、深复制、异常安全 | 副本修改独立，自赋值正确，失败后原状态有效 |
| 图形接口 | 虚函数、切片、析构 | 经基类接口调用正确，不按值丢失派生部分 |
| 泛型工具 | 推导、重载、特化 | 支持条件明确，不满足条件时诊断可理解 |

边界验证应检查结果和对象状态，不能只确认“不崩溃”。错误代码可以用于阅读编译诊断，但不要实际运行具有未定义行为的示例来推测规则。

### 11.3 知识自检与参考资料

读完后应能回答：

1. 初始化和赋值有何区别？对象存储期与生命周期为什么不同？
2. 为什么非空指针仍可能无效？数组在哪些场合会退化？
3. 为什么复制一个指针不等于复制它拥有的资源？
4. 为什么名字查找、重载匹配、访问控制和虚调用是不同步骤？
5. 模板定义为什么常放头文件？构造失败为什么不能只指望自身析构？

本文件的旧版规则可对照 [WG21 公开审阅稿语言章节](https://www.open-std.org/jtc1/sc22/open/n2356/) 和委员会缺陷修正资料阅读；这些公开文稿不是对某个已出版标准及其全部勘误的替代。核对某条规则时，应同时确认所读资料的版本与相关缺陷修正。

版本模式的正式说明见 [GCC 标准文档](https://gcc.gnu.org/onlinedocs/gcc/Standards.html)。后续语法请按现代特性指南的最低版本分别学习。

### 11.4 本次验证记录（2026-10-07）

本文 13 个完整程序已使用 MSVC 19.51、`/std:c++14 /EHsc /W4 /permissive- /utf-8` 编译并链接，分别运行后核对全部预期输出。名字遮蔽示例保留其用于教学的遮蔽警告；其余完整程序未出现编译警告。

MSVC 不提供严格的 C++98/03 模式，因此上述运行核对与版本归属审读是不同检查。旧版语法及规则另外对照文中的 WG21 资料和缺陷修正核实；在支持相应模式的工具链上，还可以使用第 1.1 节的严格编译命令。
