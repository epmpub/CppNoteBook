 **C++ Ranges 工厂函数** 

---

### **1. 工厂函数概述**
工厂函数是 C++ Ranges 库中用于 **生成特定范围（Range）** 的工具，通常以惰性求值方式创建序列，适用于构建数据流管道。这些函数位于 `<ranges>` 头文件中，属于 `std::views` 命名空间。

---

### **2. 常用工厂函数**
#### **(1) `std::views::iota`**
生成一个 **整数序列**，支持起始值和结束值。
```cpp
#include <ranges>
#include <iostream>

int main() {
    // 生成 [1, 2, 3, 4, 5]
    auto nums = std::views::iota(1, 6);
    for (int x : nums) std::cout << x << " "; 

    // 无限序列（需用 take 截断）
    auto inf_nums = std::views::iota(10) | std::views::take(3); // 10, 11, 12
}
```

#### **(2) `std::views::generate`**
通过 **函数生成器** 动态生成元素。
```cpp
#include <ranges>
#include <iostream>

int main() {
    int counter = 0;
    auto gen = std::views::generate([&counter] { return ++counter; })
               | std::views::take(3); // 1, 2, 3
    for (int x : gen) std::cout << x << " ";
}
```

#### **(3) `std::views::repeat`（C++23）**
生成 **重复元素的无限序列**。
```cpp
#include <ranges>
#include <iostream>

int main() {
    // 重复 "Hi" 3 次
    auto repeated = std::views::repeat("Hi") | std::views::take(3);
    for (auto s : repeated) std::cout << s << " "; // Hi Hi Hi
}
```

#### **(4) `std::views::empty`**
生成一个 **空范围**，常用于占位或默认值。
```cpp
auto empty_range = std::views::empty<int>;
```

#### **(5) `std::views::single`（C++23）**
生成 **只含单个元素** 的范围。
```cpp
auto single_elem = std::views::single(42); // [42]
```

---

### **3. 组合使用示例**
#### **生成斐波那契数列**
```cpp
#include <ranges>
#include <iostream>

int main() {
    int a = 0, b = 1;
    auto fib = std::views::generate([a, b]() mutable {
        int next = a + b;
        a = b;
        b = next;
        return a;
    }) | std::views::take(10); // 前10项：1, 1, 2, 3, 5, 8, 13, 21, 34, 55

    for (int x : fib) std::cout << x << " ";
}
```

#### **生成二维坐标网格**
```cpp
#include <ranges>
#include <iostream>

int main() {
    // 生成 (0,0), (0,1), (1,0), (1,1)
    auto grid = std::views::cartesian_product(
        std::views::iota(0, 2),
        std::views::iota(0, 2)
    );

    for (auto [x, y] : grid) {
        std::cout << "(" << x << ", " << y << ") ";
    }
}
```

---

### **4. 性能与注意事项**
| **特性**     | **优点**                     | **注意事项**                   |
| ------------ | ---------------------------- | ------------------------------ |
| **惰性求值** | 节省内存，适合处理大规模数据 | 避免在生成器函数中保留外部状态 |
| **无限序列** | 灵活生成动态数据流           | 需结合 `take` 或条件终止遍历   |
| **类型安全** | 编译时检查元素类型           | 确保生成器返回值类型一致       |

---

### **5. C++23 新增工厂函数**
#### **(1) `std::views::chunk_by`**
根据条件将范围分块。
```cpp
auto numbers = {1, 3, 2, 4, 5, 7, 6};
auto chunks = numbers | std::views::chunk_by([](int a, int b) { return a < b; });
// 分块结果：[[1, 3], [2, 4, 5, 7], [6]]
```

#### **(2) `std::views::slide`**
生成滑动窗口视图。
```cpp
auto nums = std::views::iota(1, 5); // [1, 2, 3, 4]
auto sliding = nums | std::views::slide(2); 
// 窗口：[[1,2], [2,3], [3,4]]
```

---

### **6. 总结**
| **工厂函数**        | **适用场景**     | **C++ 版本** |
| ------------------- | ---------------- | ------------ |
| `iota`              | 生成整数序列     | C++20        |
| `generate`          | 动态生成元素     | C++20        |
| `repeat`            | 重复元素无限序列 | C++23        |
| `cartesian_product` | 生成笛卡尔积     | C++23        |
| `chunk_by`          | 条件分块         | C++23        |

合理使用工厂函数，可以简化代码并提升性能，尤其在处理流式数据时表现优异。