# C++ 标准库指南

> 本文讲标准库组件的使用方法与契约，示例以 **C++17** 为基线。表格标出主要设施的最低版本；基础容器与流早于这一版本。线程与同步详见并发专题，底层内存和系统接口详见系统编程专题。

[返回总导航](C++Guide.md) · [核心语言](C++CoreLanguageGuide.md) · [现代特性](C++ModernFeaturesGuide.md) · [并发](C++ConcurrencyGuide.md) · [底层与系统](C++SystemProgrammingGuide.md) · [工程化](C++BuildEngineeringGuide.md)

**示例约定：** 完整程序包含头文件和 `main()`，分别编译；片段需要补齐上下文。头文件、前置条件、返回值、异常行为、复杂度和失效规则都是 API 契约的一部分。

## 阅读导航

1. [整数与基础工具类型](#integers)
2. [字符串与借用视图](#strings)
3. [容器与迭代器](#containers)
4. [算法与函数对象](#algorithms)
5. [可选值、联合值与通用调用](#utilities)
6. [智能指针与资源管理](#memory)
7. [输入输出、文件系统与时间](#io)
8. [综合练习：成绩统计器](#project)
9. [查阅方法与练习](#practice)

<a id="integers"></a>
## 1. 整数与基础工具类型

### 1.1 固定宽度整数：`std::intN_t` 与 `std::uintN_t`

这些类型从 C++11 起由 `<cstdint>` 提供。名称中的 `u` 表示无符号，数字表示**位数**，不是字节数。例如 `std::uint32_t` 是恰好 32 位、没有填充位的无符号整数。

| 有符号类型 | 取值范围 | 无符号类型 | 取值范围 |
| --- | --- | --- | --- |
| `std::int8_t` | −128 ～ 127 | `std::uint8_t` | 0 ～ 255 |
| `std::int16_t` | −32,768 ～ 32,767 | `std::uint16_t` | 0 ～ 65,535 |
| `std::int32_t` | −2,147,483,648 ～ 2,147,483,647 | `std::uint32_t` | 0 ～ 4,294,967,295 |
| `std::int64_t` | −9,223,372,036,854,775,808 ～ 9,223,372,036,854,775,807 | `std::uint64_t` | 0 ～ 18,446,744,073,709,551,615 |

这些精确宽度类型在标准中是**可选的**：实现存在满足相应要求的整数类型时才提供对应别名。常见桌面工具链通常提供表中类型，但可移植代码不能假设每个平台都有它们；类型列表与可选性见 [Microsoft `<cstdint>` 文档](https://learn.microsoft.com/en-us/cpp/standard-library/cstdint?view=msvc-170)。

它们是已有整数类型的别名，不是额外定义的强类型。例如某平台的 `std::uint32_t` 可能就是 `unsigned int`，不能把二者当作不同类型来重载函数。

常见选择：

- 协议明确规定“16 位字段”或“32 位字段”时，使用对应固定宽度类型。
- 需要负值的偏移量、差值，选择范围足够的有符号类型。
- 容器长度和索引通常沿用容器接口的类型或 `std::size_t`，不随手改成 `std::uint32_t`。
- 普通小范围计数不要求固定宽度时，可以继续使用 `int`，并验证实际范围。

固定宽度不规定**端序**。把 `std::uint32_t` 的内存直接写到网络或文件中，仍可能因大端 / 小端而得到不同字节顺序；序列化应按格式规定显式编码。在 `CHAR_BIT == 8` 的平台上，32 位占 4 字节；不能将这个前提推广到所有实现。

### 1.2 相关整数类型：`least`、`fast`、最大宽度与指针

| 类型族 | 含义 | 注意事项 |
| --- | --- | --- |
| `std::int_leastN_t` / `std::uint_leastN_t` | 至少 N 位，满足要求且存储大小最小的整数类型 | 不保证恰好 N 位；N 为 8、16、32、64 的类型由标准要求提供 |
| `std::int_fastN_t` / `std::uint_fastN_t` | 至少 N 位，由实现选择通常运算较快的类型 | 不保证在你的算法中最快，也不保证存储最小 |
| `std::intmax_t` / `std::uintmax_t` | 最大宽度整数类型 | 不应假定一定是 64 位 |
| `std::intptr_t` / `std::uintptr_t` | 可保存 `void*` 转换结果并支持转回的整数类型 | C++17 中为可选类型；不把整数地址运算当作通用指针操作 |
| `std::size_t` | `sizeof` 结果使用的无符号类型 | 常用于大小；不能假定它与 `std::uint64_t` 相同 |
| `std::ptrdiff_t` | 同一数组内合法指针相减所得的有符号类型 | 结果需可表示；不用于任意两个对象的地址相减 |

前四行类型来自 `<cstdint>`；`std::size_t`、`std::ptrdiff_t` 可通过 `<cstddef>` 获得。上述标准版本说明以本文的 C++17 基线为准，相关声明可查 [C++17 标准草案](https://timsong-cpp.github.io/cppwp/n4659/cstdint.syn)。

### 1.3 声明、边界查询与输出示例

**完整程序，C++17；要求工具链提供使用到的精确宽度类型：**

```cpp
#include <cstdint>
#include <iostream>
#include <limits>

int main() {
    const std::uint32_t packetLength{1024};
    const std::int32_t offset{-12};
    const std::uint8_t marker{200};
    const std::uint64_t largeId = UINT64_C(1) << 40;

    static_assert(std::numeric_limits<std::uint32_t>::digits == 32);

    std::cout << "packet length: " << packetLength << '\n';
    std::cout << "offset: " << offset << '\n';
    std::cout << "uint8 value: " << static_cast<unsigned int>(marker) << '\n';
    std::cout << "uint32 max: "
              << std::numeric_limits<std::uint32_t>::max() << '\n';
    std::cout << "large ID: " << largeId << '\n';
    std::cout << "-1 converted to uint32: "
              << static_cast<std::uint32_t>(-1) << '\n';
}
```

输出：

```text
packet length: 1024
offset: -12
uint8 value: 200
uint32 max: 4294967295
large ID: 1099511627776
-1 converted to uint32: 4294967295
```

`std::numeric_limits<T>::min()`、`max()` 可查询整数类型边界；`digits` 表示非符号位的有效位数，因此 `std::int32_t` 的 `digits` 是 31。也可以使用 `<cstdint>` 的 `INT32_MIN`、`INT32_MAX`、`UINT32_MAX` 等对应宏。

`UINT64_C(1)` 构造适合对应 `std::uint_least64_t` 的整数常量表达式，避免把普通 `int` 字面量直接左移 40 位。`INT32_C(...)`、`UINT32_C(...)` 等宏对应的是 `least` 类型族，宏名不带 `std::`。使用 `printf` 等格式化接口时，可用 `<cinttypes>` 的 `PRId32`、`PRIu32` 等宏，不要假定 `%lu` 一定匹配 `std::uint32_t`。

### 1.4 固定宽度类型的常见陷阱

- **8 位整数输出可能走字符重载。** `std::uint8_t`、`std::int8_t` 经常分别是 `unsigned char`、`signed char` 的别名。用流显示数值时，先转成 `unsigned int`、`int`；读取数值时，也可以先读到较宽整数，校验范围后再转换。
- **表达式结果不一定保留原位宽。** 较小的整数参与运算时会发生整数提升。在常见实现中，两个 `std::uint8_t` 相加先得到 `int`；`auto total = a + b;` 不意味着 `total` 是 8 位类型。赋回 8 位无符号类型时，超出范围的结果才按其范围转换。
- **无符号类型不能表达负数。** 整数转换为无符号整数类型时按模规则得到可表示值；例如转换 `-1` 为 `std::uint32_t` 得到 `4,294,967,295`，不是报错。先判断非负和上界，再转换外部输入。
- **有符号与无符号混用需检查转换。** 在 `int` 为 32 位、`std::uint32_t` 为 `unsigned int` 的常见实现上，`-1 < std::uint32_t{1}` 为假，因为比较前 `-1` 被转成很大的无符号值。不能仅凭数学直觉判断混合类型比较。
- **类型转换不等于范围检查。** 将不能表示的整数值转为有符号整数类型，在 C++17 中结果由实现定义；有符号算术溢出则是未定义行为。浮点数转为整数时，若截断后超出目标类型范围，也会造成未定义行为。固定宽度不会自动避免这些问题。

整数提升和整数转换的规则可分别查阅 [标准草案的整数提升条款](https://eel.is/c++draft/conv.prom) 与 [C++17 整数转换条款](https://timsong-cpp.github.io/cppwp/n4659/conv.integral)。

<a id="strings"></a>
## 2. 字符串与借用视图

| 类型 | 最低版本 | 是否拥有字符 | 用途 |
| --- | --- | --- | --- |
| `std::string` | C++98 | 是 | 存储、拼接、修改字符序列 |
| `std::wstring` | C++98 | 是 | `wchar_t` 字符序列，编码依实现约定 |
| `std::u16string` / `std::u32string` | C++11 | 是 | 相应字符单元序列 |
| `std::string_view` | C++17 | 否 | 只读借用一段字符序列 |

`std::string::size()` 统计 `char` 元素数。对 UTF-8，它通常不等于 Unicode 码点数或人眼看到的字符数；字符串类型不自动完成 Unicode 分词、规范化或大小写转换。

**片段，需要 `<string>` 与 `<string_view>`：**

```cpp
std::string owned = "hello world";
std::string copied = owned.substr(0, 5); // 新字符串
std::string_view borrowed = owned;
std::string_view prefix = borrowed.substr(0, 5); // 仍借用 owned

// std::string_view bad = std::string("temporary");
// 临时 string 在该语句结束后销毁，bad 随即悬空
```

视图不延长源对象生命周期。原字符串销毁或重新分配后，视图可能失效。`string_view::data()` 不保证在视图长度处有空字符终止符，不能直接交给任意要求 C 字符串的接口。

`string::c_str()` 提供空字符终止的序列，但字符串本身可能含内嵌空字符，所以 `strlen(c_str())` 不一定等于 `size()`。

<a id="containers"></a>
## 3. 容器与迭代器

### 3.1 按操作模式选择

| 类型 | 最低版本 | 特点 | 常见用途 |
| --- | --- | --- | --- |
| `vector` | C++98 | 连续存储，尾部插入均摊常数复杂度 | 通用动态序列 |
| `array` | C++11 | 固定大小、连续存储 | 固定数量数据 |
| `deque` | C++98 | 两端增删、随机访问，整体不保证连续 | 双端队列 |
| `list` / `forward_list` | C++98 / C++11 | 节点式双向 / 单向链表 | 节点操作与拼接 |
| `map` / `set` 及 `multi` 版本 | C++98 | 有序关联，主要查找对数复杂度 | 有序字典和集合 |
| `unordered_map` / `unordered_set` 等 | C++11 | 哈希组织，主要查找平均常数、最坏线性复杂度 | 无顺序要求的查找 |
| `stack` / `queue` / `priority_queue` | C++98 | 容器适配器 | 限制访问方式的序列 |

容器选择还要考虑缓存、内存开销、键比较和句柄失效。链表的局部插入复杂度低，不代表整体性能必然更好。

`std::vector<bool>` 是特殊实现，使用代理引用等机制，不保证存在连续的 `bool` 对象；表中的普通 `vector` 连续存储说明应排除它。详见 [C++17 `vector<bool>` 条款](https://timsong-cpp.github.io/cppwp/n4659/vector.bool)。

### 3.2 元素数量与容量

`size()` 是已构造元素数；`vector::capacity()` 是重新分配前可容纳的数量。`reserve()` 不增加元素数量，`resize()` 才改变数量。

**片段，需要 `<vector>`：**

```cpp
std::vector<int> values;
values.reserve(10);
// values[0] = 1; // 错误：没有这个元素
values.push_back(1);
values.at(0) = 2;
```

`vector::at()` 越界抛异常，`operator[]` 需要调用者保证合法下标。空容器不能直接调用 `front()`、`back()`。

`vector<int>(3, 7)` 创建三个 `7`，`vector<int>{3, 7}` 创建两个元素 `3` 和 `7`。列表初始化改变的不只是书写形式。

### 3.3 迭代与失效

算法常使用 `[first, last)` 半开区间。尾后迭代器 `end()` 不指向元素，不能解引用。范围循环按值复制、按引用访问或只读访问，应按需要选择。

| 操作 | 主要失效规则 |
| --- | --- |
| `vector` 重新分配 | 所有元素指针、引用和迭代器失效 |
| `vector` 无重新分配的插入 | 插入位置及之后的引用、迭代器失效，原尾后迭代器失效 |
| `vector::erase` | 删除位置及之后的引用和迭代器失效 |
| `map` / `set` 插入 | 不使已有元素迭代器、引用失效 |
| `map` / `set` 删除 | 被删除元素的句柄失效 |
| 哈希容器重新哈希 | 迭代器失效，已有元素指针和引用保持有效 |

这不是完整规则表，具体要查对应操作。访问字典的 `operator[]` 可能插入缺失键；只读查询用 `find()`，需要存在的元素可用 `at()`。后者对 `map` 是 C++11 新增接口。

<a id="algorithms"></a>
## 4. 算法与函数对象

**完整程序，C++17：**

```cpp
#include <algorithm>
#include <iostream>
#include <map>
#include <numeric>
#include <sstream>
#include <string>
#include <vector>

int main() {
    std::vector<int> values{5, 2, 3, 2, 1};
    std::sort(values.begin(), values.end());
    values.erase(std::unique(values.begin(), values.end()), values.end());
    for (int value : values) { std::cout << value << ' '; }
    std::cout << "\nsum="
              << std::accumulate(values.begin(), values.end(), 0) << '\n';

    std::istringstream input("blue red blue");
    std::map<std::string, int> frequency;
    std::string word;
    while (input >> word) { ++frequency[word]; }
    for (const auto& [name, count] : frequency) {
        std::cout << name << ':' << count << '\n';
    }
}
```

输出：

```text
1 2 3 5
sum=11
blue:2
red:1
```

第一行实际还带一个末尾空格。`unique` 只处理连续重复值并返回新逻辑结尾，不自行缩短容器，所以先排序，再调用 `erase`。

常见算法：

| 算法 | 用途 | 前置条件或注意事项 |
| --- | --- | --- |
| `find` / `find_if` | 查找值或条件 | 未找到返回区间终点 |
| `count_if` | 计数 | 谓词不应破坏输入 |
| `sort` / `stable_sort` | 排序 | 随机访问迭代器，比较器满足严格弱序 |
| `lower_bound` | 找到边界 | 入门时先按同一比较规则排序 |
| `transform` | 映射到输出 | 输出空间需有效，或使用插入迭代器 |
| `remove_if` | 逻辑移除 | 不直接改变容器大小，配合擦除 |
| `accumulate` | 顺序累加 | 初始值决定累加器类型 |

`list` 使用成员 `sort()`。比较器不能用 `<=`；浮点数据含 NaN 时应先处理或定义合适比较策略。累加浮点数用 `0.0`，整数仍需确认总和可表示。

谓词可以是普通函数、重载 `operator()` 的对象或 Lambda。保存引用的回调需要检查有效期；算法可能复制函数对象，不应随意依赖单个实例中的调用计数。

<a id="utilities"></a>
## 5. 可选值、联合值与通用调用

| 组件 | 最低版本 | 用途 | 注意事项 |
| --- | --- | --- | --- |
| `pair` | C++98 | 两个值组合 | 简单组合，有业务意义时可用命名结构体 |
| `tuple` | C++11 | 多值组合 | 位置访问容易损失语义 |
| `optional` | C++17 | 有值 / 无值 | 不携带失败原因 |
| `variant` | C++17 | 固定候选类型的安全联合 | 使用 `visit` / `get_if` 访问 |
| `any` | C++17 | 运行期类型擦除 | 需知道真实类型才能取回 |
| `function` | C++11 | 类型擦除的可调用对象 | 目标需可复制，可能有额外开销 |
| `reference_wrapper` | C++11 | 显式保存引用语义 | 不延长被引用对象生命周期 |
| `invoke` | C++17 | 统一调用函数、成员指针等 | 需要 `<functional>` |

`optional::value()` 无值时抛出 `bad_optional_access`，`operator*` 不做同样检查。`variant` 的类型访问失败方式取决于接口，`get_if` 用空指针表达不匹配。

**片段，需要 `<variant>` 与 `<iostream>`，C++17：**

```cpp
std::variant<int, std::string> value = 42; // 还需 <string>
std::visit([](const auto& item) { std::cout << item << '\n'; }, value);
```

选择工具时先考虑语义：正常缺失用 `optional`，固定多种表示用 `variant`，真正需要开放类型集合时才考虑 `any`。

<a id="memory"></a>
## 6. 智能指针与资源管理

| 类型 | 最低版本 | 所有权 |
| --- | --- | --- |
| `unique_ptr` | C++11 | 独占，可移动，不可复制 |
| `shared_ptr` | C++11 | 共享，通过控制块管理 |
| `weak_ptr` | C++11 | 观察共享对象，不维持其生存 |
| `make_unique` | C++14 | 创建独占所有者 |
| `make_shared` | C++11 | 创建共享所有者 |

**完整程序，C++14：**

```cpp
#include <iostream>
#include <memory>
#include <string>
#include <utility>

struct Person { std::string name; };

int main() {
    auto first = std::make_unique<Person>(Person{"Alice"});
    auto second = std::move(first);
    if (!first) { std::cout << second->name << '\n'; }

    auto shared = std::make_shared<Person>(Person{"Bob"});
    std::weak_ptr<Person> observer = shared;
    if (auto locked = observer.lock()) {
        std::cout << locked->name << '\n';
    }
}
```

输出 `Alice` 和 `Bob` 两行。`unique_ptr` 移动后源指针为空，是该类型的具体保证。`weak_ptr::lock()` 尝试取得共享所有权；先查 `expired()` 再访问不能替代它。

不要从同一裸指针独立创建多个共享控制块；双向共享所有权可能形成循环，应将观察方向改成 `weak_ptr`。引用计数管理不代表所指对象线程安全，同一个智能指针变量的并发读写也需要遵守同步规则。

资源所有权首先可用值对象或直接成员表达；函数仅借用对象时，通常接收对象引用或借用指针即可。系统资源自定义释放见底层专题。

<a id="io"></a>
## 7. 输入输出、文件系统与时间

`iostream`、`fstream`、`sstream` 提供标准流、文件流和字符串流。打开失败、读取失败与正常文件结尾应区别处理。

**完整程序，C++17：**

```cpp
#include <filesystem>
#include <fstream>
#include <iostream>
#include <stdexcept>
#include <string>

int main(int argc, char* argv[]) {
    if (argc != 2) {
        std::cerr << "usage: demo <file>\n";
        return 1;
    }
    try {
        const std::filesystem::path path(argv[1]);
        std::ifstream input(path);
        if (!input) { throw std::runtime_error("cannot open file"); }
        std::string line;
        while (std::getline(input, line)) { std::cout << line << '\n'; }
        if (input.bad() || (input.fail() && !input.eof())) {
            throw std::runtime_error("file read failed");
        }
    } catch (const std::exception& error) {
        std::cerr << error.what() << '\n';
        return 1;
    }
}
```

文件路径编码仍取决于平台、运行时和接口约定；`filesystem::path` 不自动修复来源编码不明确的窄字符参数。

- 使用 `while (getline(...))`，不使用 `while (!eof())`。
- 混用 `>>` 与 `getline` 要处理剩余换行，`std::ws` 还会跳过其他空白。
- `ofstream` 默认打开可能截断文件；追加用 `std::ios::app`，二进制用 `std::ios::binary`。
- 重要输出应显式完成关闭或刷新并检查流状态，不能只依赖析构关闭报告失败。
- `std::endl` 换行并刷新，不需立即刷新时用 `'\n'`。

`std::chrono`（C++11）表达时钟、时间点和时长。测量间隔常用 `steady_clock`；`system_clock` 表达系统时间，可能因校时跳变。数值时长应保留单位，必要时用 `duration_cast` 明确转换。

`filesystem`（C++17）提供路径与目录操作，可通过异常或 `error_code` 重载报告失败；权限、竞争变化与实际文件系统行为仍需处理。

<a id="project"></a>
## 8. 综合练习：成绩统计器

目标：读取多条 `姓名 分数` 记录，拒绝非法输入，按分数降序输出并计算平均值。分数为 `0` 到 `100` 的整数，姓名为不含空白的单个字段；输入 `quit` 或文件结尾结束。

**完整程序，C++17：**

```cpp
#include <algorithm>
#include <charconv>
#include <iomanip>
#include <iostream>
#include <numeric>
#include <optional>
#include <sstream>
#include <string>
#include <string_view>
#include <system_error>
#include <utility>
#include <vector>

struct Student {
    std::string name;
    int score{};
};

std::optional<int> parseScore(std::string_view text) {
    if (text.empty()) {
        return std::nullopt;
    }
    int score = 0;
    const char* begin = text.data();
    const char* end = begin + text.size();
    const auto [position, error] = std::from_chars(begin, end, score);

    if (error != std::errc{} || position != end || score < 0 || score > 100) {
        return std::nullopt;
    }
    return score;
}

int main() {
    std::vector<Student> students;
    std::string line;
    std::cout << "Enter: name score; enter quit to finish.\n";

    while (std::getline(std::cin, line)) {
        if (line == "quit") {
            break;
        }

        std::istringstream input(line);
        std::string name;
        std::string scoreText;
        std::string extra;
        if (!(input >> name >> scoreText) || (input >> extra)) {
            std::cerr << "Invalid format; expected: name score\n";
            continue;
        }

        const auto score = parseScore(scoreText);
        if (!score) {
            std::cerr << "Invalid score; expected an integer from 0 to 100\n";
            continue;
        }
        students.push_back(Student{std::move(name), *score});
    }

    if (std::cin.bad() || (std::cin.fail() && !std::cin.eof())) {
        std::cerr << "Input read failed\n";
        return 1;
    }
    if (students.empty()) {
        std::cout << "No valid records\n";
        return 0;
    }

    std::sort(students.begin(), students.end(),
        [](const Student& left, const Student& right) {
            if (left.score != right.score) {
                return left.score > right.score;
            }
            return left.name < right.name;
        });

    const double total = std::accumulate(students.begin(), students.end(), 0.0,
        [](double result, const Student& student) {
            return result + student.score;
        });

    for (const auto& student : students) {
        std::cout << student.name << ' ' << student.score << '\n';
    }
    const double average = total / static_cast<double>(students.size());
    std::cout << "Average: " << std::fixed << std::setprecision(2) << average << '\n';
}
```

输入：

```text
Alice 90
Bob 80
Carol 90
Dave 101
Eve 7abc
quit
```

有效记录的结果：

```text
Alice 90
Carol 90
Bob 80
Average: 86.67
```

此外会输出提示，并向标准错误报告 `Dave`、`Eve` 的非法分数。`std::from_chars` 不跳过前导空白，整数解析不接受前导 `+`；本例先把分数读取为一个独立字段，再要求完整消费字段，因此不会接受 `7abc` 这种部分有效输入。

本例把重复姓名当作多条记录；如果希望“一人一条”，需要定义重复时拒绝、覆盖还是合并。它还没有实现持久化、Unicode 姓名排序、输入规模限制和顶层异常处理，这些可以作为后续练习。

建议验证：

| 输入情形 | 预期行为 |
| --- | --- |
| 直接输入 `quit` | 输出 `No valid records`，不除以零 |
| 分数为 `0`、`100` | 接受边界值 |
| `-1`、`101`、很长的整数 | 拒绝越界数据 |
| `3.5`、`7abc`、缺少分数 | 拒绝非法格式 |
| 分数相同 | 按姓名的字符串比较顺序排列 |
| 输入最后一行后直接 EOF | 正常处理已有记录 |

<a id="practice"></a>
## 9. 查阅方法与练习

查 API 时依次确认：头文件、最低版本、参数语义、前置条件、返回值、异常、复杂度、句柄失效和同步要求。示例能编译不代表满足所有运行期前提。

练习：词频统计 → 按条件筛选排序 → 解析带错误输入的配置 → 为资源选择所有权模型 → 给成绩统计器添加文件导入和输入规模限制。

进一步阅读 [STL 专题](LearnCppSTLGuide.md)、[Microsoft 标准库文档](https://learn.microsoft.com/en-us/cpp/standard-library/cpp-standard-library-reference?view=msvc-170)。版本归属见现代特性指南，并发库见并发专题。
