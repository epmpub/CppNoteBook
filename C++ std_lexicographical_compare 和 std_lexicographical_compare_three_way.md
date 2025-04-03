# std::lexicographical_compare 和 std::lexicographical_compare_three_way

在 C++ 中，std::lexicographical_compare 和 std::lexicographical_compare_three_way 是标准库中的算法，定义在 <algorithm> 头文件中。它们用于比较两个序列（例如字符串、数组或容器）的字典序（lexicographical order）。前者是传统的布尔比较（C++98 引入），而后者是 C++20 引入的三路比较版本，返回更丰富的比较结果。以下是对两者的详细解释：

------

std::lexicographical_compare

定义

- **头文件**：<algorithm>

- **函数签名**：

  cpp

  ```cpp
  template<class InputIt1, class InputIt2, class Compare>
  bool lexicographical_compare(InputIt1 first1, InputIt1 last1,
                               InputIt2 first2, InputIt2 last2,
                               Compare comp);
  
  // 默认版本（使用 operator<）
  template<class InputIt1, class InputIt2>
  bool lexicographical_compare(InputIt1 first1, InputIt1 last1,
                               InputIt2 first2, InputIt2 last2);
  ```

  - first1, last1：第一个序列的范围。
  - first2, last2：第二个序列的范围。
  - comp：可选的比较函数（默认使用 operator<）。

- **行为**：

  - 按字典序比较两个序列，返回 true 如果第一个序列小于第二个序列，否则返回 false。
  - 字典序规则：
    - 从头开始逐元素比较。
    - 如果一个元素小于另一个，则比较结束。
    - 如果一个序列是另一个的前缀（即较短），则较短的序列较小。

- **返回值**：

  - true：[first1, last1) < [first2, last2)。
  - false：[first1, last1) >= [first2, last2)。

示例

cpp

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> v1 = {1, 2, 3};
    std::vector<int> v2 = {1, 2, 4};

    bool result = std::lexicographical_compare(
        v1.begin(), v1.end(), 
        v2.begin(), v2.end()
    );

    std::cout << std::boolalpha << "v1 < v2: " << result << "\n";
    return 0;
}
```

- **输出**：v1 < v2: true
- **说明**：
  - v1 = {1, 2, 3}，v2 = {1, 2, 4}。
  - 前两个元素相等，第三个元素 3 < 4，因此 v1 < v2。

自定义比较器

cpp

```cpp
#include <iostream>
#include <algorithm>
#include <string>

int main() {
    std::string s1 = "abc";
    std::string s2 = "ABC";

    bool result = std::lexicographical_compare(
        s1.begin(), s1.end(), 
        s2.begin(), s2.end(),
        [](char a, char b) { return std::tolower(a) < std::tolower(b); }
    );

    std::cout << std::boolalpha << "s1 < s2 (case-insensitive): " << result << "\n";
    return 0;
}
```

- **输出**：s1 < s2 (case-insensitive): false
- **说明**：忽略大小写后，"abc" == "ABC"，但 s1 不小于 s2。

------

std::lexicographical_compare_three_way

定义

- **头文件**：<algorithm>（C++20）

- **函数签名**：

  cpp

  ```cpp
  template<class InputIt1, class InputIt2, class Compare>
  auto lexicographical_compare_three_way(InputIt1 first1, InputIt1 last1,
                                         InputIt2 first2, InputIt2 last2,
                                         Compare comp);
  
  // 默认版本（使用 operator<=>）
  template<class InputIt1, class InputIt2>
  auto lexicographical_compare_three_way(InputIt1 first1, InputIt1 last1,
                                         InputIt2 first2, InputIt2 last2);
  ```

  - 参数与 std::lexicographical_compare 类似。
  - comp：可选的比较函数，默认使用三路比较运算符 <=>。

- **行为**：

  - 按字典序进行三路比较，返回一个比较结果类型（通常是 std::strong_ordering 或类似类型）。
  - 三路比较结果：
    - 小于：第一个序列小于第二个序列。
    - 等于：两个序列相等。
    - 大于：第一个序列大于第二个序列。

- **返回值**：

  - 返回类型由 comp 或默认 <=> 运算符决定，通常是：
    - std::strong_ordering::less（小于）。
    - std::strong_ordering::equal（等于）。
    - std::strong_ordering::greater（大于）。

示例

cpp

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> v1 = {1, 2, 3};
    std::vector<int> v2 = {1, 2, 4};

    auto result = std::lexicographical_compare_three_way(
        v1.begin(), v1.end(), 
        v2.begin(), v2.end()
    );

    if (result < 0) std::cout << "v1 < v2\n";
    else if (result == 0) std::cout << "v1 == v2\n";
    else std::cout << "v1 > v2\n";
    return 0;
}
```

- **输出**：v1 < v2
- **说明**：
  - v1 = {1, 2, 3}，v2 = {1, 2, 4}。
  - 第三个元素 3 < 4，返回 std::strong_ordering::less。

自定义三路比较

cpp

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <compare>

int main() {
    std::string s1 = "abc";
    std::string s2 = "ABC";

    auto result = std::lexicographical_compare_three_way(
        s1.begin(), s1.end(), 
        s2.begin(), s2.end(),
        [](char a, char b) { 
            return std::tolower(a) <=> std::tolower(b); 
        }
    );

    if (result < 0) std::cout << "s1 < s2\n";
    else if (result == 0) std::cout << "s1 == s2\n";
    else std::cout << "s1 > s2\n";
    return 0;
}
```

- **输出**：s1 == s2
- **说明**：忽略大小写后，"abc" == "ABC"，返回 std::strong_ordering::equal。

------

主要区别

| 特性     | std::lexicographical_compare | std::lexicographical_compare_three_way |
| -------- | ---------------------------- | -------------------------------------- |
| 引入版本 | C++98                        | C++20                                  |
| 返回值   | bool（小于或不小于）         | 三路比较结果（如 strong_ordering）     |
| 默认比较 | operator<                    | operator<=>                            |
| 信息量   | 仅判断小于                   | 提供小于、等于、大于三种结果           |
| 适用场景 | 简单的大小比较               | 需要完整比较关系的场景                 |

------

注意事项

- **复杂度**：两者均为 O(min(n1, n2))，其中 n1 和 n2 是两个序列的长度。
- **要求**：
  - 输入范围必须有效。
  - 比较器必须满足严格弱序（lexicographical_compare）或三路比较语义（lexicographical_compare_three_way）。
- **C++20 支持**：lexicographical_compare_three_way 需要编译器支持 C++20 和 <=> 运算符。
- **异常**：如果比较器抛出异常，行为未定义。

------

总结

- **std::lexicographical_compare**：传统的字典序比较，返回布尔值，适合只需要判断大小的场景。
- **std::lexicographical_compare_three_way**：C++20 的现代版本，利用三路比较提供更丰富的结果，适合需要完整比较关系的场景。 两者都是处理序列比较的强大工具，选择取决于具体需求和 C++ 版本支持。