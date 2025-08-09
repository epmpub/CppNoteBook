std::mismatch

这段代码展示了 C++ 标准库中 <algorithm> 头文件中的 std::mismatch 函数的用法。std::mismatch 用于比较两个范围内的元素，找到第一个不匹配的位置。它可以接受默认比较（相等性）或自定义谓词。以下是对代码的逐步解释。

```C++
#include <algorithm>
#include <string>
#include <vector>
#include <iostream>

int main() {
    std::string text1 = "Welcome to the underworld!";
    std::string text2 = "Welcome to the overworld!";
    auto it = std::mismatch(text1.begin(), text1.end(),
        text2.begin());
    // *it.first == 'u', *it.second == 'o'

    std::cout << "*it.first == '" << *it.first << 
        "' *it.second == '" << *it.second << "'\n";

    std::vector<double> data = {6.0, 11.0, 2.1};
    std::vector<double> args = {1.0, 0.5, 2.0};
    // Alternative way to think about mismatch,
    // a find with per-element argument.
    auto res = std::mismatch(data.begin(), data.end(),
        args.begin(), [](double elem, double arg) {
            return elem*arg >= 5;
        });
    // *res.first == 2.1, *res.second == 2.0

    std::cout << "*res.first == '" << *res.first << 
        "' *res.second == '" << *res.second << "'\n";
}
```



------

**基础知识**

- **std::mismatch**:
  - 功能：比较两个范围内的元素，返回第一个不匹配的元素对的位置。
  - 返回值：一个 std::pair，包含两个迭代器：
    - first: 第一个范围中不匹配的迭代器。
    - second: 第二个范围中对应的迭代器。
  - 如果两个范围完全匹配，返回的迭代器指向第一个范围的 end() 和第二个范围的对应位置。
- **重载**:
  1. 默认比较：使用 == 检查相等性。
  2. 自定义谓词：使用用户提供的二元谓词。

------

**代码分解**

**1. 默认比较：字符串比较**

cpp

```cpp
std::string text1 = "Welcome to the underworld!";
std::string text2 = "Welcome to the overworld!";
auto it = std::mismatch(text1.begin(), text1.end(), text2.begin());
// *it.first == 'u', *it.second == 'o'
```

- **操作**:
  - 比较 text1 和 text2 从开始到结束的字符。
  - 使用默认比较（==），检查两个字符串的字符是否相等。
- **过程**:
  - text1: "Welcome to the underworld!"
  - text2: "Welcome to the overworld!"
  - 逐字符比较：
    - 'W' == 'W', 'e' == 'e', ..., 'e' == 'e'（前 14 个字符 "Welcome to the " 相同）。
    - 'u' != 'o'（第 15 个字符不匹配）。
- **结果**:
  - it.first: 指向 text1 中的 'u'（text1.begin() + 14）。
  - it.second: 指向 text2 中的 'o'（text2.begin() + 14）。
- **解释**: 找到两个字符串第一个不同的字符位置。

------

**2. 自定义谓词：向量比较**

cpp

```cpp
std::vector<double> data = {6.0, 11.0, 2.1};
std::vector<double> args = {1.0, 0.5, 2.0};
auto res = std::mismatch(data.begin(), data.end(),
    args.begin(), [](double elem, double arg) {
        return elem * arg >= 5;
    });
// *res.first == 2.1, *res.second == 2.0
```

- **操作**:
  - 比较 data 和 args 中的元素，使用自定义谓词。
  - 谓词：elem * arg >= 5，返回 true 表示“匹配”，false 表示“不匹配”。
- **过程**:
  - data: {6.0, 11.0, 2.1}。
  - args: {1.0, 0.5, 2.0}。
  - 逐元素计算：
    1. 6.0 * 1.0 = 6.0 >= 5 → true（匹配）。
    2. 11.0 * 0.5 = 5.5 >= 5 → true（匹配）。
    3. 2.1 * 2.0 = 4.2 < 5 → false（不匹配）。
- **结果**:
  - res.first: 指向 data 中的 2.1（data.begin() + 2）。
  - res.second: 指向 args 中的 2.0（args.begin() + 2）。
- **解释**: 找到第一个不满足 elem * arg >= 5 的元素对。
- **注释中的观点**: 可以将 std::mismatch 视为一种“带参数的查找”，找到第一个不满足条件的元素。

------

**关键点**

1. **范围要求**:
   - 第一个范围：[first1, last1)（如 text1.begin(), text1.end()）。
   - 第二个范围：从 first2 开始，至少与第一个范围等长，否则行为未定义。
2. **默认 vs 自定义**:
   - 默认比较检查相等性。
   - 自定义谓词允许灵活定义“匹配”的含义。
3. **返回值**:
   - 如果没有不匹配，first 指向 end1，second 指向对应位置。

------

**中文解释**

**1. 字符串比较**

- **输入**:
  - text1 = "Welcome to the underworld!"
  - text2 = "Welcome to the overworld!"
- **操作**: 用 == 比较两个字符串的字符。
- **结果**: 在 'u' 和 'o' 处发现第一个不匹配。
- **输出**: it.first 指向 'u'，it.second 指向 'o'。

**2. 向量比较**

- **输入**:
  - data = {6.0, 11.0, 2.1}
  - args = {1.0, 0.5, 2.0}
- **操作**: 用谓词 elem * arg >= 5 检查每对元素。
- **结果**: 在 2.1 * 2.0 = 4.2 < 5 处发现第一个不匹配。
- **输出**: res.first 指向 2.1，res.second 指向 2.0。

------

**总结**

- **std::mismatch** 是一个通用工具，用于找到两个范围中第一个不匹配的元素对。
- **默认用法**: 检查相等性，适用于字符串或简单序列。
- **自定义用法**: 通过谓词支持复杂条件，扩展了其应用场景（如条件查找）。
- **灵活性**: 可以看作“带参数的查找”，在算法设计中非常实用。

如果你有进一步问题或需要更多示例，请告诉我！