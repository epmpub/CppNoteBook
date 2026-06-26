C++ 中与**数组（Array）**相关的 Type Traits

### 1. 主要数组相关的 Type Traits

| 类型萃取工具                   | 作用                                   | 示例                      |
| ------------------------------ | -------------------------------------- | ------------------------- |
| `std::is_array_v<T>`           | 判断是否是数组类型                     | `true` / `false`          |
| `std::remove_extent_t<T>`      | 移除**最外层**数组维度                 | `int[5]` → `int`          |
| `std::remove_all_extents_t<T>` | 移除**所有**数组维度（降维到元素类型） | `int[3][4][5]` → `int`    |
| `std::rank_v<T>`               | 获取数组的维度数量（几维数组）         | `int[3][4]` → `2`         |
| `std::extent_v<T, N>`          | 获取第 `N` 维的大小（从0开始）         | `int[3][4]` 的第0维是 `3` |

---

### 2. 完整演示代码

```cpp
#include <iostream>
#include <type_traits>
#include <string>

template<typename T>
void show_array_traits(const std::string& name)
{
    using Clean = std::remove_cvref_t<T>;

    std::cout << "=== " << name << " ===\n";
    std::cout << "is_array          : " << std::is_array_v<Clean> << "\n";
    std::cout << "rank              : " << std::rank_v<Clean> << "\n";
    std::cout << "extent[0]         : " << std::extent_v<Clean, 0> << "\n";
    std::cout << "extent[1]         : " << std::extent_v<Clean, 1> << "\n";
    std::cout << "remove_extent     : " 
              << typeid(std::remove_extent_t<Clean>).name() << "\n";
    std::cout << "remove_all_extents: " 
              << typeid(std::remove_all_extents_t<Clean>).name() << "\n";
    std::cout << std::string(50, '-') << "\n\n";
}

int main()
{
    show_array_traits<int[10]>("一维数组: int[10]");
    show_array_traits<int[5][6]>("二维数组: int[5][6]");
    show_array_traits<double[3][4][5]>("三维数组: double[3][4][5]");
    show_array_traits<const char[20]>("const char[20]");
    show_array_traits<int>("普通 int（非数组）");

    // 实际使用示例
    using Arr = int[10][20];
    using ElementType = std::remove_all_extents_t<Arr>;   // 得到 int

    std::cout << "最终元素类型: " << typeid(ElementType).name() << "\n";

    return 0;
}
```

---

### 3. 关键说明

- **`std::remove_extent_t<T>`**  
  只移除**最外面一层**数组。
  - `int[10]` → `int`
  - `int[5][6]` → `int[6]`（仍保留内层）

- **`std::remove_all_extents_t<T>`**  
  移除**所有**数组层，得到最基本的元素类型。
  - `int[5][6][7]` → `int`

- **`std::extent_v<T, N>`**  
  如果指定的维度不存在，返回 `0`。

---

### 4. 常见实际用途

```cpp
// 1. 获取数组的原始元素类型
template<typename T>
using ElementType = std::remove_all_extents_t<T>;

// 2. 遍历多维数组时判断维度
template<typename T>
void process_array(T& arr)
{
    if constexpr (std::rank_v<T> == 2)
    {
        // 处理二维数组
    }
}
```

---

**输出示例**（部分）：
```
=== 一维数组: int[10] ===
is_array          : 1
rank              : 1
extent[0]         : 10
remove_extent     : int
remove_all_extents: int
```

需要我再补充 `std::decay_t` 与数组的关系，或者写一个**多维数组打印模板**的例子吗？