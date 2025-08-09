# std::views::split

简要定义

std::views::split 是 C++20 引入的 Ranges 视图适配器（C++23 增强），定义在 <ranges> 中，用于将输入范围按分隔符分割成子范围。

cpp

```cpp
#include <ranges>

auto view = std::views::split(range, delimiter);
```

- **range**: 输入范围（std::ranges::input_range）。
- **delimiter**: 分隔符（C++20 为单元素，C++23 可为范围）。
- 返回：std::ranges::split_view，迭代时产生 std::ranges::subrange。

------

行为

- **分割**：
  - 按 delimiter 将 range 分成多个子范围，分隔符不包含在结果中。
- **C++20**：delimiter 是单个元素。
- **C++23**：delimiter 可为范围（如字符串）。
- **惰性求值**：仅在迭代时计算。

------

示例代码

示例 1：C++20 单字符分割

cpp

```cpp
#include <ranges>
#include <iostream>
#include <string>

int main() {
    std::string str = "hello,world,again";
    auto view = std::views::split(str, ',');

    for (auto sub : view) {
        std::cout << std::string_view(sub.begin(), sub.end()) << "\n";
    }

    return 0;
}
```

输出

```text
hello
world
again
```

示例 2：C++23 范围分割

cpp

```cpp
#include <ranges>
#include <iostream>
#include <string>

int main() {
    std::string str = "oneSEPtwoSEPthree";
    std::string sep = "SEP";
    auto view = std::views::split(str, sep);

    for (auto sub : view) {
        std::cout << std::string_view(sub.begin(), sub.end()) << "\n";
    }

    return 0;
}
```

输出

```text
one
two
three
```

示例 3：处理连续分隔符

cpp

```cpp
#include <ranges>
#include <iostream>
#include <string>

int main() {
    std::string str = "a,,b,c";
    auto view = std::views::split(str, ',');

    for (auto sub : view) {
        std::string_view sv(sub.begin(), sub.end());
        std::cout << "[" << sv << "]\n";
    }

    return 0;
}
```

输出

```text
[a]
[]
[b]
[c]
```

- **解释**：连续的 , 产生空子范围。

------

关键特性

- **返回类型**：std::ranges::split_view<R, P>。
- **元素类型**：std::ranges::subrange，需手动转为可打印类型。
- **时间复杂度**：
  - 构造：O(1)。
  - 迭代：O(n)，n 是输入范围大小。
  - 每次查找：O(k)，k 是分隔符长度。

------

C++20 vs C++23

| 特性       | C++20             | C++23                  |
| ---------- | ----------------- | ---------------------- |
| 分隔符类型 | 单元素（如 char） | 范围（如 std::string） |
| 标准支持   | -std=c++20        | -std=c++23             |
| 示例       | split(str, ',')   | split(str, "SEP")      |

------

使用场景

1. **字符串处理**：

   - 分割 CSV 或路径。

   cpp

   ```cpp
   auto parts = std::views::split(path, '/');
   ```

2. **数据解析**：

   - 按标记分割序列。

3. **与 Ranges 组合**：

   - 配合 transform 将子范围转为其他类型。

------

注意事项

1. **C++ 版本**：
   - C++20：单元素分隔符。
   - C++23：范围分隔符。
2. **空范围**：
   - 输入为空，结果为空。
   - 分隔符为空，行为未定义。
3. **连续分隔符**：
   - 产生空子范围，需处理。

------

扩展示例：与 transform 结合

cpp

```cpp
#include <ranges>
#include <iostream>
#include <string>

int main() {
    std::string str = "10,20,30";
    auto view = std::views::split(str, ',') 
              | std::views::transform([](auto r) {
                    return std::stoi(std::string(r.begin(), r.end()));
                });

    for (int x : view) {
        std::cout << x << " ";
    }
    std::cout << "\n";

    return 0;
}
```

输出

```text
10 20 30
```

------

总结

std::views::split 是一个灵活的 Ranges 工具，用于将范围按分隔符分割成子范围。C++20 支持单元素分隔符，C++23 扩展到范围分隔符。它提供惰性求值，适合字符串解析和数据分割。如果你需要更具体的内容（比如实现细节、性能优化），请告诉我！