# C++ 设计模式学习指南

> 本文以 **C++17** 为基线，介绍 GoF（Gang of Four，四人组）的 23 种经典设计模式。重点是识别变化、划分职责和管理对象生命周期，而不是机械套用类图。代码使用标准库，不依赖 Qt。

## 阅读导航

1. 设计模式与基本设计原则
2. 创建型模式：如何创建对象
3. 结构型模式：如何组织对象
4. 行为型模式：如何分配职责和协作
5. 相似模式辨析与选型
6. 现代 C++ 与 Qt 项目中的实践
7. 学习顺序与练习

**示例约定：** 每个 `cpp` 代码块都是独立的完整程序，包含所需头文件和 `main()`。请分别保存为 `main.cpp`，不要把所有示例直接拼接到同一文件。示例为了突出模式而简化业务；线程安全、异常和生命周期限制在对应章节说明。

GCC / Clang 编译示例：

```bash
g++ -std=c++17 -Wall -Wextra -pedantic main.cpp -o demo
```

MSVC 编译示例（在开发者命令提示符中）：

```bat
cl /std:c++17 /EHsc /W4 /utf-8 main.cpp
```

## 一、设计模式与基本设计原则

### 1.1 什么是设计模式

设计模式是对反复出现的设计问题的经验总结。它描述问题背景、参与角色、协作方式和取舍，不是固定的代码模板，也不是具体的算法。

例如：文件导出方式经常增加，可以将导出算法封装成策略；多个界面需要响应同一个模型的变化，可以使用观察者；一段流程固定，但其中几个步骤随业务变化，可以使用模板方法。

选择模式前，先明确三个问题：

1. 当前重复或耦合发生在哪里？
2. 哪一部分会变化，哪一部分应该保持稳定？
3. 引入抽象后的收益是否超过额外的复杂度？

### 1.2 基本设计原则

| 原则 | 含义 | C++ 中的实践 |
| --- | --- | --- |
| 单一职责（SRP） | 一个模块围绕一类职责组织，避免多个独立变化原因混在一起 | 分离界面显示、业务计算和持久化 |
| 开闭原则（OCP） | 为预期变化提供扩展点，减少修改稳定代码 | 使用策略、接口或模板参数替换实现 |
| 里氏替换（LSP） | 子类型应满足基类型的行为契约 | 不强化前置条件，不破坏结果和不变量 |
| 接口隔离（ISP） | 客户端不应依赖自己不需要的操作 | 拆分读取接口和写入接口 |
| 依赖倒置（DIP） | 高层业务与底层实现通过抽象协作 | 构造函数接收存储接口，而不是在内部创建数据库对象 |
| 最少知识原则 | 对象只了解完成职责所需的直接协作者 | 避免跨越多层对象连续调用内部细节 |
| 优先组合 | 通过组合独立对象复用和替换行为 | 用成员策略代替不断扩大的继承树 |

“优先组合”不意味着禁止继承。需要表达稳定的子类型关系和运行时多态时，继承仍然适用。

### 1.3 C++ 示例的共同约定

- 多态基类通过基类指针销毁对象时，析构函数应为 `virtual`。
- 默认用值对象或 `std::unique_ptr` 表达所有权；真正需要共享所有权时才用 `std::shared_ptr`。
- 引用和裸指针通常表达非拥有关系，必须保证被引用对象活得足够久。
- `std::move` 允许移动，但不会独立完成资源转移；最终行为由目标类型的操作决定。
- `const` 成员函数不自动保证线程安全；智能指针也不自动保护其所指对象。
- 示例中的 `assert` 用于演示结果；生产代码的输入校验不能依赖可能被 `NDEBUG` 禁用的断言。

### 1.4 23 种模式总览

| 分类 | 模式 | 主要解决的问题 |
| --- | --- | --- |
| 创建型 | 单例 Singleton | 受控地提供唯一实例 |
| 创建型 | 工厂方法 Factory Method | 将具体产品的创建交给子类 |
| 创建型 | 抽象工厂 Abstract Factory | 创建相互匹配的一族产品 |
| 创建型 | 建造者 Builder | 分步骤构造复杂对象 |
| 创建型 | 原型 Prototype | 根据已有对象复制新对象 |
| 结构型 | 适配器 Adapter | 转换不兼容的接口 |
| 结构型 | 桥接 Bridge | 让抽象和实现分别扩展 |
| 结构型 | 组合 Composite | 统一处理单个对象和对象树 |
| 结构型 | 装饰器 Decorator | 按需叠加对象行为 |
| 结构型 | 外观 Facade | 提供简化的子系统入口 |
| 结构型 | 享元 Flyweight | 共享可复用的内在状态 |
| 结构型 | 代理 Proxy | 控制对真实对象的访问 |
| 行为型 | 责任链 Chain of Responsibility | 沿处理链寻找或组合处理者 |
| 行为型 | 命令 Command | 将操作封装成对象 |
| 行为型 | 解释器 Interpreter | 表达并解释简单语言的规则 |
| 行为型 | 迭代器 Iterator | 隐藏集合表示并提供遍历 |
| 行为型 | 中介者 Mediator | 集中协调对象间的交互 |
| 行为型 | 备忘录 Memento | 保存并恢复对象状态 |
| 行为型 | 观察者 Observer | 在状态变化时通知订阅者 |
| 行为型 | 状态 State | 根据内部状态改变行为 |
| 行为型 | 策略 Strategy | 替换完成同一任务的算法 |
| 行为型 | 模板方法 Template Method | 固定流程并开放部分步骤 |
| 行为型 | 访问者 Visitor | 对稳定的类型集合添加操作 |

## 二、创建型模式

### 2.1 单例（Singleton）

**意图：** 限制某个类的实例创建，并提供统一访问点。适用于确实具有唯一性、生命周期明确的服务。普通业务对象通常更适合显式构造和依赖注入。

