

#  C++ 容器操作的异常安全性（Exception Safety)

```C++
#include <vector>
#include <stdexcept>
#include <iostream>
#include <set>

// A custom type that can throw on copy as requested
struct TroubleMaker {
    TroubleMaker() {}
    TroubleMaker(const TroubleMaker&) {
        if (now_ > 0) --now_;
        if (now_ == 0) throw std::runtime_error("I'm failing");
    }
    auto operator<=>(const TroubleMaker&) const = default;
    static int now_;
};

int TroubleMaker::now_ = -1;

int main() {
    std::vector<TroubleMaker> data{{}, {}, {}, {}, {}};

    std::vector<TroubleMaker> strong;
    try {
        // Throw an exception when copying the 3rd element
        TroubleMaker::now_ = 3;
        // std::vector provides strong exception guarantee
        strong.insert(strong.end(), data.begin(), data.end());
        // Note: technically this specific combination isn't strong 
        // guarantee as described by the standard, but both GCC and 
        // Clang implement it as strong guarantee
    } catch (...) {
        // strong.size() == 0
        // because an exception was thrown, state was not changed
        std::cerr << "strong.size() == " << strong.size() << '\n';
    }

    std::multiset<TroubleMaker> weak;
    try {
        // Throw an exception when copying the 3rd element
        TroubleMaker::now_ = 3;
        // map/set provide basic exception guarantee
        weak.insert(data.begin(), data.end());
    } catch (...) {
        // weak.size() == 2
        // invariants are maintained, resources are not leaked
        // but the operation partially succeeded
        std::cerr << "weak.size() == " << weak.size() << '\n';
    }
}
```

这段代码展示了 C++ 中容器操作的异常安全性（Exception Safety），通过自定义类型 TroubleMaker 和其拷贝构造函数中的异常抛出，比较了 std::vector 和 std::multiset 在插入操作时的异常保证级别。以下是对代码的详细解释：

------

**代码结构**

1. **头文件**：
   - <vector>：提供 std::vector。
   - <stdexcept>：提供 std::runtime_error。
   - <iostream>：提供 std::cerr。
   - <set>：提供 std::multiset。
2. **主要内容**：
   - 定义 TroubleMaker 类，可控地抛出异常。
   - 测试 std::vector::insert 的强异常保证。
   - 测试 std::multiset::insert 的基本异常保证。

------

**代码逐步解释**

**1. TroubleMaker 类型**

cpp

```cpp
struct TroubleMaker {
    TroubleMaker() {}
    TroubleMaker(const TroubleMaker&) {
        if (now_ > 0) --now_;
        if (now_ == 0) throw std::runtime_error("I'm failing");
    }
    auto operator<=>(const TroubleMaker&) const = default;
    static int now_;
};
int TroubleMaker::now_ = -1;
```

- **TroubleMaker**：
  - 默认构造函数：空实现。
  - 拷贝构造函数：
    - now_ 是静态计数器，控制异常抛出时机。
    - 如果 now_ > 0，递减计数器。
    - 如果 now_ == 0，抛出 std::runtime_error。
  - operator<=>：C++20 三路比较运算符，默认实现（用于 std::multiset 的排序）。
  - now_：静态成员，初始为 -1（不抛异常）。
- **目的**：
  - 模拟在特定拷贝操作时抛出异常，用于测试容器的异常安全性。

------

**2. 初始化测试数据**

cpp

```cpp
std::vector<TroubleMaker> data{{}, {}, {}, {}, {}};
```

- **data**：
  - 创建包含 5 个 TroubleMaker 对象的 std::vector。
  - 使用默认构造函数初始化，每个对象构造时不抛异常。

------

**3. std::vector 的强异常保证**

cpp

```cpp
std::vector<TroubleMaker> strong;
try {
    TroubleMaker::now_ = 3;
    strong.insert(strong.end(), data.begin(), data.end());
} catch (...) {
    std::cerr << "strong.size() == " << strong.size() << '\n';
}
```

- **TroubleMaker::now_ = 3**：

  - 设置计数器为 3，表示第 3 次拷贝时抛出异常。

- **strong.insert**：

  - 将 data 的 5 个元素插入到 strong 的末尾。
  - 插入过程：
    1. 拷贝第 1 个元素：now_ = 2。
    2. 拷贝第 2 个元素：now_ = 1。
    3. 拷贝第 3 个元素：now_ = 0，抛出异常。

- **强异常保证（Strong Exception Guarantee）**：

  - 如果操作因异常失败，对象状态保持不变（回滚）。
  - std::vector::insert 在标准中未明确要求强保证，但 GCC 和 Clang 的实现提供了强保证。

- **结果**：

  - 异常抛出后，strong 未被修改，strong.size() == 0。

- **输出**：

  ```text
  strong.size() == 0
  ```

------

**4. std::multiset 的基本异常保证**

cpp

```cpp
std::multiset<TroubleMaker> weak;
try {
    TroubleMaker::now_ = 3;
    weak.insert(data.begin(), data.end());
} catch (...) {
    std::cerr << "weak.size() == " << weak.size() << '\n';
}
```

- **TroubleMaker::now_ = 3**：

  - 同上，第 3 次拷贝抛出异常。

- **weak.insert**：

  - 将 data 的 5 个元素插入到 std::multiset。
  - 插入过程：
    1. 插入第 1 个元素：成功，now_ = 2。
    2. 插入第 2 个元素：成功，now_ = 1。
    3. 插入第 3 个元素：now_ = 0，抛出异常。

- **基本异常保证（Basic Exception Guarantee）**：

  - 标准要求 std::multiset::insert 提供基本保证：
    - 无资源泄漏。
    - 容器保持有效状态（不变性维持）。
    - 但可能部分完成操作。

- **结果**：

  - 前 2 个元素已插入，异常后停止，weak.size() == 2。

- **输出**：

  ```text
  weak.size() == 2
  ```

------

**关键技术点**

1. **异常安全性**：
   - **强保证**：操作要么完全成功，要么无变化。
   - **基本保证**：操作可能部分完成，但容器有效且无泄漏。
   - **无保证**：操作失败可能导致未定义行为（未在此展示）。
2. **std::vector::insert**：
   - 标准中未强制强保证，但主流实现（如 GCC、Clang）提供。
   - 内部可能使用临时缓冲区，失败时丢弃。
3. **std::multiset::insert**：
   - 标准要求基本保证。
   - 红黑树结构逐个插入，异常时保留已插入元素。
4. **TroubleMaker**：
   - 通过静态计数器模拟异常，测试容器行为。

------

**输出总结**

```text
strong.size() == 0
weak.size() == 2
```

------

**可能的改进或注意事项**

1. **异常类型**：
   - 使用 catch (const std::runtime_error& e) 捕获具体异常并打印信息。
2. **验证内容**：
   - 检查 weak 中的元素值（需为 TroubleMaker 添加输出支持）。
3. **标准澄清**：
   - 注释提到 vector::insert 的强保证非标准要求，可添加编译器版本验证。

------

**总结**

- **std::vector**：展示强异常保证，异常后状态不变。
- **std::multiset**：展示基本异常保证，异常后部分完成。
- **用途**：理解容器在异常场景下的行为，确保程序健壮性。

如果你有具体问题（例如其他容器的异常保证），欢迎提问！