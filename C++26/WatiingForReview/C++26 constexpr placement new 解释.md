C++26 constexpr placement new 解释

```c
#include <memory>      // std::construct_at (可选对比)
#include <cstddef>     // std::byte
#include <print>

struct Point {
    int x, y;
    constexpr Point(int a, int b) : x(a), y(b) {}
    constexpr Point() = default;
};

// 使用 union 提供对齐且类型安全的存储（C++26 constexpr placement new 推荐方式）
union Storage {
    Point points[3];
    std::byte bytes[sizeof(Point) * 3];  // 仅用于对齐/大小
};

constexpr int test() {
    Storage s{};   // 零初始化 union

    // C++26: placement new 在 constexpr 中可用
    Point* p1 = new (&s.points[0]) Point(1, 2);      // direct initialization
    Point* p2 = new (&s.points[1]) Point();          // default initialization
    Point* p3 = new (&s.points[2]) Point{10, 20};    // list initialization

    int sum = p1->x + p2->x + p3->x;

    // 必须手动调用析构函数（constexpr 中要求）
    p1->~Point();
    p2->~Point();
    p3->~Point();

    return sum;
}

int main() {
    static_assert(test() == 11);   // 编译期验证通过！

    std::println("Result: {}", test());
    return 0;
}
```

**✅ 修复后的完整可编译示例（C++26 `constexpr` placement new）**

错误原因：在 `constexpr` 上下文中，编译器对存储类型（`std::byte` 数组）和目标类型（`Point`）的**相似性（similar types）**要求非常严格。`std::byte` 数组不能直接安全地用于 `placement new` 的 `constexpr` cast。

### 修复方案：使用 `union` 提供正确的存储类型

```cpp
#include <memory>      // std::construct_at (可选对比)
#include <cstddef>     // std::byte
#include <print>

struct Point {
    int x, y;
    constexpr Point(int a, int b) : x(a), y(b) {}
    constexpr Point() = default;
};

// 使用 union 提供对齐且类型安全的存储（C++26 constexpr placement new 推荐方式）
union Storage {
    Point points[3];
    std::byte bytes[sizeof(Point) * 3];  // 仅用于对齐/大小
};

constexpr int test() {
    Storage s{};   // 零初始化 union

    // C++26: placement new 在 constexpr 中可用
    Point* p1 = new (&s.points[0]) Point(1, 2);      // direct initialization
    Point* p2 = new (&s.points[1]) Point();          // default initialization
    Point* p3 = new (&s.points[2]) Point{10, 20};    // list initialization

    int sum = p1->x + p2->x + p3->x;

    // 必须手动调用析构函数（constexpr 中要求）
    p1->~Point();
    p2->~Point();
    p3->~Point();

    return sum;
}

int main() {
    static_assert(test() == 11);   // 编译期验证通过！

    std::println("Result: {}", test());
    return 0;
}
```

### 编译命令

```bash
g++ -std=c++26 -Wall -Wextra -O3 main.cpp -o main
# 或
clang++ -std=c++26 -Wall -Wextra -O3 main.cpp -o main
```

---

### 其他可选修复方式

1. **使用 `std::aligned_storage`（较老风格）**：
   ```cpp
   alignas(Point) std::byte raw[sizeof(Point)*3];
   Point* p1 = new (raw) Point(1, 2);   // 仍可能报错，取决于编译器
   ```

2. **对比 `std::construct_at`**（C++20+，更安全但功能较弱）：
   ```cpp
   Point* p = std::construct_at(reinterpret_cast<Point*>(raw), 1, 2);
   ```

---

**关键点总结**：

- `union` 是目前在 `constexpr` 中使用 `placement new` 最可靠的方式。
- 必须**手动析构**对象。
- `new (ptr) T(...)` 现在是 `constexpr`（提案 **P2747R2**）。
- 编译器支持：GCC/Clang 已较好支持，MSVC 正在跟进。