```cpp
#include <cassert>
#include <string>

class AppConfig {
public:
    static AppConfig& instance() {
        static AppConfig config;
        return config;
    }
    AppConfig(const AppConfig&) = delete;
    AppConfig& operator=(const AppConfig&) = delete;
    ~AppConfig() = default;
    const std::string& theme() const { return theme_; }
private:
    AppConfig() = default;
    const std::string theme_ = "dark";
};

int main() {
    auto& a = AppConfig::instance();
    auto& b = AppConfig::instance();
    assert(&a == &b);
    assert(a.theme() == "dark");
}
```

**注意：** C++11 起，函数局部静态对象的初始化具有线程安全保证，但之后对可变成员的访问仍需同步。本例仅提供不可变配置。单例会隐藏依赖、增加测试隔离难度；还要考虑静态对象的销毁顺序和动态库边界，不能把这个写法理解为任意部署环境下的“全进程唯一”。

### 2.2 工厂方法（Factory Method）

**意图：** 创建者定义产品接口和使用流程，由具体创建者决定创建哪种产品。适用于框架流程稳定、产品实现需要扩展的场景。

```cpp
#include <cassert>
#include <memory>
#include <string>

struct Transport {
    virtual ~Transport() = default;
    virtual std::string deliver() const = 0;
};
struct Truck final : Transport {
    std::string deliver() const override { return "road"; }
};
struct Ship final : Transport {
    std::string deliver() const override { return "sea"; }
};
class Logistics {
public:
    virtual ~Logistics() = default;
    virtual std::unique_ptr<Transport> create() const = 0;
    std::string plan() const { return create()->deliver(); }
};
struct RoadLogistics final : Logistics {
    std::unique_ptr<Transport> create() const override {
        return std::make_unique<Truck>();
    }
};
struct SeaLogistics final : Logistics {
    std::unique_ptr<Transport> create() const override {
        return std::make_unique<Ship>();
    }
};

int main() {
    RoadLogistics road;
    SeaLogistics sea;
    assert(road.plan() == "road");
    assert(sea.plan() == "sea");
}
```

**注意：** `create(type)` 内部通过 `switch` 创建不同对象通常称为“简单工厂”，不是 GoF 工厂方法。简单工厂也很实用；只有确实需要子类扩展创建行为时，才引入创建者继承层次。

### 2.3 抽象工厂（Abstract Factory）

**意图：** 通过一个工厂创建多个相关产品，保证产品属于同一系列。典型场景是整套主题控件或一组配套的平台服务。

```cpp
#include <cassert>
#include <memory>
#include <string>

struct Button {
    virtual ~Button() = default;
    virtual std::string style() const = 0;
};
struct Menu {
    virtual ~Menu() = default;
    virtual std::string style() const = 0;
};
struct LightButton final : Button {
    std::string style() const override { return "light"; }
};
struct LightMenu final : Menu {
    std::string style() const override { return "light"; }
};
struct DarkButton final : Button {
    std::string style() const override { return "dark"; }
};
struct DarkMenu final : Menu {
    std::string style() const override { return "dark"; }
};
struct UiFactory {
    virtual ~UiFactory() = default;
    virtual std::unique_ptr<Button> button() const = 0;
    virtual std::unique_ptr<Menu> menu() const = 0;
};
struct LightFactory final : UiFactory {
    std::unique_ptr<Button> button() const override {
        return std::make_unique<LightButton>();
    }
    std::unique_ptr<Menu> menu() const override {
        return std::make_unique<LightMenu>();
    }
};
struct DarkFactory final : UiFactory {
    std::unique_ptr<Button> button() const override {
        return std::make_unique<DarkButton>();
    }
    std::unique_ptr<Menu> menu() const override {
        return std::make_unique<DarkMenu>();
    }
};

bool consistent(const UiFactory& factory) {
    return factory.button()->style() == factory.menu()->style();
}
int main() {
    LightFactory light;
    DarkFactory dark;
    assert(consistent(light));
    assert(consistent(dark));
}
```

**取舍：** 增加一个产品系列比较容易；增加一种产品类型，需要修改工厂接口及所有具体工厂。工厂帮助维持系列一致性，客户端仍需避免自行混用不同工厂生成的产品。

### 2.4 建造者（Builder）

**意图：** 将复杂对象的构造拆成明确步骤，在最终构建时检查整体约束。适用于配置选项较多、构造函数参数难以阅读的对象。

```cpp
#include <cassert>
#include <stdexcept>
#include <string>
#include <utility>

struct Request {
    std::string url;
    int timeoutMs;
    bool retry;
};
class RequestBuilder {
public:
    RequestBuilder& url(std::string value) {
        request_.url = std::move(value);
        return *this;
    }
    RequestBuilder& timeout(int value) {
        request_.timeoutMs = value;
        return *this;
    }
    RequestBuilder& retry(bool value) {
        request_.retry = value;
        return *this;
    }
    Request build() const {
        if (request_.url.empty() || request_.timeoutMs <= 0) {
            throw std::invalid_argument("invalid request");
        }
        return request_;
    }
private:
    Request request_{"", 1000, false};
};

int main() {
    auto request = RequestBuilder{}.url("/users").timeout(3000)
                                   .retry(true).build();
    assert(request.url == "/users" && request.retry);
}
```

**说明：** 本例是现代常见的链式 Builder。经典 GoF 建造者还可由指挥者（Director）组织步骤，并让不同建造者产生不同表示；步骤简单时无需额外引入 Director。本例产品为可修改的值对象；若构造后的约束必须持续成立，应封装成员并限制修改。

### 2.5 原型（Prototype）

**意图：** 从已有对象复制出新对象，尤其适合在不知道具体动态类型时复制多态对象。

```cpp
#include <cassert>
#include <memory>
#include <string>
#include <utility>

struct Shape {
    virtual ~Shape() = default;
    virtual std::unique_ptr<Shape> clone() const = 0;
    virtual std::string label() const = 0;
};
class Circle final : public Shape {
public:
    explicit Circle(std::string label) : label_(std::move(label)) {}
    std::unique_ptr<Shape> clone() const override {
        return std::make_unique<Circle>(*this);
    }
    std::string label() const override { return label_; }
private:
    std::string label_;
};

int main() {
    std::unique_ptr<Shape> source = std::make_unique<Circle>("circle");
    auto copy = source->clone();
    assert(source.get() != copy.get());
    assert(source->label() == copy->label());
}
```

