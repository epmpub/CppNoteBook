C++26 中 std::println() 打印空行（Blank Lines）

是对 C++23 std::print/std::println 的一个小但实用的完善（P3142R0，作为 C++23 的 Defect Report）。 

en.cppreference.com

1. 新增重载（C++26）

cpp

```cpp
#include <print>

void println();                    // (3) since C++26
void println(std::FILE* stream);   // (4) since C++26
```

- 无参数版本：直接输出一个换行符（\n），等价于 std::print("\n") 或 std::println("")。
- 这使得打印空行变得更简洁、自然。
- 使用对比

cpp

```cpp
// C++23 方式（稍显繁琐）
std::println("Hello");
std::println("");          // 空行
std::println("World");

// C++26 方式（推荐）
std::println("Hello");
std::println();            // 空行，更清晰
std::println("World");
```

输出效果：



```text
Hello

World
```

3. 完整示例

cpp

```cpp
#include <print>

int main() {
    std::println("Ingredient list:");
    std::println("3 large bananas");
    std::println("2 large eggs");
    std::println();                    // 空行分隔
    std::println("280 g flour");
    std::println("150 g sugar");
    std::println();                    // 另一空行
    std::println("Bake at 350°F");
}
```

4. 动机（Why this change?）

- std::println("") 虽然能工作，但传递一个“空字符串”显得多余且不够语义化。
- 空行在输出格式化中非常常见（分隔段落、表格、日志等），标准库应提供最自然的写法。
- 提高代码可读性：空行在源代码中视觉上更清晰，维护时更容易插入/删除。
- 其他相关行为

- std::println(fmt, args...) 仍然在格式化内容后自动追加 \n。
- 新增的无参版本仅输出一个 \n，不进行任何格式化。
- 所有主流实现已在 C++23 模式下提前支持这些重载（即使正式在 C++26 标准化）。
- 特性测试宏

cpp

```cpp
#if __cpp_lib_print >= 202403L
// 支持无参 println()（C++26 DR）
#endif
```

总结：C++26 的这个小改动让 std::println() 使用起来更加符合直觉，尤其在需要频繁插入空行来提升输出可读性的场景中。现在你可以直接写 std::println(); 来打印空白行，而不再需要 std::println("");。这是一个“生活质量”级别的改进！