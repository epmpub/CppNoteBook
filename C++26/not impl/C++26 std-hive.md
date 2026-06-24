**C++26 `std::hive`** 是 C++26 引入的一个全新**序列容器**（sequence container），定义在 `<hive>` 头文件中。它特别适合**频繁插入和删除**、同时需要**良好迭代性能**的场景。

### 核心设计理念

`std::hive` 是一种 **bucket-based（分桶）** 容器，也被称为“**内存位置重用容器**”。它源于游戏开发中广泛使用的 **bucket array / object pool / colony** 模式。

- **多个独立内存块（blocks / buckets）**：元素分散存储在多个独立分配的内存块中。
- **Skipfield（跳跃字段）**：每个块伴随一个元数据数组，用于快速跳过已删除的元素，实现高效迭代。
- **重用已擦除元素的内存位置**：删除元素后，其内存槽会被标记为可用，后续插入会优先复用这些位置，而不是总是分配新内存。
- **指针/引用/迭代器稳定性**：除非显式调用 `shrink_to_fit()` 或容器析构，否则**指向活跃元素的指针和迭代器永远不会失效**（即使发生插入和删除）。

### 关键特性

| 特性                       | 说明                                                         |
| -------------------------- | ------------------------------------------------------------ |
| **插入（insert/emplace）** | 平均 **O(1)**，位置未指定（unordered-like），优先复用已删除槽位 |
| **删除（erase）**          | **O(1)**（按迭代器或值）                                     |
| **迭代**                   | **O(N)**（常数时间跳过已删除元素）                           |
| **内存管理**               | 块容量按实现定义的因子增长（通常 ~1.69）；支持用户限制 min/max 块容量 |
| **稳定性**                 | 指针、引用、迭代器在插入/删除时保持有效                      |
| **不支持**                 | `operator==` / `!=`（因为是 unordered），`reserve()`（使用块机制） |

- **AllocatorAwareContainer**：支持自定义分配器（包括 `std::pmr::hive`）。
- **ReversibleContainer**：支持正向和反向迭代。

### 适用场景（优势明显）

- **游戏开发**：实体系统（Entity Component System）、粒子系统、AI 单位等（大量 spawn/despawn）。
- **高频修改的集合**：对象池、事件队列、网络连接管理等。
- **需要指针稳定性的场景**：指向容器内元素的指针必须长期有效。
- **比 `std::vector` 更好**：频繁删除不会导致大量移动/复制。
- **比 `std::list` / `std::deque` 更好**：缓存友好（块内元素连续），迭代更快。

### 基本用法示例

```cpp
#include <hive>
#include <string>
#include <iostream>

struct Entity {
    std::string name;
    int health = 100;
};

int main() {
    std::hive<Entity> entities;

    // 插入
    entities.emplace("Player", 100);
    auto it = entities.emplace("Enemy1");

    // 删除
    entities.erase(it);                    // O(1)

    // 迭代（自动跳过已删除元素）
    for (const auto& e : entities) {
        std::cout << e.name << " (health: " << e.health << ")\n";
    }

    // 容量控制
    std::hive_limits limits{8, 512};       // min=8, max=512 元素/块
    entities.reshape(limits);              // 调整块大小策略
}
```

### 与其他容器的对比

| 容器            | 插入/删除 | 迭代性能 | 指针稳定性 | 内存布局             | 典型用途                 |
| --------------- | --------- | -------- | ---------- | -------------------- | ------------------------ |
| `std::vector`   | 差        | 优秀     | 差         | 连续                 | 少修改                   |
| `std::deque`    | 中        | 良好     | 差         | 分块                 | 中等修改                 |
| `std::list`     | 优秀      | 差       | 优秀       | 链表                 | 极少迭代                 |
| **`std::hive`** | **优秀**  | **良好** | **优秀**   | **分块 + skipfield** | **高频插入/删除 + 迭代** |

### 注意事项

- **无序**：插入位置未指定，不能依赖元素顺序。
- **无 `==` 比较**：因为顺序不固定。
- **内存开销**：每个块有 skipfield 元数据（通常每个元素 1-2 字节额外开销）。
- **Feature Test Macro**：
  ```cpp
  #if __cpp_lib_hive >= 202502L
  #include <hive>
  #endif
  ```

**提案**：P0447R28（Matt Bentley）  
**cppreference**：[`std::hive`](https://en.cppreference.com/w/cpp/container/hive)

`std::hive` 是 C++26 中为**高性能、真实世界应用**量身定制的容器，填补了 `vector` 与 `list` 之间的长期空白，尤其受游戏引擎和系统编程开发者欢迎。

需要性能对比示例、与 `plf::colony` 的关系、或具体 API（如 `reshape`、`hive_limits`）的详细说明吗？随时告诉我！