**注意：** 本例字符串按值复制。若对象包含指针、句柄或子对象，必须定义克隆的含义：哪些资源深拷贝、哪些资源共享、哪些资源不能复制。`std::unique_ptr<Derived>` 不能像裸指针那样用作虚函数的协变返回类型，因此覆盖函数仍返回 `std::unique_ptr<Shape>`。

## 三、结构型模式

### 3.1 适配器（Adapter）

**意图：** 将已有对象的接口转换为客户端期望的接口。常见于旧系统、第三方库和接口迁移。

```cpp
#include <cassert>

class LegacyThermometer {
public:
    double fahrenheit() const { return 86.0; }
};
struct TemperatureReader {
    virtual ~TemperatureReader() = default;
    virtual double celsius() const = 0;
};
class ThermometerAdapter final : public TemperatureReader {
public:
    explicit ThermometerAdapter(const LegacyThermometer& device)
        : device_(device) {}
    double celsius() const override {
        return (device_.fahrenheit() - 32.0) * 5.0 / 9.0;
    }
private:
    const LegacyThermometer& device_;
};

int main() {
    LegacyThermometer device;
    ThermometerAdapter adapter(device);
    assert(adapter.celsius() == 30.0);
}
```

**注意：** 本例采用对象适配器，通过组合包装旧接口。`device_` 不拥有设备，设备必须比适配器活得久。适配时还要转换单位、错误语义和调用约束，不能只修改函数名称。

### 3.2 桥接（Bridge）

**意图：** 将高层抽象与底层实现分离，让两个变化维度分别扩展。例如“形状类型”和“渲染后端”独立变化。

```cpp
#include <cassert>
#include <string>

struct Renderer {
    virtual ~Renderer() = default;
    virtual std::string render(const std::string& shape) const = 0;
};
struct VectorRenderer final : Renderer {
    std::string render(const std::string& shape) const override {
        return "vector:" + shape;
    }
};
struct RasterRenderer final : Renderer {
    std::string render(const std::string& shape) const override {
        return "raster:" + shape;
    }
};
class Shape {
public:
    explicit Shape(const Renderer& renderer) : renderer_(renderer) {}
    virtual ~Shape() = default;
    virtual std::string draw() const = 0;
protected:
    const Renderer& renderer_;
};
struct Circle final : Shape {
    using Shape::Shape;
    std::string draw() const override { return renderer_.render("circle"); }
};
struct Square final : Shape {
    using Shape::Shape;
    std::string draw() const override { return renderer_.render("square"); }
};

int main() {
    VectorRenderer vector;
    RasterRenderer raster;
    Circle circle(vector);
    Square square(raster);
    assert(circle.draw() == "vector:circle");
    assert(square.draw() == "raster:square");
}
```

**取舍：** 避免为所有组合创建 `VectorCircle`、`RasterCircle` 等类，但会增加一层间接调用。渲染器是非拥有引用，必须保持有效。桥接通常是提前分离变化维度；适配器通常是对已有不兼容接口进行补救。

### 3.3 组合（Composite）

**意图：** 用相同接口处理单个对象和对象的树状组合。适用于目录树、场景树、组织结构等。

```cpp
#include <cassert>
#include <memory>
#include <stdexcept>
#include <utility>
#include <vector>

struct Node {
    virtual ~Node() = default;
    virtual int size() const = 0;
};
class File final : public Node {
public:
    explicit File(int bytes) : bytes_(bytes) {}
    int size() const override { return bytes_; }
private:
    int bytes_;
};
class Folder final : public Node {
public:
    void add(std::unique_ptr<Node> child) {
        if (!child) { throw std::invalid_argument("null child"); }
        children_.push_back(std::move(child));
    }
    int size() const override {
        int total = 0;
        for (const auto& child : children_) { total += child->size(); }
        return total;
    }
private:
    std::vector<std::unique_ptr<Node>> children_;
};

int main() {
    auto sub = std::make_unique<Folder>();
    sub->add(std::make_unique<File>(20));
    Folder root;
    root.add(std::make_unique<File>(10));
    root.add(std::move(sub));
    assert(root.size() == 30);
}
```

**注意：** 本例由父节点独占子节点，是一棵树。若改为共享节点形成图，还要处理环和重复遍历。真实文件大小应选择合适的整数类型并考虑溢出；深层递归也可能消耗大量栈空间。

### 3.4 装饰器（Decorator）

**意图：** 在保持同一接口的情况下，把附加行为逐层包装到对象上。适用于日志、编码、压缩和其他可组合功能。

```cpp
#include <cassert>
#include <memory>
#include <stdexcept>
#include <string>
#include <utility>

struct Text {
    virtual ~Text() = default;
    virtual std::string read() const = 0;
};
struct PlainText final : Text {
    std::string read() const override { return "hello"; }
};
class TextDecorator : public Text {
public:
    explicit TextDecorator(std::unique_ptr<Text> inner)
        : inner_(std::move(inner)) {
        if (!inner_) { throw std::invalid_argument("null text"); }
    }
protected:
    std::unique_ptr<Text> inner_;
};
struct Brackets final : TextDecorator {
    using TextDecorator::TextDecorator;
    std::string read() const override { return "[" + inner_->read() + "]"; }
};
struct LogPrefix final : TextDecorator {
    using TextDecorator::TextDecorator;
    std::string read() const override { return "log:" + inner_->read(); }
};

int main() {
    std::unique_ptr<Text> text = std::make_unique<LogPrefix>(
        std::make_unique<Brackets>(std::make_unique<PlainText>()));
    assert(text->read() == "log:[hello]");
}
```

