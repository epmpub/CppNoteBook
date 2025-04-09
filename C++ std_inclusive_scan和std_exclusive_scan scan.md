# *std::inclusive_scan*和*std::exclusive_scan* scan

*std::inclusive_scan*和*std::exclusive_scan* scan 是*std::reduce*变体，它们发出每个部分结果而不是产生单个值。

对于 inclusive_scan，第一个输出值已经包含第一个输入元素；对于 exclude_scan，第一个输出值是初始值。



```C++
#include <vector>
#include <numeric>
#include <print>

int main() {
    std::vector<int> data{2, 3, 4, 5, 6, 7};

    std::vector<int> incl;
    std::inclusive_scan(data.begin(), data.end(),
                        std::back_inserter(incl));
    // implicit init == int{} == 0
    // incl == {2, 5, 9, 14, 20, 27}

    std::println("incl == {}", incl);

    // Same as std::reduce, if the operation isn't associative,
    // the results will be non-deterministic
    std::inclusive_scan(data.begin(), data.end(), 
                        incl.begin(), std::plus<>{}, 100);
    // incl == {102, 105, 109, 114, 120, 127}

    std::println("incl == {}", incl);

    std::vector<int> excl;
    std::exclusive_scan(data.begin(), data.end(), 
                        std::back_inserter(excl), 1);
    // init is mandatory for exclusive_scan
    // excl == {1, 3, 6, 10, 15, 21}

    std::println("excl == {}", excl);

    std::vector<int> product;
    std::exclusive_scan(data.begin(), data.end(), 
        std::back_inserter(product),
        1, std::multiplies<>{});
    // product == {1, 2, 6, 24, 120, 720}

    std::println("product == {}", product);
}
```

这段代码展示了 C++17 中 <numeric> 库提供的 std::inclusive_scan 和 std::exclusive_scan 函数，用于对容器中的元素进行扫描（累积计算），并将结果存储到目标容器中。代码使用了加法和乘法两种操作，并通过 <print> 库（C++23）输出结果。以下是对代码的详细解释：

------

**代码结构**

1. **头文件**：
   - <vector>：提供 std::vector。
   - <numeric>：提供 std::inclusive_scan 和 std::exclusive_scan。
   - <print>：提供 C++23 的 std::println。
2. **主要内容**：
   - 使用 inclusive_scan 计算包含当前元素的累加和。
   - 使用 exclusive_scan 计算不包含当前元素的累积结果。
   - 展示不同初始值和操作符的效果。

------

**代码逐步解释**

**1. 初始化数据**

cpp

```cpp
std::vector<int> data{2, 3, 4, 5, 6, 7};
```

- **data**：包含 {2, 3, 4, 5, 6, 7} 的向量，作为输入。

------

**2. std::inclusive_scan（默认加法）**

cpp

```cpp
std::vector<int> incl;
std::inclusive_scan(data.begin(), data.end(), std::back_inserter(incl));
// incl == {2, 5, 9, 14, 20, 27}
std::println("incl == {}", incl);
```

- **std::inclusive_scan**：
  - 计算包含当前元素的累积结果。
  - 签名：inclusive_scan(first, last, result, [op], [init])。
  - 这里未指定操作符（默认 std::plus<>）和初始值（默认 int{} 即 0）。
- **计算过程**：
  - incl[0] = data[0] = 2。
  - incl[1] = data[0] + data[1] = 2 + 3 = 5。
  - incl[2] = data[0] + data[1] + data[2] = 2 + 3 + 4 = 9。
  - incl[3] = 2 + 3 + 4 + 5 = 14。
  - incl[4] = 2 + 3 + 4 + 5 + 6 = 20。
  - incl[5] = 2 + 3 + 4 + 5 + 6 + 7 = 27。
- **std::back_inserter**：
  - 动态插入结果到 incl，无需预分配空间。
- **输出**：incl == {2, 5, 9, 14, 20, 27}。

------

**3. std::inclusive_scan（指定初始值）**

cpp

```cpp
std::inclusive_scan(data.begin(), data.end(), incl.begin(), std::plus<>{}, 100);
// incl == {102, 105, 109, 114, 120, 127}
std::println("incl == {}", incl);
```

- **参数**：
  - incl.begin()：覆盖之前的 incl。
  - std::plus<>{}：显式指定加法操作。
  - 100：初始值。
