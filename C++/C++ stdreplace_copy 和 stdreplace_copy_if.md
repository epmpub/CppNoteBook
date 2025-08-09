

# C++ std::replace_copy 和 std::replace_copy_if

当我们需要复制某个范围的内容，并在复制时替换某些元素时，我们可以使用*std::replace_copy*和*std::replace_copy_if*算法。

两种算法都支持其范围版本的投影。



```C++
#include <algorithm>
#include <vector>
#include <iostream>
#include <iomanip>

int main() {
    std::vector<int> in{1,2,3,4,5};
    std::vector<int> out(5);

    std::replace_copy(in.begin(), in.end(), // copy from in
        out.begin(), // to out
        2, -1); // replacing elements of value 2 with -1
    // out == {1, -1, 3, 4, 5}

    for (auto v : out)
        std::cout << v << " ";
    std::cout << "\n";

    std::replace_copy_if(in.begin(), in.end(), // copy from in
        out.begin(), // to out
        [](int v) { return v % 2 != 0; }, // replacing odd elements
        0); // with zero
    // out == {0, 2, 0, 4, 0}

    for (auto v : out)
        std::cout << v << " ";
    std::cout << "\n";

    std::vector<std::string> labels{"a", "b", "hello", "bye", "e"};
    std::vector<std::string> out2(5);

    // Example with a projection
    std::ranges::replace_copy(labels, out2.begin(),
        1, "---", // Replace strings of length 1 with "---"
        [](const std::string& s) { return s.length(); });
    // out2 == {"---", "---", "hello", "bye", "---"}
    
    for (auto v : out2)
        std::cout << std::quoted(v) << " ";
    std::cout << "\n";
}
```



这段代码展示了 C++ 中用于复制并替换元素的算法：std::replace_copy 和 std::replace_copy_if（<algorithm>），以及 C++20 的 std::ranges::replace_copy（<ranges>）。代码通过示例展示了它们的功能，包括直接值替换、条件替换和使用投影的替换。以下是逐步解释。

------

代码概览

- 使用 std::replace_copy 将特定值替换并复制到新容器。
- 使用 std::replace_copy_if 根据条件替换并复制。
- 使用 std::ranges::replace_copy 结合投影替换并复制。

------

关键组件

1. **头文件**

cpp

```cpp
#include <algorithm>
#include <vector>
#include <iostream>
#include <iomanip>
```

- <algorithm>：提供 std::replace_copy、std::replace_copy_if 和 std::ranges::replace_copy。
- <vector>：提供 std::vector。
- <iostream>：用于输出。
- <iomanip>：提供 std::quoted 用于格式化字符串。
- **std::replace_copy 示例**

cpp

```cpp
std::vector<int> in{1,2,3,4,5};
std::vector<int> out(5);
std::replace_copy(in.begin(), in.end(), out.begin(), 2, -1);
for (auto v : out)
    std::cout << v << " ";
std::cout << "\n";
```

- **in 和 out**：

  - in = {1, 2, 3, 4, 5}。
  - out：预分配 5 个元素，初始值无关紧要（将被覆盖）。

- **std::replace_copy**：

  - 原型：replace_copy(first, last, result, old_value, new_value)。
  - 从 [first, last) 复制到 result，将 old_value 替换为 new_value。

- **调用**：

  - 将 2 替换为 -1，其他值不变。
  - out = {1, -1, 3, 4, 5}。

- **输出**：

  ```text
  1 -1 3 4 5
  ```

- **std::replace_copy_if 示例**

cpp

```cpp
std::replace_copy_if(in.begin(), in.end(), out.begin(),
    [](int v) { return v % 2 != 0; }, 0);
for (auto v : out)
    std::cout << v << " ";
std::cout << "\n";
```

- **std::replace_copy_if**：

  - 原型：replace_copy_if(first, last, result, pred, new_value)。
  - 从 [first, last) 复制到 result，若 pred 为真则替换为 new_value。

- **调用**：

  - 谓词 v % 2 != 0 检查奇数。
  - 奇数替换为 0，偶数保持不变。
  - out = {0, 2, 0, 4, 0}。

- **输出**：

  ```text
  0 2 0 4 0
  ```

- **std::ranges::replace_copy 示例**

cpp

```cpp
std::vector<std::string> labels{"a", "b", "hello", "bye", "e"};
std::vector<std::string> out2(5);
std::ranges::replace_copy(labels, out2.begin(),
    1, "---", [](const std::string& s) { return s.length(); });
for (auto v : out2)
    std::cout << std::quoted(v) << " ";
std::cout << "\n";
```

- **labels 和 out2**：

  - labels = {"a", "b", "hello", "bye", "e"}。
  - out2：预分配 5 个元素。

- **std::ranges::replace_copy**：

  - 原型：replace_copy(range, result, old_value, new_value, proj)。
  - 从 range 复制到 result，将投影值等于 old_value 的元素替换为 new_value。

- **调用**：

  - 投影 s.length() 返回字符串长度。
  - old_value = 1，替换长度为 1 的字符串为 "---"。
  - labels 中 "a", "b", "e" 的长度为 1。
  - out2 = {"---", "---", "hello", "bye", "---"}。

- **std::quoted**：

  - 为字符串添加引号，便于阅读。

- **输出**：

  ```text
  "---" "---" "hello" "bye" "---"
  ```

------

为什么这样工作？

1. **std::replace_copy**：
   - 直接比较元素值，替换并复制。
2. **std::replace_copy_if**：
   - 使用谓词灵活指定替换条件。
3. **std::ranges::replace_copy**：
   - 支持投影，基于投影值比较，增强灵活性。
4. **复制语义**：
   - 所有函数不修改输入，仅填充输出。

------

输出

```text
1 -1 3 4 5
0 2 0 4 0
"---" "---" "hello" "bye" "---"
```

------

使用场景

- **数据转换**：
  - 替换特定值或满足条件的元素。
- **预处理**：
  - 生成修改后的副本。
- **复杂类型**：
  - 使用投影处理结构体或容器元素。

------

总结

- std::replace_copy 将 2 替换为 -1。
- std::replace_copy_if 将奇数替换为 0。
- std::ranges::replace_copy 将长度为 1 的字符串替换为 "---"。
- 代码展示了替换复制算法的多样性和应用。