**注意：** 包装顺序可能改变结果。本例若交换两层顺序，会得到 `[log:hello]`。层数过多会增加调试难度；需要显式识别包装对象的客户端，也可能破坏接口透明性。

### 3.5 外观（Facade）

**意图：** 给复杂子系统提供简洁、符合客户端用途的入口。外观负责协调已有能力，而不应不断吸收所有业务逻辑。

```cpp
#include <cassert>
#include <string>

struct VideoSystem {
    std::string prepare() const { return "video ready"; }
};
struct AudioSystem {
    std::string prepare() const { return "audio ready"; }
};
class PlayerFacade {
public:
    std::string play() const {
        return video_.prepare() + "; " + audio_.prepare();
    }
private:
    VideoSystem video_;
    AudioSystem audio_;
};

int main() {
    PlayerFacade player;
    assert(player.play() == "video ready; audio ready");
}
```

**取舍：** 客户端依赖更少，但外观可能发展成职责过多的大类。示例用字符串模拟准备流程；真实流程需要定义失败时的清理和回滚顺序。外观不一定禁止客户端直接使用子系统。

### 3.6 享元（Flyweight）

**意图：** 在大量相似对象之间共享内在状态，把每次使用不同的外在状态交给调用者管理。

```cpp
#include <cassert>
#include <map>
#include <memory>
#include <string>

class Glyph {
public:
    explicit Glyph(char symbol) : symbol_(symbol) {}
    std::string draw(int x) const {
        return std::string(1, symbol_) + "@" + std::to_string(x);
    }
private:
    const char symbol_;  // 内在状态：字符身份
};
class GlyphFactory {
public:
    std::shared_ptr<const Glyph> get(char symbol) {
        auto found = cache_.find(symbol);
        if (found != cache_.end()) { return found->second; }
        auto glyph = std::make_shared<const Glyph>(symbol);
        cache_.emplace(symbol, glyph);
        return glyph;
    }
private:
    std::map<char, std::shared_ptr<const Glyph>> cache_;
};
struct GlyphOccurrence {
    std::shared_ptr<const Glyph> glyph;
    int x;  // 外在状态：本次出现的位置
};

int main() {
    GlyphFactory factory;
    GlyphOccurrence a{factory.get('A'), 10};
    GlyphOccurrence b{factory.get('A'), 20};
    assert(a.glyph == b.glyph);
    assert(a.glyph->draw(a.x) == "A@10");
    assert(b.glyph->draw(b.x) == "A@20");
}
```

**注意：** 本例通过只读对象共享状态，工厂持有强引用，工厂存在期间缓存不会自动释放。真实缓存需要容量策略；也可用 `weak_ptr` 允许对象在无人使用时释放。工厂的查找和插入不是线程安全的。仅仅使用 `shared_ptr` 并不等于享元，还要明确内在状态与外在状态。

### 3.7 代理（Proxy）

**意图：** 提供与真实对象相同的接口，在访问前后进行控制。常见形式包括延迟加载、权限控制、缓存和远程代理。

```cpp
#include <cassert>
#include <memory>
#include <string>
#include <utility>

struct Image {
    virtual ~Image() = default;
    virtual std::string display() = 0;
};
class RealImage final : public Image {
public:
    explicit RealImage(std::string path) : path_(std::move(path)) {}
    std::string display() override { return path_; }
private:
    std::string path_;
};
class LazyImage final : public Image {
public:
    explicit LazyImage(std::string path) : path_(std::move(path)) {}
    bool loaded() const { return static_cast<bool>(image_); }
    std::string display() override {
        if (!image_) { image_ = std::make_unique<RealImage>(path_); }
        return image_->display();
    }
private:
    std::string path_;
    std::unique_ptr<RealImage> image_;
};

int main() {
    LazyImage image("photo.png");
    assert(!image.loaded());
    const auto result = image.display();
    assert(image.loaded() && result == "photo.png");
}
```

**说明：** 这里用延迟创建模拟加载，没有执行图片文件 I/O。并发调用 `display()` 时需要同步。代理强调控制访问，装饰器强调叠加职责，二者即使类结构相似，设计目的也不同。

## 四、行为型模式

### 4.1 责任链（Chain of Responsibility）

**意图：** 将请求沿一组处理者传递，让发送者不必绑定某个具体处理者。可以在首个成功处理后停止，也可以让多个处理者依次参与，但必须明确终止规则。

```cpp
#include <cassert>
#include <memory>
#include <utility>

class Handler {
public:
    virtual ~Handler() = default;
    void setNext(std::unique_ptr<Handler> next) { next_ = std::move(next); }
    virtual bool handle(int value) const {
        return next_ ? next_->handle(value) : false;
    }
private:
    std::unique_ptr<Handler> next_;
};
struct SmallRequest final : Handler {
    bool handle(int value) const override {
        if (value > 0 && value < 100) { return true; }
        return Handler::handle(value);
    }
};
struct LargeRequest final : Handler {
    bool handle(int value) const override {
        if (value >= 100) { return true; }
        return Handler::handle(value);
    }
};

int main() {
    SmallRequest chain;
    chain.setNext(std::make_unique<LargeRequest>());
    assert(chain.handle(50));
    assert(chain.handle(500));
    assert(!chain.handle(-1));
}
```

**注意：** 本例采用“首个匹配者处理”的规则。处理顺序影响结果，应定义无人处理、异常和超时的行为。责任链不要隐藏关键业务顺序，也不要把所有条件判断都改写成大量处理者类。

### 4.2 命令（Command）

**意图：** 把请求封装成对象，使调用者与实际执行者分离，并支持排队、记录、撤销等能力。命令并不天然可撤销，撤销逻辑需要单独设计。

