# *std::views::lazy_split* 

```C++
#include <string_view>
#include <ranges>
#include <algorithm>
#include <iostream>
#include <cstdint>
#include <charconv>
#include <print>

int main() {
    std::string_view text = "the quick brown fox jumps over the lazy dog";
    for (auto word: text | 
        std::views::lazy_split(' ') | // split by word
        std::views::filter([](auto v) { // filter out "the"
            // equal can operate on input ranges
            return not std::ranges::equal(
                v, std::string_view("the"));
        })) {
        // copy can operate on input ranges
        std::ranges::copy(word,
            std::ostream_iterator<char>(std::cout, ""));
        std::cout << ", ";
    }
    std::cout << '\n';
    // prints: quick, brown, fox, jumps, over, lazy, dog,

    std::string_view ip_addr = "2001:0db8:85a3:0000:0000:8a2e:0370:7334";
    std::array<uint16_t, 8> parsed;
    std::ranges::copy(
        ip_addr | 
        std::views::split(':') | // split by ':'
        std::views::transform([](auto v) -> uint16_t { // parse
            uint16_t result = 0;
            // can't use views::lazy_split
            // from_chars requires a contiguous range
            std::from_chars(v.data(), v.data()+v.size(), result, 16);
            return result;
        }),
        parsed.begin());
    // parsed == {0x2001, 0xdb8, 0x85a3, 0x0, 0x0, 0x8a2e, 0x370, 0x7334}

    for (uint16_t hunk : parsed)
        std::print("{:0>4x} ", hunk);
    std::println("");
}
```

这段代码展示了 C++20 和 C++23 中引入的一些现代特性，特别是 <ranges> 库、std::string_view、<charconv> 和 <print> 的使用。代码分为两部分：第一部分处理字符串分割和过滤，第二部分解析 IPv6 地址并格式化输出。

以下是对代码的详细解释：

------

**代码结构**

1. **头文件**：
   - <string_view>：提供 std::string_view，一个轻量级的非拥有字符串视图。
   - <ranges>：引入 C++20 的范围库，支持视图（views）和范围操作。
   - <algorithm>：提供 std::ranges::copy 等算法。
   - <iostream>：用于标准输入输出。
   - <cstdint>：定义整数类型（如 uint16_t）。
   - <charconv>：提供 std::from_chars，用于高效地将字符串转换为数字。
   - <print>：C++23 中引入的格式化输出工具（std::print 和 std::println）。
2. **两个主要功能块**：
   - **字符串处理**：从文本中提取单词，过滤掉 "the" 并打印。
   - **IPv6 地址解析**：将 IPv6 地址字符串解析为 16 位整数数组并格式化输出。

------

**第一部分：字符串处理**

cpp

```cpp
std::string_view text = "the quick brown fox jumps over the lazy dog";
for (auto word: text | 
    std::views::lazy_split(' ') | // split by word
    std::views::filter([](auto v) { // filter out "the"
        return not std::ranges::equal(
            v, std::string_view("the"));
    })) {
    std::ranges::copy(word,
        std::ostream_iterator<char>(std::cout, ""));
    std::cout << ", ";
}
std::cout << '\n';
// 输出: quick, brown, fox, jumps, over, lazy, dog,
```

**逐步解释**

1. **std::string_view text**：
   - 定义一个字符串视图，内容为 "the quick brown fox jumps over the lazy dog"。使用 std::string_view 而不是 std::string 可以避免不必要的内存分配。
2. **范围管道（| 操作符）**：
   - C++20 的 <ranges> 库引入了视图（views），允许以声明式的方式处理数据。
   - text | std::views::lazy_split(' ')：
     - std::views::lazy_split 将 text 按空格 ' ' 分割为子范围（单词）。
     - “lazy” 表示分割是惰性的，只有在迭代时才会计算，避免一次性生成所有子字符串。
     - 结果是一个视图，包含子范围如 {"the", "quick", "brown", ...}。
   - | std::views::filter(...)：
     - 过滤掉等于 "the" 的单词。
     - std::ranges::equal(v, std::string_view("the")) 比较每个子范围 v 和 "the" 是否相等。
     - not 取反，表示保留不相等的单词。
3. **范围 for 循环**：
   - for (auto word: ...) 遍历过滤后的视图，word 是每个单词的子范围（类型为视图，例如 std::ranges::subrange）。