- **计算过程**：
  - incl[0] = 100 + data[0] = 100 + 2 = 102。
  - incl[1] = 100 + data[0] + data[1] = 100 + 2 + 3 = 105。
  - incl[2] = 100 + 2 + 3 + 4 = 109。
  - incl[3] = 100 + 2 + 3 + 4 + 5 = 114。
  - incl[4] = 100 + 2 + 3 + 4 + 5 + 6 = 120。
  - incl[5] = 100 + 2 + 3 + 4 + 5 + 6 + 7 = 127。
- **注意**：
  - 如果操作符非关联（如减法），结果可能不确定，但加法是关联的，结果稳定。
- **输出**：incl == {102, 105, 109, 114, 120, 127}。

------

**4. std::exclusive_scan（加法）**

cpp

```cpp
std::vector<int> excl;
std::exclusive_scan(data.begin(), data.end(), std::back_inserter(excl), 1);
// excl == {1, 3, 6, 10, 15, 21}
std::println("excl == {}", excl);
```

- **std::exclusive_scan**：
  - 计算不包含当前元素的累积结果。
  - 签名：exclusive_scan(first, last, result, init, [op])。
  - 初始值 1 是必需的，默认操作符为 std::plus<>。
- **计算过程**：
  - excl[0] = 1（初始值）。
  - excl[1] = 1 + data[0] = 1 + 2 = 3。
  - excl[2] = 1 + 2 + 3 = 6。
  - excl[3] = 1 + 2 + 3 + 4 = 10。
  - excl[4] = 1 + 2 + 3 + 4 + 5 = 15。
  - excl[5] = 1 + 2 + 3 + 4 + 5 + 6 = 21。
- **特点**：
  - 每个元素是前缀和，不包括当前元素。
- **输出**：excl == {1, 3, 6, 10, 15, 21}。

------

**5. std::exclusive_scan（乘法）**

cpp

```cpp
std::vector<int> product;
std::exclusive_scan(data.begin(), data.end(), 
    std::back_inserter(product), 1, std::multiplies<>{});
// product == {1, 2, 6, 24, 120, 720}
std::println("product == {}", product);
```

- **参数**：
  - 初始值 1。
  - std::multiplies<>{}：乘法操作。
- **计算过程**：
  - product[0] = 1（初始值）。
  - product[1] = 1 * data[0] = 1 * 2 = 2。
  - product[2] = 1 * 2 * 3 = 6。
  - product[3] = 1 * 2 * 3 * 4 = 24。
  - product[4] = 1 * 2 * 3 * 4 * 5 = 120。
  - product[5] = 1 * 2 * 3 * 4 * 5 * 6 = 720。
- **效果**：
  - 计算阶乘的前缀积。
- **输出**：product == {1, 2, 6, 24, 120, 720}。

------

**关键技术点**

1. **std::inclusive_scan**：
   - 包含当前元素的前缀和。
   - 初始值可选，默认 0。
2. **std::exclusive_scan**：
   - 不包含当前元素的前缀和。
   - 初始值必需。
3. **操作符**：
   - 默认 std::plus<>，可自定义（如 std::multiplies<>）。
   - 非关联操作可能导致不确定结果，但加法和乘法是关联的。
4. **std::back_inserter**：
   - 动态扩展目标容器，适合空向量。
5. **<print>**：
   - C++23 的格式化输出，简洁展示容器内容。

------

**输出总结**

```text
incl == {2, 5, 9, 14, 20, 27}
incl == {102, 105, 109, 114, 120, 127}
excl == {1, 3, 6, 10, 15, 21}
product == {1, 2, 6, 24, 120, 720}
```

------

**可能的改进或注意事项**

1. **性能**：
   - 可结合 <execution> 使用并行版本（如 std::execution::par）。
2. **输出验证**：
   - 当前仅打印向量，可添加计算结果的校验。
3. **初始值含义**：
   - exclusive_scan 的初始值影响整个序列，应根据需求选择。

------

**总结**

- **inclusive_scan**：计算包含当前项的累积结果。
- **exclusive_scan**：计算不含当前项的累积结果。
- **用途**：前缀和、阶乘等场景。
- **现代 C++**：结合 C++17 和 C++23 特性，代码简洁高效。

如果你有具体问题（例如并行版本），欢迎提问！