```cpp
#include <cassert>
#include <cstddef>
#include <stdexcept>
#include <string>
#include <utility>

class Editor {
public:
    void append(const std::string& text) { text_ += text; }
    void truncate(std::size_t size) { text_.resize(size); }
    const std::string& text() const { return text_; }
private:
    std::string text_;
};
struct Command {
    virtual ~Command() = default;
    virtual void execute() = 0;
    virtual void undo() = 0;
};
class AppendCommand final : public Command {
public:
    AppendCommand(Editor& editor, std::string text)
        : editor_(editor), text_(std::move(text)) {}
    void execute() override {
        if (executed_) { throw std::logic_error("already executed"); }
        previousSize_ = editor_.text().size();
        editor_.append(text_);
        executed_ = true;
    }
    void undo() override {
        if (!executed_) { throw std::logic_error("not executed"); }
        editor_.truncate(previousSize_);
        executed_ = false;
    }
private:
    Editor& editor_;
    std::string text_;
    std::size_t previousSize_ = 0;
    bool executed_ = false;
};

int main() {
    Editor editor;
    AppendCommand command(editor, "hello");
    command.execute();
    assert(editor.text() == "hello");
    command.undo();
    assert(editor.text().empty());
}
```

**限制：** 本例要求编辑器在执行与撤销之间没有不受管理的修改。多命令场景通常由历史管理器按后进先出顺序撤销；还要保证接收者在命令执行时仍然存在。网络发送、付款等外部操作不能用简单状态回退假装撤销，需要业务补偿。

### 4.3 解释器（Interpreter）

**意图：** 为简单语言的规则建立表达式对象，并根据上下文解释这些表达式。适用于小型表达式、规则树和受限 DSL（领域专用语言）。

```cpp
#include <cassert>
#include <memory>
#include <stdexcept>
#include <utility>

struct Expression {
    virtual ~Expression() = default;
    virtual int evaluate() const = 0;
};
class Number final : public Expression {
public:
    explicit Number(int value) : value_(value) {}
    int evaluate() const override { return value_; }
private:
    int value_;
};
class Add final : public Expression {
public:
    Add(std::unique_ptr<Expression> left, std::unique_ptr<Expression> right)
        : left_(std::move(left)), right_(std::move(right)) {
        if (!left_ || !right_) { throw std::invalid_argument("null operand"); }
    }
    int evaluate() const override {
        return left_->evaluate() + right_->evaluate();
    }
private:
    std::unique_ptr<Expression> left_, right_;
};

int main() {
    Add expression(std::make_unique<Number>(1),
        std::make_unique<Add>(std::make_unique<Number>(2),
                              std::make_unique<Number>(3)));
    assert(expression.evaluate() == 6);
}
```

**说明：** 本例手动构造语法树，没有包含字符串分词和语法解析。真实解释器还需要上下文、错误定位、整数溢出和递归深度检查。规则复杂时，应评估成熟的解析工具，而不是为每条语法不断扩充继承树。

### 4.4 迭代器（Iterator）

**意图：** 通过统一遍历接口访问集合元素，让客户端不依赖集合内部组织方式。C++ 标准库已经提供成熟的迭代器机制，通常无需自行发明遍历框架。

```cpp
#include <cassert>
#include <vector>

class Numbers {
public:
    void add(int value) { values_.push_back(value); }
    auto begin() const { return values_.cbegin(); }
    auto end() const { return values_.cend(); }
private:
    std::vector<int> values_;
};

int main() {
    Numbers numbers;
    numbers.add(10);
    numbers.add(20);
    int sum = 0;
    for (int value : numbers) { sum += value; }
    assert(sum == 30);
}
```

**说明：** 本例复用 `vector` 的只读迭代器。范围 `for` 在这里通过 `begin()` / `end()` 获取迭代器并逐个访问元素。遍历期间增加元素可能使原有迭代器失效；不同容器的失效规则不同，迭代器类别也决定可以使用哪些算法。

### 4.5 中介者（Mediator）

**意图：** 将多个对象之间的交互规则集中到协调者中，让对象不必互相了解。常见于表单控件联动和工作流协调。

```cpp
#include <cassert>
#include <string>
#include <utility>

enum class Event { TextChanged };
struct Mediator {
    virtual ~Mediator() = default;
    virtual void notify(Event event) = 0;
};
class TextBox {
public:
    explicit TextBox(Mediator& mediator) : mediator_(mediator) {}
    void setText(std::string text) {
        text_ = std::move(text);
        mediator_.notify(Event::TextChanged);
    }
    const std::string& text() const { return text_; }
private:
    Mediator& mediator_;
    std::string text_;
};
class Button {
public:
    void setEnabled(bool value) { enabled_ = value; }
    bool enabled() const { return enabled_; }
private:
    bool enabled_ = false;
};
class Dialog final : public Mediator {
public:
    Dialog() : textBox_(*this) {}
    Dialog(const Dialog&) = delete;
    Dialog& operator=(const Dialog&) = delete;
    void input(std::string text) { textBox_.setText(std::move(text)); }
    bool canSubmit() const { return button_.enabled(); }
    void notify(Event event) override {
        if (event == Event::TextChanged) {
            button_.setEnabled(!textBox_.text().empty());
        }
    }
private:
    TextBox textBox_;
    Button button_;
};

int main() {
    Dialog dialog;
    dialog.input("Alice");
    assert(dialog.canSubmit());
    dialog.input("");
    assert(!dialog.canSubmit());
}
```

**注意：** 文本框和按钮没有直接依赖彼此，联动规则由对话框管理。本例禁止复制，避免成员中的中介者引用仍指向旧对象。复杂协调者应按业务拆分，并防止联动通知形成递归循环。

### 4.6 备忘录（Memento）

**意图：** 在不向外暴露对象内部表示的前提下保存状态，之后由对象自己恢复。适用于快照、编辑历史和检查点。

