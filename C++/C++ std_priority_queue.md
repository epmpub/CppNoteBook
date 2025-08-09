# std::priority_queue

在 C++ 中，std::priority_queue 是一个基于优先级队列的容器适配器，定义在 <queue> 头文件中。它提供了一种数据结构，其中元素按照某种优先级顺序排列，默认情况下是最大值优先（大顶堆）。std::priority_queue 通常用于需要快速访问最高（或最低）优先级元素的场景，例如任务调度、Dijkstra 算法等。

以下是对 std::priority_queue 的详细解释：

------

定义和基本概念

- **头文件**：<queue>

- **底层实现**：

  - 默认使用 std::vector 作为底层容器。
  - 通过堆算法（std::make_heap, std::push_heap, std::pop_heap）维护优先级顺序。

- **模板参数**：

  cpp

  ```cpp
  template<
      class T,
      class Container = std::vector<T>,
      class Compare = std::less<typename Container::value_type>
  > class priority_queue;
  ```

  - T：存储的元素类型。
  - Container：底层容器类型（默认 std::vector<T>）。
  - Compare：比较器（默认 std::less<T>，即大顶堆，最大值优先）。

- **默认行为**：

  - 最大值优先（堆顶是最大元素）。
  - 可以通过自定义比较器改为最小值优先（小顶堆）。

------

主要成员函数

1. **构造函数**：

   cpp

   ```cpp
   std::priority_queue<int> pq; // 空优先级队列
   std::vector<int> vec{3, 1, 4, 1, 5};
   std::priority_queue<int> pq2(vec.begin(), vec.end()); // 从范围初始化
   ```

   - 默认构造空队列。
   - 可以从迭代器范围构造并自动建堆。

2. **push(const T& value)**：

   - 插入元素并调整堆。
   - 时间复杂度：O(log n)。

3. **pop()**：

   - 移除堆顶元素并调整堆。
   - 时间复杂度：O(log n)。

4. **top()**：

   - 返回堆顶元素的引用（只读）。
   - 时间复杂度：O(1)。

5. **empty()**：

   - 检查队列是否为空，返回 bool。
   - 时间复杂度：O(1)。

6. **size()**：

   - 返回队列中元素数量。
   - 时间复杂度：O(1)。

------

示例代码

1. 默认大顶堆

cpp

```cpp
#include <iostream>
#include <queue>

int main() {
    std::priority_queue<int> pq;
    pq.push(3);
    pq.push(1);
    pq.push(4);
    pq.push(1);
    pq.push(5);

    while (!pq.empty()) {
        std::cout << pq.top() << " "; // 输出堆顶元素
        pq.pop();                     // 移除堆顶
    }
    std::cout << "\n";
    return 0;
}
```

- **输出**：5 4 3 1 1
- **说明**：
  - 默认使用 std::less<int>，形成大顶堆。
  - 每次 top() 返回最大值，pop() 移除后调整堆。
- 自定义小顶堆

cpp

```cpp
#include <iostream>
#include <queue>
#include <vector>

int main() {
    // 使用 std::greater<int> 改为小顶堆
    std::priority_queue<int, std::vector<int>, std::greater<int>> pq;
    pq.push(3);
    pq.push(1);
    pq.push(4);
    pq.push(1);
    pq.push(5);

    while (!pq.empty()) {
        std::cout << pq.top() << " ";
        pq.pop();
    }
    std::cout << "\n";
    return 0;
}
```

- **输出**：1 1 3 4 5
- **说明**：
  - 使用 std::greater<int> 作为比较器，形成小顶堆。
  - 堆顶是最小值。
- 自定义类型

cpp

```cpp
#include <iostream>
#include <queue>
#include <string>

struct Task {
    int priority;
    std::string name;
    Task(int p, std::string n) : priority(p), name(n) {}
    bool operator<(const Task& other) const { // 定义优先级顺序
        return priority < other.priority;      // 大顶堆：高优先级优先
    }
};

int main() {
    std::priority_queue<Task> pq;
    pq.emplace(1, "Low");
    pq.emplace(3, "High");
    pq.emplace(2, "Medium");

    while (!pq.empty()) {
        std::cout << pq.top().name << " (priority: " << pq.top().priority << ")\n";
        pq.pop();
    }
    return 0;
}
```

- **输出**：

  ```text
  High (priority: 3)
  Medium (priority: 2)
  Low (priority: 1)
  ```

- **说明**：

  - 自定义类型需要重载 < 运算符（或提供比较器）。
  - emplace 直接构造元素，避免拷贝。

------

与其他容器的比较

1. **与 std::queue 的区别**：
   - std::queue 是 FIFO（先进先出），而 std::priority_queue 基于优先级。
2. **与 std::set 的区别**：
   - std::set 保持全局排序且支持快速查找，而 std::priority_queue 只保证堆顶是最大/最小值。
   - priority_queue 不支持遍历所有元素。
3. **与手写堆的区别**：
   - 封装了堆操作，简化使用。
   - 不提供直接访问底层堆数组的能力。

------

注意事项

- **不可遍历**：只能访问堆顶元素，无法直接迭代所有元素。
- **线程安全**：std::priority_queue 本身不是线程安全的，多线程访问需要外部同步。
- **时间复杂度**：
  - 插入/删除：O(log n)
  - 访问顶部：O(1)
  - 构建堆：O(n)（从范围构造时）
- **底层容器要求**：
  - 必须支持随机访问（如 std::vector 或 std::deque）。
  - 默认使用 std::vector。

------

总结

std::priority_queue 是一个高效的优先级队列实现，适用于需要快速获取最大或最小元素的场景。通过灵活的比较器和容器选择，它可以轻松适配不同需求。无论是任务调度还是图算法，它都是 C++ 标准库中一个强大而便捷的工具。