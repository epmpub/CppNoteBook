**Class Type Non-Type Template Parameter**（类类型非类型模板参数）

在 C++20 之前，字符串字面量不能作为模板参数使用。即使在 C++20 中支持扩展的非类型模板参数（NTTP），也不能直接写成：

```cpp
MyPrint<"hello world">(); // 错误
```

需要先定义一个可作为 NTTP 的结构体来包装字符串。

C++20 的正确写法：

```cpp
#include <iostream>

template<size_t N>
struct FixedString {
    char value[N];

    constexpr FixedString(const char (&str)[N]) {
        for (size_t i = 0; i < N; ++i) {
            value[i] = str[i];
        }
    }
};

template<FixedString Str>
void MyPrint() {
    std::cout << Str.value << std::endl;
}

int main() {
    MyPrint<"hello world">();
}
```

输出：

```text
hello world
```

原理：

```cpp
MyPrint<"hello world">();
```

实际上编译器会推导为：

```cpp
MyPrint<FixedString{"hello world"}>();
```

其中：

```cpp
template<FixedString Str>
```

是 C++20 新增的 **Class Type Non-Type Template Parameter**（类类型非类型模板参数）。

如果是 C++17 或更早版本，则必须写成：

```cpp
constexpr char str[] = "hello world";

template<const char* S>
void MyPrint() {
    std::cout << S << '\n';
}

int main() {
    MyPrint<str>();
}
```

但这种方式只能接受具有外部链接（external linkage）的字符数组，不能直接写字符串字面量。

因此：

- C++17：`MyPrint<str>()`
- C++20：`MyPrint<"hello world">()`（通过 `FixedString` 包装）
- C++23/C++26：仍然沿用 C++20 的 `FixedString` 技术，没有进一步简化到直接接受字符串字面量作为模板参数。