```cpp
#include <cassert>
#include <string>
#include <utility>

class Editor {
public:
    class Snapshot {
    public:
        Snapshot(const Snapshot&) = default;
        Snapshot& operator=(const Snapshot&) = default;
        ~Snapshot() = default;
    private:
        friend class Editor;
        explicit Snapshot(std::string text) : text_(std::move(text)) {}
        std::string text_;
    };
    void setText(std::string text) { text_ = std::move(text); }
    const std::string& text() const { return text_; }
    Snapshot save() const { return Snapshot(text_); }
    void restore(const Snapshot& snapshot) { text_ = snapshot.text_; }
private:
    std::string text_;
};

int main() {
    Editor editor;
    editor.setText("version 1");
    auto snapshot = editor.save();
    editor.setText("version 2");
    editor.restore(snapshot);
    assert(editor.text() == "version 1");
}
```

**取舍：** 外部可以保存快照，但无法直接修改其内容。完整快照可能占用大量内存，必要时使用差量记录或共享不可变数据。恢复内存状态不会自动恢复文件、网络等外部系统；跨实例恢复是否允许也需要明确约束。

### 4.7 观察者（Observer）

**意图：** 当被观察对象发生变化时通知订阅者，而不让被观察对象依赖具体的界面或业务对象。

```cpp
#include <cassert>
#include <cstddef>
#include <functional>
#include <map>
#include <memory>
#include <stdexcept>
#include <utility>

class Subject {
public:
    using Callback = std::function<void(int)>;
    using Token = std::size_t;
    Token subscribe(Callback callback) {
        if (!callback) { throw std::invalid_argument("empty callback"); }
        const Token token = next_++;
        observers_.emplace(token, std::make_shared<Callback>(std::move(callback)));
        return token;
    }
    void unsubscribe(Token token) { observers_.erase(token); }
    void setValue(int value) {
        value_ = value;
        const auto snapshot = observers_;
        for (const auto& item : snapshot) { (*item.second)(value); }
    }
    int value() const { return value_; }
private:
    int value_ = 0;
    Token next_ = 1;
    std::map<Token, std::shared_ptr<Callback>> observers_;
};

int main() {
    Subject subject;
    int received = 0;
    const auto token = subject.subscribe([&received](int value) {
        received = value;
    });
    subject.setValue(42);
    assert(received == 42 && subject.value() == 42);
    subject.unsubscribe(token);
    subject.setValue(7);
    assert(received == 42);
}
```

**限制：** 本例只用于单线程，通知前复制订阅表，因此本轮通知期间的订阅和取消只影响后续通知。回调抛异常会中断本轮通知；生产代码需确定异常和重入策略。回调捕获的对象必须仍然有效，可用 RAII 订阅句柄自动取消，或用 `weak_ptr` 防止访问已销毁对象。

订阅表通过智能指针保留同一个回调对象，快照不会重新复制回调内部的可变状态。这里的共享所有权用于保证本轮通知期间回调仍然存在，并不延长回调所借用对象的生命周期。

### 4.8 状态（State）

**意图：** 把不同状态下的行为封装为状态对象，让上下文根据内部状态执行操作和发生转换。适用于播放器、连接协议和订单流程。

```cpp
#include <cassert>
#include <memory>
#include <string>
#include <utility>

struct State {
    virtual ~State() = default;
    virtual const char* name() const = 0;
    virtual std::unique_ptr<State> press() const = 0;
};
struct Stopped final : State {
    const char* name() const override { return "stopped"; }
    std::unique_ptr<State> press() const override;
};
struct Playing final : State {
    const char* name() const override { return "playing"; }
    std::unique_ptr<State> press() const override;
};
std::unique_ptr<State> Stopped::press() const {
    return std::make_unique<Playing>();
}
std::unique_ptr<State> Playing::press() const {
    return std::make_unique<Stopped>();
}
class Player {
public:
    std::string stateName() const { return state_->name(); }
    void press() {
        auto next = state_->press();
        state_ = std::move(next);
    }
private:
    std::unique_ptr<State> state_ = std::make_unique<Stopped>();
};

int main() {
    Player player;
    assert(player.stateName() == "stopped");
    player.press();
    assert(player.stateName() == "playing");
    player.press();
    assert(player.stateName() == "stopped");
}
```

**注意：** 状态转换由当前状态决定。新状态先构造成功，再替换旧状态；替换发生在旧状态的成员函数返回后，避免在调用过程中销毁当前状态对象。真实流程还需处理非法事件、进入/退出动作和并发事件。状态很少且规则稳定时，枚举加明确的转换表可能更简单。

### 4.9 策略（Strategy）

**意图：** 将完成同一任务的不同算法封装并替换，让使用者无需了解算法细节。C++ 中既可以使用多态接口，也可以使用函数对象、模板参数或 `std::function`。

```cpp
#include <cassert>
#include <functional>
#include <stdexcept>
#include <utility>

class Checkout {
public:
    using Pricing = std::function<int(int)>;
    explicit Checkout(Pricing pricing) : pricing_(std::move(pricing)) {
        if (!pricing_) { throw std::invalid_argument("empty strategy"); }
    }
    int payable(int cents) const {
        if (cents < 0) { throw std::invalid_argument("negative price"); }
        return pricing_(cents);
    }
private:
    Pricing pricing_;
};

int main() {
    Checkout regular([](int cents) { return cents; });
    Checkout promotion([](int cents) { return cents - cents / 10; });
    assert(regular.payable(1000) == 1000);
    assert(promotion.payable(1000) == 900);
}
```

**说明：** 两个策略使用相同输入输出约定，选择由客户端作出。本例以“分”为单位，并将优惠金额向下取整；真实定价需要统一货币、舍入和策略结果约束。函数策略适合简洁行为；需要复杂状态或多项相关操作时，可以使用策略类接口。

### 4.10 模板方法（Template Method）

**意图：** 在基类中定义固定流程，让子类实现或覆盖其中的步骤。这里的“模板”是算法骨架的含义，不是 C++ 的 `template` 关键字。

```cpp
#include <cassert>
#include <string>

class DataJob {
public:
    virtual ~DataJob() = default;
    std::string run() const { return save(process(load())); }
protected:
    virtual std::string load() const = 0;
    virtual std::string process(const std::string& data) const = 0;
    virtual std::string save(const std::string& data) const = 0;
};
class TextJob final : public DataJob {
protected:
    std::string load() const override { return "hello"; }
    std::string process(const std::string& data) const override {
        return "[" + data + "]";
    }
    std::string save(const std::string& data) const override {
        return "saved:" + data;
    }
};

int main() {
    TextJob concrete;
    const DataJob& job = concrete;
    assert(job.run() == "saved:[hello]");
}
```

