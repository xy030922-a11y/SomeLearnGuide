# C++ 底层与系统编程指南

> 本文以 **C++17** 为可移植示例基线，重点解释对象与存储、二进制数据、C 接口和系统资源。Windows / POSIX API 单独标明平台范围，它们不属于 ISO 标准 C++。

返回 [C++Guide 总导航](C++Guide.md)。配套主题：

- [核心语言](C++CoreLanguageGuide.md)：类型、指针、引用、对象生命周期与 RAII。
- [现代特性](C++ModernFeaturesGuide.md)：固定宽度整数、智能指针及后续标准特性。
- [标准库](C++StandardLibraryGuide.md)：容器、流、文件系统与错误类型。
- [并发](C++ConcurrencyGuide.md)：内存模型、原子操作和线程同步。
- [构建与工程](C++BuildEngineeringGuide.md)：编译、调试、检测工具与性能分析。

**示例约定：** 本文包含 3 个完整程序，分别保存为 `main.cpp` 编译。示例 1 要求实现提供 `std::uint8_t`、`std::uint32_t`，且 `CHAR_BIT == 8`；另两个示例不依赖固定整数位宽。

## 阅读导航

1. [值、对象表示与外部格式](#representation)
2. [位宽、字节与整数运算](#integer-operations)
3. [大端字段编码](#endian-example)
4. [原始存储与对象生命周期](#storage)
5. [对象别名与 memcpy](#aliasing)
6. [C 接口与资源安全](#c-resources)
7. [文件描述符与平台句柄](#native-handles)
8. [错误返回与部分完成](#partial-io)
9. [性能与练习](#practice)
10. [编译示例](#build)

<a id="representation"></a>
## 1. 先区分值、对象表示与外部格式

一个整数的数学值、它在当前机器中的内存字节，以及协议规定的字节序列，是三个不同层面。系统编程需要明确在哪一层操作。

| 概念 | 含义 | 常见误解 |
| --- | --- | --- |
| 值 | 类型允许表示的数据 | 相同值就一定有相同原始字节 |
| 对象表示 | 对象占用的全部字节，包括可能的填充 | 结构体成员大小之和等于 `sizeof` |
| 值表示 | 决定对象值的那些位 | 每一位都必须参与表示值 |
| 外部格式 | 文件或协议规定的编码 | 可以直接写出结构体内存 |

结构体可能因对齐包含填充，整数宽度和端序也可能因实现不同而改变。含指针、`std::string` 或虚函数的对象更不能直接作为持久化格式。即使类型可以平凡复制，其对象表示也不因此成为跨平台协议。[C++17 标准草案：对象表示](https://timsong-cpp.github.io/cppwp/n4659/basic.types)。

不要用 `std::memcmp` 代替一般对象的值比较：相等的成员值可能对应不同填充字节，字节排序也不等于成员的数值排序。位域的排列和宽度规则同样不足以定义可移植的网络格式。

<a id="integer-operations"></a>
## 2. 位宽、字节与整数运算

### 2.1 字节不总是 8 位

标准保证 `sizeof(char) == 1`，这里的单位是 C++ 字节。每个字节有多少位由 `<climits>` 中的 `CHAR_BIT` 给出；不能仅凭 `sizeof(int) == 4` 推断所有平台的 `int` 都是 32 位。

常用查询：

| 工具 | 头文件 | 用途 |
| --- | --- | --- |
| `sizeof(T)` | 语言运算符 | 对象占用的字节数 |
| `alignof(T)` | 语言运算符 | 类型的对齐要求 |
| `CHAR_BIT` | `<climits>` | 一个字节的位数 |
| `std::numeric_limits<T>::digits` | `<limits>` | 不计符号位的有效位数 |
| `std::size_t` / `std::ptrdiff_t` | `<cstddef>` | 大小 / 同一数组内合法指针差 |
| `std::uint32_t` 等 | `<cstdint>` | 精确位宽的整数别名 |

`std::uint32_t` 如果存在，就有恰好 32 个值位且没有填充位；精确位宽类型是可选的。`least` 和 `fast` 类型保证至少所需位数，具体宽度可能更大。完整类型列表与使用陷阱见 [标准库整数章节](C++StandardLibraryGuide.md#integers)。[C++17 标准草案：`<cstdint>`](https://timsong-cpp.github.io/cppwp/n4659/cstdint.syn)。

### 2.2 运算之前先确认提升与边界

- 有符号整数算术溢出是未定义行为；不能依赖它自动回绕。
- 无符号运算按结果类型的范围取模，但小整数可能先提升为 `int`。两个 `std::uint8_t` 相加，不保证结果类型仍为 8 位。
- 位移次数必须非负，且小于左操作数经过整数提升后的类型位宽。不要把普通 `int` 字面量左移超过其合法范围。
- 符号与无符号混合比较可能先发生转换；先校验非负与上界，再把输入转成长度或字段类型。
- 固定位宽并不保证转换安全：范围检查、算术检查与类型转换是不同操作。

### 2.3 端序与格式边界

大端格式把高位字节放在前面，小端格式把低位字节放在前面。`std::uint32_t` 只规定位宽，不规定内存字节顺序。

C++17 没有标准的 `std::endian` 或 `std::byteswap`；它们分别来自 C++20 和 C++23。需要一个固定外部格式时，可直接按数值移位与掩码编码，不必先检测主机端序。

<a id="endian-example"></a>
## 3. 完整示例：显式编码一个 32 位大端字段

约定外部格式是 4 个 8 位字节，高位字节在前。下面按整数值编码与解码，不读取主机上的整数内存表示。

**完整程序，要求 C++17，并满足开头列出的位宽条件：**

```cpp
#include <array>
#include <climits>
#include <cstdint>
#include <iostream>

static_assert(CHAR_BIT == 8, "This format requires 8-bit bytes");

using Bytes4 = std::array<std::uint8_t, 4>;

Bytes4 encodeBigEndian(std::uint32_t value) {
    return {
        static_cast<std::uint8_t>((value >> 24) & 0xffu),
        static_cast<std::uint8_t>((value >> 16) & 0xffu),
        static_cast<std::uint8_t>((value >> 8) & 0xffu),
        static_cast<std::uint8_t>(value & 0xffu)
    };
}

std::uint32_t decodeBigEndian(const Bytes4& bytes) {
    return (static_cast<std::uint32_t>(bytes[0]) << 24)
         | (static_cast<std::uint32_t>(bytes[1]) << 16)
         | (static_cast<std::uint32_t>(bytes[2]) << 8)
         | static_cast<std::uint32_t>(bytes[3]);
}

int main() {
    const std::uint32_t value = 0x12345678u;
    const Bytes4 bytes = encodeBigEndian(value);

    std::cout << "bytes:" << std::hex;
    for (std::uint8_t byte : bytes) {
        std::cout << ' ' << static_cast<unsigned int>(byte);
    }
    std::cout << "\nvalue: 0x" << decodeBigEndian(bytes) << '\n';
    return 0;
}
```

预期输出：

```text
bytes: 12 34 56 78
value: 0x12345678
```

这里先转为 `std::uint32_t` 再移位，避免让字节类型按不合适的宽度处理高位。输出字节时先转为整数，避免 `std::uint8_t` 的字符别名使流把它显示成字符。

`std::array` 保证解码函数收到 4 个元素；处理真实文件或网络缓冲区时，必须先校验剩余数据长度，再提取字段。长度字段还要限制最大值，并检查大小计算是否溢出，例如先确认 `count <= maximum / elementSize` 再计算乘积。

可移植二进制格式还需要明确版本、字段长度、符号编码、文本编码及可选字段规则。浮点数要另行规定外部表示，不能默认所有实现都是相同 IEEE 754 布局。

<a id="storage"></a>
## 4. 原始存储、对齐与对象生命周期

### 4.1 分配存储不等于构造对象

| 操作 | 分配存储 | 构造 / 析构对象 |
| --- | --- | --- |
| `new T(...)` | 是 | 构造 `T` |
| `delete pointer` | 释放匹配存储 | 析构对象 |
| `::operator new(size)` | 是 | 不调用 `T` 的构造函数 |
| `std::malloc(size)` | 是 | 不调用 C++ 构造函数 |
| `::new (address) T(...)` | 使用已有存储 | 在该位置构造 `T` |
| `pointer->~T()` | 否 | 析构对象，不释放存储 |

placement new 要求位置非空、存储足够大、对齐满足 `T`，并且存储可用于该对象。对象构造完成后才能按 `T` 使用；析构后不能继续读取其成员。规则见 [C++17 生命周期条款](https://timsong-cpp.github.io/cppwp/n4659/basic.life) 和 [存储分配条款](https://timsong-cpp.github.io/cppwp/n4659/new.delete)。

`alignas(T)` 可以让局部缓冲区满足 `T` 的对齐。若动态分配过度对齐类型的存储，应使用匹配的 C++17 对齐分配与释放接口；不要假设普通 `malloc` 或普通 `operator new(size)` 满足任意扩展对齐。

分配与释放必须配对：`new` / `delete`、`new[]` / `delete[]`、`malloc` / `free`；直接使用带对齐参数的 `operator new` 时，释放也必须遵守对应接口契约。给已有局部缓冲区 placement new 得到的指针，不能交给普通 `delete`。

### 4.2 完整示例：在对齐缓冲区中构造对象

**完整程序，要求 C++17：**

```cpp
#include <cstddef>
#include <iostream>
#include <memory>
#include <new>
#include <string>

struct Record {
    int id;
    std::string label;
};

struct DestroyRecord {
    void operator()(Record* pointer) const noexcept {
        pointer->~Record();
    }
};

int main() {
    alignas(Record) std::byte storage[sizeof(Record)];
    std::unique_ptr<Record, DestroyRecord> record(
        ::new (static_cast<void*>(storage)) Record{7, "local storage"});

    std::cout << record->id << ' ' << record->label << '\n';
    return 0;
}
```

预期输出：

```text
7 local storage
```

存储先声明，管理对象的 `unique_ptr` 后声明，因此退出作用域时先析构 `Record`，再结束存储的生命周期。自定义 deleter 只调用析构函数，不释放局部缓冲区；普通默认 deleter 不适合这里。

本例直接使用 placement new 返回的指针，不通过旧指针猜测新对象。`std::launder` 用于特定的对象替换与访问情形，不能替代对齐、构造或生命周期管理。普通应用优先使用普通对象、容器或 `std::make_unique`；自管存储适合确有需要的内存池或底层组件。

<a id="aliasing"></a>
## 5. 对象别名与 `std::memcpy`

将某个地址 `reinterpret_cast` 为另一个类型的指针，不会让该处自动变成目标类型的有效对象。通过不允许的类型访问对象会违反别名规则；满足地址对齐也不能解决类型与生命周期问题。

C++17 允许通过 `char`、`unsigned char` 或 `std::byte` 访问对象表示。不要因为 `signed char` 名字中有 `char` 就假设它也具有同样的通用别名权限。[C++17 标准草案：允许的对象访问类型](https://timsong-cpp.github.io/cppwp/n4659/basic.lval)。

对于 `std::is_trivially_copyable_v<T>` 为真的类型，可在满足条件时复制其对象表示：例如将两个独立、已构造且有效的完整 `T` 对象之间的全部字节用 `std::memcpy` 复制，目标随后具有源对象的值。[C++17 标准草案：平凡可复制类型](https://timsong-cpp.github.io/cppwp/n4659/basic.types)。

使用时还要保证：

- 源与目标范围有效，复制大小正确；`memcpy` 要求不重叠，重叠字节复制用 `memmove`。
- 不用字节复制代替 `std::string`、智能指针等对象的复制构造或赋值。
- 复制对象字节与定义外部协议是不同任务；后者仍要显式处理端序与格式。
- 从任意输入字节还原某个类型前，必须确认其表示合法；任意位模式未必是有效指针或有效 `bool`。
- 在本指南的 C++17 显式生命周期写法中，先构造目标对象，再进行允许的表示复制；不要仅凭一个强制转换就从原始缓冲区读取 `T`。

<a id="c-resources"></a>
## 6. C 接口与资源安全

C 库常用“创建函数返回指针，另一个函数释放”的模式。立即把成功获取的资源放进 RAII 对象，可以让正常返回、提前返回和异常路径都执行清理。

`std::unique_ptr<T, Deleter>` 不只适用于 `delete`：deleter 可调用 `fclose`、库的销毁函数或其他匹配释放函数。`get()` 借用原指针；`release()` 转移出所有权，此后必须由新的所有者负责清理。

`extern "C"` 指定 C 语言链接，但不会让 C++ 类、模板或 `std::string` 自动具有可跨编译器使用的 ABI。跨 C 边界应使用约定明确的数据类型、长度和所有权规则；C++ 异常应在边界内处理并转换为接口规定的错误表示。

### 6.1 完整示例：给标准 `FILE*` 配置 deleter

**完整程序，要求 C++17：** 执行时创建临时流，写入并读回一行；临时流获取与 I/O 都可能失败。

```cpp
#include <cstdio>
#include <iostream>
#include <memory>

struct FileCloser {
    void operator()(std::FILE* file) const noexcept {
        if (file != nullptr) {
            std::fclose(file);
        }
    }
};

using File = std::unique_ptr<std::FILE, FileCloser>;

int main() {
    File file(std::tmpfile());
    if (!file) {
        std::cerr << "tmpfile failed\n";
        return 1;
    }

    if (std::fputs("hello\n", file.get()) == EOF
        || std::fflush(file.get()) == EOF) {
        std::cerr << "write failed\n";
        return 1;
    }
    if (std::fseek(file.get(), 0, SEEK_SET) != 0) {
        std::cerr << "seek failed\n";
        return 1;
    }

    char buffer[32]{};
    if (std::fgets(buffer, static_cast<int>(sizeof(buffer)), file.get())
        == nullptr) {
        std::cerr << (std::ferror(file.get())
            ? "read failed\n" : "unexpected end of file\n");
        return 1;
    }
    std::cout << buffer;

    if (std::fclose(file.release()) != 0) {
        std::cerr << "close failed\n";
        return 1;
    }
    return 0;
}
```

成功时输出：

```text
hello
```

失败时返回非零并输出错误提示。正常完成路径显式 `release()` 后关闭，检查关闭结果；提前返回路径由 deleter 兜底关闭。关闭后不能再次使用原 `FILE*`。C 文件接口来自 `<cstdio>`，属于标准库；它们的具体语义沿用 C 标准。[C++17 标准草案：C 文件接口](https://timsong-cpp.github.io/cppwp/n4659/c.files)。

析构函数适合保证清理，但不适合通过抛异常报告写回失败。对重要输出应提供显式完成或关闭接口，检查结果，再让析构处理剩余清理；缓冲刷新成功本身也不保证数据已经持久化到物理存储。

<a id="native-handles"></a>
## 7. 文件描述符、句柄与平台边界

以下为平台 API 概览，不能直接当作标准 C++ 接口使用。

| 方面 | POSIX | Windows Win32 |
| --- | --- | --- |
| 常见文件资源标识 | `int` 文件描述符 | `HANDLE` |
| 打开文件 | `open` | `CreateFileW` |
| 打开失败 | `-1`，按契约读取 `errno` | `INVALID_HANDLE_VALUE`，调用 `GetLastError` |
| 读写 | `read` / `write` | `ReadFile` / `WriteFile` |
| 关闭普通文件资源 | `close` | `CloseHandle` |
| 原生路径接口 | 按系统规则解释路径字节 | `W` 接口接收宽字符路径 |

POSIX 的非负描述符都可能有效，包括 `0`，所以不能用 `if (!fd)` 判断失败。Windows 不同创建接口可能使用不同失败哨兵，也可能要求不同关闭函数，必须按对应 API 文档封装。[POSIX `open`](https://pubs.opengroup.org/onlinepubs/9799919799/functions/open.html)、[Microsoft `CreateFileW`](https://learn.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-createfilew)、[Microsoft `CloseHandle`](https://learn.microsoft.com/en-us/windows/win32/api/handleapi/nf-handleapi-closehandle)。

原生资源通常适合可移动、禁止复制的 RAII 包装类。需要独立拥有同一底层资源时，应调用平台的复制资源接口；复制一个整数或句柄值本身并不创建新所有权。`FILE*`、POSIX 描述符与 Win32 句柄也不能相互强制转换来替代正式转换接口。

<a id="partial-io"></a>
## 8. 错误返回与部分完成

系统接口可能成功、失败，或只完成一部分请求。读取或写入一个缓冲区时，要根据实际完成字节数推进位置，区分 EOF、暂不可用、可恢复中断和永久错误，不能假设一次调用完成全部数据。[POSIX `read` 返回规则](https://pubs.opengroup.org/onlinepubs/9799919799/functions/read.html)。

只有函数契约要求检查错误状态时，才读取 `errno` 或 `GetLastError`。在日志、清理或其他函数调用之前先保存错误码，避免错误信息被覆盖；成功返回后遗留的错误码不一定与本次操作有关。[Microsoft `GetLastError` 契约](https://learn.microsoft.com/en-us/windows/win32/api/errhandlingapi/nf-errhandlingapi-getlasterror)。

关闭资源的失败也有平台语义：应核对资源是否仍然有效、是否允许重试以及编号是否可能复用。不要把所有 I/O 失败都套进同一个“重试到成功”的循环。封装层应明确获取失败、部分完成和关闭失败分别如何向调用者报告。

资源安全不等于线程安全。一个包装类拥有资源，不代表多个线程同时调用它就是安全的；同步和关闭时序见 [并发指南](C++ConcurrencyGuide.md)。

<a id="practice"></a>
## 9. 测量性能与实践练习

先确定瓶颈是分配、拷贝、缓存访问、计算还是系统调用，再选择优化。连续容器、减少分配与批量 I/O 往往值得测量，但应以真实输入和目标平台的数据判断收益。

测量经过时间可使用 `<chrono>` 的 `std::chrono::steady_clock`。`high_resolution_clock` 不保证是单调时钟。使用优化构建，重复测量代表性输入，记录运行环境，并保证计算结果被有效使用，避免测量到被编译器消除的工作。

不要用 `volatile` 代替线程同步，也不要将它视为通用的基准测试屏障。内存映射设备访问和平台屏障需要遵守目标平台与工具链契约；原子类型是否无锁也应查询具体实现。详见 [并发指南](C++ConcurrencyGuide.md) 和 [构建与工程指南](C++BuildEngineeringGuide.md)。

| 练习 | 目标 | 应验证的行为 |
| --- | --- | --- |
| 32 位编码器 | 扩展示例 1 的输入与输出 | `0`、`1`、`0x80000000`、`0xffffffff` 往返正确 |
| 二进制记录解析 | 定义版本、长度与字段顺序 | 截断、超长、未知版本和大小溢出被拒绝 |
| 单对象存储池 | 显式构造、析构和复用 | 构造失败不析构不存在的对象，释放无重复 |
| 原生文件包装 | 隔离 Windows / POSIX 实现 | 获取失败、移动、提前返回与关闭结果处理正确 |
| I/O 性能比较 | 比较小块与批量读写 | 使用相同数据、相同完成条件与多次测量 |

先把格式、生命周期与错误处理写清楚，再进行性能优化；每一次优化都应保留原有正确性契约。

<a id="build"></a>
## 10. 编译示例

GCC / Clang 使用 `-std=c++17 -Wall -Wextra -Wpedantic`，例如：

```bash
g++ -std=c++17 -Wall -Wextra -Wpedantic main.cpp -o demo
```

MSVC 开发者终端：

```bat
cl /std:c++17 /EHsc /W4 /permissive- /utf-8 /D_CRT_SECURE_NO_WARNINGS main.cpp /Fe:demo.exe
```

最后一个定义用于关闭 MSVC 对本示例标准 C 文件接口的特定弃用诊断，不表示这些接口被 ISO C++ 弃用，也不替代返回值和缓冲区检查。MSVC 的说明见 [C4996 文档](https://learn.microsoft.com/en-us/cpp/error-messages/compiler-warnings/compiler-warning-level-3-c4996?view=msvc-170)。
