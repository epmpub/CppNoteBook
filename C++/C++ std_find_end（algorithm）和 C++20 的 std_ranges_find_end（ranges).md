# std::find_end（<algorithm>）和 C++20 的 std::ranges::find_end（<ranges>

```C++
#include <algorithm>
#include <vector>
#include <print>

int main() {
    std::vector<int> haystack{1,2,3,4,5,1,2,3,4,5};
    std::vector<int> needle{3,4};

    auto it = std::find_end(
        haystack.begin(), haystack.end(),
        needle.begin(), needle.end());
    // (it - haystack.begin()) == 7
    std::println("(it - haystack.begin()) == {}", (it - haystack.begin()));   

    // Ranges version returns the found instance as a subrange
    auto rng = std::ranges::find_end(haystack, needle);
    // rng == {3,4}
    std::println("rng == {}", rng);

    // Both versions support custom comparator
    std::string sentence = "Word word WORD wORD";
    std::string word = "word";

    auto case_sen = std::ranges::find_end(sentence, word);
    // case_sen == "word";
    auto case_ins = std::ranges::find_end(sentence, word, [](char l, char r){
        return std::tolower(l) == std::tolower(r);
    });
    // case_ins == "wORD"

    std::println("case_sen == {}", case_sen);
    std::println("case_ins == {}", case_ins);
}
```



这段代码展示了 C++ 中查找子序列最后出现位置的两种算法：传统的 std::find_end（<algorithm>）和 C++20 的 std::ranges::find_end（<ranges>）。代码通过示例展示了它们的用法，包括默认比较和自定义比较器。以下是逐步解释。

------

代码概览

- 使用 std::find_end 查找向量中子序列 {3, 4} 的最后出现位置。
- 使用 std::ranges::find_end 返回子范围，并支持自定义比较器查找字符串中的单词。

------

关键组件

1. **头文件**

cpp

```cpp
#include <algorithm>
#include <vector>
#include <print>
```

- <algorithm>：提供 std::find_end。
- <vector>：提供 std::vector。
- <print>：C++23 的 std::println 用于格式化输出。
- **std::find_end 示例**

cpp

```cpp
std::vector<int> haystack{1,2,3,4,5,1,2,3,4,5};
std::vector<int> needle{3,4};
auto it = std::find_end(haystack.begin(), haystack.end(),
                        needle.begin(), needle.end());
std::println("(it - haystack.begin()) == {}", (it - haystack.begin()));
```

- **haystack 和 needle**：

  - haystack = {1, 2, 3, 4, 5, 1, 2, 3, 4, 5}。
  - needle = {3, 4}。

- **std::find_end**：

  - 原型：find_end(first1, last1, first2, last2)。
  - 在 [first1, last1) 中查找 [first2, last2) 最后出现的位置。
  - 返回迭代器，指向子序列的起始位置，或 last1（未找到）。

- **查找过程**：

  - {3, 4} 出现在位置 2（{3, 4, 5}）和位置 7（{3, 4, 5}）。
  - 返回最后出现的位置，it 指向 haystack[7]。

- **输出**：

  - (it - haystack.begin()) == 7。

  ```text
  (it - haystack.begin()) == 7
  ```

- **std::ranges::find_end 示例**

cpp

```cpp
auto rng = std::ranges::find_end(haystack, needle);
std::println("rng == {}", rng);
```

- **std::ranges::find_end**：

  - 原型：find_end(range, subrange)。
  - 返回一个 subrange，表示最后匹配的子序列。

- **查找过程**：

  - 同上，找到位置 7 的 {3, 4}。
  - rng 是 [haystack[7], haystack[9])。

- **输出**：

  - rng == {3, 4}（假设 std::println 支持 subrange，实际需自定义格式化）。

  ```text
  rng == {3, 4}
  ```

- **自定义比较器示例**

cpp

```cpp
std::string sentence = "Word word WORD wORD";
std::string word = "word";
auto case_sen = std::ranges::find_end(sentence, word);
auto case_ins = std::ranges::find_end(sentence, word, [](char l, char r){
    return std::tolower(l) == std::tolower(r);
});
std::println("case_sen == {}", case_sen);
std::println("case_ins == {}", case_ins);
```

- **sentence 和 word**：

  - sentence = "Word word WORD wORD"。
  - word = "word"。

- **case_sen**：

  - 默认比较（==），区分大小写。
  - 查找 "word"：
    - 出现位置："Word" (0-4)、"word" (5-9)、"WORD" (10-14)、"wORD" (15-19)。
    - 最后匹配 "word" 在位置 5。
  - case_sen 是 [sentence[5], sentence[9])，即 "word"。

- **case_ins**：

  - 自定义比较器：忽略大小写。
  - 查找 "word"：
    - 所有 "Word", "word", "WORD", "wORD" 都匹配。
    - 最后匹配 "wORD" 在位置 15。
  - case_ins 是 [sentence[15], sentence[19])，即 "wORD".

- **输出**：

  ```text
  case_sen == word
  case_ins == wORD
  ```

------

为什么这样工作？

1. **std::find_end**：
   - 返回迭代器，指向最后匹配的子序列开头。
   - 使用线性搜索，从右向左查找。
2. **std::ranges::find_end**：
   - 返回 subrange，更现代化。
   - 支持自定义比较器。
3. **比较器**：
   - 默认 == 区分大小写。
   - 自定义 tolower 实现忽略大小写。

------

输出

```text
(it - haystack.begin()) == 7
rng == {3, 4}
case_sen == word
case_ins == wORD
```

- 注意：rng 和 subrange 的输出可能需要自定义格式化支持，此处假设简化为 {3, 4}。

------

使用场景

- **模式匹配**：
  - 查找子序列的最后出现。
- **文本处理**：
  - 区分或忽略大小写的搜索。
- **数据分析**：
  - 检测重复模式。

------

总结

- std::find_end 找到 {3, 4} 在位置 7，返回迭代器。
- std::ranges::find_end 返回 {3, 4} 的子范围。
- 自定义比较器在字符串中找到 "word"（区分大小写）和 "wORD"（忽略大小写）。
- 代码展示了传统和 Ranges 算法的用法及灵活性。
- 