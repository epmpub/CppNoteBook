# std::generate_n

std::generate_n 是 C++ 标准模板库 (STL) 中的一个算法函数，定义在 <algorithm> 头文件中。它用于将一个生成器函数的连续结果赋值给指定数量的元素，通常用于填充容器或范围。

**std::generate_n 是 std::generate 的变体，区别在于它只生成指定数量的元素，而不是填充整个范围。**

以下是对 std::generate_n 的详细解释：

------

定义

cpp

```cpp
template <class OutputIterator, class Size, class Generator>
OutputIterator generate_n(OutputIterator first, Size n, Generator gen);
```

- **first**: 指向要填充的第一个元素位置的输出迭代器。
- **n**: 要生成的元素数量（类型通常是整数，如 size_t）。
- **gen**: 一个无参数的生成器函数（或函数对象），每次调用时返回一个值。
- **返回值**: 一个迭代器，指向生成元素之后的下一个位置（即 first + n）。

------

行为

- std::generate_n 从 first 开始，连续调用 gen() 函数 n 次，并将结果依次赋值给 [first, first + n) 范围内的元素。
- 生成器 gen 必须是可调用的（可以是函数、lambda 表达式、函数对象等），并且不接受参数。

------

示例代码

以下是一些使用 std::generate_n 的例子：

示例 1：填充固定值

cpp

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec(5); // 创建一个大小为 5 的向量

    // 用 lambda 表达式生成固定值 42
    std::generate_n(vec.begin(), 5, []() { return 42; });

    // 输出结果
    for (int x : vec) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

输出

```text
42 42 42 42 42
```

------

示例 2：生成递增序列

cpp

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec(5);
    int value = 0;

    // 用 lambda 表达式生成递增序列
    std::generate_n(vec.begin(), 5, [&value]() { return value++; });

    // 输出结果
    for (int x : vec) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

输出

```text
0 1 2 3 4
```

------

示例 3：使用函数对象

cpp

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

struct RandomGenerator {
    int operator()() {
        return rand() % 100; // 生成 0-99 的随机数
    }
};

int main() {
    std::vector<int> vec(5);

    // 使用函数对象生成随机数
    std::generate_n(vec.begin(), 5, RandomGenerator{});

    // 输出结果
    for (int x : vec) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

输出（随机）

```text
83 12 45 67 9
```

------

与 std::generate 的区别

| 特性     | std::generate              | std::generate_n             |
| -------- | -------------------------- | --------------------------- |
| 参数     | 接受范围 [first, last)     | 接受起始位置和数量 first, n |
| 填充范围 | 填充整个范围 [first, last) | 只填充 n 个元素             |
| 返回值   | 无返回值（void）           | 返回 first + n 的迭代器     |
| 使用场景 | 已知完整范围时使用         | 只需填充部分元素时使用      |

示例对比：

cpp

```cpp
std::vector<int> vec(5);
std::generate(vec.begin(), vec.end(), []() { return 1; });    // 填充整个向量
std::generate_n(vec.begin(), 3, []() { return 2; });          // 只填充前 3 个元素
```

------

时间复杂度

- **O(n)**: std::generate_n 调用生成器 n 次，复杂度取决于 n 和 gen 的执行时间。
- 如果 gen 本身的复杂度是常数，则总复杂度为 O(n)。

------

注意事项

1. **范围检查**：
   - 调用者必须确保 [first, first + n) 是有效的范围，否则会导致未定义行为（如越界访问）。
   - 例如，vec 的大小必须至少为 n，否则需要先用 reserve 或 resize 调整容器。
2. **生成器状态**：
   - 如果生成器需要维持状态（如递增计数器），需要使用可变对象（如 lambda 中的捕获变量或函数对象）。
3. **异常安全**：
   - 如果 gen 抛出异常，std::generate_n 会停止执行，已赋值的元素保持不变。

------

实际应用

- **初始化容器**：快速填充容器以特定模式的值。
- **测试数据生成**：生成随机数或序列用于测试。
- **部分更新**：只更新容器中的一部分元素。

------

总结

std::generate_n 是一个灵活的工具，用于将生成器的输出赋值给指定数量的元素。它与 std::generate 类似，但通过显式指定数量 n 提供了更精确的控制。结合 lambda 表达式或函数对象，它可以轻松实现各种生成逻辑。

如果你有具体的使用场景或问题，欢迎继续提问！