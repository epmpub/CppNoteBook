# std::ranges::fold_left && std::ranges::fold_left_first

## std::ranges::fold_right , std::ranges::fold_right_last

std::ranges::fold_left 是 C++23 引入的一个算法，定义在 <ranges> 头文件中。它是 Ranges 库中的折叠（fold）操作，用于从左到右（left-to-right）对一个范围的元素应用二元操作（binary operation），并结合一个初始值生成最终结果。这类似于传统的 std::accumulate，但更现代化，支持 Ranges 和更灵活的接口。

以下是对 std::ranges::fold_left 的详细解释：

------

定义

cpp

```cpp
#include <ranges>

namespace std::ranges {
    template <input_range R, class T, indirectly_binary_left_foldable<T, iterator_t<R>> F>
    constexpr auto fold_left(R&& r, T init, F f);
}
```

- **R**: 输入范围，满足 std::ranges::input_range。
- **T**: 初始值的类型。
- **F**: 二元操作函数（如加法、乘法）。
- 返回类型：折叠后的结果，类型由 init 和 f 推导。

语法

cpp

```cpp
auto result = std::ranges::fold_left(range, init, binary_op);
```

- **range**: 要折叠的范围。
- **init**: 初始值。
- **binary_op**: 二元函数，形如 f(accum, element)。

------

行为

- **fold_left** 从左到右遍历 range，将 binary_op 应用于当前累积值（从 init 开始）和每个元素。

- **数学表示**：

  - 对于范围 {e1, e2, e3, ..., en}，结果是：

    ```text
    f(...f(f(f(init, e1), e2), e3)..., en)
    ```

- **惰性**：不像视图，它是立即求值的算法。

------

前提条件

- **范围要求**：
  - 必须是 std::ranges::input_range（支持输入迭代）。
- **二元操作**：
  - F 必须是一个二元函数，接受累积值和元素，返回新累积值。
  - 类型需满足 indirectly_binary_left_foldable（即可以通过迭代器间接调用）。
- **初始值**：
  - init 的类型需与 binary_op 的返回值兼容。

------

示例代码

示例 1：基本求和

cpp

```cpp
#include <ranges>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> vec = {1, 2, 3, 4};

    auto sum = std::ranges::fold_left(vec, 0, std::plus<>{});

    std::cout << "Sum: " << sum << "\n";
    return 0;
}
```

输出

```text
Sum: 10
```

- **解释**：
  - init = 0，binary_op = std::plus<>{}。
  - 计算：(((0 + 1) + 2) + 3) + 4 = 10。

示例 2：自定义操作

cpp

```cpp
#include <ranges>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> vec = {1, 2, 3, 4};

    auto result = std::ranges::fold_left(vec, 1, [](int acc, int x) {
        return acc * x;
    });

    std::cout << "Product: " << result << "\n";
    return 0;
}
```

输出

```text
Product: 24
```

- **解释**：
  - init = 1，binary_op = 乘法。
  - 计算：(((1 * 1) * 2) * 3) * 4 = 24。

示例 3：字符串拼接

cpp

```cpp
#include <ranges>
#include <iostream>
#include <vector>
#include <string>

int main() {
    std::vector<std::string> words = {"Hello", " ", "World"};

    auto result = std::ranges::fold_left(words, std::string{}, std::plus<>{});

    std::cout << "Result: " << result << "\n";
    return 0;
}
```

输出

```text
Result: Hello World
```

- **解释**：
  - init = ""，binary_op = 字符串拼接。
  - 计算：(("" + "Hello") + " ") + "World" = "Hello World"。

------

返回类型

- 返回 binary_op 的结果类型，通常与 init 类型相同（或由 binary_op 推导）。
- 如果范围为空，返回 init。

------

时间复杂度

- **O(n)**：
  - n 是范围的大小。
  - 每次迭代调用一次 binary_op。

------

与 std::accumulate 的对比

| 特性     | std::ranges::fold_left | std::accumulate        |
| -------- | ---------------------- | ---------------------- |
| 引入版本 | C++23                  | C++98                  |
| 输入类型 | Ranges（更通用）       | 迭代器对               |
| 操作顺序 | 左折叠（明确）         | 左折叠（默认）         |
| 并行支持 | 无（顺序执行）         | 无（C++17 有并行版本） |
| 灵活性   | 高（支持 Ranges）      | 中（仅迭代器）         |