**取舍：** 固定流程集中维护，但子类依赖基类的步骤约定。需要在运行时频繁替换步骤时，组合策略可能更灵活。不要在基类构造或析构中期待虚函数调用到派生类实现。本例用字符串模拟读取、处理和保存。

### 4.11 访问者（Visitor）

**意图：** 将对象结构与操作分离，在一组相对稳定的具体类型上添加新操作，而不不断修改这些类型。

```cpp
#include <cassert>
#include <string>
#include <utility>

struct NumberNode;
struct TextNode;
struct Visitor {
    virtual ~Visitor() = default;
    virtual void visit(const NumberNode& node) = 0;
    virtual void visit(const TextNode& node) = 0;
};
struct Node {
    virtual ~Node() = default;
    virtual void accept(Visitor& visitor) const = 0;
};
struct NumberNode final : Node {
    explicit NumberNode(int value) : value(value) {}
    void accept(Visitor& visitor) const override { visitor.visit(*this); }
    int value;
};
struct TextNode final : Node {
    explicit TextNode(std::string value) : value(std::move(value)) {}
    void accept(Visitor& visitor) const override { visitor.visit(*this); }
    std::string value;
};
struct SumVisitor final : Visitor {
    void visit(const NumberNode& node) override { sum += node.value; }
    void visit(const TextNode&) override {}
    int sum = 0;
};
struct RenderVisitor final : Visitor {
    void visit(const NumberNode& node) override {
        output += std::to_string(node.value);
    }
    void visit(const TextNode& node) override { output += node.value; }
    std::string output;
};

int main() {
    NumberNode number(42);
    TextNode text("answer=");
    const Node* nodes[] = {&text, &number};
    SumVisitor sum;
    RenderVisitor render;
    for (const Node* node : nodes) {
        node->accept(sum);
        node->accept(render);
    }
    assert(sum.sum == 42);
    assert(render.output == "answer=42");
}
```

**说明：** 第一次动态分派选择具体节点的 `accept()`；该函数中的 `*this` 具有具体节点类型，从而选择对应的 `visit()` 重载，再动态分派到具体访问者。这是经典访问者的双重分派。

**取舍：** 增加新操作比较容易，增加新节点类型需要更新访问者接口和所有访问者。C++17 中，封闭类型集合也可以用 `std::variant` 配合 `std::visit` 表达访问；它是相关思路的一种实现选择，不是所有多态层次的直接替代品。

## 五、相似模式辨析与选型

### 5.1 容易混淆的模式

| 对比 | 核心区别 | 判断问题 |
| --- | --- | --- |
| 简单工厂 / 工厂方法 | 前者集中选择产品，后者由创建者子类扩展创建行为 | 是否需要通过继承开放创建扩展点？ |
| 工厂方法 / 抽象工厂 | 前者关注产品创建扩展点，后者关注一族相关产品 | 是否需要整套产品相互匹配？ |
| 抽象工厂 / 建造者 | 前者选择产品系列，后者组织构造步骤 | 复杂性来自产品配套，还是对象构建过程？ |
| 适配器 / 桥接 | 前者协调已有接口，后者分离独立变化维度 | 接口已经不兼容，还是正在设计可独立扩展的两部分？ |
| 装饰器 / 代理 | 前者叠加职责，后者控制访问 | 是增加组合功能，还是控制真实对象何时及如何被访问？ |
| 外观 / 中介者 | 前者简化子系统入口，后者管理协作者之间的交互 | 是客户端使用太复杂，还是对象之间互相耦合？ |
| 状态 / 策略 | 前者围绕内部状态与转换，后者围绕算法选择 | 行为变化来自状态迁移，还是客户端选择另一算法？ |
| 策略 / 模板方法 | 前者通过组合替换算法，后者通过继承实现流程步骤 | 需要替换行为，还是保持固定流程并扩展步骤？ |
| 观察者 / 中介者 | 前者广播变化，后者决定交互规则 | 需要通知订阅者，还是集中协调联动逻辑？ |
| 命令 / 备忘录 | 前者封装操作，后者封装状态快照 | 要记录“做了什么”，还是保存“当时是什么状态”？ |

这些模式可以组合。例如撤销系统可以用命令管理操作顺序，再用备忘录保存难以反向计算的状态；主题系统可以用抽象工厂创建控件，再用装饰器添加附加显示行为。

### 5.2 从问题出发选择模式

| 观察到的问题 | 优先考虑 | 先检查的简单方案 |
| --- | --- | --- |
| 构造对象时参数太多、步骤有约束 | 建造者 | 配置结构体、命名清晰的工厂函数 |
| 客户端到处创建具体类型 | 工厂方法或简单工厂 | 集中创建并通过构造函数传入依赖 |
| 产品必须属于同一系列 | 抽象工厂 | 一个明确的系列配置与创建入口 |
| 老接口无法直接接入 | 适配器 | 小型转换函数 |
| 多个独立维度导致组合类爆炸 | 桥接 | 组合两个独立对象 |
| 层级对象需要统一操作 | 组合 | 简单树结构与遍历函数 |
| 功能需要按不同顺序叠加 | 装饰器 | 函数组合或小型包装对象 |
| 大量对象重复保存相同数据 | 享元 | 先测量内存，再评估共享只读数据 |
| 初始化昂贵且不一定被使用 | 延迟加载代理 | 局部延迟创建逻辑 |
| 一段算法有多个实现 | 策略 | Lambda、函数对象或模板参数 |
| 状态相关分支难以维护 | 状态 | 显式状态枚举和转换表 |
| 多处需要感知模型变化 | 观察者 | 生命周期明确的回调机制 |
| 操作需要排队或撤销 | 命令 | 保存函数对象或操作记录 |
| 控件之间互相调用越来越多 | 中介者 | 在一个协调模块内集中规则 |
| 对稳定类型集合不断增加操作 | 访问者 | 普通函数、`variant` 和 `visit` |

