# C++ std::ranges::mismatch

C++20 引入的 <ranges> 库提供了一个与 std::mismatch 类似的功能，即 **std::ranges::mismatch**。它属于 Ranges 库的一部分，旨在改进传统算法，使其更现代化、更易用，并支持范围（range）而不仅仅是迭代器对。std::ranges::mismatch 在功能上与 std::mismatch 等价，但提供了更简洁的语法和对范围对象的直接支持。

以下是对 std::ranges::mismatch 的解释，并将其与你的代码对比。

------

**基础知识**

- **std::ranges::mismatch**:
  - 功能：比较两个范围内的元素，返回第一个不匹配的元素对。
  - 返回值：一个 std::ranges::mismatch_result 对象，包含两个迭代器（in1 和 in2），分别指向第一个范围和第二个范围的不匹配位置。
  - 支持默认比较（==）和自定义谓词。
- **头文件**: <ranges>。
- **优点**:
  - 可以直接接受范围对象（如 std::vector、std::string），无需手动指定 begin() 和 end()。
  - 与 Ranges 库的其他功能（如视图）无缝集成。

------

**与你的代码对比**

**1. 默认比较：字符串比较**

**原始代码（使用 std::mismatch）**:

cpp

```cpp
#include <algorithm>
#include <string>

std::string text1 = "Welcome to the underworld!";
std::string text2 = "Welcome to the overworld!";
auto it = std::mismatch(text1.begin(), text1.end(), text2.begin());
// *it.first == 'u', *it.second == 'o'
```

**使用 std::ranges::mismatch**:

cpp

```cpp
#include <ranges>
#include <string>
#include <iostream>

int main() {
    std::string text1 = "Welcome to the underworld!";
    std::string text2 = "Welcome to the overworld!";
    auto result = std::ranges::mismatch(text1, text2);
    std::cout << *result.in1 << " " << *result.in2 << "\n"; // 输出: u o
}
```

- **变化**:
  - 直接传入 text1 和 text2（范围对象），无需显式调用 begin() 和 end()。
  - 返回类型是 std::ranges::mismatch_result，成员为 in1（第一个范围的迭代器）和 in2（第二个范围的迭代器）。
- **结果**: 仍然找到 'u' 和 'o' 作为第一个不匹配的字符对。

------

**2. 自定义谓词：向量比较**

**原始代码（使用 std::mismatch）**:

cpp

```cpp
#include <algorithm>
#include <vector>

std::vector<double> data = {6.0, 11.0, 2.1};
std::vector<double> args = {1.0, 0.5, 2.0};
auto res = std::mismatch(data.begin(), data.end(),
    args.begin(), [](double elem, double arg) {
        return elem * arg >= 5;
    });
// *res.first == 2.1, *res.second == 2.0
```

**使用 std::ranges::mismatch**:

cpp

```cpp
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    std::vector<double> data = {6.0, 11.0, 2.1};
    std::vector<double> args = {1.0, 0.5, 2.0};
    auto res = std::ranges::mismatch(data, args,
        [](double elem, double arg) { return elem * arg >= 5; });
    std::cout << *res.in1 << " " << *res.in2 << "\n"; // 输出: 2.1 2.0
}
```

- **变化**:
  - 直接传入 data 和 args 作为范围。
  - 自定义谓词作为第三个参数，逻辑不变。
  - 返回值是 std::ranges::mismatch_result，用 res.in1 和 res.in2 访问迭代器。
- **结果**: 仍然找到 2.1 和 2.0 作为第一个不满足 elem * arg >= 5 的元素对。

------

**关键特性**

1. **语法简化**:

   - std::ranges::mismatch 接受范围对象，省去了手动指定迭代器的麻烦。
   - 示例：std::mismatch(data.begin(), data.end(), args.begin()) → std::ranges::mismatch(data, args)。

2. **返回类型**:

   - std::mismatch 返回 std::pair<Iterator1, Iterator2>。

   - std::ranges::mismatch 返回 std::ranges::mismatch_result<Iterator1, Iterator2>，成员名为 in1 和 in2，更具语义化。

   - mismatch_result 是模板别名，可以用结构化绑定解构：

     cpp

     ```cpp
     auto [in1, in2] = std::ranges::mismatch(data, args, pred);
     ```

3. **支持投影**:

   - std::ranges::mismatch 还支持投影函数（projection），允许在比较前对元素进行变换。例如：

     cpp

     ```cpp
     auto res = std::ranges::mismatch(data, args,
         std::less{}, // 比较器
         [](double x) { return x * 2; }, // data 的投影
         [](double y) { return y; });    // args 的投影
     ```

4. **范围长度**:

   - 如果两个范围长度不同，std::ranges::mismatch 只比较较短的范围长度（与 std::mismatch 类似，但更明确）。

------

**完整示例**

以下是将你的代码改用 Ranges 库的完整版本：

cpp

```cpp
#include <ranges>
#include <string>
#include <vector>
#include <iostream>

int main() {
    // 字符串比较
    std::string text1 = "Welcome to the underworld!";
    std::string text2 = "Welcome to the overworld!";
    auto result1 = std::ranges::mismatch(text1, text2);
    std::cout << "Mismatch: " << *result1.in1 << " " << *result1.in2 << "\n"; // u o

    // 向量比较
    std::vector<double> data = {6.0, 11.0, 2.1};
    std::vector<double> args = {1.0, 0.5, 2.0};
    auto result2 = std::ranges::mismatch(data, args,
        [](double elem, double arg) { return elem * arg >= 5; });
    std::cout << "Mismatch: " << *result2.in1 << " " << *result2.in2 << "\n"; // 2.1 2.0
}
```

- **输出**:

  ```text
  Mismatch: u o
  Mismatch: 2.1 2.0
  ```

------

**中文解释**

**功能**

- std::ranges::mismatch 是 Ranges 库中的等价算法，用于找到两个范围中第一个不匹配的元素对。

**与 std::mismatch 的区别**

- **输入**: 接受范围对象（如 std::string、std::vector），无需显式迭代器。
- **返回**: 返回 mismatch_result，用 in1 和 in2 表示不匹配位置。
- **扩展**: 支持投影和更灵活的比较器。

**示例**

1. **字符串**: 比较 text1 和 text2，找到 'u' 和 'o'。
2. **向量**: 用谓词 elem * arg >= 5 比较 data 和 args，找到 2.1 和 2.0。

**优点**

- 更简洁的语法。
- 与 Ranges 生态（如视图、适配器）集成更好。

------

**总结**

std::ranges::mismatch 是 std::mismatch 的现代化替代品，功能相同但更符合 C++20 的 Ranges 设计理念。它简化了代码，提高了可读性，同时保留了自定义谓词的灵活性。如果你使用 C++20，推荐优先考虑 Ranges 版本，尤其是在处理范围对象时。

如果你有更多问题或想探讨其他 Ranges 算法，请告诉我！