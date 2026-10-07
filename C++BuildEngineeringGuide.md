# C++ 编译链接与工程化指南

> 本文讲如何把源代码组织、构建、调试和交付为可维护的工程。示例项目使用 **C++17 + CMake 3.16**。编译器、构建系统和测试工具不属于 C++ 语言标准；实际可用功能以安装的工具链为准。

[返回总导航](C++Guide.md) · [核心语言](C++CoreLanguageGuide.md) · [现代特性](C++ModernFeaturesGuide.md) · [标准库](C++StandardLibraryGuide.md) · [并发](C++ConcurrencyGuide.md) · [底层与系统](C++SystemProgrammingGuide.md)

## 阅读导航

1. [编译与链接过程](#pipeline)
2. [头文件、翻译单元与 ODR](#organization)
3. [完整 CMake 项目](#cmake)
4. [依赖、库与 ABI](#dependencies)
5. [调试与诊断](#debugging)
6. [测试、静态分析与性能](#quality)
7. [交付与工程检查](#delivery)

<a id="pipeline"></a>
## 1. 编译与链接过程

通常可以分为预处理、编译、汇编和链接。源文件连同包含内容形成翻译单元，各翻译单元可生成目标文件，链接器再解析外部符号并连接运行时和库。

| 阶段 | 主要工作 | 常见问题 |
| --- | --- | --- |
| 预处理 | 包含头文件、条件编译、展开宏 | 缺失头文件、宏污染 |
| 编译 | 语法与类型检查、生成代码 | 类型不匹配、非法表达式 |
| 汇编 | 生成目标文件 | 工具或目标平台配置问题 |
| 链接 | 解析符号、连接目标文件和库 | 未定义符号、重复定义、架构不匹配 |
| 加载与运行 | 操作系统加载程序和动态库 | 缺失运行库、环境或路径问题 |

GCC / Clang 的分步示例：

```bash
g++ -std=c++17 -Wall -Wextra -Wpedantic -g -c main.cpp -o main.o
g++ main.o -o demo
```

一次构建多个源文件：

```bash
g++ -std=c++17 -Wall -Wextra -Wpedantic -g main.cpp math_utils.cpp -o demo
```

MSVC 开发者终端：

```bat
cl /std:c++17 /EHsc /W4 /permissive- /utf-8 /Zi main.cpp math_utils.cpp /Fe:demo.exe
```

`/EHsc` 设置异常处理，`/utf-8` 设置源和执行字符集，`/Zi` 生成调试信息。具体模式见 [MSVC 官方选项说明](https://learn.microsoft.com/en-us/cpp/build/reference/std-specify-language-standard-version?view=msvc-170)，GCC 参数见 [官方手册](https://gcc.gnu.org/onlinedocs/gcc/Invoking-GCC.html)。

编译选项和链接选项不同，Sanitizer、线程库等设施还可能需要两阶段一致配置。CMake 应通过目标属性统一表达这些要求。

<a id="organization"></a>
## 2. 头文件、翻译单元与 ODR

头文件一般放声明、类型定义、模板和合适的内联定义；普通非内联函数实现放 `.cpp`。模板定义通常需要在实例化点可见，也可以通过显式实例化安排实现。

- 头文件使用保护宏；`#pragma once` 是工具链广泛支持的指令，但不是 ISO C++ 指令。
- 普通非内联函数或变量在多个翻译单元重复定义可能违反 ODR（单一定义规则）。
- `inline` 允许符合要求的多处定义，不要求编译器一定展开调用；定义仍需满足一致性要求。
- 包含自己使用的头文件，避免依赖传递包含。
- 头文件全局不写 `using namespace std;`。
- 不通过 `#include "other.cpp"` 拼接正常工程的实现。
- 头文件中的宏和不同编译配置可能让“看似相同”的定义实际不一致，应统一配置。

接口应说明参数的所有权、有效期、失败方式和同步要求。将实现细节藏在 `.cpp` 能降低重新编译和耦合；必要时可使用 Pimpl，但要考虑动态分配和间接访问成本。

<a id="cmake"></a>
## 3. 完整 CMake 项目

以下文件组成**一个完整项目**，代码块不能各自单独编译。

```text
cpp-guide-demo/
  CMakeLists.txt
  include/math_utils.hpp
  src/math_utils.cpp
  src/main.cpp
  tests/math_test.cpp
```

`include/math_utils.hpp`：

```cpp
#ifndef CPP_GUIDE_MATH_UTILS_HPP
#define CPP_GUIDE_MATH_UTILS_HPP

namespace guide {
int add(int left, int right);
}

#endif
```

`src/math_utils.cpp`：

```cpp
#include "math_utils.hpp"

namespace guide {
int add(int left, int right) {
    return left + right;
}
}
```

`src/main.cpp`：

```cpp
#include "math_utils.hpp"
#include <iostream>

int main() {
    std::cout << guide::add(2, 3) << '\n';
    return 0;
}
```

`tests/math_test.cpp`：

```cpp
#include "math_utils.hpp"
#include <iostream>

int main() {
    if (guide::add(2, 3) != 5 || guide::add(-2, 2) != 0) {
        std::cerr << "addition result mismatch\n";
        return 1;
    }
    return 0;
}
```

这里 `add` 的接口约定是结果可由 `int` 表示。若要处理任意外部整数，须增加溢出检查或调整结果类型。测试使用实际分支，避免在 `NDEBUG` 下被禁用。

`CMakeLists.txt`：

```cmake
cmake_minimum_required(VERSION 3.16)
project(CppGuideDemo VERSION 1.0 LANGUAGES CXX)

add_library(guide_math src/math_utils.cpp)
add_library(guide::math ALIAS guide_math)
target_include_directories(guide_math PUBLIC
    "${CMAKE_CURRENT_SOURCE_DIR}/include"
)
target_compile_features(guide_math PUBLIC cxx_std_17)
set_target_properties(guide_math PROPERTIES CXX_EXTENSIONS OFF)

add_executable(guide_demo src/main.cpp)
target_link_libraries(guide_demo PRIVATE guide::math)
set_target_properties(guide_demo PROPERTIES CXX_EXTENSIONS OFF)

include(CTest)
if(BUILD_TESTING)
    add_executable(guide_math_test tests/math_test.cpp)
    target_link_libraries(guide_math_test PRIVATE guide::math)
    set_target_properties(guide_math_test PROPERTIES CXX_EXTENSIONS OFF)
    add_test(NAME math_results COMMAND guide_math_test)
endif()

set(guide_targets guide_math guide_demo)
if(BUILD_TESTING)
    list(APPEND guide_targets guide_math_test)
endif()
foreach(guide_target IN LISTS guide_targets)
    if(MSVC)
        target_compile_options(${guide_target} PRIVATE /W4 /permissive- /utf-8)
    else()
        target_compile_options(${guide_target} PRIVATE -Wall -Wextra -Wpedantic)
    endif()
endforeach()
```

`PUBLIC` 要求用于库本身并传播给使用者，`PRIVATE` 只用于当前目标，`INTERFACE` 只描述使用者要求。`cxx_std_17` 表达最低语言级别，不证明标准库的每项功能均存在，见 [CMake 编译特性文档](https://cmake.org/cmake/help/latest/manual/cmake-compile-features.7.html)。

以下 `ctest --test-dir` 命令要求 CMake/CTest **3.20 或以后**；项目本身的 CMake 配置仍支持 3.16。单配置生成器，例如通常的 Ninja / Makefiles 配置：

```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Debug
cmake --build build
ctest --test-dir build --output-on-failure
```

Visual Studio 等多配置生成器：

```powershell
cmake -S . -B build
cmake --build build --config Debug
ctest --test-dir build -C Debug --output-on-failure
```

使用 CMake/CTest 3.16–3.19 时，先进入 `build`，单配置运行 `ctest --output-on-failure`，多配置运行 `ctest -C Debug --output-on-failure`。该选项的版本说明见 [CTest 官方文档](https://cmake.org/cmake/help/latest/manual/ctest.1.html#cmdoption-ctest-test-dir)。Debug / Release 的选择方式取决于生成器；切换编译器、架构或生成器通常应使用新的构建目录。

<a id="dependencies"></a>
## 4. 依赖、库与 ABI

### 4.1 静态库与动态库

静态库通常在链接时提供目标代码；动态库还涉及运行时加载、导出符号和部署。不同平台对应 `.a` / `.lib`、`.so` / `.dylib` / `.dll` 等格式，不能只凭后缀判断所有工具链关系。

库的头文件声明存在，不代表实现已被链接。引入依赖优先使用它提供的 CMake 导入目标，使包含路径、定义和传递依赖一起表达。

### 4.2 ABI 与二进制兼容

ABI 涉及名称修饰、调用约定、对象布局、异常和运行库等。语言标准相同不保证二进制兼容。

连接库前确认：目标架构、编译器及相关版本、标准库实现、运行时配置、构建模式、关键编译定义和导出约定。尤其不要在不了解兼容规则时，跨动态库边界随意传递标准库对象或跨模块分配释放资源。

`extern "C"` 用于 C 语言链接约定，不会让任意 C++ 类型自动成为 C 接口。库边界通常要明确数据格式、错误表示、资源创建和释放函数。

### 4.3 可重复构建

记录依赖版本和工具链版本，固定可重复取得的依赖来源。构建脚本不应默认依赖某台机器的绝对路径；环境差异通过工具链文件、配置参数或合适的 Presets 表达。Presets 本身有 CMake 版本要求，不能无条件套到所有旧安装。

<a id="debugging"></a>
## 5. 调试与诊断

1. 保存稳定输入和预期结果，确认问题能复现。
2. 区分编译、链接、加载、崩溃和逻辑错误。
3. 查看真实编译命令，确认标准模式、宏、包含路径和库。
4. 使用调试信息设置断点，查看变量、调用栈、对象生命周期和线程。
5. 缩小成最小复现，修改原因并验证相关边界。

| 现象 | 首先检查 |
| --- | --- |
| 头文件找不到 | 包含路径、依赖目标、文件大小写 |
| 未定义符号 | 实现文件是否加入目标，签名和库是否匹配 |
| 重复定义 | ODR、头文件定义、重复加入源文件 |
| 架构不匹配 | x86 / x64 / ARM 等目标配置 |
| 运行时库找不到 | 部署路径、加载规则、运行库 |
| Debug 正常、Release 失败 | 未定义行为、断言副作用、初始化与时序 |

查看 CMake 真实命令可使用 `cmake --build build --verbose`。运行崩溃时不要只看最后一行；定位首次出现无效状态的位置更有价值。

<a id="quality"></a>
## 6. 测试、静态分析与性能

测试核对可观察行为和边界：空输入、范围端点、格式错误、资源失败、状态转换及历史缺陷。单元测试关注独立逻辑，集成测试检查模块协作。没有崩溃不是充分验证。

静态分析可补充编译器诊断；clang-tidy 等工具需要与真实构建参数一致，可利用工具链支持的编译数据库。结果要结合接口语义判断。

对支持相应选项的 GCC / Clang 配置，可使用：

```bash
g++ -std=c++17 -g -O1 -fsanitize=address,undefined -fno-omit-frame-pointer main.cpp -o demo
```

AddressSanitizer 检查多类内存错误，UndefinedBehaviorSanitizer 检查多类未定义行为。检测范围与平台限制见 [ASan](https://clang.llvm.org/docs/AddressSanitizer.html) 和 [UBSan](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html) 文档。并发检查工具通常需要独立配置，不把不同 Sanitizer 的组合当作普遍可用。

性能分析使用代表性输入、合适优化级别和重复测量。先确认正确性，再查看分配次数、算法复杂度、缓存和锁竞争；不要凭 Debug 单次耗时判断最终性能。

<a id="delivery"></a>
## 7. 交付与工程检查

交付前确认入口和运行方式、目标平台、依赖库、资源路径、配置格式、错误日志及文件权限。用干净构建和代表性运行环境验证，避免依赖开发机的偶然状态。

实用检查清单：

- 项目能从配置说明开始构建，源文件与构建目录分开。
- 编译参数由目标表达，语言标准和依赖版本明确。
- 警告有解释，重要缺陷修复后有对应验证。
- 用户输入与资源失败有实际处理逻辑。
- 所有权和线程退出路径清楚，运行库与资源部署齐全。

练习：将成绩统计器拆成解析库、命令行入口和行为测试；再增加文件导入，验证非法输入和打开失败。参考 [CMake 官方文档](https://cmake.org/cmake/help/latest/) 与工具链的官方部署说明。
