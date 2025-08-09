# std::logical_and*，*std::logical_or*和*std::logical_not

三个函数对象*std::logical_and*，*std::logical_or*和*std::logical_not模拟相应逻辑运算符**&& （与）*，*|| （或）*，*! （非）*的功能。

*请注意，函数调用的所有参数都会在函数调用之前以未指定的顺序进行求值。这与&&*和*||*运算符的短路求值形成对比。



```C++
#include <iostream>
#include <vector>
#include <algorithm>
#include <functional>

int main() {
    std::vector<std::optional<int>> data{1,{},2,3,{},4,5,6,{},{}};

    // Partition the vector with empty optionals first
    std::partition(data.begin(), data.end(), std::logical_not<>{});
    // same as:
    std::partition(data.begin(), data.end(),
        [](const auto& o) { return not o.has_value(); });

    for (auto& v : data)
        if (v)
            std::cout << *v << ", ";
        else
            std::cout << "{}, ";
    std::cout << '\n';

    std::vector<bool> in1{false, true, true, false, false, true, false, true};
    std::vector<bool> in2{true, true, false, false, true, false, true, false};
    std::vector<bool> out1(in1.size());
    
    // element-wise reduction using logical AND from in1 and in2 to out1
    std::transform(in1.begin(), in1.end(), in2.begin(), out1.begin(),
        std::logical_and<>{});
    for (auto v : out1)
        std::cout << v;
    std::cout << "\n";

    std::vector<bool> out2(in1.size());

    // element-wise reduction using logical AND from in1 and in2 to out1
    std::transform(in1.begin(), in1.end(), in2.begin(), out2.begin(),
        std::logical_or<>{});

    for (auto v : out2)
        std::cout << v;
    std::cout << "\n";

    std::cout << "\n";

    // Important: because std::logical_and, std::logical_or are 
    // function objects, they do not have short-circuit logic 
    // of && and || operators.
    auto fn1 = []{ std::cout << "fn1()\n"; return true; };
    auto fn2 = []{ std::cout << "fn2()\n"; return true; };
    auto fn3 = []{ std::cout << "fn3()\n"; return false; };
    auto fn4 = []{ std::cout << "fn4()\n"; return false; };
   
    // Only fn1 is called
    bool r1 = fn1() || fn2();
    std::cout << "\n";
    // Both fn1 and fn2 is called in unspecified order
    bool r2 = std::logical_or<>{}(fn1(), fn2());
    std::cout << "\n";
    // Only fn3 is called
    bool r3 = fn3() && fn4();
    std::cout << "\n";
    // Both fn3 and fn4 is called in unspecified order
    bool r4 = std::logical_and<>{}(fn3(), fn4());
}
```





这段代码展示了 C++ 中 <algorithm> 和 <functional> 库的用法，特别是 std::partition、std::transform 以及逻辑操作函数对象（如 std::logical_and 和 std::logical_or）的应用。代码通过示例展示了这些工具的行为，并对比了函数对象与内置逻辑运算符（如 && 和 ||）的区别。以下是逐步解释。

------

代码概览

- 使用 std::partition 对包含 std::optional 的向量重新排序。
- 使用 std::transform 对两个布尔向量进行元素级逻辑操作。
- 对比逻辑函数对象和短路逻辑运算符的行为。

------

关键组件

1. **头文件**

cpp

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <functional>
```

- <iostream>：提供 std::cout。
- <vector>：提供 std::vector。
- <algorithm>：提供 std::partition 和 std::transform。
- <functional>：提供 std::logical_not、std::logical_and 和 std::logical_or。
- **main 函数**

**分区操作**

cpp

```cpp
std::vector<std::optional<int>> data{1,{},2,3,{},4,5,6,{},{}};
std::partition(data.begin(), data.end(), std::logical_not<>{});
for (auto& v : data)
    if (v)
        std::cout << *v << ", ";
    else
        std::cout << "{}, ";