## 六、现代 C++ 与 Qt 项目中的实践

### 6.1 先设计所有权，再设计协作

| 关系 | 常用表达 | 例子 |
| --- | --- | --- |
| 独占拥有 | 值成员、`std::unique_ptr<T>` | 文件夹拥有节点、代理拥有真实对象 |
| 共享拥有 | `std::shared_ptr<T>` | 多个享元使用者共享只读字形 |
| 借用对象 | `T&`、非拥有 `T*` | 适配器借用设备、命令借用接收者 |
| 不延长生命周期的观察 | `std::weak_ptr<T>` | 回调在使用对象前检查它是否仍然存在 |

避免“为了安全全部使用 `shared_ptr`”。共享所有权会使销毁时机更难推断，双向强引用还会形成循环；应先明确谁负责结束对象的生命周期。

含有非拥有引用的类也要考虑复制和移动：复制后引用通常仍然指向原对象，不会自动重绑定到新对象的成员。本指南的中介者示例因此禁止复制。

### 6.2 运行时多态与编译时多态

- **运行时选择实现：** 可使用虚函数接口或 `std::function`，适用于动态配置和实现集合较开放的场景。
- **编译时确定实现：** 可使用模板策略和函数对象，便于静态检查及优化，但可能增加模板复杂度和代码体积。
- **类型集合固定、操作不断增加：** 可评估 `std::variant` 与 `std::visit`，同时接受类型集合变更时需要更新相关访问操作的成本。

设计模式描述的是协作意图，并不强制要求每个角色都成为一个含虚函数的类。简单策略可以只是 Lambda，简单外观可以只是一个协调函数。

### 6.3 异常、线程与资源释放

资源由 RAII 对象管理，减少手动 `new` / `delete`。工厂返回智能指针，构建失败时让已完成的子对象自动清理；状态切换先建立新状态，再替换旧状态。

对缓存、订阅表、命令队列等共享可变数据，需要明确线程归属或同步方式。不要因为局部静态初始化或智能指针引用计数具有某些并发保证，就推断整个模式实现是线程安全的。

执行用户回调时应谨慎持锁，否则可能产生重入问题或死锁；使用快照或锁外通知也要明确取消订阅和对象销毁的语义。

### 6.4 应用于当前目录中的 Qt 学习项目

可以围绕一个“小型文档编辑器”练习模式，而不是把每个按钮都设计成新的抽象层：

1. 文档模型管理内容，界面显示模型，导出服务处理格式转换。
2. 用策略封装不同导出算法，必要时用工厂统一创建导出器。
3. 用命令管理编辑操作和历史顺序，用备忘录保存复杂操作前的状态。
4. 用观察者思路通知多个视图更新；Qt 的信号与槽提供了相关的解耦通信机制。
5. 用中介者或协调类集中管理“选中内容后启用哪些操作”等界面联动规则。
6. 只有出现多套配套界面或多维实现变化时，再评估抽象工厂和桥接。

Qt 对象的父子管理与 C++ 智能指针管理要有明确的所有权边界，避免两个独立机制同时负责删除同一对象。涉及信号与槽时，还应结合对象生命周期、连接方式和线程归属设计通知行为；本文的纯 C++ 回调示例不能直接代表 Qt 的全部连接语义。

## 七、学习顺序与练习

### 7.1 推荐顺序

1. 先掌握类、虚函数、RAII、引用、智能指针、Lambda 和基本 STL。
2. 学习策略、简单工厂、工厂方法、适配器、外观，理解如何隔离变化。
3. 学习观察者、命令、装饰器、组合，理解对象协作与生命周期。
4. 学习状态、模板方法、建造者、抽象工厂，理解流程和构造扩展点。
5. 最后学习桥接、原型、责任链、中介者、备忘录、享元、访问者、解释器，并理解标准库迭代器如何落实遍历抽象。
6. 单独评估单例的代价，练习把全局访问改为显式依赖注入。

### 7.2 实践任务

| 练习 | 要求 | 应检查的边界 |
| --- | --- | --- |
| 替换价格策略 | 增加满额优惠，并保持调用方接口稳定 | 负数、舍入、结果不能为负 |
| 扩展主题工厂 | 增加高对比度主题和一种新控件 | 比较增加系列与增加产品类型的修改范围 |
| 文本包装器 | 组合前缀、括号和脱敏处理 | 顺序变化、空输入、异常传播 |
| 编辑器历史 | 支持多个命令的撤销和重做 | 空历史、撤销后新编辑、接收者生命周期 |
| 模型订阅 | 自动取消订阅并支持通知期间退订 | 回调对象销毁、重入、回调抛异常 |
| 对象树 | 增加节点查询和删除 | 空树、深层树、节点所有权 |
| 状态机 | 为播放器增加暂停状态 | 非法事件、完整转换表、连续事件 |
| 享元缓存 | 测量大量重复对象的内存消耗 | 缓存容量、无人使用的对象释放、并发访问 |

### 7.3 常见误区

- 为了“使用模式”提前搭建复杂继承树，实际变化点却尚未出现。
- 用单例替代所有依赖传递，导致测试互相影响。
- 将简单工厂等同于 GoF 工厂方法，忽略创建者的扩展机制。
- 用裸指针同时表达拥有和借用关系，却没有写清楚生命周期。
- 把 `clone()` 的成员复制误认为所有资源都已经深拷贝。
- 观察者只实现订阅，不实现退订和销毁规则。
- 状态转换只覆盖正常路径，没有验证非法事件。
- 把“可以撤销内存操作”误认为“可以撤销所有外部副作用”。

判断一个设计是否合适，应看它是否让真实的变化更局部、职责更清楚、生命周期更可靠，而不是看其中出现了多少模式名称。
