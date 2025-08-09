# C++ 20 std::atomic<<std::shared_ptr>>, std::atomic<<std::weak_ptr>>

While the *std::shared_ptr* has limited use in single-threaded code, in multi-threaded code, the notion of shared ownership (even if temporary) is fairly common.

However, while multiple instances of *std::shared_ptr* that point to the same memory can be managed safely in multi-threaded code, operations on a single instance of a *std::shared_ptr* are not thread-safe.

This is why C++20 introduced *std::atomic<<std::shared_ptr<T>>>*and *std::atomic<<std::weak_ptr<T>>*, which can be used to implement thread-safe shared data structures.

Unfortunately, the GCC libstdc++ implementation is not lock-free (Clang libc++ doesn’t implement it), so the resulting performance can be disappointing.



```C++
#include <memory>
#include <atomic>
#include <optional>

// Example of a simple thread-safe 
// and resource-safe stack datastructure
template <typename T>
struct Stack {
    struct Node {
        T value;
        std::shared_ptr<Node> prev;
    };

    void push(T value) {
        // Make a new node, setting the current head as its previous value
        auto active = std::make_shared<Node>(std::move(value), head_.load());

        // Another thread can come in a modify the current head,
        // so check if active->prev is still head_
        // - if it's not, update active->prev to the current value of head_ and loop
        // - if it is, update head_ to active
        while (not head_.compare_exchange_weak(active->prev, active));
    }

    std::optional<T> pop() {
        // Load the current head
        auto active = head_.load();

        // Another thread can come in and modify the current head,
        // so check if the head has changed (i.e. head_ != active)
        // - if it has changed, update active to the current head and loop
        // - if it hasn't changed, update the head to the previous element on the stack
        while (active != nullptr && not head_.compare_exchange_weak(active, active->prev));

        // If we didn't run out of elements, return the value
        if (active != nullptr) return {std::move(active->value)};
        else return std::nullopt;
    }

private:
    std::atomic<std::shared_ptr<Node>> head_;
};

#include <thread>
#include <array>
#include <ranges>

int main() {
    Stack<int> stack;
    std::array<std::jthread, 4> writers;
    std::array<std::jthread, 4> readers;

    // start the writers
    for (auto& t : writers)
        t = std::jthread{[&stack]{ 
            // write 100 values
            for (auto _ : std::views::repeat(true, 100))
                stack.push(42);
        }};
    // start the readers
    for (auto& t : readers)
        t = std::jthread{[&stack]{ 
            // read 100 values
            for (auto _ : std::views::repeat(true, 100))
                while (stack.pop() == std::nullopt);
        }};
}
```



这段代码实现了一个线程安全的、基于链表的无锁栈（lock-free stack），使用 C++ 的现代特性（如智能指针、原子操作和线程支持）来确保资源安全和并发访问的正确性。以下是对代码的详细解释：

------

**代码结构**

1. **头文件**：
   - <memory>：提供 std::shared_ptr 和 std::make_shared。
   - <atomic>：提供 std::atomic 用于无锁操作。
   - <optional>：提供 std::optional 表示可选值。
   - <thread>：提供 std::jthread（C++20）。
   - <array>：提供 std::array。
   - <ranges>：提供 std::views::repeat（C++20）。
2. **主要内容**：
   - Stack<T>：线程安全无锁栈。
   - main：测试多线程推入和弹出。

------

**代码逐步解释**

**1. Stack<T> 模板**

cpp

```cpp
template <typename T>
struct Stack {
    struct Node {
        T value;
        std::shared_ptr<Node> prev;
    };
```

- **Node**：
  - 链表节点，包含值 value 和指向前一个节点的智能指针 prev。
  - 使用 std::shared_ptr 确保节点生命周期安全。

cpp

```cpp
private:
    std::atomic<std::shared_ptr<Node>> head_;
```

- **head_**：
  - 栈顶指针，使用 std::atomic 保证线程安全的读写。

------

