# std::barrier 和 std::latch

std::barrier 和 std::latch 是 C++20 引入的两个线程同步工具，分别用于协调多线程的执行。它们都定义在 <barrier> 和 <latch> 头文件中，提供了比传统的互斥锁和条件变量更高层次的同步机制。以下是对它们的详细解释和对比。

------

1. std::barrier

定义

std::barrier 是一个可重用的同步原语，用于让一组线程在某个“阶段”（phase）完成时同步。它允许多个线程到达一个屏障点（barrier point），等待所有参与线程到达后一起继续执行。

头文件

cpp

```cpp
#include <barrier>
```

构造函数

cpp

```cpp
std::barrier b(num_threads, completion_function = std::noop); // num_threads 是参与线程数
```

- num_threads：预期到达屏障的线程数。
- completion_function：可选的完成函数，在所有线程到达后、屏障释放前调用。

主要成员函数

- **arrive_and_wait()**：
  - 当前线程到达屏障并等待其他线程。
  - 当所有 num_threads 个线程到达后，完成函数（如果有）被调用，然后所有线程继续。
- **arrive_and_drop()**：
  - 当前线程到达屏障但退出参与，不等待后续阶段。

示例代码

cpp

```cpp
#include <iostream>
#include <barrier>
#include <thread>
#include <vector>

void worker(std::barrier<>& b, int id) {
    std::cout << "线程 " << id << " 开始阶段 1\n";
    b.arrive_and_wait(); // 等待所有线程完成阶段 1
    std::cout << "线程 " << id << " 开始阶段 2\n";
    b.arrive_and_wait(); // 等待所有线程完成阶段 2
}

int main() {
    const int num_threads = 3;
    std::barrier sync_point(num_threads);

    std::vector<std::thread> threads;
    for (int i = 0; i < num_threads; ++i) {
        threads.emplace_back(worker, std::ref(sync_point), i);
    }

    for (auto& t : threads) {
        t.join();
    }
    return 0;
}
```

输出（可能顺序不同）

```text
线程 0 开始阶段 1
线程 1 开始阶段 1
线程 2 开始阶段 1
线程 0 开始阶段 2
线程 1 开始阶段 2
线程 2 开始阶段 2
```

特点

- **可重用**：同一个 std::barrier 对象可以用于多个同步阶段。
- **阶段性同步**：适合需要分阶段执行的任务。
- **完成函数**：可以在所有线程到达时执行额外逻辑（如重置状态）。

------

2. std::latch

定义

std::latch 是一个一次性同步原语，用于等待一组线程完成某个任务。它类似于一个计数器，初始化时设置一个计数值，线程通过减少计数来“到达”，当计数减到 0 时，所有等待的线程被释放。

头文件

cpp

```cpp
#include <latch>
```

构造函数

cpp

```cpp
std::latch l(count); // count 是初始计数值
```

- count：需要“到达”的次数，通常是线程或任务的数量。

主要成员函数

- **count_down()**：
  - 将计数器减 1，不阻塞。
- **wait()**：
  - 阻塞当前线程，直到计数器减到 0。
- **try_wait()**：
  - 检查计数器是否为 0，返回 true 如果已完成，否则返回 false。
- **arrive_and_wait()**：
  - 相当于 count_down() 后立即调用 wait()。

示例代码

cpp

```cpp
#include <iostream>
#include <latch>
#include <thread>
#include <vector>

void worker(std::latch& l, int id) {
    std::cout << "线程 " << id << " 正在工作\n";
    l.count_down(); // 完成工作，减少计数
}

int main() {
    const int num_threads = 3;
    std::latch latch(num_threads);

    std::vector<std::thread> threads;
    for (int i = 0; i < num_threads; ++i) {
        threads.emplace_back(worker, std::ref(latch), i);
    }

    latch.wait(); // 主线程等待所有工作线程完成
    std::cout << "所有线程完成\n";

    for (auto& t : threads) {
        t.join();
    }
    return 0;
}
```

输出（可能顺序不同）

```text
线程 0 正在工作
线程 1 正在工作
线程 2 正在工作
所有线程完成
```

特点

- **一次性**：计数器减到 0 后，std::latch 无法重用。
- **简单同步**：适合等待一组任务完成一次的场景。
- **灵活性**：可以由多个线程多次调用 count_down()，只要总次数匹配初始值。

------

std::barrier 与 std::latch 的对比

| 特性     | std::barrier             | std::latch                  |
| -------- | ------------------------ | --------------------------- |
| 可重用性 | 可重用（支持多阶段）     | 一次性（计数器到 0 后失效） |
| 线程数量 | 固定参与线程数           | 计数器可以由任意线程减少    |
| 等待行为 | 所有线程到达后一起继续   | 计数到 0 后等待线程继续     |
| 完成函数 | 支持                     | 不支持                      |
| 使用场景 | 多阶段同步（如并行算法） | 一次性任务完成（如初始化）  |

------

使用场景

- **std::barrier**：
  - 并行计算中的阶段同步（如矩阵运算的每一步）。
  - 需要多次同步的循环任务。
- **std::latch**：
  - 等待一组线程完成初始化或一次性任务。
  - 主线程等待多个工作线程完成前置工作。

------

注意事项

1. **线程安全性**：
   - 两者的成员函数都是线程安全的，但需要确保正确使用（如避免超出计数或线程数）。
2. **性能**：
   - std::latch 通常更轻量，因为它是一次性工具。
   - std::barrier 更复杂，支持重用和完成函数。
3. **异常**：
   - 如果构造参数（如 num_threads 或 count）为负数，会抛出 std::system_error。

------

总结

- **std::barrier** 是一个强大的工具，适用于需要多阶段同步的场景，支持重用和完成逻辑。
- **std::latch** 更简单，适合一次性等待任务完成的场景，类似于一个倒计数器。

两者都是 C++20 提供的现代化同步工具，替代了部分传统条件变量的复杂用法。如果你有具体问题或使用场景，欢迎进一步讨论！