

### Oxford variadic comma

用 **C++26** 标准，并结合 **Oxford variadic comma** 的语法特性，来实现一个类似 C 风格的可变参数函数 `print_values_legacy`。C++26 引入了对 **经典 C 风格可变参数 (`...`)** 的改进支持，也允许在参数列表中直接使用 **Oxford variadic comma**。

下面我给出一个可编译的示例，完整实现 `print_values_legacy`：

```cpp
#include <iostream>
#include <cstdarg>

// C 风格可变参数函数，使用 Oxford variadic comma
void print_values_legacy(int count ... ) {
    va_list args;
    va_start(args, count);

    for (int i = 0; i < count; ++i) {
        int value = va_arg(args, int);
        std::cout << value;
        if (i < count - 1) std::cout << ", "; // Oxford comma 风格
    }
    va_end(args);
    std::cout << '\n';
}

int main() {
    print_values_legacy(3, 10, 20, 30);
    print_values_legacy(5, 1, 2, 3, 4, 5);
}
```

### 解析

1. `int count ...`：这里的 `count` 指定了可变参数的数量，这是 **Oxford variadic comma** 的写法（C++26 允许这种更自然的语法）。
2. `va_list args; va_start(args, count);`：初始化可变参数列表。
3. `va_arg(args, int)`：依次读取 `int` 类型参数。
4. Oxford comma 风格：在打印列表时，如果有多个元素，会在倒数第二个和最后一个之间加逗号。

输出效果：

```
10, 20, 30
1, 2, 3, 4, 5
```

------

如果你愿意，我可以给一个 **更现代 C++26 版本**，完全去掉 `va_list`，用 **模板 + Oxford variadic comma** 实现零开销可变参数打印函数，比 C 风格更安全。

你希望我直接给这个现代版本吗？