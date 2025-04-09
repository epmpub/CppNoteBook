

# std::format 自定义格式化程序

通过实现自定义格式化程序，支持使用 std::format 格式化用户类型。

*当通过std::format*和*std::print*启用自定义类型的格式化时，我们不仅必须指定如何打印我们的类型，还必须指定如何解析格式字符串。

虽然打印是运行时操作，但格式解析是在编译时完成的，需要 constexpr 实现

```C++
#include <format>
#include <print>

struct CustomObject {};

// Specialization for the formatter type:
template <> struct std::formatter<CustomObject> {
    // We do not parse anything, but we still need to advance 
    // the iterator over the corresponding {} in the format.
    // This happens at compile-time.
    constexpr auto parse(std::format_parse_context& ctx) {
        auto it = ctx.begin();
        while (it != ctx.end() && *it != '}') 
            ++it;
        return it;
    }
    // Runtime formatting, we simply add "CustomObject" to the buffer
    auto format(const CustomObject&, auto& ctx) const {
        return std::format_to(ctx.out(), "CustomObject");
    }
};

struct Wrapper {
    int value;
};

// Since our wrapper is effectively an int
// we can inherit from the int formatter.
template <typename CharT>
struct std::formatter<Wrapper, CharT> : std::formatter<int, CharT> {  
    // parse() is inherited, we need a format method matching our type
    auto format(const Wrapper& v, auto& ctx) const {
        return std::formatter<int, CharT>::format(v.value, ctx);
    }
};

struct Greeter {};

// Custom format example: {}, {:u} or {:w}
template <> struct std::formatter<Greeter> {
    void unexpected_format_specification_for_greeter(){}
    constexpr auto parse(std::format_parse_context& ctx) {
        auto it = ctx.begin();
        while (it != ctx.end() && *it != '}') {
            switch (*it) {
                case 'w': object = "World"; break;
                case 'u': object = "Universe"; break;
                // we call a non-constexpr function to trigger an error
                default: unexpected_format_specification_for_greeter();
            }
            ++it;
        }
        return it;
    }
    auto format(const Greeter&, auto& ctx) const {
        return std::format_to(ctx.out(), "Hello {}!", object);
    }
    std::string_view object = "World";
};

int main() {
    auto str1 = std::format("{}, {}", CustomObject{}, CustomObject{});
    // str1 == "CustomObject, CustomObject"

    std::println("{}, {}", CustomObject{}, CustomObject{});

    auto str2 = std::format("0x{:X}", Wrapper{255});
    // str2 == "0xFF"

    std::println("0x{:X}", Wrapper{255});

    auto str3 = std::format("{:u}", Greeter{});
    // str3 == "Hello Universe!"

    std::println("{:w}", Greeter{});
    std::println("{:u}", Greeter{});

    // std::format("{:Q}", Greeter{}); // Wouldn't compile:
    // call to non-'constexpr' function
    // ...unexpected_format_specification_for_greeter()
}
```

这段代码展示了 C++20 中 <format> 库的自定义格式化支持，通过为用户定义类型特化 std::formatter 来扩展 std::format 和 C++23 的 std::println 的功能。代码定义了三个结构体（CustomObject、Wrapper 和 Greeter），并为它们提供了不同的格式化逻辑。以下是对代码的详细解释：

------

**代码结构**

1. **头文件**：
   - <format>：提供 std::format 和格式化基础设施。
   - <print>：C++23 的 std::println，依赖 <format>。
2. **主要内容**：
   - 为 CustomObject 定义简单格式化。
   - 为 Wrapper 继承 int 的格式化。
   - 为 Greeter 实现带自定义格式说明符的格式化。
3. **验证**：
   - 在 main 中测试格式化结果。

------

**代码逐步解释**

**1. CustomObject 的格式化**

cpp

```cpp
struct CustomObject {};

template <> struct std::formatter<CustomObject> {
    constexpr auto parse(std::format_parse_context& ctx) {
        auto it = ctx.begin();
        while (it != ctx.end() && *it != '}') 
            ++it;
        return it;
    }
    auto format(const CustomObject&, auto& ctx) const {
        return std::format_to(ctx.out(), "CustomObject");
    }
};
```

- **std::formatter<CustomObject>**：
  - 特化 std::formatter 以支持 CustomObject。
- **parse**：
  - 处理格式字符串中的说明符（例如 {}）。
  - 这里不解析具体说明符，仅跳过 {} 中的内容，直到遇到 }。
  - constexpr：在编译期执行，确保格式解析是静态的。
  - 返回迭代器，指向 } 后的位置。
- **format**：
  - 将 "CustomObject" 写入输出缓冲区。
  - 参数：
    - const CustomObject&：要格式化的对象（未使用）。
    - ctx：格式化上下文，提供输出迭代器 ctx.out()。
  - 返回输出迭代器。
- **效果**：
  - 无论格式说明符如何，总是输出 "CustomObject"。

------

**2. Wrapper 的格式化（继承）**

