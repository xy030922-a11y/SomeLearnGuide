# C++ STL 常用组件与泛型编程教程目录 / C++ STL Components and Generic Programming Tutorial Contents

> 本教程以 **C++17** 为统一基线，依次介绍 STL 的容器、迭代器、算法、函数对象、适配器和分配器，并补充字符串、智能指针、常用工具类型与工程实践。**C++20 / C++23** 内容单独标注，需使用支持相应特性的编译器与标准库。STL 是 C++ 标准库的重要组成部分；本文涉及的智能指针等配套标准库设施不全部属于狭义 STL。除非特别说明，学习示例应兼容 C++17。\
> This tutorial uses **C++17** as its consistent baseline and covers STL containers, iterators, algorithms, function objects, adapters, and allocators, followed by strings, smart pointers, utility types, and practical projects. **C++20 / C++23** topics are explicitly marked and require suitable compiler and standard-library support. The STL is an important part of the C++ standard library; supporting facilities such as smart pointers extend beyond the STL in its narrow sense. Unless otherwise stated, learning examples should be compatible with C++17.

## 教程导读 / Tutorial Guide

1. STL、C++ 标准库与泛型编程的关系 / The relationship between the STL, the C++ standard library, and generic programming
2. 六大组件的职责与协作：容器、算法、迭代器、函数对象、适配器和分配器 / Roles and cooperation of the six traditional STL components
3. 学习前置知识：类、模板、引用、指针、重载与对象生命周期 / Prerequisites: classes, templates, references, pointers, overloading, and object lifetime
4. C++11、C++14、C++17、C++20 与 C++23 的学习边界 / Learning boundaries across C++ language and library versions
5. MSVC、GCC、Clang 与 MSVC STL、libstdc++、libc++ 的区别 / Distinguishing compilers from standard-library implementations
6. Qt Creator 或其他 IDE 中的纯 C++ 控制台项目 / Creating a plain C++ console project in Qt Creator or another IDE
7. CMake、`target_compile_features(... cxx_std_17)` 与关闭非标准扩展 / CMake, target language features, and disabling nonstandard extensions
8. 标准头文件、`std` 命名空间与避免依赖传递包含 / Standard headers, the std namespace, and avoiding transitive-include dependencies
9. 第一个 STL 程序：`std::vector`、`std::sort()` 与结果遍历 / A first STL program using vector, sort(), and traversal
10. 阅读 API 文档：头文件、版本、前置条件、返回值、复杂度和失效规则 / Reading API documentation for headers, versions, preconditions, results, complexity, and invalidation
11. 学习顺序：基础容器与遍历、常用算法、关联容器、内存与现代扩展 / Learning order from basic containers and traversal to algorithms, associative containers, memory, and modern extensions
12. 每章练习方法：最小示例、边界输入、错误示例与性能比较 / Chapter exercises using minimal examples, boundary inputs, incorrect usage, and performance comparisons

---

## 第一篇：STL 基础——类型、对象与通用接口 / Part I: STL Foundations — Types, Objects, and Common Interfaces

### 第 1 章：STL 体系与泛型编程 / Chapter 1: STL Architecture and Generic Programming

1. 容器负责存储、算法负责处理、迭代器连接二者 / Containers for storage, algorithms for processing, and iterators as the connection
2. 函数模板与类模板的基本使用 / Basic use of function templates and class templates
3. 模板参数、默认模板参数与类型推导 / Template parameters, default arguments, and type deduction
4. 泛型接口与面向对象接口的适用场景 / Use cases for generic and object-oriented interfaces
5. `value_type`、`size_type`、`difference_type` 与嵌套类型 / Common nested types and their meanings
6. 依赖类型中的 `typename` 与模板代码的阅读 / Reading template code and dependent type names
7. C++17 类模板实参推导（CTAD）与适用限制 / Class template argument deduction and its limitations
8. 从编译错误定位缺失操作、类型不匹配和迭代器能力不足 / Diagnosing missing operations, type mismatches, and insufficient iterator capabilities

### 第 2 章：值语义、引用与对象生命周期 / Chapter 2: Value Semantics, References, and Object Lifetime

1. 对象、指针、引用与所有权的区别 / Distinguishing objects, pointers, references, and ownership
2. 初始化、赋值、复制构造与析构 / Initialization, assignment, copy construction, and destruction
3. STL 容器的值语义：复制元素及元素自身的共享行为 / Container value semantics, element copies, and sharing within element types
4. `const T&`、`T&`、按值传参和返回值 / Const references, mutable references, value parameters, and return values
5. `auto`、`auto&`、`const auto&` 与意外复制 / Type deduction, references, and accidental copies
6. 范围 `for` 的按值、按引用与只读遍历 / Value, reference, and read-only range-based loops
7. RAII、作用域结束与容器元素的自动销毁 / RAII, scope exit, and automatic element destruction
8. 悬空指针、悬空引用与临时对象有效期 / Dangling pointers, dangling references, and temporary-object lifetime
9. 容器销毁时裸指针元素与所指对象的区别 / Distinguishing destruction of raw-pointer elements from destruction of pointees

### 第 3 章：移动语义与原位构造 / Chapter 3: Move Semantics and In-Place Construction

1. 左值、右值、左值引用与右值引用 / Lvalues, rvalues, lvalue references, and rvalue references
2. `std::move()`：将表达式转换为可供移动的值类别 / Casting expressions to enable move operations
3. 移动构造、移动赋值与资源转移 / Move construction, move assignment, and resource transfer
4. 移动后标准库对象通常有效但状态未指定，具体类型可能有更强保证 / Generally valid but unspecified moved-from states, with stronger guarantees for some types
5. `const` 对象、复制回退与不能仅凭 `std::move()` 判断是否移动 / Const objects, copy fallback, and why a move cast does not guarantee a move
6. 转发引用、引用折叠与 `std::forward()` / Forwarding references, reference collapsing, and perfect forwarding
7. `push_back()`、`emplace_back()` 与实参构造方式 / Comparing insertion of existing values with in-place construction
8. `insert()`、`emplace()` 与关联容器插入失败时的构造行为 / Insertion, emplacement, and construction when associative insertion fails
9. `noexcept` 移动构造与容器重新分配 / Noexcept move construction and container reallocation
10. C++17 保证的复制消除、可选 NRVO 与返回局部对象 / Guaranteed copy elision, optional NRVO, and returning local objects

### 第 4 章：复杂度、存储布局与容器通用操作 / Chapter 4: Complexity, Storage Layout, and Common Container Operations

1. `O(1)`、`O(log n)`、`O(n)` 与 `O(n log n)` / Common asymptotic complexity classes
2. 最坏、平均与均摊复杂度的区别 / Worst-case, average-case, and amortized complexity
3. 连续存储、分段存储、节点存储与缓存局部性 / Contiguous, segmented, and node-based storage, and cache locality
4. 默认构造、数量构造、区间构造与初始化列表 / Default, count, range, and initializer-list construction
5. `std::vector<int>(10, 1)` 与 `std::vector<int>{10, 1}` / Distinguishing count-value construction from initializer-list construction
6. `empty()`、`size()`、`max_size()` 的含义与接口差异 / Meaning and availability of common size-query operations
7. `begin()`、`end()`、`cbegin()` 与 `cend()` / Mutable and read-only traversal boundaries
8. `clear()`、`swap()`、赋值与容器资源管理 / Clearing, swapping, assignment, and resource management
9. `operator[]`、`at()`、`front()`、`back()` 的适用容器与边界前提 / Container-specific element access and its preconditions
10. 元素数量、容量、已分配内存与对象有效期的区别 / Distinguishing element count, capacity, allocated memory, and object lifetime
11. 有符号与无符号混用、空容器下标和倒序循环 / Signed-unsigned mixing, empty-container indices, and reverse loops

### 第 5 章：常用工具类型与结构化绑定 / Chapter 5: Utility Types and Structured Bindings

1. `std::pair`、`std::make_pair()` 与键值对 / Value pairs and key-value representation
2. `std::tuple`、`std::make_tuple()` 与多返回值 / Tuples and multiple return values
3. `std::get()`、`std::tuple_size` 与按位置访问 / Positional access and tuple metadata
4. `std::tie()`、`std::ignore` 与已有变量的组合赋值 / Assigning tuples to existing variables and ignoring fields
5. C++17 结构化绑定与 `auto [key, value]`、`auto& [key, value]` / Structured bindings by value and by reference
6. `std::optional`：可缺失的值、`has_value()` 与 `value_or()` / Optional values, presence checks, and fallback values
7. `std::variant`、`std::get_if()` 与 `std::visit()` / Type-safe alternatives and visitation
8. `std::any` 与 `std::any_cast()`：运行时类型擦除 / Runtime type erasure and checked value extraction
9. `std::reference_wrapper`：在容器中保存引用语义 / Storing reference semantics in containers
10. `std::swap()`、`std::exchange()` 与值替换 / Swapping values and replacing a value while retrieving the old one

---

## 第二篇：顺序容器与字符串 / Part II: Sequence Containers and Strings

