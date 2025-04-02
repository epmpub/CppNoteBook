# std::ranges::fold_left_with_iter 

std::ranges::fold_left_with_iter 是 C++23 引入的一个 Ranges 算法，定义在 <algorithm> 头文件中。它是对传统 std::accumulate 的增强版本，用于对范围进行**左折叠（left fold）**操作，同时返回折叠结果和结束迭代器。与其他折叠算法不同，它不仅计算折叠值，还提供迭代停止的位置，这在需要额外上下文时非常有用。

以下是对 std::ranges::fold_left_with_iter 的详细解释：

------

定义

cpp

```cpp
#include <algorithm>

namespace std::ranges {
    template<input_iterator I, sentinel_for<I> S, class T, indirectly_binary_left_foldable<T, I> F>
    constexpr auto fold_left_with_iter(I first, S last, T init, F f);

    template<input_range R, class T, indirectly_binary_left_foldable<T, iterator_t<R>> F>
    constexpr auto fold_left_with_iter(R&& r, T init, F f);
}
```

- **I**: 输入迭代器类型。
- **S**: 哨兵类型，与 I 兼容。
- **T**: 初始值类型。
- **F**: 二元操作函数类型。
- **first**: 范围起始迭代器。
- **last**: 范围结束哨兵。
- **init**: 初始值。
- **f**: 二元操作函数（如 std::plus<>）。
- **R**: 输入范围类型。

返回类型

- 返回一个 fold_left_with_iter_result 结构，包含：
  - **in**: I 类型，指向折叠结束的迭代器。
  - **value**: U 类型，折叠结果（U 是 f 的返回类型，通常是 T 的退化类型）。

cpp

```cpp
template<class I, class T>
struct fold_left_with_iter_result {
    [[no_unique_address]] I in;
    [[no_unique_address]] T value;
};
```

------

行为

- **std::ranges::fold_left_with_iter**：
  - 从 init 开始，依次对范围 [first, last) 的每个元素应用二元操作 f，从左到右折叠。
  - 计算方式为：f(f(f(init, *first), *(first+1)), ...)。
  - 如果范围为空，返回 {first, init}。
- **返回**：
  - in: 指向 last 的迭代器（或停止位置）。
  - value: 折叠结果。
- **应用次数**：
  - 恰好 ranges::distance(first, last) 次 f。

------

前提条件

- **范围**：
  - [first, last) 必须是有效范围。
- **迭代器**：
  - I 必须满足 input_iterator。
  - S 必须是 I 的哨兵。
- **操作**：
  - F 必须是 indirectly_binary_left_foldable<T, I>，即支持 f(T, *I) 的调用。
- **类型**：
  - T 必须可移动构造。

------

示例代码

示例 1：基本用法（求和）

cpp

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> vec = {1, 2, 3, 4};

    auto result = std::ranges::fold_left_with_iter(vec, 0, std::plus<>{});

    std::cout << "Sum: " << result.value << "\n"; // 折叠结果
    std::cout << "End position: " << (result.in == vec.end()) << "\n"; // 是否到达末尾

    return 0;
}
```

输出

```text
Sum: 10
End position: 1
```

- **解释**：
  - 从 0 开始，计算 0 + 1 + 2 + 3 + 4 = 10。
  - result.in 指向 vec.end()。

示例 2：自定义操作

cpp

```cpp
#include <algorithm>
#include <iostream>
#include <vector>
#include <string>

int main() {
    std::vector<int> vec = {1, 2, 3};

    auto result = std::ranges::fold_left_with_iter(
        vec, std::string("start"),
        [](std::string s, int x) { return s + "-" + std::to_string(x); }
    );

    std::cout << "Result: " << result.value << "\n";

    return 0;
}
```

输出

```text
Result: start-1-2-3
```

- **解释**：
  - 从 "start" 开始，依次追加 "-1", "-2", "-3"。

示例 3：部分折叠

cpp

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};

    auto result = std::ranges::fold_left_with_iter(
        vec.begin(), vec.begin() + 3, 0, std::multiplies<>{}
    );

    std::cout << "Product: " << result.value << "\n";
    std::cout << "Stopped at: " << *result.in << "\n";

    return 0;
}
```

输出

```text
Product: 6
Stopped at: 4
```

- **解释**：
  - 只折叠前 3 个元素：1 * 2 * 3 = 6。
  - result.in 指向第 4 个元素。

------

时间复杂度

- **O(n)**：
  - n 是范围大小（ranges::distance(first, last)）。
  - 每次迭代调用一次 f。

------

与其他折叠算法的对比

| 算法                      | 初始值 | 返回类型                     | 用途                    |
| ------------------------- | ------ | ---------------------------- | ----------------------- |
| fold_left                 | 有     | U                            | 仅返回折叠值            |
| fold_left_with_iter       | 有     | in_value_result              | 返回折叠值和结束迭代器  |
| fold_left_first           | 无     | optional<U>                  | 用首元素作为初始值      |
| fold_left_first_with_iter | 无     | in_value_result<optional<U>> | 首元素初始值+结束迭代器 |

------

使用场景

1. **需要折叠值和位置**：

   - 计算累积结果并跟踪处理到的位置。

   cpp

   ```cpp
   auto [it, sum] = std::ranges::fold_left_with_iter(vec, 0, std::plus<>{});
   ```

2. **调试或验证**：

   - 检查折叠是否完整覆盖范围。

3. **部分处理**：

   - 在自定义条件下提前终止并获取停止点。

------

注意事项

1. **C++23 要求**：
   - 需要 -std=c++23 和支持 C++23 的编译器。
2. **空范围**：
   - 返回 {first, init}，无 f 调用。
3. **异常**：
   - 如果 f 或迭代器操作抛出异常，则传播。
4. **Niebloid**：
   - 它是算法函数对象，不能显式指定模板参数。

------

总结

std::ranges::fold_left_with_iter 是 C++23 中一个强大的折叠工具，扩展了 std::accumulate 的功能。它通过返回结束迭代器提供了更多上下文，适合需要同时获取结果和位置的场景。与 fold_left 相比，它更灵活，但也稍复杂。如果你有具体问题或想探讨其他 C++23 折叠算法（如 fold_right），请告诉我！