- **fold_left** 是现代化的替代品，命名更明确（“左折叠”），与 Ranges 生态集成。

------

使用场景

1. **聚合计算**：

   - 求和、乘积、最大值等。

   cpp

   ```cpp
   auto sum = std::ranges::fold_left(vec, 0, std::plus<>{});
   ```

2. **字符串处理**：

   - 拼接字符串或构建格式化输出。

3. **复杂累积**：

   - 使用自定义操作处理复杂数据。

------

注意事项

1. **C++23 要求**：

   - 需要 -std=c++23 和支持 C++23 的编译器（如 GCC 13+、Clang 16+）。

2. **空范围**：

   - 如果 range 为空，返回 init。

   cpp

   ```cpp
   std::vector<int> empty;
   auto result = std::ranges::fold_left(empty, 42, std::plus<>{}); // 42
   ```

3. **操作顺序**：

   - 严格从左到右，不像 std::accumulate 可优化为其他顺序。

4. **异常**：

   - 如果 binary_op 抛出异常，fold_left 中止，结果未定义。

------

扩展：与 Ranges 视图结合

cpp

```cpp
#include <ranges>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    auto even = vec | std::views::filter([](int x) { return x % 2 == 0; });

    auto sum = std::ranges::fold_left(even, 0, std::plus<>{});

    std::cout << "Sum of evens: " << sum << "\n";
    return 0;
}
```

输出

```text
Sum of evens: 6
```

- **解释**：过滤偶数 {2, 4}，然后求和。

------

总结

std::ranges::fold_left 是 C++23 中一个强大的折叠算法，从左到右对范围应用二元操作。它替代了传统的 std::accumulate，与 Ranges 集成，支持自定义操作。适用于聚合、拼接等场景，时间复杂度为 O(n)。如果你有具体问题或想探讨其他折叠算法（如 fold_right），欢迎继续提问！



std::ranges::fold_left_first 是 C++23 引入的一个算法，定义在 <ranges> 头文件中。它是 std::ranges::fold_left 的变体，用于从左到右（left-to-right）折叠一个范围的元素，但与 fold_left 不同的是，它不接受显式的初始值，而是使用范围的第一个元素作为初始值。这使得它特别适合处理非空范围的折叠操作。

以下是对 std::ranges::fold_left_first 的详细解释：

------

定义

cpp

```cpp
#include <ranges>

namespace std::ranges {
    template <input_range R, indirectly_binary_left_foldable<range_value_t<R>, iterator_t<R>> F>
    constexpr auto fold_left_first(R&& r, F f);
}
```

- **R**: 输入范围，满足 std::ranges::input_range。
- **F**: 二元操作函数。
- 返回类型：std::optional<T>，其中 T 是折叠结果类型。如果范围为空，返回空 optional。

语法

cpp

```cpp
auto result = std::ranges::fold_left_first(range, binary_op);
```

- **range**: 要折叠的范围。
- **binary_op**: 二元函数，形如 f(left, right)。

------

行为

- **fold_left_first**：

  - 如果范围非空，取第一个元素作为初始值，从第二个元素开始折叠。

  - 对剩余元素从左到右应用 binary_op。

  - 数学表示：对于范围 {e1, e2, e3, ..., en}，结果是：

    ```text
    f(...f(f(e1, e2), e3)..., en)
    ```

- **空范围**：

  - 如果范围为空，返回 std::nullopt。

- **单元素范围**：

  - 如果范围只有一个元素，返回该元素（不调用 binary_op）。

------

前提条件

- **范围要求**：
  - 必须是 std::ranges::input_range。
  - 不能保证为空（空范围返回 std::nullopt）。
- **二元操作**：
  - F 必须是二元函数，接受两个元素，返回折叠结果。
  - 类型需满足 indirectly_binary_left_foldable。
- **非空性**：
  - 如果范围可能为空，需检查返回值。

------

示例代码

示例 1：基本求和

cpp

