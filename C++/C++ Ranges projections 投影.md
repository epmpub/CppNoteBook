

# Ranges projections  投影

Support for projections is one of the improvements introduced with the C++20 range versions of standard algorithms.

Projections are applied before elements are passed to the corresponding invocable.

Algorithms that operate on multiple source ranges provide a separate projection for each range.

Note that taking an address of standard functions (including members) is undefined behaviour.

```c++
#include <ranges>
#include <algorithm>
#include <vector>

struct User {
    int64_t id;
    std::string name;
};

using namespace std::literals;

std::vector<User> users{{37,"Eliana Green"}, {23, "Logan Sterling"},
                        {1, "Isla Bennett"}, {7, "Marcel Jones"}};

// Sort users by id (any invocable will work)
std::ranges::sort(users, {}, &User::id);
// {} instantiates the default comparator: std::ranges::less
// std::ranges::sort(users, std::ranges::less{}, &User::id);

// users == {{1,"Isla..."}, {7,"Marcel..."},
//           {23,"Logan..."}, {37,"Eliana..."}}

// Find by name
auto it = std::ranges::find(users, "Eliana Green"s, &User::name);
// it->id == 37, it->name == "Eliana Green"

std::vector<int> first{1,2,3,4,5};
std::vector<int> second{1,2,3,4,5};
std::vector<int> out;

std::ranges::transform(first, second, std::back_inserter(out), 
    [](int left, int right) { return left * right; }, // transformation
    [](int left) { return left + 10; },   // projection, first range
    [](int right) { return right / 2; }); // projection, second range
// out == 0 (11*0), 12 (12*1), 13 (13*1), 28 (14*2), 30 (15*2)
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/W1GYv1aET)

这段代码使用了 C++20 的 Ranges 库中的 std::ranges::transform 算法，对两个输入范围（first 和 second）进行变换，并将结果存储到输出容器 out 中。

与传统 std::transform 不同，Ranges 版本支持额外的投影函数（projections），**允许在变换前对输入元素进行预处理**。让我逐行解释代码的工作原理。

------

完整代码

cpp

```cpp
std::vector<int> first{1, 2, 3, 4, 5};
std::vector<int> second{1, 2, 3, 4, 5};
std::vector<int> out;

std::ranges::transform(first, second, std::back_inserter(out), 
    [](int left, int right) { return left * right; }, // transformation
    [](int left) { return left + 10; },   // projection, first range
    [](int right) { return right / 2; }); // projection, second range
// out == 0 (11*0), 12 (12*1), 13 (13*1), 28 (14*2), 30 (15*2)
```

------

逐步解释

1. **输入和输出定义**

cpp

```cpp
std::vector<int> first{1, 2, 3, 4, 5};
std::vector<int> second{1, 2, 3, 4, 5};
std::vector<int> out;
```

- **first**：第一个输入范围，包含 {1, 2, 3, 4, 5}。
- **second**：第二个输入范围，包含 {1, 2, 3, 4, 5}。
- **out**：输出容器，初始为空，用于存储变换结果。

------

2. **std::ranges::transform 调用**

cpp

```cpp
std::ranges::transform(first, second, std::back_inserter(out), 
    [](int left, int right) { return left * right; }, 
    [](int left) { return left + 10; }, 
    [](int right) { return right / 2; });