cpp

```cpp
struct Wrapper {
    int value;
};

template <typename CharT>
struct std::formatter<Wrapper, CharT> : std::formatter<int, CharT> {  
    auto format(const Wrapper& v, auto& ctx) const {
        return std::formatter<int, CharT>::format(v.value, ctx);
    }
};
```

- **Wrapper**：
  - 包含一个 int 成员 value。
- **std::formatter<Wrapper, CharT>**：
  - 继承 std::formatter<int, CharT>，重用 int 的格式化逻辑。
  - CharT：字符类型（例如 char 或 wchar_t），支持多字符集。
- **parse**：
  - 继承自 std::formatter<int, CharT>，无需显式定义。
  - 支持 int 的格式说明符（如 :X 表示十六进制）。
- **format**：
  - 将 Wrapper::value 传递给 int 的格式化器。
- **效果**：
  - Wrapper 的格式化等同于其 value 的格式化。

------

**3. Greeter 的自定义格式化**

cpp

```cpp
struct Greeter {};

template <> struct std::formatter<Greeter> {
    void unexpected_format_specification_for_greeter(){}
    constexpr auto parse(std::format_parse_context& ctx) {
        auto it = ctx.begin();
        while (it != ctx.end() && *it != '}') {
            switch (*it) {
                case 'w': object = "World"; break;
                case 'u': object = "Universe"; break;
                default: unexpected_format_specification_for_greeter();
            }
            ++it;
        }
        return it;
    }
    auto format(const Greeter&, auto& ctx) const {
        return std::format_to(ctx.out(), "Hello {}!", object);
    }
    std::string_view object = "World";
};
```

- **Greeter**：
  - 空结构体，仅用于格式化演示。
- **std::formatter<Greeter>**：
  - 支持自定义格式说明符 {}、:w 和 :u。
- **parse**：
  - 解析格式说明符：
    - :w：设置 object = "World"。
    - :u：设置 object = "Universe"。
    - 其他：调用非 constexpr 函数 unexpected_format_specification_for_greeter，触发编译错误。
  - object 是成员变量，存储解析结果。
- **format**：
  - 输出 "Hello <object>!"，例如 "Hello World!" 或 "Hello Universe!"。
- **限制**：
  - 非法说明符（如 :Q）因调用非 constexpr 函数而在编译期失败。

------

**4. main 函数测试**

cpp

```cpp
auto str1 = std::format("{}, {}", CustomObject{}, CustomObject{});
// str1 == "CustomObject, CustomObject"
std::println("{}, {}", CustomObject{}, CustomObject{});
```

- **std::format**：
  - 格式化两个 CustomObject，结果为 "CustomObject, CustomObject"。
- **std::println**：
  - 输出相同内容并换行。

cpp

```cpp
auto str2 = std::format("0x{:X}", Wrapper{255});
// str2 == "0xFF"
std::println("0x{:X}", Wrapper{255});
```

- **{:X}**：
  - 十六进制大写格式，255 输出为 "FF"。
- **结果**：
  - "0xFF"，Wrapper 的 value 被格式化为十六进制。

cpp

```cpp
auto str3 = std::format("{:u}", Greeter{});
// str3 == "Hello Universe!"
std::println("{:w}", Greeter{});
std::println("{:u}", Greeter{});
```

- **{:u}**：
  - 设置 object = "Universe"，输出 "Hello Universe!"。
- **{:w}**：
  - 设置 object = "World"，输出 "Hello World!"。

cpp

```cpp
// std::format("{:Q}", Greeter{}); // 不合法，编译错误
```

- **原因**：
  - :Q 未在 parse 中定义，调用非 constexpr 函数导致编译失败。

------

**关键技术点**

1. **std::formatter**：
   - C++20 的格式化接口，必须实现 parse 和 format。
   - parse 是 constexpr，在编译期解析格式字符串。
2. **继承格式化器**：
   - Wrapper 重用 int 的格式化逻辑，简化实现。
3. **自定义说明符**：
   - Greeter 通过 parse 支持 :w 和 :u，并在非法输入时触发编译错误。
4. **<print>**：
   - C++23 的便捷输出，依赖 <format>。

------

**输出总结**

```text
CustomObject, CustomObject
0xFF
Hello World!
Hello Universe!
```

------

**可能的改进或注意事项**

1. **错误处理**：
   - Greeter 的错误处理可抛出 std::format_error 代替非 constexpr 函数。
2. **扩展性**：
   - 为 CustomObject 添加动态内容（如成员变量）。
3. **字符类型**：
   - CustomObject 和 Greeter 可模板化 CharT 支持宽字符。

------

**总结**

- **CustomObject**：简单输出固定字符串。
- **Wrapper**：继承 int 格式化，复用现有逻辑。
- **Greeter**：支持自定义格式说明符，展示灵活性。
- **用途**：扩展 <format> 到用户类型。

如果你有具体问题（例如如何添加新说明符），欢迎提问！