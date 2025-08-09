# C++ std::erase（C++20 ) auto 关键字

```c++

#include <vector>
#include <algorithm>
#include <iostream>

struct S {
    S() { puts("S()"); }
    S(const S&) { puts("S(const S&)"); }
    S(S&&) { puts("S(S&&)"); }
    ~S() { puts("~S()"); }
};

int main() {
    std::vector<int> data{1,2,2,1};
    std::erase(data, data[0]); // OOPS
    // data == {2,1} (implementation specific result)
    for (auto v : data)
        std::cout << v << " ";
    std::cout << "\n";

    data = {1,2,2,1};
    std::erase(data, auto(data[0])); // correct
    // data == {2,2}
    for (auto v : data)
        std::cout << v << " ";
    std::cout << "\n";

    // direct initialization, all the way through, no copies
    auto v = auto(auto(auto(S{})));
}
```

这段代码涉及 C++ 中的 std::erase（C++20 引入）、auto 关键字的用法，以及对象构造和析构的行为。我将逐步解释代码的每一部分，并分析其输出和潜在问题。

------

代码结构

1. **结构体 S**：定义了一个简单的类，用于跟踪对象的构造和析构。
2. **主函数**：
   - 使用 std::vector 和 std::erase 删除特定元素，展示两种用法（正确和错误的）。
   - 使用 auto 关键字进行类型推导，结合 S 的构造行为。

------

逐步分析

1. 结构体 S 的定义

cpp

```cpp
struct S {
    S() { puts("S()"); }              // 默认构造函数
    S(const S&) { puts("S(const S&)"); } // 拷贝构造函数
    S(S&&) { puts("S(S&&)"); }        // 移动构造函数
    ~S() { puts("~S()"); }            // 析构函数
};
```

- 这个结构体用于输出构造和析构的日志，帮助我们观察对象的生命周期。
- 提供了默认构造、拷贝构造和移动构造三种方式。

------

2. 第一个 std::erase 用法（错误）

cpp

```cpp
std::vector<int> data{1, 2, 2, 1};
std::erase(data, data[0]); // OOPS
// data == {2, 1} (implementation specific result)
for (auto v : data)
    std::cout << v << " ";
std::cout << "\n";
```

- **初始化**: data 被初始化为 {1, 2, 2, 1}。
- **问题**: std::erase(data, data[0]) 的调用有未定义行为的风险。
  - data[0] 是 data 中的第一个元素（值为 1）。
  - std::erase（C++20 引入）会移除容器中所有等于第二个参数的元素。
  - 但 data[0] 是一个引用，指向 data 内的元素。当 std::erase 修改 data（例如移除第一个 1）时，data[0] 可能变成悬垂引用或指向被移动后的元素，导致未定义行为。
- **可能的结果**: 
  - 在某些实现中，data[0] 在移除后可能指向下一个元素（例如 2），导致 std::erase 只移除了部分 1，结果可能是 {2, 1}。
  - 具体行为依赖实现，未定义行为可能导致崩溃或其他意外结果。
- **输出**: {2, 1}（注释中提到的实现特定结果，但不可靠）。

------

3. 第二个 std::erase 用法（正确）

cpp

```cpp
data = {1, 2, 2, 1};
std::erase(data, auto(data[0])); // correct
// data == {2, 2}
for (auto v : data)
    std::cout << v << " ";
std::cout << "\n";
```

- **初始化**: data 被重新赋值为 {1, 2, 2, 1}。
- **修正**: 使用 auto(data[0])。
  - data[0] 是 int&（对 data 中第一个元素的引用）。
  - auto 推导会移除引用，得到 int 类型，并拷贝 data[0] 的值（1）。
  - 因此，std::erase(data, 1) 接收的是一个独立的 int 值，而不是容器内的引用。
- **执行**: std::erase 安全地移除所有值为 1 的元素，结果是 {2, 2}。
- **输出**: 2 2。

------

4. auto 和 S 的构造

cpp

```cpp
auto v = auto(auto(auto(S{})));
```

- **逐步分解**:

  1. S{}: 创建一个临时 S 对象，调用默认构造函数。
     - 输出: S()
  2. auto(S{}): auto 推导类型为 S，结果是 S 类型的一个副本。
     - 因为是临时对象传递给 auto，可能触发移动构造（S(S&&)），但编译器可能优化为直接构造。
     - 输出: S(S&&)（如果移动）或无额外输出（如果优化）。
  3. auto(auto(S{})): 再次推导，类型仍是 S，可能触发另一次移动。
     - 输出: S(S&&)（如果移动）。
  4. auto(auto(auto(S{}))): 再推导，仍然是 S，赋值给 v。
     - 输出: S(S&&)（如果移动）。

- **优化可能**: 现代编译器（带 RVO，Return Value Optimization）可能省略不必要的拷贝或移动，直接构造 v，只输出一次 S()。

- **析构**: 每次临时对象销毁时调用析构函数。

- **实际输出**（未优化情况下）:

  ```text
  S()         // 初始 S{}
  S(S&&)      // 第一次 auto
  ~S()        // 初始临时对象析构
  S(S&&)      // 第二次 auto
  ~S()        // 上一个临时对象析构
  S(S&&)      // 赋值给 v
  ~S()        // 上一个临时对象析构
  ```

  最后 v 析构时:

  ```text
  ~S()        // v 离开作用域
  ```

- **优化后输出**: 如果 RVO 生效，可能只有:

  ```text
  S()         // 直接构造 v
  ~S()        // v 析构
  ```

------

完整输出（假设无优化）

```text
2 1         // 第一个 std::erase (未定义行为的结果)
2 2         // 第二个 std::erase (正确结果)
S()         // S{} 创建
S(S&&)      // 第一次 auto
~S()        // 临时对象析构
S(S&&)      // 第二次 auto
~S()        // 临时对象析构
S(S&&)      // 赋值给 v
~S()        // 临时对象析构
~S()        // v 析构
```

优化后输出（更现实）

如果编译器优化生效:

```text
2 1         // 第一个 std::erase
2 2         // 第二个 std::erase
S()         // 直接构造 v
~S()        // v 析构
```

------

关键点

1. **std::erase 的陷阱**:
   - 直接使用容器内元素的引用（如 data[0]）可能导致未定义行为。
   - 使用 auto 强制拷贝值是安全的做法。
2. **auto 的推导**:
   - auto 移除引用并推导值类型。
   - 多重 auto 对值类型无影响，但可能触发构造/移动。
3. **构造和析构**:
   - S 的日志展示了对象的生命周期。
   - 编译器优化（如 RVO）会减少不必要的拷贝或移动。

如果你有具体问题或想深入某部分，请告诉我！