```

- **std::ranges::transform**：
  - 是 C++20 Ranges 库中的算法，用于对范围元素应用变换。
  - 此版本接受两个输入范围，并支持投影函数。
- **参数**：
  1. **first**: 第一个输入范围。
  2. **second**: 第二个输入范围。
  3. **std::back_inserter(out)**: 输出迭代器，动态追加结果到 out。
  4. **[](int left, int right) { return left \* right; }**: 变换函数（transformation），对两个输入元素执行操作。
  5. **[](int left) { return left + 10; }**: 第一个范围的投影函数（projection），在变换前处理 first 的元素。
  6. **[](int right) { return right / 2; }**: 第二个范围的投影函数，在变换前处理 second 的元素。

------

3. **执行过程**

- **std::ranges::transform** 对两个范围的对应元素逐对处理：
  1. 对 first 的每个元素应用投影 left + 10。
  2. 对 second 的每个元素应用投影 right / 2。
  3. 将投影后的结果传入变换函数 left * right，结果追加到 out。
- **范围大小**：
  - first 和 second 都有 5 个元素，结果 out 也将有 5 个元素。

逐步计算

| 索引 | first[i] | 投影 first[i] + 10 | second[i] | 投影 second[i] / 2 | 变换 (proj_first * proj_second) | out[i] |
| ---- | -------- | ------------------ | --------- | ------------------ | ------------------------------- | ------ |
| 0    | 1        | 11                 | 1         | 0 (1 / 2)          | 11 * 0                          | 0      |
| 1    | 2        | 12                 | 2         | 1 (2 / 2)          | 12 * 1                          | 12     |
| 2    | 3        | 13                 | 3         | 1 (3 / 2)          | 13 * 1                          | 13     |
| 3    | 4        | 14                 | 4         | 2 (4 / 2)          | 14 * 2                          | 28     |
| 4    | 5        | 15                 | 5         | 2 (5 / 2)          | 15 * 2                          | 30     |

- **注意**：
  - second[i] / 2 使用整数除法，向下取整（如 3 / 2 = 1）。

------

4. **结果**

cpp

```cpp
// out == 0 (11*0), 12 (12*1), 13 (13*1), 28 (14*2), 30 (15*2)
```

- **out** 最终包含 {0, 12, 13, 28, 30}。
- **验证**：
  - out[0] = 11 * 0 = 0
  - out[1] = 12 * 1 = 12
  - out[2] = 13 * 1 = 13
  - out[3] = 14 * 2 = 28
  - out[4] = 15 * 2 = 30

------

时间复杂度

- **O(n)**：
  - n 是输入范围的大小（这里是 5）。
  - 每个元素执行一次投影和变换。

------

使用场景

1. **双范围变换**：
   - 对两个范围的对应元素进行操作（如点积、逐元素运算）。
2. **预处理输入**：
   - 使用投影函数在变换前调整数据。
3. **动态输出**：
   - 使用 std::back_inserter 填充结果容器。

------

注意事项

1. **C++20 要求**：
   - 需要 -std=c++20 和支持 Ranges 的编译器。
2. **范围大小匹配**：
   - first 和 second 的大小必须相同，否则行为未定义（只处理较短范围的长度）。
3. **投影函数**：
   - 可选参数，若省略则直接使用原始元素。
4. **整数除法**：
   - right / 2 是整数除法，结果向下取整。

------

等价传统代码

cpp

```cpp
#include <vector>
#include <algorithm>
#include <iostream>

int main() {
    std::vector<int> first{1, 2, 3, 4, 5};
    std::vector<int> second{1, 2, 3, 4, 5};
    std::vector<int> out;

    for (size_t i = 0; i < first.size(); ++i) {
        int proj_first = first[i] + 10;
        int proj_second = second[i] / 2;
        out.push_back(proj_first * proj_second);
    }

    for (int x : out) {
        std::cout << x << " ";
    }
    std::cout << "\n";

    return 0;
}
```

- **输出**：0 12 13 28 30
- **缺点**：手动循环，不如 Ranges 版本简洁。

------

总结

这段代码使用 std::ranges::transform 对两个向量 first 和 second 进行变换：

- **投影**：first 每个元素加 10，second 每个元素除以 2。
- **变换**：将投影后的值相乘。
- **结果**：存储在 out 中，最终为 {0, 12, 13, 28, 30}。 Ranges 版本通过投影和变换函数提供了声明式、灵活的方式，比传统循环更现代化。如果你有具体问题或想调整代码，请告诉我！