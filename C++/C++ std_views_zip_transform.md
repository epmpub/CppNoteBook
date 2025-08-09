# std::views::zip_transform

std::views::zip_transform 是 C++23 引入的一个 Ranges 视图适配器，定义在 <ranges> 头文件中。它结合了 std::views::zip 和 std::ranges::transform 的功能，用于将多个输入范围“压缩”（zip）在一起，并对每组对应的元素应用一个变换函数，生成一个新的视图。这是一个惰性求值的工具，非常适合处理多个范围的同步操作。

以下是对 std::views::zip_transform 的详细解释：

------

定义

cpp

```cpp
#include <ranges>

namespace std::views {
    inline constexpr auto zip_transform = /* unspecified */;
}
```

- **zip_transform**：

  - 接受一个变换函数和多个输入范围。
  - 将范围的对应元素打包成元组，然后应用变换函数。

- **语法**：

  cpp

  ```cpp
  auto view = std::views::zip_transform(transform_fn, range1, range2, ...);
  ```

  - **transform_fn**: 变换函数，接受多个参数（对应每个范围的元素）。
  - **range1, range2, ...**: 输入范围，需满足 std::ranges::input_range。

- 返回类型：std::ranges::zip_transform_view。

------

行为

- **std::views::zip_transform**：
  - 将多个范围的对应元素“压缩”成一组（类似 std::views::zip）。
  - 对每组元素应用 transform_fn，生成新元素。
  - 视图长度等于最短输入范围的长度。
- **惰性求值**：
  - 不复制数据，仅在迭代时计算。
- **元素类型**：
  - 由 transform_fn 的返回值决定。

------

前提条件

- **范围要求**：
  - 所有输入范围必须满足 std::ranges::input_range。
- **变换函数**：
  - 必须接受与范围数量相匹配的参数。
  - 参数类型需与各范围的元素类型兼容。
- **非空**：
  - 如果所有范围为空，结果视图为空。

------

示例代码

示例 1：基本用法（两个范围相加）

cpp

```cpp
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v1 = {1, 2, 3};
    std::vector<int> v2 = {4, 5, 6};

    auto view = std::views::zip_transform(
        [](int a, int b) { return a + b; }, v1, v2
    );

    std::cout << "Sum: ";
    for (int x : view) {
        std::cout << x << " ";
    }
    std::cout << "\n";

    return 0;
}
```

输出

```text
Sum: 5 7 9
```

- **解释**：
  - v1 和 v2 的元素配对：{1, 4}, {2, 5}, {3, 6}。
  - 变换函数计算和：1+4=5, 2+5=7, 3+6=9。

示例 2：三个范围相乘

cpp

```cpp
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v1 = {1, 2, 3};
    std::vector<int> v2 = {2, 2, 2};
    std::vector<int> v3 = {3, 4, 5};

    auto view = std::views::zip_transform(
        [](int a, int b, int c) { return a * b * c; }, v1, v2, v3
    );

    std::cout << "Product: ";
    for (int x : view) {
        std::cout << x << " ";
    }
    std::cout << "\n";

    return 0;
}
```

输出

```text
Product: 6 16 30
```

- **解释**：
  - 配对：{1, 2, 3}, {2, 2, 4}, {3, 2, 5}。
  - 计算：1*2*3=6, 2*2*4=16, 3*2*5=30。

示例 3：不同长度范围

cpp

```cpp
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v1 = {1, 2, 3, 4};
    std::vector<int> v2 = {10, 20}; // 较短

    auto view = std::views::zip_transform(
        [](int a, int b) { return a + b; }, v1, v2
    );

    std::cout << "Sum: ";
    for (int x : view) {
        std::cout << x << " ";
    }
    std::cout << "\n";

    return 0;
}
```

输出

```text
Sum: 11 22
```

- **解释**：
  - 只处理最短范围的长度（2）：1+10=11, 2+20=22。

------

返回视图的类型

- **std::ranges::zip_transform_view<F, R1, R2, ...>**：
  - F 是变换函数类型。
  - R1, R2, ... 是输入范围类型。
- **元素类型**：
  - decltype(transform_fn(*ranges::begin(r1), *ranges::begin(r2), ...))。

------

时间复杂度

- **构造**：O(1)，视图是惰性的。
- **迭代**：
  - 每次访问：O(1)（调用变换函数）。
  - 总复杂度：O(n)，n 是最短范围的大小。

------

使用场景

1. **多范围同步操作**：

   - 对多个容器执行逐元素计算。

   cpp

   ```cpp
   auto sums = std::views::zip_transform(std::plus<>{}, v1, v2);
   ```

2. **数据融合**：

   - 合并多个来源的数据。

3. **与 Ranges 组合**：

   - 配合其他视图（如 filter）处理。

------

注意事项

1. **C++23 要求**：
   - 需要 -std=c++23 和支持 C++23 的编译器。
2. **范围长度**：
   - 视图长度由最短范围决定，超出的元素被忽略。
3. **引用性**：
   - 变换函数操作的是引用，修改会影响原始范围（若非 const）。
4. **函数签名**：
   - transform_fn 参数数量必须与范围数量匹配。

------

与其他视图/算法的对比

| 工具                      | 功能                  | 输入                | 返回类型       |
| ------------------------- | --------------------- | ------------------- | -------------- |
| std::views::zip           | 压缩多个范围为元组    | 多个范围            | 元组视图       |
| std::ranges::transform    | 对单个/双范围应用变换 | 1-2 个范围 + 输出   | 无（写入输出） |
| std::views::zip_transform | 压缩并变换多个范围    | 多个范围 + 变换函数 | 变换结果视图   |

------

替代方案（C++20）

在 C++20 中，可以用 std::views::zip（range-v3）加 std::views::transform 模拟：

cpp

```cpp
#include <range/v3/view/zip.hpp>
#include <range/v3/view/transform.hpp>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v1 = {1, 2, 3};
    std::vector<int> v2 = {4, 5, 6};

    auto view = ranges::views::zip(v1, v2) | ranges::views::transform(
        [](auto pair) { return pair.first + pair.second; }
    );

    for (int x : view) {
        std::cout << x << " ";
    }
    std::cout << "\n";

    return 0;
}
```

- **输出**：5 7 9
- **缺点**：需要 range-v3 库。

------

总结

std::views::zip_transform 是 C++23 中一个强大的视图适配器，结合了范围压缩和变换功能。它允许对多个范围的对应元素应用自定义操作，生成惰性视图，适合多数据源的同步处理。相比 C++20 的替代方案，它更简洁且内置于标准库。如果你有具体问题或想扩展用法，请告诉我！