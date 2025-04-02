C++ 查找所有满足条件的元素

在 C++ 中，std::find 只能查找第一个满足条件的元素。如果你需要查找**所有满足条件的元素**，需要使用其他方法或组合标准库中的工具。以下是几种实现方式的详细说明，涵盖传统方法和 C++20 Ranges 库的现代方法，具体取决于你的需求（如返回迭代器、索引还是新容器）。

------

1. **传统方法：使用循环**

最直接的方式是手动遍历范围，检查每个元素，并收集满足条件的元素。

示例：查找所有偶数

cpp

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5, 6};
    std::vector<int> result;

    for (int i : vec) {
        if (i % 2 == 0) { // 条件：偶数
            result.push_back(i);
        }
    }

    std::cout << "Even numbers: ";
    for (int x : result) {
        std::cout << x << " ";
    }
    std::cout << "\n";

    return 0;
}
```

输出

```text
Even numbers: 2 4 6
```

- **优点**：简单直观，可定制。
- **缺点**：需要手动管理循环和结果容器。
- **时间复杂度**：O(n)，n 是范围大小。

------

2. **使用 std::copy_if**

std::copy_if 是 <algorithm> 中的算法，可以将满足条件的元素复制到新容器中。

示例：复制所有大于 3 的元素

cpp

```cpp
#include <vector>
#include <algorithm>
#include <iostream>

int main() {
    std::vector<int> vec = {1, 5, 2, 4, 3, 6};
    std::vector<int> result;

    std::copy_if(vec.begin(), vec.end(), std::back_inserter(result),
                 [](int x) { return x > 3; });

    std::cout << "Numbers > 3: ";
    for (int x : result) {
        std::cout << x << " ";
    }
    std::cout << "\n";

    return 0;
}
```

输出

```text
Numbers > 3: 5 4 6
```

- **解释**：
  - std::back_inserter 动态追加元素到 result。
  - Lambda 函数定义条件。
- **优点**：标准库支持，简洁。
- **时间复杂度**：O(n)。

------

3. **使用 std::find_if 多次查找**

如果你需要迭代器位置，可以使用 std::find_if 循环查找所有匹配元素。

示例：查找所有正数的迭代器

cpp

```cpp
#include <vector>
#include <algorithm>
#include <iostream>

int main() {
    std::vector<int> vec = {-1, 2, -3, 4, 5, -6};
    std::vector<std::vector<int>::iterator> positions;

    auto it = vec.begin();
    while (it != vec.end()) {
        it = std::find_if(it, vec.end(), [](int x) { return x > 0; });
        if (it != vec.end()) {
            positions.push_back(it);
            ++it; // 移动到下一个位置
        }
    }

    std::cout << "Positive numbers: ";
    for (auto pos : positions) {
        std::cout << *pos << " ";
    }
    std::cout << "\n";

    return 0;
}
```

输出

```text
Positive numbers: 2 4 5
```

- **解释**：
  - 每次找到一个满足条件的元素后，移动迭代器继续查找。
- **优点**：返回迭代器，可用于修改原容器。
- **缺点**：代码较复杂。
- **时间复杂度**：O(n^2)（最坏情况每次查找扫描剩余范围）。

------

4. **使用 C++20 Ranges：std::ranges::filter_view**

C++20 引入的 Ranges 库提供了更现代的解决方案。std::views::filter 创建一个视图，只包含满足条件的元素。

示例：过滤所有偶数

cpp

```cpp
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5, 6};

    auto even = vec | std::views::filter([](int x) { return x % 2 == 0; });

    std::cout << "Even numbers: ";
    for (int x : even) {
        std::cout << x << " ";
    }
    std::cout << "\n";

    return 0;
}
```

输出

```text
Even numbers: 2 4 6
```

- **解释**：
  - std::views::filter 返回一个惰性视图，仅在迭代时过滤。
  - 使用管道操作符 | 组合。
- **优点**：
  - 声明式，简洁。
  - 惰性求值，不复制数据。
- **时间复杂度**：构造视图 O(1)，迭代 O(n)。
- **要求**：C++20 支持（-std=c++20）。

转换为容器

如果需要结果容器，可以用 std::ranges::copy：

cpp

```cpp
std::vector<int> result;
std::ranges::copy(even, std::back_inserter(result));
```

------

5. **使用 std::for_each 收集结果**

std::for_each 可以结合 lambda 收集满足条件的元素。

示例：查找所有负数

cpp

```cpp
#include <vector>
#include <algorithm>
#include <iostream>

int main() {
    std::vector<int> vec = {1, -2, 3, -4, -5, 6};
    std::vector<int> result;

    std::for_each(vec.begin(), vec.end(), [&result](int x) {
        if (x < 0) {
            result.push_back(x);
        }
    });

    std::cout << "Negative numbers: ";
    for (int x : result) {
        std::cout << x << " ";
    }
    std::cout << "\n";

    return 0;
}
```

输出

```text
Negative numbers: -2 -4 -5
```

- **优点**：灵活，可执行额外操作。
- **时间复杂度**：O(n)。

------

6. **自定义条件与复杂类型**

对于复杂类型，可以定义自定义谓词。

示例：查找特定字符串

cpp

```cpp
#include <vector>
#include <ranges>
#include <iostream>
#include <string>

int main() {
    std::vector<std::string> vec = {"cat", "dog", "camel", "bird"};

    auto long_words = vec | std::views::filter([](const std::string& s) { 
        return s.size() > 3; 
    });

    std::cout << "Words longer than 3: ";
    for (const auto& w : long_words) {
        std::cout << w << " ";
    }
    std::cout << "\n";

    return 0;
}
```

输出

```text
Words longer than 3: camel bird
```

------

选择方法的指南

| 需求           | 推荐方法                 | 原因                 |
| -------------- | ------------------------ | -------------------- |
| 返回新容器     | std::copy_if / Ranges    | 简洁，标准支持       |
| 返回迭代器位置 | std::find_if 循环        | 适合需要修改原始数据 |
| 惰性访问       | std::views::filter       | 无复制，现代 C++     |
| 复杂逻辑       | 手动循环 / std::for_each | 最大灵活性           |

------

性能对比

- **线性方法**（copy_if, filter, for_each）：O(n)。
- **多次查找**（find_if 循环）：O(n^2)，不推荐大数据。
- **Ranges 视图**：构造 O(1)，迭代 O(n)，内存高效。

------

总结

要查找所有满足条件的元素：

1. **简单场景**：用 std::copy_if 或手动循环。
2. **现代 C++**：用 std::views::filter（C++20），优雅且高效。
3. **需要位置**：用 std::find_if 多次调用。 选择取决于你的需求（结果类型、性能、C++ 版本）。如果你有具体场景或代码需要优化，请告诉我，我会进一步协助！