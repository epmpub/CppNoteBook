# std::find_first_of

```C++
#include <algorithm>
#include <vector>
#include <string>
#include <iostream>

int main() {
    std::vector<int> haystack{1, 2, 3, 4, 5, 6};
    std::vector<int> needle{6, 4, 2};

    // find the first element in haystack matching either 2, 4, or 6
    auto it = std::find_first_of(haystack.begin(), haystack.end(), 
        needle.begin(), needle.end());
    // *it == 2

    std::cout << "*it == " << *it << "\n";

    std::string text = "The quick brown fox jumps over the lazy dog";
    std::string vowels = "aeiou";
    auto first = std::find_first_of(text.begin(), text.end(),
        vowels.begin(), vowels.end(),
            [](char l, char r) { // custom comparator
            return std::tolower(l) == std::tolower(r);
        });
    // *first == 'e'

    std::cout << "*first == " << *first << "\n";
}
```



这段代码展示了 C++ 标准库中 <algorithm> 提供的 std::find_first_of 函数的用法，用于在容器中查找第一个匹配指定集合中任意元素的元素。代码包含两个示例：一个使用默认比较器，另一个使用自定义比较器。以下是逐步解释。

------

代码概览

- 使用 std::find_first_of 在 haystack（向量）和 text（字符串）中查找匹配 needle 和 vowels 中任意元素的第一个位置。
- 展示了默认比较和自定义比较器的应用。

------

关键组件

1. **头文件**

cpp

```cpp
#include <algorithm>
#include <vector>
#include <string>
```

- <algorithm>：提供 std::find_first_of。
- <vector>：提供 std::vector。
- <string>：提供 std::string 和 std::tolower。
- **向量示例**

cpp

```cpp
std::vector<int> haystack{1, 2, 3, 4, 5, 6};
std::vector<int> needle{6, 4, 2};

auto it = std::find_first_of(haystack.begin(), haystack.end(), 
    needle.begin(), needle.end());
// *it == 2
```

- **haystack 和 needle**：
  - haystack 是搜索目标，包含 {1, 2, 3, 4, 5, 6}。
  - needle 是查找集合，包含 {6, 4, 2}。
- **std::find_first_of**：
  - 原型：find_first_of(first1, last1, first2, last2, [pred])。
  - 在 [first1, last1)（haystack）中查找第一个元素，使其等于 [first2, last2)（needle）中的任意元素。
  - 默认使用 == 比较。
- **查找过程**：
  - 检查 haystack 中的每个元素：
    - 1：不在 needle 中。
    - 2：在 needle 中（匹配 {2}），停止。
  - 返回迭代器 it，指向 haystack[1]。
- **结果**：
  - *it == 2。
  - 如果未找到，返回 haystack.end()。
- **字符串示例（自定义比较器）**

cpp

```cpp
std::string text = "The quick brown fox jumps over the lazy dog";
std::string vowels = "aeiou";
auto first = std::find_first_of(text.begin(), text.end(),
    vowels.begin(), vowels.end(),
        [](char l, char r) { // custom comparator
        return std::tolower(l) == std::tolower(r);
    });
// *first == 'e'
```

- **text 和 vowels**：
  - text 是搜索目标，内容为 "The quick ..."。
  - vowels 是查找集合，包含 "aeiou"。
- **std::find_first_of（带比较器）`**：
  - 第五个参数是自定义比较器（lambda），用于定义匹配规则。
  - [](char l, char r) { return std::tolower(l) == std::tolower(r); }：
    - l 是 text 中的字符。
    - r 是 vowels 中的字符。
    - 将两字符转换为小写后比较。
- **查找过程**：
  - 检查 text 中的每个字符：
    - 'T' → t，不在 "aeiou" 中。
    - 'h' → h，不在。
    - 'e' → e，在 vowels 中（匹配 'e'），停止。
  - 返回迭代器 first，指向 text[2]。
- **结果**：
  - *first == 'e'。
- **注意**：
  - 自定义比较器使查找忽略大小写（如 'E' 也能匹配 'e'）。

------

为什么这样工作？

1. **std::find_first_of**：
   - 线性搜索 haystack 或 text，直到找到第一个匹配 needle 或 vowels 中任意元素的元素。
   - 时间复杂度：O(n * m)，其中 n 是 haystack 长度，m 是 needle 长度。
2. **默认比较器**：
   - 使用 ==，直接比较元素值。
3. **自定义比较器**：
   - 替换默认 ==，提供灵活的匹配规则（如忽略大小写）。

------

输出

- 无运行时输出，代码展示返回值：
  - 向量示例：*it == 2。
  - 字符串示例：*first == 'e'。

------

使用场景

- **集合查找**：
  - 在列表中查找符合某些条件的第一个元素（如数字、字符）。
- **文本处理**：
  - 查找特定字符集（如元音）的首次出现。
- **自定义规则**：
  - 使用比较器实现复杂的匹配逻辑。

------

总结

- std::find_first_of 在容器中查找匹配集合中任意元素的第一个位置。
- 向量示例使用默认比较，找到 2。
- 字符串示例使用自定义比较器，找到 'e'（忽略大小写）。
- 代码展示了该算法的灵活性和实用性。