> 本篇按存储布局与访问方式组织容器学习，分类可参照 [Microsoft C++ 标准库容器文档](https://learn.microsoft.com/en-us/cpp/standard-library/stl-containers?view=msvc-170)。\
> This part organizes containers by storage layout and access patterns; see the [Microsoft C++ standard-library container documentation](https://learn.microsoft.com/en-us/cpp/standard-library/stl-containers?view=msvc-170) for an overview.

### 第 6 章：std::array——固定大小数组 / Chapter 6: std::array — Fixed-Size Arrays

1. `<array>`、`std::array<T, N>` 与编译期元素数量 / The array header, element types, and compile-time extent
2. 聚合初始化、值初始化与未初始化标量元素 / Aggregate initialization, value initialization, and uninitialized scalar elements
3. `operator[]`、`at()`、`front()` 与 `back()` / Indexed, checked, and endpoint access
4. `data()`、连续存储与 C 接口交互 / Contiguous storage and interoperability with C APIs
5. `size()`、`empty()` 与零长度数组的边界 / Size queries and boundaries for zero-length arrays
6. `fill()`、`swap()` 与逐元素操作 / Filling, swapping, and element-wise operations
7. `std::get()`、结构化绑定与元组协议 / Indexed tuple access, structured bindings, and the tuple protocol
8. 原生数组、`std::array` 和 `std::vector` 的选型 / Choosing between built-in arrays, array, and vector

### 第 7 章：std::vector——动态数组 / Chapter 7: std::vector — Dynamic Arrays

1. `<vector>`、连续存储与动态增长 / The vector header, contiguous storage, and dynamic growth
2. 构造、赋值、`assign()` 与区间初始化 / Construction, assignment, and initialization from ranges
3. `size()`、`capacity()`、`reserve()` 与 `resize()` / Distinguishing element count, capacity reservation, and resizing
4. `reserve()` 不创建元素，不能用下标写入容量范围内的未构造位置 / Reserved capacity does not create elements or permit indexing beyond size
5. `push_back()`、`emplace_back()` 与尾部插入的均摊复杂度 / Appending, emplacement, and amortized complexity
6. `insert()`、`erase()`、`pop_back()` 与元素搬移 / Insertion, erasure, removal from the back, and element relocation
7. `data()`、`operator[]`、`at()` 与缓冲区访问 / Buffer access and indexed element access
8. 重新分配使全部元素引用、指针和迭代器失效 / Reallocation invalidates references, pointers, and iterators to all elements
9. 未重新分配时插入、删除位置及之后的失效规则 / Invalidation at and after insertion or erasure positions without reallocation
10. `clear()` 保留容量与 `shrink_to_fit()` 的非强制收缩请求 / Retained capacity after clearing and non-binding capacity-reduction requests
11. 增长倍率由实现决定，避免每次追加前都 `reserve(size() + 1)` / Implementation-dependent growth factors and avoiding repeated one-element reservations
12. `std::vector<bool>` 特化、代理引用与非普通 `bool` 数组语义 / The vector<bool> specialization, proxy references, and differences from ordinary bool arrays

> `reserve()`、重新分配与容量收缩的语义可核对 [C++ 工作草案的 vector.capacity 条款](https://eel.is/c++draft/vector.capacity)；工作草案中的较新版本接口需与本教程基线区分。\
> For reservation, reallocation, and capacity reduction, consult [vector.capacity in the C++ working draft](https://eel.is/c++draft/vector.capacity), while distinguishing newer interfaces from this tutorial's baseline.

### 第 8 章：std::deque——双端队列 / Chapter 8: std::deque — Double-Ended Queues

1. `<deque>`、随机访问与非连续存储 / The deque header, random access, and non-contiguous storage
2. 分段存储的常见实现方式与标准保证的区别 / Common segmented implementations versus standard guarantees
3. `push_front()`、`push_back()` 与两端追加 / Insertion at both ends
4. `emplace_front()`、`emplace_back()` 与原位构造 / In-place construction at both ends
5. `pop_front()`、`pop_back()` 与非空前提 / Endpoint removal and non-empty preconditions
6. `operator[]`、`at()`、迭代器与随机访问 / Indexed access, checked access, and random-access iterators
7. 中间 `insert()`、`erase()` 的移动成本 / Element-relocation costs for insertion and erasure in the middle
8. 两端插入使迭代器失效，但保留已有元素的引用和指针 / Endpoint insertion invalidates iterators while preserving references and pointers to existing elements
9. 删除首端、尾端或中间元素时不同的失效规则 / Different invalidation rules for front, back, and middle erasure
10. 与 `vector` 的内存布局、访问开销和适用场景比较 / Comparing memory layout, access overhead, and use cases with vector

### 第 9 章：std::list 与 std::forward_list——链表 / Chapter 9: std::list and std::forward_list — Linked Lists

1. `<list>`、`<forward_list>` 与双向、单向链表 / Headers for doubly and singly linked lists
2. 节点存储、额外指针开销与遍历局部性 / Node storage, pointer overhead, and traversal locality
3. 已知位置的插入删除成本与查找位置的线性成本 / Insertion and erasure at known positions versus linear search for positions
4. `std::list` 的 `insert()`、`erase()`、`push_front()` 与 `push_back()` / Common list insertion, erasure, and endpoint operations
5. `before_begin()`、`insert_after()`、`erase_after()` 与前驱迭代器 / Forward-list operations and predecessor iterators
6. `forward_list` 不提供 `size()`、`back()` 和反向遍历接口 / Missing size, back-element, and reverse-traversal interfaces in forward_list
7. `splice()`、`splice_after()` 与节点转移、分配器相等前提 / Node transfer and allocator-equality preconditions
8. 成员 `sort()`、`merge()` 与排序、合并的前置条件 / Member sorting, merging, and their preconditions
9. 成员 `remove()`、`remove_if()`、`unique()` 与实际节点删除 / Member removal, deduplication, and physical node erasure
10. `reverse()`、节点稳定性与已删除元素的迭代器失效 / Reversal, node stability, and invalidation for erased elements
11. 不支持随机访问：不能直接使用 `std::sort()` / Lack of random access and why std::sort() cannot be used directly

### 第 10 章：std::string——字符串操作 / Chapter 10: std::string — String Operations

1. `<string>`、`std::basic_string` 与 `std::string` / The string header, the basic_string template, and string
2. 构造、赋值、长度与包含内嵌空字符的字符串 / Construction, assignment, length, and embedded null characters
3. `append()`、`operator+=`、`insert()`、`erase()` 与 `replace()` / Appending, insertion, erasure, and replacement
4. `find()`、`rfind()`、`find_first_of()` 与 `std::string::npos` / Searching and detecting missing matches
5. `substr()`、`compare()` 与字符串比较 / Owning substrings and string comparisons
6. `c_str()`、`data()`、空字符终止与指针有效期 / Character buffers, null termination, and pointer lifetime
7. C++17 非 const `data()` 与仅修改有效元素范围 / Mutable data access in C++17 and valid write bounds
8. `std::getline()`、空格读取与流状态处理 / Reading complete lines and handling stream state
9. `std::stoi()`、`std::stod()`、`std::to_string()` 与错误处理 / Numeric conversions and error handling
10. C++17 `std::from_chars()`、`std::to_chars()` 与返回状态检查 / Character conversions and checking error codes and returned pointers
11. 字节、编码单元、Unicode 码点与 UTF-8 多字节字符 / Bytes, code units, Unicode code points, and multibyte UTF-8 characters
12. 小字符串优化是实现细节，不能假设固定阈值 / Small-string optimization as an implementation detail without a fixed portable threshold

### 第 11 章：std::string_view 与位集合工具 / Chapter 11: std::string_view and Bitset Utilities

1. `<string_view>`、C++17 `std::string_view` 与只读字符串视图 / Read-only string views in C++17
2. 从字面量、`std::string` 与指针加长度构造视图 / Constructing views from literals, strings, and pointer-length pairs
3. `substr()`、`remove_prefix()` 与 `remove_suffix()`：调整视图 / Adjusting views without copying the underlying characters
4. 视图不拥有数据：源对象销毁、修改或重分配导致的有效期问题 / Non-owning views and lifetime issues caused by source destruction, mutation, or reallocation
5. `data()` 不保证视图末尾以空字符终止 / No guarantee of null termination at the end of a view
6. 函数参数与返回视图时的生命周期约定 / Lifetime contracts for view parameters and returned views
7. `<bitset>`、`std::bitset<N>` 与固定大小位集合 / Fixed-size collections of bits
8. `set()`、`reset()`、`flip()`、`test()`、`count()` 与位查询 / Bit modification, checked access, and population counting
9. `all()`、`any()`、`none()`、位运算与移位 / Aggregate bit queries, bitwise operations, and shifts
10. `to_string()`、`to_ulong()`、`to_ullong()` 与转换异常 / String and integer conversions and conversion exceptions
11. `bitset` 不提供普通容器迭代器；动态位存储与 `vector<bool>` 的取舍 / The absence of ordinary container iterators and tradeoffs for dynamic bit storage

---

## 第三篇：关联容器与容器适配器 / Part III: Associative Containers and Container Adapters

### 第 12 章：std::set 与 std::multiset——有序集合 / Chapter 12: std::set and std::multiset — Ordered Sets

1. `<set>`、有序键集合与唯一键、多重键 / Ordered key collections with unique or equivalent keys
2. 平衡搜索树的常见实现与标准复杂度要求 / Common balanced-search-tree implementations and standard complexity requirements
3. `insert()`、`emplace()` 与插入结果 / Insertion, emplacement, and interpreting insertion results
4. `find()` 的对数查找；`multiset::count()` 的复杂度为 `O(log n + 匹配数量)` / Logarithmic lookup with find(); multiset::count() has complexity O(log n + number of matches)
5. `lower_bound()`、`upper_bound()` 与 `equal_range()` / Member boundary queries and equivalent-key ranges
6. `erase(key)`、`erase(iterator)` 与多重键删除数量 / Erasing by key or iterator and the number of removed duplicates
7. `std::less<T>`、`std::greater<T>` 与自定义排序 / Ascending, descending, and custom orderings
8. 比较器定义键等价：`!comp(a, b) && !comp(b, a)` / Comparator-defined key equivalence
9. 元素键不可通过迭代器直接修改 / Keys cannot be modified directly through iterators
10. 排序去重、区间查询与 `vector` 排序方案的比较 / Deduplication, interval queries, and comparison with sorting a vector

### 第 13 章：std::map 与 std::multimap——有序映射 / Chapter 13: std::map and std::multimap — Ordered Maps

1. `<map>`、键值映射与 `std::pair<const Key, T>` / Ordered mappings and their key-value element type
2. `insert()`、`emplace()`、`emplace_hint()` 与插入位置提示 / Insertion, emplacement, and position hints
3. `operator[]` 的缺失键插入行为与映射值初始化 / Missing-key insertion and mapped-value initialization through subscripting
4. `at()`、`find()` 与避免只读查询意外插入 / Checked access and lookup without accidental insertion
5. C++17 `try_emplace()`：键存在时不构造映射值，但调用实参仍会求值 / Avoiding mapped-value construction for existing keys while still evaluating call arguments
6. C++17 `insert_or_assign()`：插入或更新映射值 / Inserting or assigning a mapped value
7. 结构化绑定遍历、只读键与可修改映射值 / Structured bindings, immutable keys, and mutable mapped values
8. `multimap` 的重复键、`equal_range()` 与无 `operator[]` 接口 / Equivalent keys, range queries, and the absence of subscripting in multimap
9. `lower_bound()`、`upper_bound()` 与按键区间处理 / Processing ordered key intervals
10. 有序索引、计数统计与一对多关系建模 / Ordered indexes, frequency counting, and one-to-many relationships

### 第 14 章：无序关联容器——哈希表 / Chapter 14: Unordered Associative Containers — Hash Tables

1. `<unordered_map>`、`<unordered_set>` 与四类无序关联容器 / Headers and the four unordered associative container types
2. `std::unordered_map`、`std::unordered_set` 与唯一键 / Hash-based mappings and sets with unique keys
3. `std::unordered_multimap`、`std::unordered_multiset` 与等价键组 / Hash-based containers with equivalent-key groups
4. 哈希函数、相等判断、桶与冲突处理 / Hash functions, equality predicates, buckets, and collisions
5. `insert()`、`emplace()`、`find()`、`count()` 与 `erase()` / Insertion, lookup, counting, and erasure
6. `unordered_map` 的 `operator[]`、`at()`、`try_emplace()` 与 `insert_or_assign()` / Access and update operations for unordered mappings
7. 单键查找平均常数、最坏线性的复杂度边界 / Average constant-time and worst-case linear-time single-key lookup
8. 无序遍历与不可依赖的输出顺序 / Unspecified traversal order and avoiding order-dependent output
9. `equal_range()` 与等价键查询；不提供有序上下界查询 / Equivalent-key lookup without ordered boundary queries
10. 重新哈希使迭代器失效，但不使未删除元素的引用和指针失效 / Rehashing invalidates iterators while preserving references and pointers to surviving elements

### 第 15 章：哈希策略与自定义键 / Chapter 15: Hash Policies and Custom Keys

1. `std::hash<T>`、自定义哈希函数对象与模板参数 / Standard hashing, custom hash function objects, and template parameters
2. `KeyEqual` 与“等价键必须具有相同哈希值”的约束 / Equality predicates and the requirement that equivalent keys have equal hash values
3. 结构体、组合键与多个字段的哈希组合 / Hashing structures, composite keys, and multiple fields
4. 大小写无关键的哈希与相等规则保持一致 / Consistent hashing and equality for case-insensitive keys
5. `bucket_count()`、`bucket_size()`、`bucket()` 与桶内遍历 / Bucket inspection and local iteration
6. `load_factor()`、`max_load_factor()` 与时间、空间取舍 / Load factors and time-space tradeoffs
7. `reserve()` 按预期元素数量规划，`rehash()` 按桶数量约束规划 / Reserving for expected elements versus requesting a bucket-count lower bound
8. 批量插入前预留容量与重新哈希成本 / Reserving before bulk insertion and the cost of rehashing
9. 不把 `std::hash` 当作加密哈希或跨进程持久化标识 / Standard hashes are not cryptographic hashes or portable persistent identifiers
10. 重复键、冲突数据与哈希质量的测试 / Testing duplicate keys, collision-heavy inputs, and hash quality

### 第 16 章：比较器、异构查找与节点句柄 / Chapter 16: Comparators, Heterogeneous Lookup, and Node Handles

1. 严格弱序的非自反性、传递性与等价关系 / Irreflexivity, transitivity, and equivalence under strict weak ordering
2. 多字段排序、升降序组合与相同主键的次级排序 / Multi-field orderings and tie-breaking
3. 禁止用 `<=` 作为严格排序比较器 / Why less-than-or-equal is not a strict ordering predicate
4. 比较器所依赖状态的稳定性与修改外部数据的风险 / Stable comparison state and risks from changing external data
5. C++14 透明比较器 `std::less<>` 与有序容器异构查找 / Transparent comparators and heterogeneous lookup in ordered containers
6. C++17 `extract()`、节点句柄与脱离容器后的键修改 / Node extraction, node handles, and key modification outside a container
7. 节点重新插入、唯一键冲突与未插入节点的处理 / Node reinsertion, key conflicts, and handling rejected nodes
8. C++17 `merge()`：转移兼容节点与保留冲突键 / Merging compatible nodes and retaining conflicting keys
9. 节点转移中的分配器相等要求、引用与迭代器规则 / Allocator-equality requirements and reference and iterator validity during node transfer
10. C++20 无序容器异构查找的透明哈希与透明相等谓词（扩展） / Transparent hashing and equality for heterogeneous unordered lookup in C++20

> 有序键的等价关系与无序容器规则可参照工作草案的 [关联容器要求](https://eel.is/c++draft/associative.reqmts)和[无序关联容器要求](https://eel.is/c++draft/unord.req)；较新接口按章节版本标记学习。\
> For ordered-key equivalence and unordered-container rules, see the working draft's [associative-container requirements](https://eel.is/c++draft/associative.reqmts) and [unordered-container requirements](https://eel.is/c++draft/unord.req), observing the version labels in this tutorial.

### 第 17 章：std::stack、std::queue 与 std::priority_queue / Chapter 17: std::stack, std::queue, and std::priority_queue

1. `<stack>`、`<queue>` 与容器适配器的受限接口 / Adapter headers and their restricted interfaces
2. `std::stack`：后进先出、`push()`、`top()` 与 `pop()` / Stack operations and last-in-first-out behavior
3. `std::queue`：先进先出、`front()`、`back()` 与 `pop()` / Queue operations and first-in-first-out behavior
4. `std::priority_queue`：优先级访问与堆结构 / Priority access and heap organization
5. 默认大顶堆、`std::greater<T>` 小顶堆与自定义优先级 / Default max-heaps, min-heaps, and custom priorities
6. `emplace()`、`empty()`、`size()` 与非空访问前提 / Emplacement, state queries, and non-empty access preconditions
7. `pop()` 不返回被删除值，先读取或移动再移除 / Reading or moving a value before removal because pop() returns no value
8. `priority_queue::top()` 返回 const 引用与移动取出的限制 / Const access to the highest-priority element and limitations on moving it out
9. 默认底层容器与自定义底层容器需要满足的操作 / Default underlying containers and requirements for alternative containers
10. 无公开遍历接口、无任意位置删除与堆优先级更新限制 / No public traversal or arbitrary erasure, and limits on updating priorities
11. 表达式求值、广度优先遍历、任务调度与 Top-K / Expression evaluation, breadth-first traversal, task scheduling, and Top-K selection

---

## 第四篇：迭代器与区间 / Part IV: Iterators and Ranges

### 第 18 章：迭代器模型与分类 / Chapter 18: Iterator Models and Categories

1. 迭代器作为容器与算法之间的接口 / Iterators as the interface between containers and algorithms
2. 输入迭代器：读取元素与单遍遍历约束 / Input iterators: reading elements and single-pass constraints
3. 输出迭代器：写入元素与目标位置要求 / Output iterators: writing elements and destination requirements
4. 前向迭代器：多遍遍历保证 / Forward iterators and the multipass guarantee
5. 双向迭代器：递增、递减与边界条件 / Bidirectional iterators: increment, decrement, and boundaries
6. 随机访问迭代器：常数时间跳转、相减与下标访问 / Random-access iterators: constant-time jumps, subtraction, and indexing
7. 连续迭代器：C++17 的连续性要求与 C++20 的 `std::contiguous_iterator` 概念 / Contiguous iterators: C++17 requirements and the C++20 `std::contiguous_iterator` concept
8. `std::iterator_traits`：`value_type`、`difference_type`、`reference`、`pointer` 与 `iterator_category` / Iterator-associated types through `std::iterator_traits`
9. 原生指针、容器迭代器与代理引用；`std::vector<bool>` 的特殊性 / Raw pointers, container iterators, proxy references, and the `std::vector<bool>` specialization
10. `iterator`、`const_iterator` 与迭代器对象自身的 `const` / `iterator`, `const_iterator`, and const-qualified iterator objects
11. 自定义迭代器的操作与语义要求；避免依赖 C++17 已弃用的 `std::iterator` / Operations and semantic requirements for custom iterators; avoiding `std::iterator`, deprecated in C++17

### 第 19 章：迭代器操作与区间 / Chapter 19: Iterator Operations and Ranges

1. `[first, last)`：左闭右开区间、空区间与尾后迭代器 / Half-open ranges, empty ranges, and past-the-end iterators
2. `begin()`、`end()`、`cbegin()` 与 `cend()`：可修改与只读遍历 / Mutable and read-only traversal
3. `std::begin()`、`std::end()` 与原生数组的统一访问 / Uniform access to containers and built-in arrays
4. `rbegin()`、`rend()`、`crbegin()` 与 `crend()`：反向区间 / Reverse ranges
5. `std::advance()`：原地移动；负距离要求双向或随机访问迭代器 / Advancing in place; negative distances require bidirectional or random-access iterators
6. `std::next()` 与 `std::prev()`：返回移动后的迭代器副本 / Returning advanced iterator copies
7. `std::distance()`：随机访问迭代器为常数复杂度，其他适用迭代器为线性复杂度 / Constant-time distance for random-access iterators and linear-time distance otherwise
8. `std::advance()`、`std::next()` 与 `std::prev()` 的复杂度取决于迭代器类别 / Iterator categories determine the complexity of advancement operations
9. 区间有效性、可达性与同一序列内的比较；不得解引用 `end()` / Valid ranges, reachability, comparisons within a sequence, and non-dereferenceable `end()`
10. `std::size()`、`std::empty()` 与 `std::data()`（C++17）：统一查询与数据访问 / Uniform size, emptiness, and data access in C++17
11. 基于范围的 `for` 与 `auto`、`auto&`、`const auto&` 的选择 / Range-based `for` and choosing value or reference bindings

### 第 20 章：迭代器适配器 / Chapter 20: Iterator Adapters

1. `std::reverse_iterator` 与 `std::make_reverse_iterator()` / Reverse iterators and their factory function
2. 反向迭代器的 `base()`：指向被反向访问元素之后的位置 / `base()` points one position after the element accessed through a reverse iterator
3. `std::back_insert_iterator` 与 `std::back_inserter()`：通过 `push_back()` 追加 / Appending through `push_back()`
4. `std::front_insert_iterator` 与 `std::front_inserter()`：通过 `push_front()` 插入及顺序变化 / Front insertion through `push_front()` and its effect on order
5. `std::insert_iterator` 与 `std::inserter()`：在指定位置调用容器插入操作 / Inserting through a container at a specified position
6. 插入迭代器的容器接口要求与动态扩展目标区间 / Container interface requirements and growing output sequences
7. `std::istream_iterator`：格式化输入、单遍读取与流结束状态 / Formatted input, single-pass reading, and end-of-stream state
8. `std::ostream_iterator`：格式化输出与分隔符 / Formatted output and delimiters
9. `std::istreambuf_iterator` 与 `std::ostreambuf_iterator`：流缓冲区字符访问 / Character access through stream buffers
10. `std::move_iterator` 与 `std::make_move_iterator()`：调整解引用结果以支持移动 / Adapting dereference results to enable moving
11. 移动迭代器不保证一定发生移动；元素的 `const` 与可用重载影响结果 / Move iterators do not guarantee a move; constness and available overloads affect the result

### 第 21 章：迭代器失效与安全遍历 / Chapter 21: Iterator Invalidation and Safe Traversal

1. 区分迭代器、指针、引用与尾后迭代器的失效规则 / Distinguishing invalidation rules for iterators, pointers, references, and past-the-end iterators
2. `std::vector` 扩容：重新分配使全部迭代器、指针与引用失效 / Vector reallocation invalidates all iterators, pointers, and references
3. `std::vector` 未扩容的插入与擦除：受影响位置及其后的失效范围 / Invalidation at and after affected positions when vector insertion or erasure does not reallocate
4. `std::deque` 两端插入使迭代器失效，但保留现有元素的引用；中间修改需单独分析 / Deque end insertion invalidates iterators but preserves references to existing elements; analyze middle modifications separately
5. `std::list` 与 `std::forward_list`：插入保持现有迭代器有效，擦除使被删元素的迭代器失效 / List insertion preserves existing iterators; erasure invalidates iterators to erased elements
6. 有序关联容器：插入保持现有迭代器有效，擦除只影响被删元素 / Ordered associative containers preserve existing iterators on insertion and invalidate only erased elements on erasure
7. 无序关联容器：重哈希使迭代器失效，但不使现有元素的指针与引用失效 / Rehashing invalidates unordered-container iterators but preserves pointers and references to existing elements
8. 使用 `erase()` 或 `erase_after()` 返回的迭代器继续遍历 / Continuing traversal with the iterator returned by `erase()` or `erase_after()`
9. 缓存 `end()`、遍历中插入与基于范围的 `for` 的失效隐患 / Invalidation hazards from cached end iterators and insertion during traversal
10. 容器销毁、清空、赋值与移动操作后的有效性检查 / Checking validity after destruction, clearing, assignment, and move operations
11. 调试迭代器与 AddressSanitizer：检查越界及悬空访问，理解工具检测范围 / Debug iterators and AddressSanitizer for bounds and dangling-access checks, with awareness of detection limits

---

## 第五篇：函数对象与可调用对象 / Part V: Function Objects and Callables

### 第 22 章：函数对象与预定义操作 / Chapter 22: Function Objects and Predefined Operations

1. 函数对象与 `operator()`：将行为传递给算法 / Function objects and passing behavior to algorithms through `operator()`
2. 一元函数、二元函数、一元谓词与二元谓词 / Unary and binary functions and predicates
3. `std::plus`、`std::minus`、`std::multiplies`、`std::divides`、`std::modulus` 与 `std::negate` / Arithmetic function objects
4. `std::equal_to`、`std::not_equal_to`、`std::less` 与 `std::greater` / Equality and ordering function objects
5. `std::less_equal` 与 `std::greater_equal`：一般比较用途及其不满足排序严格弱序的原因 / Non-strict comparisons and why they do not satisfy sorting's strict weak ordering requirement
6. `std::logical_and`、`std::logical_or` 与 `std::logical_not`：函数调用实参不具备内建逻辑运算的短路规则 / Logical function objects do not short-circuit argument evaluation
7. `std::bit_and`、`std::bit_or`、`std::bit_xor` 与 `std::bit_not` / Bitwise function objects
8. `std::less<>` 等透明函数对象（C++14）：类型推导与异构比较 / Transparent function objects in C++14: type deduction and heterogeneous comparison
9. 有状态函数对象、算法内部复制与外部状态管理 / Stateful function objects, copies inside algorithms, and external state management
10. 比较器的严格弱序：非自反性、传递性与等价关系的一致性 / Strict weak ordering: irreflexivity, transitivity, and consistent equivalence
11. 自定义排序条件、多字段比较与浮点 `NaN` 的比较策略 / Custom ordering, multi-field comparison, and comparison policies for floating-point `NaN`

### 第 23 章：Lambda 表达式 / Chapter 23: Lambda Expressions

1. Lambda 的捕获列表、参数列表、返回类型与函数体 / Capture lists, parameters, return types, and bodies
2. 按值捕获、按引用捕获与显式列出依赖 / Capturing by value or reference and making dependencies explicit
3. 默认捕获 `[=]`、`[&]` 与混合捕获 / Default and mixed captures
4. `mutable`：修改按值捕获的闭包成员 / Modifying closure members captured by value
5. 初始化捕获（C++14）：重命名变量与捕获仅移动对象 / Init-capture in C++14: renaming variables and capturing move-only objects
6. 泛型 Lambda（C++14）：使用 `auto` 参数 / Generic lambdas with `auto` parameters in C++14
7. `this` 捕获与 `[*this]`（C++17）：指针捕获与对象副本的区别 / Capturing the `this` pointer versus an object copy with `[*this]` in C++17
8. 无捕获 Lambda 到函数指针的转换 / Conversion of captureless lambdas to function pointers
9. `constexpr` Lambda（C++17）与常量表达式条件 / C++17 constexpr lambdas and constant-expression requirements
10. Lambda 作为查找谓词、比较器、变换操作与删除条件 / Lambdas as search predicates, comparators, transformations, and removal conditions
11. 引用捕获的生命周期、返回闭包与异步执行中的悬空风险 / Reference-capture lifetimes and dangling references in returned or asynchronously executed closures

### 第 24 章：可调用对象包装与绑定 / Chapter 24: Callable Wrappers and Binding

1. 函数指针、成员函数指针、函数对象与 Lambda 的调用方式 / Calling function pointers, member-function pointers, function objects, and lambdas
2. `std::invoke()`（C++17）：统一调用普通可调用对象与成员指针 / Unified invocation of callables and member pointers in C++17
3. `std::function`：按函数签名进行类型擦除 / Type erasure by function signature with `std::function`
4. `std::function` 对目标可复制性的要求与仅移动闭包的限制 / Copyable target requirements and restrictions on move-only closures
5. 空 `std::function` 的判断与 `std::bad_function_call` / Detecting empty wrappers and handling `std::bad_function_call`
6. `std::function` 的分配、间接调用与模板参数方案的取舍 / Allocation and indirect-call costs versus templated callable parameters
7. `std::bind()` 与 `std::placeholders`：参数绑定、重排与存储方式 / Argument binding, reordering, and storage with `std::bind()`
8. `std::ref()`、`std::cref()` 与 `std::reference_wrapper`：显式传递引用语义 / Explicit reference semantics through reference wrappers
9. `std::mem_fn()`：将成员指针适配为可调用对象 / Adapting member pointers with `std::mem_fn()`
10. `std::not_fn()`（C++17）：对可调用对象的结果取反 / Negating callable results with `std::not_fn()` in C++17
11. 使用 Lambda 表达绑定意图与维护所引用对象的生命周期 / Expressing bindings with lambdas and maintaining referenced-object lifetimes
12. `std::apply()`（C++17）：展开元组元素以调用可调用对象 / Invoking a callable with expanded tuple elements in C++17

---

## 第六篇：标准算法与数值处理 / Part VI: Standard Algorithms and Numeric Processing

### 第 25 章：算法模型与非修改算法 / Chapter 25: Algorithm Models and Non-Modifying Algorithms

1. `<algorithm>` 与 `<numeric>`：算法分类、输入要求与返回值 / Algorithm families, input requirements, and return values
2. 迭代器类别、有效区间、谓词约束与复杂度说明 / Iterator categories, valid ranges, predicate requirements, and complexity specifications
3. `std::all_of()`、`std::any_of()` 与 `std::none_of()`：条件判断及空区间结果 / Predicate checks and their empty-range results
4. `std::find()`、`std::find_if()` 与 `std::find_if_not()`：返回位置或尾后迭代器 / Finding a position or returning the end iterator
5. `std::count()` 与 `std::count_if()`：元素计数 / Counting matching elements
6. `std::mismatch()` 与 `std::equal()`：序列比较及双区间重载 / Comparing sequences with mismatch and equality operations and two-range overloads
7. `std::search()`、`std::find_end()`、`std::find_first_of()` 与 `std::search_n()` / Searching subsequences, sets of candidates, and repeated values
8. `std::adjacent_find()`：相邻元素关系检查 / Finding adjacent elements that satisfy a relation
9. `std::for_each()` 与 `std::for_each_n()`（C++17）：逐项操作及修改元素的条件 / Applying operations to elements and conditions for modifying them
10. `std::min_element()`、`std::max_element()` 与 `std::minmax_element()`：极值位置 / Locating minimum and maximum elements
11. `std::lexicographical_compare()` 与 `std::is_permutation()`：字典序与排列等价 / Lexicographical ordering and permutation equivalence
12. `std::default_searcher`、`std::boyer_moore_searcher` 与 `std::boyer_moore_horspool_searcher`（C++17） / Reusable searcher objects introduced in C++17

### 第 26 章：复制、移动、变换与填充 / Chapter 26: Copying, Moving, Transforming, and Filling

1. `std::copy()`、`std::copy_n()` 与 `std::copy_if()`：复制整个区间、指定数量或匹配元素 / Copying a range, a count, or matching elements
2. 目标区间必须可写且足够长；`reserve()` 不创建可供赋值的元素 / Output ranges must be writable and large enough; `reserve()` does not create elements to assign to
3. `std::back_inserter()` 与预先 `resize()`：构造输出区间的两种方式 / Growing output with an inserter or preparing elements with `resize()`
4. 无执行策略的 `std::copy()`：目标起点不得位于源区间内，可用于满足前提的向左重叠复制 / Non-policy `std::copy()` requires the destination start outside the source range and supports valid leftward overlapping copies
5. `std::copy_backward()`：从尾部复制；向右重叠复制须满足目标尾位置前提 / Backward copying and the destination-end precondition for rightward overlapping copies
6. `std::copy_if()` 与带执行策略的 `std::copy()` 要求输入输出区间不重叠 / `std::copy_if()` and execution-policy `std::copy()` require non-overlapping input and output ranges
7. 三参数算法 `std::move()` 与 `std::move_backward()`：逐元素移动赋值 / Moving elements by assignment with the range algorithms
8. 单参数 `std::move()` 属于 `<utility>` 的值类别转换；与同名区间算法的职责区别 / The single-argument utility `std::move()` changes value category and differs from the range algorithm
9. `std::transform()`：一元、二元变换与允许的原地输出；逐个核对重叠和操作副作用要求 / Unary and binary transformations, permitted in-place output, and per-overload overlap and side-effect requirements
10. `std::fill()`、`std::fill_n()`、`std::generate()` 与 `std::generate_n()` / Filling with values or generating elements
11. `std::swap()`、`std::iter_swap()` 与 `std::swap_ranges()`：对象、元素及区间交换 / Swapping objects, iterator-referenced elements, and ranges

### 第 27 章：删除、去重、替换与重排 / Chapter 27: Removing, Deduplicating, Replacing, and Rearranging

1. `std::remove()` 与 `std::remove_if()`：压缩保留元素并返回逻辑尾，不改变容器大小 / Compacting retained elements and returning a logical end without changing container size
2. erase-remove 惯用法：将算法返回值到原尾位置的区间交给容器 `erase()` / Erasing the range from the algorithm's returned iterator to the original end
3. 删除算法后的尾部元素处于有效但未指定状态；尾部不等于被删除值的集合 / The remaining tail has valid but unspecified values and does not represent the removed-value set
4. `std::remove_copy()` 与 `std::remove_copy_if()`：保留原区间并输出筛选结果 / Producing filtered output while preserving the input range
5. `std::unique()`：只合并相邻等价元素，返回逻辑尾 / Removing consecutive equivalent elements and returning a logical end
6. 全局去重时的排序与等价关系设计；`std::unique_copy()` 的输出方式 / Ordering and equivalence for global deduplication and output with `std::unique_copy()`
7. 链表成员 `remove()`、`remove_if()` 与 `unique()`：直接移除节点 / List member operations that erase nodes directly
8. `std::replace()`、`std::replace_if()`、`std::replace_copy()` 与 `std::replace_copy_if()` / In-place replacement and replacement while copying
9. `std::reverse()`、`std::reverse_copy()`、`std::rotate()` 与 `std::rotate_copy()` / Reversing and rotating ranges, in place or into another range
10. `std::shuffle()`（C++11）与 `std::sample()`（C++17）：随机重排与抽样；`std::random_shuffle()` 在 C++17 已移除 / Shuffling and sampling; `std::random_shuffle()` was removed in C++17
11. `std::erase()` 与 `std::erase_if()`（C++20 扩展）：按容器支持情况简化删除 / C++20 erasure helpers, according to container support

### 第 28 章：排序、分区与第 N 个元素 / Chapter 28: Sorting, Partitioning, and the Nth Element

1. `std::sort()`：随机访问迭代器与 `O(N log N)` 比较复杂度 / Random-access iterator requirements and `O(N log N)` comparison complexity
2. `std::stable_sort()`：保留等价元素原有顺序与额外内存的取舍 / Preserving equivalent-element order and the tradeoff involving temporary memory
3. `std::partial_sort()` 与 `std::partial_sort_copy()`：得到有序的前 K 个元素 / Producing the first K elements in sorted order
4. `std::nth_element()`：定位排序后第 N 个位置的元素；两侧各自不保证有序 / Selecting the element at a sorted position without sorting either side
5. `std::is_sorted()` 与 `std::is_sorted_until()`：检查整体或前缀有序性 / Checking sorted ranges or sorted prefixes
6. `std::partition()` 与 `std::stable_partition()`：按谓词分组及稳定性 / Partitioning by a predicate with or without stability
7. `std::partition_copy()`：将两类元素写入不同目标区间 / Writing matching and non-matching elements into separate output ranges
8. `std::is_partitioned()` 与 `std::partition_point()`：检查分区与查找已有分区边界 / Checking partitions and locating a boundary in an already partitioned range
9. 比较器必须形成严格弱序；避免使用 `<=` 作为排序比较器 / Comparators must induce a strict weak ordering; avoid `<=` as a sorting comparator
10. 稳定性、比较次数、移动次数与额外空间的综合选择 / Choosing algorithms by stability, comparisons, moves, and extra storage
11. `std::list::sort()` 与 `std::forward_list::sort()`：链表使用成员排序 / Sorting linked lists through their member functions
12. `std::min()`、`std::max()`、`std::minmax()` 与 C++17 `std::clamp()`：值选择、范围限制与返回引用的生命周期 / Selecting values, clamping to bounds, and managing the lifetime of returned references

### 第 29 章：二分查找、合并与集合算法 / Chapter 29: Binary Search, Merging, and Set Algorithms

1. `std::lower_bound()`：返回首个不满足 `comp(element, value)` 的位置，输入须按该表达式分区 / Returning the first position where `comp(element, value)` is false in a range partitioned by that expression
2. `std::upper_bound()`：定位查询值之后的边界，注意比较器参数方向及分区前提 / Finding the upper boundary with the required comparison direction and partitioning
3. `std::equal_range()` 与 `std::binary_search()`：等价区间和存在性检查的分区与比较一致性要求 / Partitioning and comparison-consistency requirements for equal ranges and existence checks
4. 按同一比较器有序是常用的充分条件；查找后插入仍需维护顺序 / Sorting by a consistent comparator as a common sufficient condition and preserving order after insertion
5. 二分算法的比较次数为对数级；非随机访问迭代器的移动次数可能为线性级 / Logarithmic comparisons but potentially linear iterator traversal for non-random-access iterators
6. `std::map::lower_bound()` 与 `std::set::lower_bound()`：利用容器结构的成员查找 / Member lookup that uses ordered-container structure
7. `std::merge()`：合并两个有序区间，输出区间不得与输入重叠 / Merging two sorted ranges into a non-overlapping output range
8. `std::inplace_merge()`：合并同一序列中相邻的两个有序区间 / Merging adjacent sorted subranges in place
9. `std::includes()`：检查有序区间的包含关系与重复次数 / Checking sorted-range inclusion, including duplicate multiplicities
10. `std::set_union()`、`std::set_intersection()`、`std::set_difference()` 与 `std::set_symmetric_difference()` / Sorted-range union, intersection, difference, and symmetric difference
11. 集合算法的排序前提、重复元素计数规则与输出区间要求 / Set-algorithm ordering preconditions, duplicate-count rules, and output-range requirements

### 第 30 章：堆与排列算法 / Chapter 30: Heap and Permutation Algorithms

1. 堆的偏序性质、随机访问区间与默认最大堆 / Heap ordering, random-access ranges, and the default max-heap
2. `std::make_heap()`：以线性比较复杂度建立堆 / Building a heap with linear comparison complexity
3. `std::push_heap()`：先向容器末尾追加元素，再维护堆结构 / Appending an element before restoring the heap property
4. `std::pop_heap()`：将堆顶移到区间末尾，随后由容器 `pop_back()` 移除 / Moving the heap top to the range end before removing it through the container
5. `std::push_heap()` 与 `std::pop_heap()` 的已有堆前提和对数比较复杂度 / Existing-heap preconditions and logarithmic comparison complexity
6. `std::sort_heap()`：将已有堆转换为有序区间 / Turning an existing heap into a sorted range
7. `std::is_heap()` 与 `std::is_heap_until()`：堆有效性检查 / Checking the heap property
8. 使用 `std::greater<>` 构建最小堆，并保持各堆操作的比较器一致 / Building a min-heap with `std::greater<>` and using a consistent comparator
9. `std::next_permutation()` 与 `std::prev_permutation()`：按字典序生成排列 / Generating permutations in lexicographical order
10. 排列算法返回 `false` 时重置为边界排列；重复元素与枚举起点 / Resetting to a boundary permutation on `false`, duplicate elements, and enumeration starting points
11. 手动堆算法与 `std::priority_queue`：可控区间与封装接口的选择 / Choosing between explicit heap algorithms and the `std::priority_queue` interface

### 第 31 章：数值算法与随机数 / Chapter 31: Numeric Algorithms and Random Numbers

1. `std::accumulate()`：从左到右累积，初值类型决定累积类型 / Left-to-right accumulation with the initial value determining the accumulator type
2. `std::inner_product()`：内积与自定义乘加操作 / Inner products and custom combination operations
3. `std::partial_sum()` 与 `std::adjacent_difference()`：前缀累计与相邻差分 / Prefix accumulation and adjacent differences
4. `std::iota()`：生成连续递增值 / Filling a range with successively incremented values
5. `std::reduce()` 与 `std::transform_reduce()`（C++17）：允许重排和分组的归约 / Reductions that may reorder and regroup operations in C++17
6. `std::inclusive_scan()`、`std::exclusive_scan()` 及变换版本（C++17） / Inclusive and exclusive scans and their transform variants in C++17
7. 整数溢出、浮点舍入、结合性与归约结果的可复现性 / Integer overflow, floating-point rounding, associativity, and reproducible reduction results
8. `std::gcd()` 与 `std::lcm()`（C++17）：最大公约数、最小公倍数与表示范围前提 / Greatest common divisors, least common multiples, and representability preconditions
9. `<random>`：随机数引擎与分布的分工；`std::mt19937` / Separating random engines from distributions with `std::mt19937`
10. `std::uniform_int_distribution`、`std::uniform_real_distribution` 与 `std::normal_distribution` / Uniform integer, uniform real, and normal distributions
11. `std::random_device`、种子与 `std::seed_seq`：非确定性来源可能受实现限制 / Random devices, seeds, and seed sequences; nondeterminism may depend on the implementation
12. 固定种子与复现：同一引擎的序列规则不等于分布结果跨标准库完全一致 / Fixed seeds and reproducibility: engine-sequence guarantees do not imply identical distribution output across standard libraries

### 第 32 章：并行算法与执行策略 / Chapter 32: Parallel Algorithms and Execution Policies

1. `<execution>` 与支持执行策略的算法重载（C++17） / Execution policies and supported algorithm overloads in C++17
2. `std::execution::seq`：使用策略重载并限制为顺序执行 / Using a policy overload restricted to sequenced execution
3. `std::execution::par`：允许并行执行，不保证创建线程或获得加速 / Permitting parallel execution without guaranteeing threads or speedup
4. `std::execution::par_unseq`：允许并行及未排序执行，适用于满足约束的操作 / Permitting parallel and unsequenced execution for operations meeting the required constraints
5. `std::execution::unseq`（C++20 扩展）：允许未排序执行 / Unsequenced execution as a C++20 extension
6. 并行重载的迭代器要求、输出空间与区间重叠限制 / Iterator requirements, output capacity, and overlap restrictions for parallel overloads
7. 共享状态、数据竞争与回调中的副作用管理 / Shared state, data races, and side effects in callbacks
8. 未排序策略下的向量化安全限制；避免在回调中使用不满足要求的同步操作 / Vectorization-safety restrictions and unsuitable synchronization inside callbacks
9. 使用标准执行策略时，元素访问函数抛出的异常若逃出调用，将调用 `std::terminate()`；该规则也适用于 `seq` / Exceptions escaping element-access function invocations call `std::terminate()` with standard execution policies, including `seq`
10. 为并行化申请临时内存失败可抛出 `std::bad_alloc`；区分该失败与回调异常 / Failure to allocate temporary parallelization storage may throw `std::bad_alloc`; distinguish this from callback exceptions
11. 并行归约的运算顺序与浮点结果变化；与 `std::accumulate()` 的语义比较 / Reduction ordering, floating-point result variation, and comparison with `std::accumulate()` semantics
12. 标准库实现、并行后端与链接依赖的核对；用实际数据规模测量加速效果 / Checking library support, parallel backends, and linking dependencies, then measuring performance at realistic data sizes

---

## 第七篇：内存管理与工程质量 / Part VII: Memory Management and Engineering Quality

### 第 33 章：分配器与未初始化内存 / Chapter 33: Allocators and Uninitialized Memory

1. 存储分配、对象构造、对象销毁与存储释放的区别 / Distinguishing storage allocation, object construction, object destruction, and storage deallocation
2. `std::allocator<T>`：默认分配器与 `allocate()`、`deallocate()` / The default allocator and its allocation and deallocation operations
3. `std::allocator_traits`：统一访问分配器类型、分配操作与对象生命周期操作 / Uniform access to allocator types, allocation operations, and object lifetime operations
4. `std::allocator_traits::construct()` 与 `destroy()`：容器构造和销毁元素的接口 / Allocator-aware interfaces for constructing and destroying container elements
5. `std::allocator::construct()` 与 `destroy()`：C++17 弃用、C++20 移除；`std::allocator_traits` 对应接口继续保留 / Allocator member functions deprecated in C++17 and removed in C++20, while the corresponding allocator traits interfaces remain available
6. `rebind_alloc`、有状态分配器与分配器相等性的含义 / Allocator rebinding, stateful allocators, and the meaning of allocator equality
7. `propagate_on_container_copy_assignment`、`propagate_on_container_move_assignment` 与 `propagate_on_container_swap` / Allocator propagation during container copy assignment, move assignment, and swapping
8. `std::scoped_allocator_adaptor`：嵌套容器中的分配器传递 / Propagating allocators through nested containers
9. `std::uninitialized_copy()`、`uninitialized_copy_n()`、`uninitialized_fill()` 与 `uninitialized_fill_n()` / Copying and filling objects into uninitialized storage
10. `std::uninitialized_move()`、`uninitialized_move_n()`、`uninitialized_default_construct()` 与 `uninitialized_value_construct()`（C++17） / Moving, default-initializing, and value-initializing objects in uninitialized storage in C++17
11. `std::destroy()`、`destroy_n()` 与 `destroy_at()`（C++17），以及 `std::construct_at()`（C++20 扩展） / Explicit object destruction in C++17 and construction at an address in C++20
12. 对齐要求、部分构造失败时的清理，以及分配器和存储释放的配对规则 / Alignment requirements, cleanup after partial construction failure, and matching allocation with deallocation

### 第 34 章：多态分配器与内存资源 / Chapter 34: Polymorphic Allocators and Memory Resources

1. `<memory_resource>` 与 `std::pmr`（C++17）：运行时选择内存分配策略 / Selecting memory allocation strategies at runtime with C++17 polymorphic memory resources
2. `std::pmr::memory_resource`：按字节数和对齐要求分配内存的抽象接口 / An abstract interface for allocating memory by byte count and alignment
3. `std::pmr::polymorphic_allocator<T>`：容器类型与内存资源实现的分离 / Separating container types from memory resource implementations
4. `std::pmr::vector`、`std::pmr::string`、`std::pmr::map` 与 `std::pmr::unordered_map` / Polymorphic allocator aliases for common containers and strings
5. `std::pmr::monotonic_buffer_resource`：阶段性分配、整体释放与上游资源 / Phase-based allocation, bulk release, and upstream resources
6. `std::pmr::unsynchronized_pool_resource`：重复小块分配与外部同步要求 / Repeated small allocations and the need for external synchronization
7. `std::pmr::synchronized_pool_resource`：内存资源内部同步与容器同步的区别 / Distinguishing internal memory resource synchronization from container synchronization
8. `new_delete_resource()`、`null_memory_resource()`、`get_default_resource()` 与 `set_default_resource()` / Standard resources, allocation failure resources, and default resource selection
9. 内存资源、初始缓冲区与容器的生命周期关系：先销毁依赖资源的对象，再释放资源 / Lifetime ordering for resources, initial buffers, and containers: destroy dependent objects before releasing their resource
10. PMR 容器的复制构造与赋值：默认复制构造使用默认资源，赋值保留目标分配器 / PMR container copying: default copy construction uses the default resource, while assignment retains the destination allocator
11. 不同资源之间的移动赋值成本；分配器不相等时不能直接对 PMR 容器调用 `swap()` / Move assignment costs across resources; do not directly swap PMR containers whose allocators compare unequal
12. 嵌套 `std::pmr::vector<std::pmr::string>` 与普通 `std::string` 的分配差异 / Allocation differences between nested PMR strings and ordinary strings in PMR containers

### 第 35 章：智能指针与所有权（标准库补充） / Chapter 35: Smart Pointers and Ownership (Standard Library Supplement)

1. 智能指针属于标准库资源管理设施，本章作为狭义 STL 容器与算法之外的补充 / Smart pointers are standard library resource management facilities, covered here as a supplement to STL containers and algorithms
2. RAII、独占所有权、共享所有权与非拥有型观察 / RAII, exclusive ownership, shared ownership, and non-owning observation
3. `std::unique_ptr` 与 `std::make_unique()`：独占所有权及移动转移 / Exclusive ownership and ownership transfer through moves
4. `std::unique_ptr<T[]>`、自定义删除器、`get()`、`release()` 与 `reset()` / Array ownership, custom deleters, observation, ownership release, and replacement
5. `std::shared_ptr` 与 `std::make_shared()`：共享所有权、引用计数与控制块 / Shared ownership, reference counting, and control blocks
6. `std::allocate_shared()`、对象与控制块的分配策略，以及弱引用对控制块生命周期的影响 / Allocation strategies for shared objects and control blocks, and the effect of weak references on control block lifetime
7. `std::weak_ptr`、`expired()` 与 `lock()`：通过 `lock()` 获取有效共享所有权 / Weak observation and obtaining valid shared ownership through `lock()`
8. 双向关系与循环引用：使用 `std::weak_ptr` 表达不拥有对象的反向连接 / Breaking ownership cycles with weak back-references
9. `std::enable_shared_from_this` 与 `shared_from_this()`：复用已有控制块，避免对同一裸指针重复建立所有权 / Reusing an existing control block and avoiding duplicate ownership of the same raw pointer
10. 智能指针容器、按值传参与按引用传参，以及多态对象的安全销毁 / Containers of smart pointers, parameter passing, and safe destruction of polymorphic objects
11. 共享控制块支持不同 `std::shared_ptr` 实例的并发操作，不代表被管理对象可以无同步读写 / Concurrent operations on distinct shared pointer instances sharing a control block do not make the managed object safe for unsynchronized access
12. 同一 `std::shared_ptr` 实例的并发写入：C++17 原子自由函数与 C++20 `std::atomic<std::shared_ptr<T>>` 扩展 / Concurrent writes to the same shared pointer instance using C++17 atomic free functions or the C++20 atomic shared pointer specialization

### 第 36 章：异常安全与元素类型要求 / Chapter 36: Exception Safety and Element Type Requirements

1. 无抛出保证、强异常保证与基本异常保证 / No-throw, strong, and basic exception safety guarantees
2. 按具体操作阅读异常保证：构造、插入、擦除、赋值与交换 / Reading exception guarantees for construction, insertion, erasure, assignment, and swapping separately
3. 容器按操作提出的可构造、可复制、可移动、可赋值与可销毁要求 / Operation-specific container requirements for construction, copying, moving, assignment, and destruction
4. `CopyInsertable`、`MoveInsertable` 与分配器参与构造的关系 / How allocator-aware construction affects copy-insertion and move-insertion requirements
5. `noexcept` 移动构造对 `std::vector` 扩容时元素搬迁与异常保证的影响 / How non-throwing move construction affects element relocation and exception guarantees during vector growth
6. 可复制但移动可能抛出的元素：实现通常复制旧元素以维持强保证；仅能移动且移动抛出时应检查具体操作的保证 / Copyable elements with potentially throwing moves are commonly copied during reallocation to preserve strong guarantees; inspect operation guarantees for throwing moves of move-only elements
7. `std::move_if_noexcept()` 与 `std::is_nothrow_move_constructible_v` / Conditional move selection and compile-time detection of non-throwing move construction
8. 移动后的标准库对象通常有效但状态未指定；继续操作时仍须满足该操作的前置条件 / Moved-from standard library objects are generally valid but have unspecified state; subsequent operations must still satisfy their preconditions
9. Rule of Zero、资源封装与析构函数的异常边界 / The Rule of Zero, resource encapsulation, and exception boundaries in destructors
10. 比较器、哈希函数、相等谓词与算法回调的语义要求和抛异常行为 / Semantic requirements and exception behavior of comparators, hash functions, equality predicates, and algorithm callbacks
11. 通过临时对象构造后提交、`swap()` 与回滚设计维护对象不变量 / Preserving invariants through temporary construction, commit operations, swapping, and rollback design
12. 分配失败、元素构造失败与边界输入的验证 / Checking behavior under allocation failure, element construction failure, and boundary inputs

### 第 37 章：性能测试与线程安全 / Chapter 37: Performance Measurement and Thread Safety

1. 渐进复杂度、常数成本、缓存局部性与实际数据规模 / Asymptotic complexity, constant costs, cache locality, and realistic data sizes
2. `std::chrono::steady_clock`：测量耗时与区分时钟精度 / Measuring elapsed time and understanding clock resolution
3. Release 构建、预热、重复采样、结果分布与避免无用计算被优化掉 / Release builds, warm-up, repeated sampling, result distributions, and preventing dead-code elimination
4. 固定输入分布、记录编译器和标准库版本，以及保持对比条件一致 / Fixing input distributions, recording compiler and standard library versions, and controlling comparison conditions
5. 分别测量查找、插入、删除、遍历、构造与析构成本 / Measuring lookup, insertion, erasure, traversal, construction, and destruction separately
6. `reserve()`、批量操作、分配次数与元素复制移动次数 / Capacity reservation, bulk operations, allocation counts, and element copy and move counts
7. 容器线程安全的基本边界：不同容器对象的操作与同一对象上的并发访问 / Basic container thread-safety boundaries for separate objects and concurrent access to the same object
8. 并发只读访问与必须同步的结构修改：插入、删除、扩容和重新哈希 / Concurrent read-only access and synchronization for structural changes such as insertion, erasure, growth, and rehashing
9. 同一容器不同元素内容的并发修改保证，以及 `std::vector<bool>` 的例外 / Concurrent modification of different contained elements and the `std::vector<bool>` exception
10. 迭代器失效与数据竞争的区别，以及外部 `std::mutex` 和锁作用域 / Distinguishing iterator invalidation from data races, and using external mutexes with appropriate lock scopes
11. 并行算法中的共享状态、回调同步与执行策略限制 / Shared state, callback synchronization, and execution-policy restrictions in parallel algorithms
12. 地址检查、未定义行为检查、线程检查工具及其平台支持范围 / Address, undefined-behavior, and thread checking tools and their platform support

---

## 第八篇：现代 STL 扩展 / Part VIII: Modern STL Extensions

> 本篇第 38—40 章为 **C++20 扩展**，第 41 章为 **C++23 扩展**；使用前应检查编译器、标准库版本及相应特性测试宏。\
> Chapters 38–40 cover **C++20 extensions**, while Chapter 41 covers **C++23 extensions**; check compiler and standard library versions and the relevant feature-test macros before use.

### 第 38 章：Concepts 与 Ranges 算法（C++20） / Chapter 38: Concepts and Ranges Algorithms (C++20)

1. `<concepts>`、`concept` 与 `requires`：表达模板接口的约束 / Expressing template interface constraints with concepts and requires clauses
2. `std::same_as`、`std::convertible_to`、`std::regular` 与 `std::strict_weak_order` / Common type, conversion, regularity, and ordering concepts
3. `<ranges>` 与 `std::ranges::range`：从迭代器对到范围抽象 / Moving from iterator pairs to the range abstraction
4. `input_range`、`forward_range`、`bidirectional_range`、`random_access_range` 与 `contiguous_range` / Range categories and their supported traversal capabilities
5. `sized_range`、`common_range`、迭代器与哨兵类型 / Sized ranges, common ranges, and iterator and sentinel types
6. `std::ranges::begin()`、`end()`、`size()`、`distance()` 与范围访问 / Range access and distance measurement
7. `std::ranges::find()`、`count_if()`、`copy()` 与接受整个范围的算法重载 / Range-based overloads for searching, counting, and copying
8. `std::ranges::sort()`、`stable_sort()` 与 `lower_bound()` 的概念要求 / Concept requirements for sorting and binary search algorithms
9. 投影参数与成员指针：按对象字段查找、比较和排序 / Projections and member pointers for searching, comparing, and sorting by object fields
10. `std::ranges::in_out_result` 等结果类型与算法返回信息 / Structured algorithm result types and returned iterator information
11. `borrowed_range`、`borrowed_iterator_t` 与 `std::ranges::dangling`：范围销毁后的迭代器有效性 / Borrowed ranges, conditional iterator return types, and iterator validity after range destruction
12. 传统算法与 Ranges 算法的迁移：约束、返回类型及 C++20 Ranges 算法不提供执行策略重载 / Migrating classic algorithms while accounting for constraints, return types, and the absence of execution-policy overloads in C++20 ranges algorithms

### 第 39 章：Views 与范围管道（C++20） / Chapter 39: Views and Range Pipelines (C++20)

1. `std::ranges::view`、`viewable_range` 与 `std::views` 命名空间 / Views, adaptable ranges, and the views namespace
2. `std::views::all`、`std::ranges::ref_view` 与 `std::ranges::owning_view`：引用或拥有基础范围，`owning_view` 随 C++20 缺陷修正补入 / Referencing or owning an underlying range; `owning_view` was added through a C++20 defect correction
3. View 不保证非拥有，也不保证所有操作惰性；所有权、借用性质与求值时机分别判断 / A view is not necessarily non-owning or entirely lazy; assess ownership, borrowing, and evaluation timing independently
4. `std::views::filter` 与 `std::views::transform`：按需筛选和变换 / Filtering and transforming elements on demand
5. `std::views::take`、`drop`、`take_while` 与 `drop_while`：截取范围 / Selecting range prefixes and suffixes
6. `std::views::iota`、`single` 与 `empty`：生成数字范围、单元素范围和空范围 / Generating numeric, single-element, and empty ranges
7. `std::views::reverse`、`keys`、`values` 与 `elements`：调整遍历和投影元组元素 / Reversing traversal and selecting tuple elements
8. `std::views::split`、`join` 与 `common`：拆分、展开和适配公共迭代器类型 / Splitting, flattening, and adapting ranges to a common iterator type
9. 管道运算符 `|`、适配器组合与中间视图的类型 / Pipeline composition and intermediate view types
10. 基础容器、视图和谓词捕获对象的生命周期，以及容器修改后的失效规则 / Lifetimes of underlying containers, views, and captured objects, and invalidation after container modification
11. 重复遍历中的重复计算、状态缓存、谓词语义要求与单次遍历范围 / Repeated computation, cached state, predicate requirements, and single-pass ranges
12. C++20 中通过迭代器或 `std::ranges::copy()` 将结果写入容器；`std::ranges::to()` 需 C++23 / Materializing results with iterators or range copying in C++20; `std::ranges::to()` requires C++23

> View 的所有权规则及 `owning_view` 补充可参阅 [WG21 P2415R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2415r2.html)；作为 C++20 缺陷修正采纳的记录见 [N4902](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/n4902.html)。\
> For view ownership rules and the addition of `owning_view`, see [WG21 P2415R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2415r2.html); its adoption as a C++20 defect correction is recorded in [N4902](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/n4902.html).

### 第 40 章：span 与容器便利接口（C++20） / Chapter 40: span and Container Convenience Interfaces (C++20)

1. `<span>` 与 `std::span<T>`：连续元素序列的非拥有型视图 / A non-owning view over a contiguous sequence of elements
2. 静态范围长度、`std::dynamic_extent`、数组与容器构造 / Static extents, dynamic extents, and construction from arrays and containers
3. `std::span<T>` 与 `std::span<const T>`：视图自身的常量性与元素只读性 / Distinguishing constness of the view object from read-only access to elements
4. `data()`、`size()`、`size_bytes()`、`empty()` 与连续数据访问 / Accessing contiguous data and inspecting element and byte counts
5. `first()`、`last()` 与 `subspan()`：切分子范围及长度前置条件 / Selecting subranges and respecting extent preconditions
6. `std::as_bytes()` 与 `std::as_writable_bytes()`：对象表示的字节视图 / Byte views of object representations
7. `std::span` 不延长底层对象生命周期；C++20/23 的 `operator[]` 不提供自动越界检查 / Spans do not extend underlying object lifetimes, and C++20/23 indexing does not provide automatic bounds checking
8. `std::erase()` 与 `std::erase_if()`：适用于 `vector`、`deque`、`list`、`forward_list` 和 `basic_string` 的自由函数 / Free erase-by-value and erase-by-predicate functions for supported sequence containers and strings
9. 关联及无序关联容器提供 `std::erase_if()`；按键删除使用成员 `erase()`，固定长度 `std::array` 没有上述擦除接口 / Associative and unordered containers provide free erase-by-predicate functions and member key erasure; fixed-size arrays have no such erasure interfaces
10. 有序及无序关联容器的 `contains()`：判断键是否存在 / Testing key presence with associative and unordered container membership functions
11. `std::to_array()`、`std::ssize()` 与 `std::string::starts_with()`、`ends_with()` / Converting arrays, retrieving signed sizes, and testing string prefixes and suffixes
12. `std::less<>` 与透明哈希、透明相等谓词：C++20 无序容器的异构查找与既有有序容器接口 / Transparent comparison, hashing, and equality for C++20 unordered heterogeneous lookup and existing ordered lookup interfaces

### 第 41 章：范围与容器扩展（C++23） / Chapter 41: Range and Container Extensions (C++23)

1. `std::ranges::to()`：将满足要求的范围实体化为指定容器，支持管道组合 / Materializing compatible ranges into target containers with optional pipeline composition
2. `std::from_range` 与 `std::from_range_t`：支持范围构造的容器标记接口 / Tagged construction for containers that support range inputs
3. `assign_range()`、`insert_range()` 与 `append_range()`：`vector` 等容器的范围赋值、插入和追加；`forward_list` 使用 `insert_range_after()`，且没有 `append_range()` / Range assignment, insertion, and appending in containers such as vector; forward lists use `insert_range_after()` and do not provide `append_range()`
4. `prepend_range()`、关联容器的 `insert_range()` 与容器适配器的 `push_range()`：按容器能力选择接口 / Selecting front insertion, associative insertion, and adaptor push operations according to container capabilities
5. `std::views::zip`、`zip_transform`、`adjacent` 与 `adjacent_transform` / Combining ranges and transforming adjacent element groups
6. `std::views::chunk`、`slide`、`stride` 与 `chunk_by` / Chunking, sliding windows, strided traversal, and predicate-based grouping
7. `std::views::join_with`、`cartesian_product`、`enumerate`、`repeat`、`as_const` 与 `as_rvalue` / Joining with separators, Cartesian products, indexed traversal, repetition, const views, and rvalue views
8. `std::ranges::contains()`、`contains_subrange()`、`starts_with()`、`ends_with()` 与 `fold_left()` / Range membership, subrange tests, prefix and suffix tests, and left folds
9. `<flat_map>` 中的 `std::flat_map`、`std::flat_multimap`：基于键和值容器的有序关联适配器 / Ordered associative adaptors backed by separate key and value containers
10. `<flat_set>` 中的 `std::flat_set`、`std::flat_multiset`：基于序列容器的有序集合适配器 / Ordered set adaptors backed by sequence containers
11. Flat 容器的默认 `std::vector` 存储、对数查找、通常为线性的插入删除成本，以及迭代器失效 / Default vector storage, logarithmic lookup, generally linear insertion and erasure costs, and iterator invalidation in flat containers
12. `__cpp_lib_ranges_to_container`、`__cpp_lib_containers_ranges`、`__cpp_lib_flat_map` 与 `__cpp_lib_flat_set`：按具体功能检查标准库支持 / Checking library support for range conversion, container range interfaces, and flat containers with feature-test macros

> 范围到容器的接口设计可参阅 [WG21 P1206R7](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p1206r7.pdf)；最终 C++23 接口以 [C++23 工作草案 N4950](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/n4950.pdf) 为参考。\
> For range-to-container interface design, see [WG21 P1206R7](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p1206r7.pdf); consult the [C++23 working draft N4950](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/n4950.pdf) for the final C++23 interfaces.

---

## 第九篇：综合实践与常见问题 / Part IX: Practical Projects and Common Problems

### 第 42 章：文本统计与数据查询 / Chapter 42: Text Statistics and Data Queries

1. 单词频次统计：`string`、`unordered_map` 与输入清洗 / Word-frequency counting with strings, unordered maps, and input normalization
2. 统计结果排名：转换为 `vector<pair<...>>` 并按频次与单词排序 / Ranking counts with a vector of pairs and deterministic tie-breaking
3. Top-K 热词：完整排序、`partial_sort()` 与有界堆的取舍 / Choosing full sorting, partial sorting, or bounded heaps for Top-K words
4. 有序通讯录：`map`、唯一键、更新、删除与范围查询 / An ordered contact directory with unique keys, updates, removal, and range queries
5. 一对多索引：`multimap` 与 `map<Key, vector<Value>>` / One-to-many indexes using multimaps or mapped vectors
6. 集合运算：交集、并集、差集与一致的排序规则 / Set intersection, union, and difference with consistent ordering
7. 日志解析：`string_view` 切片、`from_chars()` 与源缓冲区有效期 / Log parsing with string views, character conversion, and source-buffer lifetime
8. 批量数据清洗：筛选、转换、排序去重与统计汇总 / Batch filtering, transformation, deduplication, and aggregation
9. 空输入、重复键、非法数字、超长行与 UTF-8 数据边界 / Handling empty input, duplicate keys, invalid numbers, long lines, and UTF-8 boundaries
10. 从命令式循环逐步重构为 STL 算法与可组合函数 / Refactoring loops into STL algorithms and composable functions

### 第 43 章：缓存、调度与图算法 / Chapter 43: Caching, Scheduling, and Graph Algorithms

1. LRU 缓存：`list` 保存访问顺序、`unordered_map` 保存节点位置 / An LRU cache with list ordering and hash-based node lookup
2. 使用 `splice()` 更新访问顺序，维护缓存容量和淘汰规则 / Updating recency through node splicing and maintaining eviction rules
3. 缓存类复制、移动后内部迭代器关系的重建或操作限制 / Rebuilding internal iterator relationships after cache copying or moving, or restricting those operations
4. 优先级任务调度：`priority_queue`、自定义比较器与序号打破平局 / Priority scheduling with custom comparators and sequence-based tie-breaking
5. 广度优先搜索：`queue`、邻接表与访问标记 / Breadth-first search with queues, adjacency lists, and visited flags
6. 深度优先遍历：递归调用与显式 `stack` / Depth-first traversal through recursion or explicit stacks
7. 非负边权最短路径：小顶堆、距离表与过期条目跳过 / Shortest paths with nonnegative weights, a min-heap, distance tables, and stale-entry checks
8. 区间合并：`vector`、排序与线性扫描 / Interval merging through sorting and linear scanning
9. 撤销与重做：栈结构、命令对象与资源生命周期 / Undo and redo with stacks, command objects, and resource lifetime
10. 数据结构不变量、复杂度分析与边界案例验证 / Checking data-structure invariants, complexity, and boundary cases

### 第 44 章：工程组织、调试与 Qt 互操作 / Chapter 44: Project Organization, Debugging, and Qt Interoperability

1. 按章节组织 CMake 目标、头文件依赖与编译标准 / Organizing CMake targets, header dependencies, and language versions by chapter
2. Debug 与 Release、编译器告警和标准库调试模式 / Build configurations, compiler warnings, and standard-library debug modes
3. MSVC 迭代器调试、libstdc++ 调试模式与构建配置一致性 / Iterator debugging, libstdc++ debug mode, and consistent build settings
4. 在工具链支持时使用 AddressSanitizer、UndefinedBehaviorSanitizer 与 ThreadSanitizer / Using supported memory, undefined-behavior, and thread sanitizers
5. 测试空区间、单元素、重复值、极值、只移动类型与抛异常类型 / Testing empty and singleton ranges, duplicates, extremes, move-only types, and throwing types
6. Qt 项目中的 STL：标准库代码独立编译，Qt 互操作示例采用 Qt 6.5.3 / Building standard-library code independently and using Qt 6.5.3 for interoperability examples
7. `QString`、`QByteArray` 与 `std::string`：显式编码和长度转换 / Explicit encoding and length conversion between Qt and standard strings
8. `QList`、`QVector` 与 `std::vector`：范围转换、元素复制与迭代器有效期 / Converting Qt and standard containers while respecting copies and iterator lifetime
9. QObject 父子所有权与智能指针管理的边界 / Coordinating QObject parent ownership with smart-pointer ownership
10. Qt 隐式共享、STL 元素复制与共享指针复制的语义差异 / Distinguishing Qt implicit sharing, STL element copies, and shared-pointer copies
11. 跨线程传递容器时的对象有效期、独占访问与同步 / Lifetime, exclusive access, and synchronization when passing containers across threads
12. 记录编译器、标准库实现、标准版本与实际性能数据 / Recording compiler and library versions, language mode, and measured performance

> Qt 互操作中的字符串编码与容器接口可参阅 [Qt 6.5 QString 文档](https://doc.qt.io/qt-6.5/qstring.html)和 [Qt 6.5 QList 文档](https://doc.qt.io/qt-6.5/qlist.html)。\
> For string encoding and container interfaces used in Qt interoperability, see the [Qt 6.5 QString reference](https://doc.qt.io/qt-6.5/qstring.html) and [Qt 6.5 QList reference](https://doc.qt.io/qt-6.5/qlist.html).

---

## 附录 / Appendices

1. STL 常用头文件与组件速查：`<vector>`、`<map>`、`<algorithm>`、`<iterator>`、`<functional>` 等 / Quick reference for common STL headers and components
2. 顺序容器、有序关联容器、无序关联容器与适配器选型表 / Selection guide for sequence, ordered, unordered, and adapter containers
3. 常用容器访问、查找、插入和删除的复杂度对照 / Complexity reference for access, lookup, insertion, and erasure
4. 迭代器类别、可用操作与算法最低要求 / Iterator categories, supported operations, and minimum algorithm requirements
5. 容器操作对迭代器、引用、指针及 `end()` 的失效规则速查 / Invalidation reference for iterators, references, pointers, and past-the-end positions
6. `size()`、`capacity()`、`reserve()`、`resize()` 与 `shrink_to_fit()` 对照 / Reference for size, capacity, reservation, resizing, and capacity reduction
7. `insert()`、`emplace()`、`try_emplace()` 与 `insert_or_assign()` 对照 / Comparing insertion, emplacement, conditional emplacement, and insertion-or-assignment
8. `remove()`、`erase()`、成员 `remove_if()` 与 C++20 `std::erase_if()` 对照 / Comparing logical removal, physical erasure, member removal, and erase_if()
9. 比较器严格弱序、哈希与相等谓词一致性检查 / Checking strict weak ordering and hash-equality consistency
10. `std::move()`、`std::forward()`、移动算法与移动迭代器对照 / Comparing move casts, forwarding, move algorithms, and move iterators
11. `string_view`、`span` 与 Ranges 视图的所有权和有效期速查 / Ownership and lifetime reference for string views, spans, and range views
12. C++11 / C++14 / C++17 / C++20 / C++23 特性与功能测试宏索引 / Feature and feature-test-macro index across language and library versions
13. 常见未定义行为、异常、断言失败与编译错误定位 / Diagnosing undefined behavior, exceptions, assertion failures, and compilation errors
14. 各章练习输入、预期输出、复杂度目标与可复现性能测试模板 / Exercise inputs, expected outputs, complexity targets, and reproducible benchmarking templates

### 文档与标准资料 / Documentation and Standards Resources

1. [Microsoft C++ 标准库参考 / Microsoft C++ Standard Library Reference](https://learn.microsoft.com/en-us/cpp/standard-library/cpp-standard-library-reference?view=msvc-170)：查询组件、头文件与 MSVC 实现说明 / Component, header, and implementation reference
2. [Microsoft STL 算法说明 / Microsoft STL Algorithm Conventions](https://learn.microsoft.com/en-us/cpp/standard-library/algorithms?view=msvc-170)：理解迭代器区间与算法文档约定 / Iterator ranges and algorithm documentation conventions
3. [C++17 工作草案 N4659 / C++17 Working Draft N4659](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2017/n4659.pdf)：核对本教程基线中的标准要求 / Requirements corresponding to this tutorial's baseline
4. [C++ 工作草案 / C++ Working Draft](https://eel.is/c++draft/)：查询标准条款；在线草案会更新，新增内容需核对标准版本 / Standard wording, with version checks required for evolving draft content
