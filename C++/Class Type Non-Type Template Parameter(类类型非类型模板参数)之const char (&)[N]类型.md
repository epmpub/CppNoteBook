## const char (&)[N]类型

这里：

```cpp
template<size_t N>
struct FixedString {
    constexpr FixedString(const char (&str)[N]) {
        // ...
    }
};
```

参数 `str` 的类型是：

```cpp
const char (&)[N]
```

即：

**“对 `const char[N]` 数组的左值引用（lvalue reference）”**

完整拆解：

```cpp
const char      // 数组元素类型
[N]             // 长度为 N 的数组
&               // 引用
```

所以：

```cpp
const char (&str)[N]
```

等价于：

```cpp
using ArrayRef = const char (&)[N];
ArrayRef str;
```

例如：

```cpp
char s1[] = "hello";
```

`"hello"` 的类型是：

```cpp
const char[6]
```

因为包含结尾的 `'\0'`：

```text
'h' 'e' 'l' 'l' 'o' '\0'
```

长度为 6。

因此：

```cpp
FixedString fs("hello");
```

编译器推导：

```cpp
N = 6
```

构造函数实际变成：

```cpp
constexpr FixedString(const char (&str)[6])
```

------

为什么不用：

```cpp
const char* str
```

因为数组传给指针会发生退化（decay）：

```cpp
void foo(const char* s);
```

调用：

```cpp
foo("hello");
```

此时：

```cpp
const char[6]
```

变成：

```cpp
const char*
```

长度信息丢失了。

而：

```cpp
template<size_t N>
void foo(const char (&s)[N]);
```

调用：

```cpp
foo("hello");
```

编译器可以推导：

```cpp
N = 6
```

长度被保留下来。

这就是 STL 中很多地方使用数组引用的原因。

例如：

```cpp
template<size_t N>
constexpr size_t length(const char (&)[N])
{
    return N - 1;
}

int main()
{
    std::cout << length("hello"); // 5
}
```

编译器自动推导：

```cpp
"hello" -> const char[6]
N = 6
```

输出：

```text
5
```

------

可以用 `decltype` 验证：

```cpp
#include <iostream>
#include <type_traits>

template<size_t N>
void test(const char (&str)[N])
{
    std::cout << std::boolalpha
              << std::is_same_v<
                    decltype(str),
                    const char (&)[N]
                 >
              << '\n';
}

int main()
{
    test("hello");
}
```

输出：

```text
true
```

所以严格地说：

```cpp
str
```

的类型不是：

```cpp
const char*
```

也不是：

```cpp
const char[N]
```

而是：

```cpp
const char (&)[N]
```

即“引用一个长度为 N 的字符数组”。这也是 `FixedString` 能够推导出字符串长度 `N` 的关键。