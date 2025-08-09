# std::cout 同步问题

```C++
void f() { std::cout << "Hello "; }

struct F {
    void operator()() const { std::cout << "parallel world "; }
};

int main()
{
    std::thread t1{f};      // f() 在独立线程中执行
    std::thread t2{F()};    // F()() 在独立线程中执行

    t1.join();
    t2.join();
}  // 剩下一个糟糕的 BUG
```



这段代码存在一个潜在的 **竞态条件（Race Condition）**，导致输出结果可能不一致。问题出在 **多个线程同时访问共享资源 `std::cout` 而没有同步机制**。

---

### **问题分析**
1. **`std::cout` 不是线程安全的**  
   - `std::cout` 是一个全局对象，多个线程同时调用 `operator<<` 会导致数据竞争（Data Race）。
   - 标准不保证 `std::cout` 的插入操作是原子的，因此可能出现输出交错（Interleaving）。

2. **可能的输出（非确定性）**  
   由于线程调度顺序不确定，可能的输出包括：
   - `Hello parallel world` ✅（理想情况）
   - `parallel world Hello` ✅（顺序交换，但仍然完整）
   - `Hparallel world ello ` ❌（字符交错，数据损坏）
   - `Hellparallel world o ` ❌（部分字符丢失）

3. **标准规定**  
   C++ 标准规定：
   > 对标准流对象（如 `std::cout`）的 **无同步并发访问** 是未定义行为（UB）。  
   > （见 [C++ Standard §[iostream.objects]/2](https://eel.is/c++draft/iostream.objects#2)）

---

### **如何修复？**
#### **方法 1：使用互斥锁（`std::mutex`）**
```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::mutex cout_mutex;  // 全局锁

void f() {
    std::lock_guard<std::mutex> lock(cout_mutex);
    std::cout << "Hello ";
}

struct F {
    void operator()() const {
        std::lock_guard<std::mutex> lock(cout_mutex);
        std::cout << "parallel world ";
    }
};

int main() {
    std::thread t1{f};
    std::thread t2{F()};

    t1.join();
    t2.join();
}
```
**输出（确定性的）**：  
`Hello parallel world` 或 `parallel world Hello`（顺序仍不确定，但不会字符交错）。

#### **方法 2：C++20 的 `std::osyncstream`（同步流）**
```cpp
#include <iostream>
#include <thread>
#include <syncstream>

void f() {
    std::osyncstream(std::cout) << "Hello ";
}

struct F {
    void operator()() const {
        std::osyncstream(std::cout) << "parallel world ";
    }
};

int main() {
    std::thread t1{f};
    std::thread t2{F()};

    t1.join();
    t2.join();
}
```
**特点**：  
- 更高效（锁粒度更细）。
- 避免手动管理 `std::mutex`。

---

### **关键点总结**
| 问题                     | 原因                | 修复方法                                      |
| ------------------------ | ------------------- | --------------------------------------------- |
| 多线程同时写 `std::cout` | 数据竞争（UB）      | 加锁（`std::mutex`）或使用 `std::osyncstream` |
| 输出顺序不确定           | 线程调度非确定性    | 如果顺序重要，需额外同步（如条件变量）        |
| 字符可能交错             | `operator<<` 非原子 | 保证每个完整消息原子化输出                    |

**最终建议**：  
- 如果只是简单调试，可以容忍非确定性顺序，但必须避免字符交错（用锁）。  
- 如果是生产代码，优先使用 `std::osyncstream`（C++20）。