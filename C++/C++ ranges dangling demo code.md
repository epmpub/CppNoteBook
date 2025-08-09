C++ ranges dangling demo code ：

------



```cpp
#include <ranges>
#include <vector>
#include <iostream>

auto get_data()
{
    return std::vector<int>{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
        | std::views::filter([](int i) { return i % 2 == 0; });
}

int main()
{
    auto data = get_data();
    for (auto i : data)
    {
        std::cout << i << ' ';
    }
    std::cout << '\n';
}
```



### 逐步解释：

1. **创建临时向量**：
   ```cpp
   std::vector<int>{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
   ```
   - 在`get_data`函数中，构造一个临时的`std::vector<int>`，包含数字1到10。这是一个右值（rvalue），因为它没有名称，直接作为临时对象存在。

2. **隐式应用`views::all`**：
   - 使用管道操作符`|`将临时向量传递给`std::views::filter`时，C++范围库会自动对左侧的操作数应用`std::views::all`。
   - 由于临时向量是右值，`std::views::all`会生成一个`std::ranges::owning_view`，该视图拥有临时向量的所有权，延长其生命周期至与视图本身一致。

3. **应用过滤视图**：
   ```cpp
   | std::views::filter([](int i) { return i % 2 == 0; })
   ```
   - `std::views::filter`接收一个谓词（此处为Lambda表达式，筛选偶数），创建一个惰性视图，仅包含满足条件的元素。

4. **返回组合视图**：
   - `get_data`函数返回由`owning_view`和`filter_view`组合而成的视图。由于`owning_view`持有临时向量的所有权，即使函数返回，向量仍不会被销毁。

5. **在`main`中使用视图**：
   ```cpp
   auto data = get_data();
   for (auto i : data)
   {
       std::cout << i << ' ';
   }
   ```
   - `data`接收从`get_data`返回的视图。遍历该视图时，`owning_view`确保底层向量仍然有效，因此能安全访问其元素。
   - 输出结果为过滤后的偶数：`2 4 6 8 10`。

### 关键点总结：
- **右值与`owning_view`**：通过将向量构造为右值，`std::views::all`自动创建`owning_view`，拥有向量所有权，延长其生命周期。
- **隐式`views::all`调用**：使用管道操作符时，编译器自动插入`views::all`，简化代码。
- **生命周期管理**：`owning_view`确保临时向量存活至视图销毁，避免悬垂引用。

此修复有效解决了原始代码中因局部向量销毁导致的悬垂范围问题，保证了视图操作的安全性。