4. **输出**：
   - std::ranges::copy(word, std::ostream_iterator<char>(std::cout, ""))：
     - 将 word 的字符复制到 std::cout，std::ostream_iterator 是一个输出迭代器。
     - 第二个参数 "" 表示字符之间无分隔符。
   - std::cout << ", "：在每个单词后添加逗号和空格。
   - 最后 std::cout << '\n' 换行。

**输出结果**

- 原始文本："the quick brown fox jumps over the lazy dog"
- 过滤掉 "the" 后："quick, brown, fox, jumps, over, lazy, dog,"

------

**第二部分：IPv6 地址解析**

cpp

```cpp
std::string_view ip_addr = "2001:0db8:85a3:0000:0000:8a2e:0370:7334";
std::array<uint16_t, 8> parsed;
std::ranges::copy(
    ip_addr | 
    std::views::split(':') | // split by ':'
    std::views::transform([](auto v) -> uint16_t { // parse
        uint16_t result = 0;
        std::from_chars(v.data(), v.data()+v.size(), result, 16);
        return result;
    }),
    parsed.begin());
// parsed == {0x2001, 0xdb8, 0x85a3, 0x0, 0x0, 0x8a2e, 0x370, 0x7334}

for (uint16_t hunk : parsed)
    std::print("{:0>4x} ", hunk);
std::println("");
```

**逐步解释**

1. **std::string_view ip_addr**：
   - 定义一个 IPv6 地址字符串视图："2001:0db8:85a3:0000:0000:8a2e:0370:7334"。
   - IPv6 地址由 8 个 16 位（4 字符十六进制）字段组成，用冒号 : 分隔。
2. **std::array<uint16_t, 8> parsed**：
   - 定义一个长度为 8 的数组，用于存储解析后的 16 位整数。
3. **范围管道解析**：
   - ip_addr | std::views::split(':')：
     - 使用 std::views::split 按冒号 : 分割字符串。
     - 结果是一个视图，包含子范围如 {"2001", "0db8", "85a3", ...}。
     - **注意：这里使用 split 而不是 lazy_split，因为后续的 std::from_chars 需要连续范围。**
   - | std::views::transform(...)：
     - 对每个子范围应用转换函数，将其从字符串解析为 uint16_t。
     - std::from_chars(v.data(), v.data()+v.size(), result, 16)：
       - v.data() 是子范围的起始指针，v.data()+v.size() 是结束指针。
       - 16 表示按十六进制解析。
       - 将结果存储在 result 中并返回。
4. **std::ranges::copy**：
   - 将转换后的 uint16_t 值复制到 parsed 数组的起始位置（parsed.begin()）。
   - 结果：parsed 包含 {0x2001, 0x0db8, 0x85a3, 0x0000, 0x0000, 0x8a2e, 0x0370, 0x7334}。
5. **格式化输出**：
   - for (uint16_t hunk : parsed)：遍历 parsed 数组。
   - std::print("{:0>4x} ", hunk)：
     - {:0>4x} 是 C++23 的格式化语法：
       - x：以小写十六进制输出。
       - 0>：用 0 填充左侧。
       - 4：至少 4 个字符。
     - 每个值后加空格。
   - std::println("")：换行并刷新输出。

**输出结果**

- parsed 中的值：{0x2001, 0x0db8, 0x85a3, 0x0000, 0x0000, 0x8a2e, 0x0370, 0x7334}
- 格式化输出："2001 0db8 85a3 0000 0000 8a2e 0370 7334"

------

**关键技术点**

1. **C++20 范围库**：
   - std::views::lazy_split 和 std::views::split 是惰性视图，用于分割字符串。
   - std::views::filter 和 std::views::transform 支持声明式数据处理。
   - | 操作符使代码更简洁，像流水线一样组合操作。
2. **std::from_chars**：
   - 比传统方法（如 std::stoi）更快**，因为它是低级、无动态分配的转换工具**。
   - **需要连续范围，**因此注释提到不能用 lazy_split（它可能生成非连续视图）。
3. **C++23 <print>**：
   - 提供现代格式化输出，替代 std::cout，语法更直观。

------

**总结**

- **第一部分**：展示了如何用范围库分割字符串、过滤单词并输出。
- **第二部分**：展示了如何解析 IPv6 地址并格式化输出，结合了范围转换和现代格式化工具。
- **代码风格**：利用 C++20/23 的新特性，代码简洁、高效且易读。

如果你有具体问题（例如某一行代码的细节），欢迎进一步提问！