# std::views::as_const

std::views::as_const 是 C++23 引入的一个 Ranges 视图适配器，定义在 <ranges> 头文件中。它的作用是将输入范围的元素转换为常量引用（const），生成一个新的视图，从而防止通过该视图修改原始数据。这在需要只读访问或确保数据不变性时非常有用，同时保持了 Ranges 视图的惰性求值特性。

以下是对 std::views::as_const 的详细解释：

------

定义

cpp

```cpp
#include <ranges>

namespace std::views {
 inline constexpr auto as_const = /* unspecified */;
}
```

- **as_const**：
- 将输入范围的元素类型包装为 const 引用。
- **语法**：

cpp

```cpp
auto view = std::views::as_const(range);
```

- **range**: 输入范围，需满足 std::ranges::input_range。
- 返回类型：std::ranges::as_const_view<R>。

------

行为

- **std::views::as_const**：
- 创建一个视图，其中每个元素是原始范围元素的 const 引用。
- 不复制数据，仅改变访问权限。
- **惰性求值**：
- 视图仅在迭代时生效。
- **元素类型**：
- 如果原始范围的引用类型是 T&，则视图的引用类型是 const T&。

------

前提条件

- **范围要求**：
- 输入范围必须满足 std::ranges::input_range。
- 元素必须可转换为 const 引用。
- **适用性**：
- 适用于任何可迭代的容器（如 std::vector、std::array）。

------

示例代码

示例 1：基本用法

cpp

```cpp
#include <ranges>
#include <vector>
#include <iostream>

int main() {
 std::vector<int> vec = {1, 2, 3, 4};

 auto view = std::views::as_const(vec);

 std::cout << "View: ";
 for (const int& x : view) {
 	std::cout << x << " ";
 }
 std::cout << "\n";

 // view[0] = 10; // 错误：不能修改
 vec[0] = 10; // 正确：原始向量可修改

 std::cout << "After modification: ";
 for (const int& x : view) {
	 std::cout << x << " ";
 }
 std::cout << "\n";

 return 0;
}
```

输出

```text
View: 1 2 3 4
After modification: 10 2 3 4
```

- **解释**：
- view 是  vec 的只读视图。
- 不能通过 view 修改元素，但原始 vec 可修改，视图反映变化。

示例 2：与管道操作符结合

cpp

```cpp
#include <ranges>
#include <vector>
#include <iostream>

int main() {
 std::vector<int> vec = {1, 2, 3, 4, 5};

 auto even_const = vec | std::views::as_const | std::views::filter(
     [](int x) { return x % 2 == 0; }
 );

 std::cout << "Even numbers (const): ";
 for (const int& x : even_const) {
	 std::cout << x << " ";
 }
 std::cout << "\n";

 return 0;
}
```

输出

```text
Even numbers (const): 2 4
```

- **解释**：
- as_const 确保 filter 视图中的元素是 const int&，不可修改。

------

返回视图的类型

- **std::ranges::as_const_view<R>**：
- R 是输入范围类型。
- **元素类型**：
- std::ranges::range_reference_t<R> 变为 const T&（若原始是 T&）。

------

时间复杂度

- **构造**：O(1)，视图是惰性的。
- **迭代**：
- 每次访问：O(1)。
- 总复杂度：O(n)，n 是范围大小。

------

使用场景

1. **只读访问**：

- 防止意外修改数据。

cpp

```cpp
auto const_view = std::views::as_const(vec);
```

1. **接口设计**：

- 传递只读视图给函数，确保数据不变性。

1. **与 Ranges 组合**：

- 在视图管道中强制常量性。

------

注意事项

1. **C++23 要求**：

- 需要 -std=c++23 和支持 C++23 的编译器。

1. **原始数据可变**：

- as_const 只限制视图的修改，原始范围仍可变。

1. **性能**：

- 无额外内存开销，仅改变引用类型。

------

与其他 方法的对比

**std::as_const（C++17）**

- **std::as_const(vec)**：
- 返回整个对象的 const 引用（const std::vector<int>&）。
- 不生成视图，适用于单对象。
- **std::views::as_const**：
- 生成范围视图，元素是 const 引用。

示例对比

cpp

```cpp
#include <vector>
#include <ranges>
#include <iostream>

int main() {
 std::vector<int> vec = {1, 2, 3};

 auto const_ref = std::as_const(vec); // const std::vector<int>&
 auto const_view = std::views::as_const(vec); // as_const_view

 // const_ref[0] = 10; // 错误
 // const_view[0] = 10; // 错误
 vec[0] = 10; // 正确

 std::cout << const_view[0] << "\n"; // 10
 return 0;
}
```

------

替代方案（C++20 前）

在 C++20 中，可以用 std::ranges::transform 模拟：

cpp

```cpp
auto const_view = vec | std::views::transform([](const int& x) -> const int& { return x; });
```

- **缺点**：不够简洁，需显式 lambda。

------

总结

std::views::as_const 是 C++23 中一个轻量级视图适配器，将范围元素转换为 const 引用，防止修改。它与 Ranges 生态无缝集成，提供只读访问的同时保留惰性求值特性。相比 std::as_const，它专注于范围而非整个对象，适合现代 C++ 编程。如果你有具体问题或想探讨用法，请告诉我！ 