**2. push 方法**

cpp

```cpp
void push(T value) {
    auto active = std::make_shared<Node>(std::move(value), head_.load());
    while (not head_.compare_exchange_weak(active->prev, active));
}
```

- **功能**：
  - 将值推入栈顶。
- **步骤**：
  1. **auto active = std::make_shared<Node>(...)**：
     - 创建新节点，值从 value 移动，prev 初始化为当前 head_。
  2. **head_.compare_exchange_weak(active->prev, active)**：
     - 比较并交换（CAS，Compare-And-Swap）：
       - 如果 head_ == active->prev，则将 head_ 更新为 active。
       - 如果不相等（其他线程修改了 head_），更新 active->prev 为新 head_，重试。
     - weak 版本可能虚假失败，但性能更高，适合循环。
- **无锁**：
  - CAS 确保线程安全，无需显式锁。

------

**3. pop 方法**

cpp

```cpp
std::optional<T> pop() {
    auto active = head_.load();
    while (active != nullptr && not head_.compare_exchange_weak(active, active->prev));
    if (active != nullptr) return {std::move(active->value)};
    else return std::nullopt;
}
```

- **功能**：
  - 从栈顶弹出一个值，若栈空则返回 std::nullopt。
- **步骤**：
  1. **auto active = head_.load()**：
     - 加载当前栈顶。
  2. **head_.compare_exchange_weak(active, active->prev)**：
     - 如果 head_ == active，将 head_ 更新为 active->prev。
     - 如果不相等（其他线程修改），更新 active 并重试。
     - active == nullptr 时退出，表示栈空。
  3. **返回值**：
     - 若 active != nullptr，返回移动后的值。
     - 否则返回 std::nullopt。
- **无锁**：
  - 使用 CAS 实现线程安全。

------

**4. main 函数测试**

cpp

```cpp
int main() {
    Stack<int> stack;
    std::array<std::jthread, 4> writers;
    std::array<std::jthread, 4> readers;

    for (auto& t : writers)
        t = std::jthread{[&stack]{ 
            for (auto _ : std::views::repeat(true, 100))
                stack.push(42);
        }};
    for (auto& t : readers)
        t = std::jthread{[&stack]{ 
            for (auto _ : std::views::repeat(true, 100))
                while (stack.pop() == std::nullopt);
        }};
}
```

- **stack**：
  - Stack<int> 实例。
- **writers**：
  - 4 个写线程，每个推送 100 个 42。
  - 使用 std::jthread（自动加入线程）。
  - std::views::repeat(true, 100)：重复 100 次。
- **readers**：
  - 4 个读线程，每个尝试弹出 100 个值。
  - 若 pop() 返回 std::nullopt，重试直到成功。
- **行为**：
  - 总共推送 4 * 100 = 400 个值，读取 4 * 100 = 400 个值。
  - 无锁设计确保并发正确性。

------

**关键技术点**

1. **无锁编程**：
   - 使用 std::atomic 和 CAS 实现线程安全。
2. **std::shared_ptr**：
   - 管理节点生命周期，防止内存泄漏。
3. **std::optional**：
   - 表示 pop 的可选返回值。
4. **std::jthread**：
   - C++20 线程类，析构时自动加入。
5. **ABA 问题**：
   - std::shared_ptr 的引用计数避免传统无锁栈的 ABA 问题。

------

**可能的改进或注意事项**

1. **性能优化**：
   - compare_exchange_weak 可替换为 strong 减少虚假失败。
2. **验证**：
   - 可添加计数器检查推送和弹出总数。
3. **异常安全**：
   - push 中 make_shared 若抛异常，需处理。
4. **输出**：
   - 可打印操作结果。

------

**总结**

- **功能**：线程安全的无锁栈。
- **实现**：使用原子操作和智能指针。
- **测试**：多线程并发推入和弹出。
- **用途**：高并发场景下的数据结构。

如果你有具体问题（例如 ABA 问题细节），欢迎提问！