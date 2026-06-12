好的，这里给出一个 **完全 C++26 风格**、基于 **Oxford variadic comma** 的可变参数打印函数示例，无需 `va_list`，类型安全且零运行时开销：

```cpp
#include <iostream>

// 使用 C++26 Oxford variadic comma
void print_values_legacy(auto ...values ... ) {
    constexpr size_t N = sizeof...(values);
    size_t i = 0;
    ((std::cout << values << (++i < N ? ", " : "")), ...); // fold expression
    std::cout << '\n';
}

int main() {
    print_values_legacy(10, 20, 30);
    print_values_legacy(1, 2, 3, 4, 5);
    print_values_legacy(42); // 单个参数也支持
}
```

### 解析

1. `auto ...values ...`
   - 这是 **C++26 Oxford variadic comma** 的写法，允许直接在函数参数列表中定义可变参数包。
2. `sizeof...(values)`
   - 计算可变参数的数量，用于判断是否添加逗号，实现 Oxford comma 风格。
3. `((std::cout << values << (++i < N ? ", " : "")), ...);`
   - **折叠表达式**：逐个输出参数，如果不是最后一个就加逗号。
4. 输出示例：

```
10, 20, 30
1, 2, 3, 4, 5
42
```

------

这个版本比 C 风格 `va_list` 安全得多，也完全利用了 C++26 新特性。

如果你希望，我可以写一个 **支持任意类型（int、double、string…）都能打印，并保持 Oxford comma 风格** 的泛型版本。

你想让我写吗？