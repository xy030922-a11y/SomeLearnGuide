# C++ 并发与多线程指南

> 本文是 [C++ Guide](C++Guide.md) 的并发专题，以 **C++17** 为编译基线，并注明 C++11、C++14、C++20 的版本边界。学习目标是管理线程生命周期、保护共享状态、传递结果与错误，并设计能够正常结束的后台工作。

前置知识见 [语言核心](C++CoreLanguageGuide.md) 与 [现代语言特性](C++ModernFeaturesGuide.md)；容器与智能指针见 [标准库](C++StandardLibraryGuide.md)。构建、测试和诊断见 [构建与工程实践](C++BuildEngineeringGuide.md)，操作系统接口见 [系统编程](C++SystemProgrammingGuide.md)。

**示例约定：** 三个“完整程序”分别保存为独立的 `main.cpp`。片段需要补齐注明的头文件和上下文。并发顺序通常不确定，不要用一次输出或一次运行时间证明线程安全。

## 阅读导航

1. [并发、并行与共享状态](#concepts)
2. [线程生命周期与异常传播](#threads)
3. [互斥锁与锁对象](#locks)
4. [完整示例：安全回收线程](#join-example)
5. [数据竞争与 happens-before](#memory-model)
6. [原子操作与内存序](#atomics)
7. [条件变量与谓词](#conditions)
8. [future、promise 与 async](#futures)
9. [完整示例：异步返回结果](#async-example)
10. [完整示例：可关闭的队列](#queue-example)
11. [线程池与生产者消费者](#pool)
12. [C++20 的协作式停止](#stop)
13. [死锁、所有权与验证](#review)

<a id="concepts"></a>
## 1. 并发、并行与共享状态

并发表示多个任务的执行过程可以重叠；并行表示多个任务在同一时刻实际执行。增加线程有调度、同步和缓存开销，任务太小或共享争用太多时，程序可能更慢。

| 功能 | 最低标准 | 主要用途 |
| --- | --- | --- |
| `thread`、`mutex`、`lock_guard`、`unique_lock` | C++11 | 创建线程，保护共享状态 |
| `condition_variable`、`future`、`promise`、`async`、`atomic` | C++11 | 等待条件，传递结果，原子访问 |
| `shared_timed_mutex` | C++14 | 共享读锁、独占写锁与定时锁定 |
| `scoped_lock`、`shared_mutex` | C++17 | 管理多个锁，提供共享读写锁 |
| `jthread`、`stop_token`、原子等待与通知 | C++20 | 管理线程退出与协作式停止 |

开始设计时先回答：任务拥有哪些数据，哪些对象被共享，谁可以修改，谁负责停止与等待？能让任务拥有独立输入和结果时，优先采用这种结构。

<a id="threads"></a>
## 2. 线程生命周期与异常传播

`std::thread` 构造成功后，工作函数可以开始执行。参数通常经过衰减后复制或移动到线程的内部存储；确实需要引用语义时可用 `<functional>` 的 `std::ref`，并确保对象活到线程结束。

- `join()` 等待线程结束，并建立线程完成与等待方之间的同步关系。
- `joinable()` 表示对象仍关联一个需要管理的线程；工作函数已经返回，线程对象也可能仍然 `joinable()`。
- `detach()` 放弃该线程的直接等待管理，不延长被引用局部对象的生命。
- `std::thread` 不能复制，可以移动；向仍然 `joinable()` 的线程对象进行移动赋值会终止程序。
- 析构时仍然 `joinable()` 的 `std::thread` 会调用 `std::terminate`，见 [C++17 线程析构条款](https://timsong-cpp.github.io/cppwp/n4659/thread.thread.destr)。

不要只在正常路径末尾写 `join()`：创建第二个线程、分配内存或其他操作失败时，已创建的第一个线程也必须回收。把等待责任放进 RAII 对象，比在各个返回分支补写等待更可靠。

普通线程工作函数中逃逸的异常不会自动传到创建线程，通常会终止程序。需要在工作函数最外层捕获，保存 `std::exception_ptr`，等待完成后用 `std::rethrow_exception` 重新抛出；也可采用 future 传递错误。

<a id="locks"></a>
## 3. 互斥锁与锁对象

| 工具 | 使用方式 | 注意事项 |
| --- | --- | --- |
| `std::mutex` | 同一时刻允许一个线程持锁 | 所有相关读写都要遵循同一保护规则 |
| `std::lock_guard<Mutex>`，C++11 | 构造时上锁，析构时解锁 | 适合简单的作用域临界区 |
| `std::unique_lock<Mutex>`，C++11 | 可移动，支持延迟加锁、解锁与重锁 | 条件变量等待通常需要它 |
| `std::scoped_lock`，C++17 | 在一个作用域管理一个或多个锁 | 多锁构造采用避免死锁的锁定算法 |

**片段，C++17；需要 `<mutex>`，位于函数体中，两个互斥锁必须是不同对象：**

```cpp
std::mutex firstMutex;
std::mutex secondMutex;
{
    std::scoped_lock lock(firstMutex, secondMutex);
    // 同时受这两个锁保护的短小操作
}
```

保护“余额不能为负”等多字段不变式时，检查与更新应处于同一临界区。即使两个字段各自是原子变量，也不能自动保证它们的组合状态一致。

尽量缩短持锁时间。不要持锁等待另一个需要此锁的任务结束，也不要轻易持锁调用未知回调、耗时 I/O 或外部库。读写锁适合某些读多写少的场景，但公平性与性能取决于实现和负载。

<a id="join-example"></a>
## 4. 完整示例：安全回收线程

**完整程序，最低 C++11。** 两个线程共增加计数器 2,000 次；每个线程使用独立错误槽位，主线程在等待结束后检查错误。

```cpp
#include <exception>
#include <iostream>
#include <mutex>
#include <thread>
#include <utility>

class JoiningThread {
public:
    explicit JoiningThread(std::thread&& thread) noexcept
        : thread_(std::move(thread)) {}
    JoiningThread(const JoiningThread&) = delete;
    JoiningThread& operator=(const JoiningThread&) = delete;

    ~JoiningThread() noexcept {
        if (thread_.joinable()) {
            try {
                thread_.join();
            } catch (...) {
                std::terminate();
            }
        }
    }

private:
    std::thread thread_;
};

int main() {
    try {
        int counter = 0;
        std::mutex mutex;
        std::exception_ptr errors[2];
        auto work = [&](int index) {
            try {
                for (int i = 0; i < 1000; ++i) {
                    std::lock_guard<std::mutex> lock(mutex);
                    ++counter;
                }
            } catch (...) {
                errors[index] = std::current_exception();
            }
        };
        {
            JoiningThread first{std::thread(work, 0)};
            JoiningThread second{std::thread(work, 1)};
        }
        for (const auto& error : errors) {
            if (error) {
                std::rethrow_exception(error);
            }
        }
        std::cout << counter << '\n'; // 2000
    } catch (const std::exception& error) {
        std::cerr << error.what() << '\n';
        return 1;
    }
}
```

如果第二次创建线程失败，`first` 仍会在异常展开时等待。被引用的计数器、互斥锁和错误槽位都声明在外层，活到两个线程完成以后。

这个教学包装器不公开底层线程，也不支持转移自身；仅在拥有它的主线程中销毁，避免等待自己。析构不向外抛异常；若有效所有权下的等待仍出现无法恢复的系统错误，这里明确选择终止，不能靠分离线程后继续销毁共享数据来“恢复”。

<a id="memory-model"></a>
## 5. 数据竞争与 happens-before

两个线程访问同一内存位置，至少一个访问会修改、至少一个不是原子访问，且这些冲突访问之间没有所需的 happens-before 关系，就构成数据竞争，导致未定义行为。多次测试碰巧成功也不能使这种程序合法。

这里的 happens-before 是语言规定的先后关系，不是根据日志时间或观察到的执行速度推断。常见建立方式包括：

- 一个线程解锁互斥锁，另一个线程随后成功取得同一个锁。
- 工作线程完成，等待它的 `join()` 成功返回。
- release 原子写入与读到相应发布值的 acquire 原子读取。

关系可以与线程内部的执行顺序组合，说明先前写入何时对之后的读取可见。数据竞争与同步关系的正式定义见 [C++17 内存模型条款](https://timsong-cpp.github.io/cppwp/n4659/intro.races)。

`volatile` 不提供线程同步。`sleep_for()`、`yield()` 也不建立共享数据的安全访问规则。对象寿命结束与仍在运行的任务之间同样需要协调，不能只关注整数读写。

<a id="atomics"></a>
## 6. 原子操作与内存序

**片段，C++11；需要 `<atomic>`：**

```cpp
std::atomic<unsigned long long> completed{0};
completed.fetch_add(1, std::memory_order_relaxed);
auto snapshot = completed.load(std::memory_order_relaxed);
```

这适合独立统计计数，且读取者不通过计数推断其他数据已经发布。`load()` 后计算再 `store()` 是两步操作，可能丢失别人的更新；需要原子增量时使用 `fetch_add()` 等读改写接口。

| 内存序 | 常见作用 | 使用边界 |
| --- | --- | --- |
| `seq_cst`，默认 | 为相应原子操作提供较易推理的全序规则 | 不把多次操作变成事务，也不自动保护其他数据 |
| `relaxed` | 保证该原子对象的访问原子性与相应一致性 | 不发布其他普通对象的写入 |
| `release` / `acquire` | 配合发布与读取共享数据 | 读取必须观察到相应发布值或满足发布序列规则 |
| `acq_rel` | 读改写操作兼有获取和发布能力 | 不能直接用于普通 `load()` 或 `store()` |

`load()` 不能使用 `release`，`store()` 不能使用 `acquire`。比较交换的成功与失败内存序也有约束；`compare_exchange_weak` 可能虚假失败，通常需要循环并正确处理被更新的 `expected`。

除非已有完整的正确性论证，先用默认内存序或锁。不要因为一次性能测试就把同步改成 `relaxed`。原子类型不保证一定无锁；“无锁”也不等于“每个线程都不会饥饿”。内存序规则见 [C++17 原子顺序条款](https://timsong-cpp.github.io/cppwp/n4659/atomics.order)。

<a id="conditions"></a>
## 7. 条件变量与谓词

条件变量用于等待“队列非空”“任务结束”等共享状态变化。`wait(lock, predicate)` 会检查谓词；不满足时释放锁并等待，醒来后重新取得锁再检查。

必须使用谓词或等价的循环：通知不保证条件成立，等待也可能发生虚假唤醒。条件变量不保存一条可供后来者消费的“通知消息”；真正的状态应保存在受同一互斥锁保护的数据中。

- 状态检查与状态修改都遵循同一互斥锁规则；同时等待同一 `condition_variable` 的线程使用同一互斥锁。
- `notify_one()` 唤醒一个等待者，`notify_all()` 通知所有等待者；最终仍由谓词决定能否继续。
- 先修改状态，释放锁后再通知是常见方式。通知时持锁也可以正确，但醒来的线程可能马上再次等锁。
- 定时等待也要检查结果与谓词；循环重试时考虑固定截止时间，避免每次重试重新获得完整超时时间。
- 销毁条件变量、队列和锁之前，确保所有使用它们的线程已经退出。

<a id="futures"></a>
## 8. future、promise 与 async

`promise<T>` 写入结果或异常，配套的 `future<T>` 等待并取得结果。`get()` 会重新抛出保存的异常，并消耗普通 future 的结果；需要多个读取者时考虑 `shared_future<T>`。

显式使用 promise 时，一个结果只能成功设置一次；提供者未设置结果就销毁时，等待方通常会收到 `broken_promise` 错误。`packaged_task` 可把可调用对象与 future 结果通道组合起来。

| `std::async` 启动策略 | 行为 |
| --- | --- |
| `std::launch::async` | 异步执行；无法启动线程时可能抛出异常 |
| `std::launch::deferred` | 延迟到第一次非定时等待，在等待者线程中执行 |
| 不指定策略 | 实现可以选择异步或延迟执行 |

定时等待延迟任务时可能返回 `future_status::deferred`，不能只区分 ready 与 timeout。不要把默认 `async` 当作保证并行，也不要把它当作可控制工作线程数量的线程池。

释放由 `async` 异步任务产生的最后一个共享状态引用时，可能等待任务完成。丢弃临时 future 可能使看似异步的调用立即等待；持锁销毁这类 future 也可能造成死锁。不是所有 future 的析构都会等待。[C++17 async 规则](https://timsong-cpp.github.io/cppwp/n4659/futures.async)

<a id="async-example"></a>
## 9. 完整示例：异步返回结果

**完整程序，最低 C++11。** 输入范围限制保证示例的整数运算不会溢出；任务异常通过 `get()` 传回。

```cpp
#include <future>
#include <iostream>
#include <stdexcept>

long long sumSquares(long long count) {
    if (count < 0 || count > 100000) {
        throw std::invalid_argument("count out of range");
    }
    long long result = 0;
    for (long long i = 1; i <= count; ++i) {
        result += 1LL * i * i;
    }
    return result;
}

int main() {
    try {
        auto result = std::async(std::launch::async, sumSquares, 1000);
        std::cout << result.get() << '\n'; // 333833500
    } catch (const std::exception& error) {
        std::cerr << error.what() << '\n';
        return 1;
    }
}
```

可以把参数改成 `-1` 验证错误传播。本例只有独立输入与返回结果，不需要用互斥锁保护计算过程。

<a id="queue-example"></a>
## 10. 完整示例：可关闭的队列

**完整程序，最低 C++17，因为使用 `std::optional`。** 主线程生产 1 到 100，异步线程消费；队列关闭后继续取完已有元素，关闭且为空时返回无值。

```cpp
#include <condition_variable>
#include <deque>
#include <future>
#include <iostream>
#include <mutex>
#include <optional>
#include <stdexcept>

class BlockingQueue {
public:
    bool push(int value) {
        {
            std::lock_guard<std::mutex> lock(mutex_);
            if (closed_) {
                return false;
            }
            values_.push_back(value);
        }
        ready_.notify_one();
        return true;
    }

    std::optional<int> pop() {
        std::unique_lock<std::mutex> lock(mutex_);
        ready_.wait(lock, [this] { return closed_ || !values_.empty(); });
        if (values_.empty()) {
            return std::nullopt;
        }
        int value = values_.front();
        values_.pop_front();
        return value;
    }

    void close() {
        {
            std::lock_guard<std::mutex> lock(mutex_);
            closed_ = true;
        }
        ready_.notify_all();
    }

private:
    std::mutex mutex_;
    std::condition_variable ready_;
    std::deque<int> values_;
    bool closed_ = false;
};

int main() {
    try {
        BlockingQueue queue;
        auto consumer = std::async(std::launch::async, [&queue] {
            int total = 0;
            while (auto value = queue.pop()) {
                total += *value;
            }
            return total;
        });
        try {
            for (int i = 1; i <= 100; ++i) {
                if (!queue.push(i)) {
                    throw std::logic_error("queue already closed");
                }
            }
        } catch (...) {
            queue.close();
            throw;
        }
        queue.close();
        std::cout << consumer.get() << '\n'; // 5050
    } catch (const std::exception& error) {
        std::cerr << error.what() << '\n';
        return 1;
    }
}
```

生产过程失败时先关闭队列，再展开异常，避免 future 等待一个永远等不到结束通知的消费者。异步任务的异常由 future 保存。future 的生命周期结束在队列销毁之前，防止回调继续访问已销毁的队列。

关闭状态也是谓词的一部分；`notify_all()` 让空队列上的等待者有机会退出。此示例不支持重开，不限制容量，不承诺处理无法恢复的底层锁故障；真实服务还需要定义取消、容量、任务失败和关闭策略。

<a id="pool"></a>
## 11. 线程池与生产者消费者

线程池通常由固定或受控数量的工作线程、任务队列和结果通道组成。提交者把任务放进队列，工作线程等待、取出任务、释放队列锁，再执行任务；任务异常保存到结果通道，不逃出工作线程。

设计时需要明确：

1. 队列是否有容量限制，满时阻塞、拒绝还是丢弃？这决定背压与内存上限。
2. 关闭后是否拒绝提交？已有任务是执行完还是取消？结果如何通知调用者？
3. 部分工作线程创建失败时，谁通知退出并等待已经创建的线程？
4. 工作任务能否提交另一个任务并同步等待？所有线程都这样等待时可能耗尽执行资源。
5. 任务捕获的数据、线程池对象与结果状态分别活到什么时候？

C++17 标准库没有通用线程池。`hardware_concurrency()` 只是提示，可能返回 0，也不能直接决定 I/O 或计算任务的最佳线程数。先用清楚的生命周期协议，再根据测量选择工作线程数量。

<a id="stop"></a>
## 12. C++20 的协作式停止

`std::jthread` 析构时若仍关联线程，会请求停止并等待。接受 `std::stop_token` 的工作函数需要检查停止请求，并在有界时间内离开；它不会强制终止计算或自动中断任意 I/O。

**片段，C++20；需要 `<thread>`、`<stop_token>` 与 `<chrono>`，放在函数体内：**

```cpp
std::jthread worker([](std::stop_token stop) {
    while (!stop.stop_requested()) {
        std::this_thread::sleep_for(std::chrono::milliseconds(10));
        // 在这里执行一小批工作；工作函数仍需处理自己的异常。
    }
});
worker.request_stop();
```

本例请求停止不会立即中断睡眠。任务使用的资源要先于线程对象声明，让线程先完成退出再销毁资源；也不要持有工作线程需要的锁去等待 jthread 析构。

普通 `condition_variable` 的等待不会自动响应停止令牌。可采用关闭标志加通知，或使用 C++20 `condition_variable_any` 的停止令牌等待重载。C++20 的原子 `wait()` / `notify_one()` / `notify_all()` 可用于等待原子值变化，通知仍不能替代实际状态更新。

<a id="review"></a>
## 13. 死锁、所有权与验证

死锁通常来自锁顺序相反、持锁等待回调、等待自身或资源耗尽。为多锁操作规定一致顺序，或使用 `scoped_lock` 等合适工具；递归互斥锁不会自动修复错误的协作协议。

`shared_ptr` 的控制块支持不同智能指针实例的相应并发操作，并不使所指对象线程安全。对同一个 `shared_ptr` 变量的并发读写仍要同步，或采用版本匹配的原子接口。异步回调可用 `weak_ptr::lock()` 临时取得所有权，但取得所有权后仍需保护对象状态。

检查清单：共享读写遵循同一协议；捕获引用不悬空；创建失败能够回收已有线程；工作异常能够报告；关闭能够唤醒等待者；销毁顺序正确；不持锁等待依赖此锁的任务。

在支持的平台上，可用 ThreadSanitizer 辅助发现数据竞争；它不能证明不存在死锁或业务逻辑错误，支持与限制见 [Clang ThreadSanitizer 官方文档](https://clang.llvm.org/docs/ThreadSanitizer.html)。验证多种调度、空队列、重复关闭、生产失败、消费者失败和关闭时仍有任务的情况，并给测试设置超时。

GCC / Clang 在常见 POSIX 线程平台上的示例编译命令：

```bash
g++ -std=c++17 -Wall -Wextra -Wpedantic -pthread main.cpp -o demo
```

MSVC 开发者终端中的编译命令：

```bat
cl /std:c++17 /EHsc /W4 /permissive- /utf-8 main.cpp /Fe:demo.exe
```

第 12 节片段需要改用 C++20 模式，并确认对应标准库支持。有关平台线程、系统句柄和阻塞 I/O 的边界，继续阅读 [系统编程指南](C++SystemProgrammingGuide.md)。