std::cout << '\n';
```

- **data**：

  - 包含 10 个 std::optional<int>，其中 5 个有值，5 个为空。

- **std::partition**：

  - 将范围按谓词分区，空 optional 在前。
  - std::logical_not<>{} 转换为 !o.has_value()，即空返回 true。

- **等价形式**：

  - [](const auto& o) { return not o.has_value(); }。

- **结果**：

  - data 变为 { {}, {}, {}, {}, {}, 1, 2, 3, 4, 5, 6 }。

- **输出**：

  ```text
  {}, {}, {}, {}, {}, 1, 2, 3, 4, 5, 6,
  ```

**逻辑与操作**

cpp

```cpp
std::vector<bool> in1{false, true, true, false, false, true, false, true};
std::vector<bool> in2{true, true, false, false, true, false, true, false};
std::vector<bool> out1(in1.size());
std::transform(in1.begin(), in1.end(), in2.begin(), out1.begin(), std::logical_and<>{});
for (auto v : out1)
    std::cout << v;
std::cout << "\n";
```

- **std::transform**：

  - 对 in1 和 in2 的对应元素应用 std::logical_and<>{}。
  - 结果存入 out1。

- **std::logical_and**：

  - 元素级逻辑与：out1[i] = in1[i] && in2[i]。

- **结果**：

  - out1 = {false, true, false, false, false, false, false, false}。

- **输出**：

  ```text
  01000000
  ```

**逻辑或操作**

cpp

```cpp
std::vector<bool> out2(in1.size());
std::transform(in1.begin(), in1.end(), in2.begin(), out2.begin(), std::logical_or<>{});
for (auto v : out2)
    std::cout << v;
std::cout << "\n";
```

- **std::logical_or**：

  - 元素级逻辑或：out2[i] = in1[i] || in2[i]。

- **结果**：

  - out2 = {true, true, true, false, true, true, true, true}。

- **输出**：

  ```text
  11101111
  ```

**短路逻辑对比**

cpp

```cpp
auto fn1 = []{ std::cout << "fn1()\n"; return true; };
auto fn2 = []{ std::cout << "fn2()\n"; return true; };
auto fn3 = []{ std::cout << "fn3()\n"; return false; };
auto fn4 = []{ std::cout << "fn4()\n"; return false; };
bool r1 = fn1() || fn2();
bool r2 = std::logical_or<>{}(fn1(), fn2());
bool r3 = fn3() && fn4();
bool r4 = std::logical_and<>{}(fn3(), fn4());
```

- **|| 和 &&**：

  - 短路求值：
    - fn1() || fn2()：fn1() 为 true，跳过 fn2()。
    - fn3() && fn4()：fn3() 为 false，跳过 fn4()。

- **std::logical_or 和 std::logical_and**：

  - 函数对象，全程求值：
    - std::logical_or<>{}(fn1(), fn2())：两个函数都调用。
    - std::logical_and<>{}(fn3(), fn4())：两个函数都调用。

- **输出**：

  ```text
  fn1()
  
  fn1()
  fn2()
  
  fn3()
  
  fn3()
  fn4()
  ```

------

为什么这样工作？

1. **std::partition**：
   - 根据谓词重新排列元素，std::logical_not 简化逻辑。
2. **std::transform**：
   - 元素级应用函数对象，std::logical_and 和 std::logical_or 无短路。
3. **短路 vs 非短路**：
   - || 和 && 是运算符，支持短路。
   - 函数对象是普通函数调用，参数先求值。

------

输出

```text
{}, {}, {}, {}, {}, 1, 2, 3, 4, 5, 6,
01000000
11101111

fn1()

fn1()
fn2()

fn3()

fn3()
fn4()
```

------

使用场景

- **分区**：
  - 按条件重新排序数据。
- **批量逻辑操作**：
  - 对向量进行元素级计算。
- **控制求值**：
  - 选择短路或全程逻辑。

------

总结

- std::partition 将空 optional 移到前面。
- std::transform 用逻辑函数对象处理布尔向量。
- 函数对象无短路，运算符有短路。
- 代码展示了算法和函数对象的应用及区别。