```cpp
#include <ranges>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> vec = {1, 2, 3, 4};

    auto result = std::ranges::fold_left_first(vec, std::plus<>{});

    std::cout << "Sum: " << *result << "\n";
    return 0;
}
```

输出

```text
Sum: 10
```

- **解释**：
  - 初始值：1（第一个元素）。
  - 计算：((1 + 2) + 3) + 4 = 10。

示例 2：空范围

cpp

```cpp
#include <ranges>
#include <iostream>
#include <vector>
#include <optional>

int main() {
    std::vector<int> empty;

    auto result = std::ranges::fold_left_first(empty, std::plus<>{});

    if (result) {
        std::cout << "Result: " << *result << "\n";
    } else {
        std::cout << "Empty range\n";
    }
    return 0;
}
```

输出

```text
Empty range
```

- **解释**：
  - 范围为空，返回 std::nullopt。

示例 3：单元素范围

cpp

```cpp
#include <ranges>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> vec = {42};

    auto result = std::ranges::fold_left_first(vec, std::plus<>{});

    std::cout << "Result: " << *result << "\n";
    return 0;
}
```

输出

```text
Result: 42
```

- **解释**：
  - 只有一个元素 42，直接返回，不调用 std::plus。

示例 4：字符串拼接

cpp

```cpp
#include <ranges>
#include <iostream>
#include <vector>
#include <string>

int main() {
    std::vector<std::string> words = {"Hello", " ", "World"};

    auto result = std::ranges::fold_left_first(words, std::plus<>{});

    std::cout << "Result: " << *result << "\n";
    return 0;
}
```

输出

```text
Result: Hello World
```

- **解释**：
  - 初始值："Hello"。
  - 计算：("Hello" + " ") + "World" = "Hello World"。

------

返回类型

- **std::optional<T>**：
  - T 是折叠结果类型，由 binary_op 和范围元素类型推导。
  - 非空范围：包含折叠结果。
  - 空范围：std::nullopt。

------

时间复杂度

- **O(n - 1)**：
  - n 是范围大小。
  - 对 n-1 个元素（从第二个开始）调用 binary_op。
- 空范围或单元素：O(1)。

------

与 std::ranges::fold_left 的对比

| 特性       | fold_left_first   | fold_left |
| ---------- | ----------------- | --------- |
| 初始值     | 第一个元素        | 显式指定  |
| 空范围处理 | 返回 std::nullopt | 返回 init |
| 返回类型   | std::optional<T>  | T         |
| 适用场景   | 非空范围折叠      | 通用折叠  |

------

使用场景

1. **无初始值聚合**：

   - 当初始值自然是范围的第一个元素时。

   cpp

   ```cpp
   auto sum = std::ranges::fold_left_first(vec, std::plus<>{});
   ```

2. **字符串拼接**：

   - 合并字符串序列，无需显式空字符串。

3. **简化代码**：

   - 避免手动提取第一个元素。

------

注意事项

1. **C++23 要求**：

   - 需要 -std=c++23 和支持 C++23 的编译器。

2. **空范围检查**：

   - 返回值是 std::optional，需检查是否存在。

   cpp

   ```cpp
   if (auto r = std::ranges::fold_left_first(vec, f)) { /* use *r */ }
   ```

3. **操作顺序**：

   - 严格左折叠，不优化为其他顺序。

4. **异常**：

   - 如果 binary_op 抛出异常，结果未定义。

------

扩展：与 Ranges 视图结合

cpp

```cpp
#include <ranges>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    auto even = vec | std::views::filter([](int x) { return x % 2 == 0; });

    auto result = std::ranges::fold_left_first(even, std::plus<>{});

    std::cout << "Sum of evens: " << (result ? *result : 0) << "\n";
    return 0;
}
```

输出

```text
Sum of evens: 6
```

- **解释**：过滤偶数 {2, 4}，折叠为 2 + 4 = 6。

------

总结

std::ranges::fold_left_first 是 C++23 中的折叠算法，使用范围的第一个元素作为初始值，从左到右应用二元操作。它返回 std::optional，处理空范围更安全。适用于无初始值需求的聚合场景，与 fold_left 相比更简洁但适用范围稍窄。如果你有具体问题或想探讨其他折叠变体，请告诉我！