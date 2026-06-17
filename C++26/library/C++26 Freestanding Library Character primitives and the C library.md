这是 C++26 Freestanding 扩展的一部分，来源于 **P2338R4 Freestanding Library: Character primitives and the C library**。

一句话概括：

> C++26 允许 Freestanding 实现提供更多与字符处理、数字转换和内存操作相关的标准库组件，使嵌入式开发能够使用这些基础设施，而不必引入完整 Hosted 运行时。

先看问题。

在 C++23 之前，Freestanding 与 Hosted 的差异很大：

```cpp
Hosted:
    string
    charconv
    cstring
    cstdlib
    ...
Freestanding:
    很多都不可用
```

例如：

```cpp
std::from_chars()
std::to_chars()
std::char_traits
std::errc
std::memcpy
std::strlen
```

这些明明不依赖：

```cpp
文件系统
线程
动态内存
异常
locale
```

却经常不保证可用。

对于嵌入式开发很不友好。

------

## 1. Freestanding `std::char_traits`

C++26 新增：

```cpp
#include <string>

std::char_traits<char>
std::char_traits<wchar_t>
std::char_traits<char8_t>
std::char_traits<char16_t>
std::char_traits<char32_t>
```

保证 Freestanding 可用。

例如：

```cpp
#include <string>

constexpr auto n =
    std::char_traits<char>::length("hello");
```

以前：

```cpp
Freestanding
↓
不保证存在
```

现在：

```cpp
Freestanding
↓
标准保证存在
```

------

### 为什么重要？

很多库需要：

```cpp
compare
copy
move
length
find
```

这些字符基础设施。

例如：

```cpp
std::basic_string_view
```

内部大量依赖：

```cpp
std::char_traits
```

------

## 2. Freestanding `std::errc`

新增：

```cpp
#include <system_error>

std::errc
```

可用。

例如：

```cpp
auto result =
    std::from_chars(first,last,value);

if(result.ec == std::errc::invalid_argument)
{
}
```

以前：

```cpp
from_chars
↓
需要 errc
↓
Freestanding 不保证
```

很尴尬。

现在完整支持。

------

## 3. Freestanding `<charconv>`

这是最重要的部分之一。

新增保证：

```cpp
#include <charconv>

std::from_chars
std::to_chars
std::from_chars_result
std::to_chars_result
```

可用。

例如：

```cpp
int x;

auto r =
    std::from_chars(
        "123",
        "123"+3,
        x
    );
```

或者：

```cpp
char buffer[32];

auto r =
    std::to_chars(
        buffer,
        buffer+32,
        123
    );
```

------

### 为什么重要？

以前嵌入式经常只能：

```cpp
atoi
strtol
sprintf
snprintf
```

这些 C API。

而：

```cpp
from_chars
to_chars
```

具有：

- 无分配
- 无异常
- locale-independent
- constexpr 友好

特别适合 Freestanding。

------

## 4. Freestanding `<cstring>`

新增大量内存操作函数：

```cpp
std::memcpy
std::memmove
std::memcmp
std::memset

std::strlen
std::strcmp
std::strncmp

std::strcpy
std::strncpy

std::strchr
std::strrchr
```

等等。

例如：

```cpp
#include <cstring>

char dst[128];

std::memcpy(
    dst,
    src,
    n
);
```

C++26 保证 Freestanding 可用。

------

### 为什么合理？

这些函数本质上只是：

```cpp
内存读写
字节比较
字符扫描
```

完全不需要操作系统。

------

## 5. Freestanding `<cstdlib>`

新增保证：

```cpp
std::abs
std::labs
std::llabs

std::div
std::ldiv
std::lldiv

std::bsearch
std::qsort
```

等基础设施。

例如：

```cpp
int x = std::abs(-42);
```

或者：

```cpp
std::qsort(
    arr,
    count,
    sizeof(int),
    cmp
);
```

------

### 注意

不是整个 `<cstdlib>`。

例如：

```cpp
std::system
std::getenv
std::atexit
```

仍然依赖 Hosted 环境。

不属于 Freestanding 保证范围。

------

## 为什么是这几个头文件？

WG21 的原则是：

如果一个组件：

```text
不依赖
    OS
    文件系统
    locale
    线程
    堆内存
```

那么它应该可以在 Freestanding 中提供。

例如：

```cpp
memcpy
strlen
char_traits
from_chars
to_chars
errc
```

都是纯算法。

因此被纳入。

而：

```cpp
fstream
locale
filesystem
thread
```

仍然属于 Hosted 世界。

------

## 对嵌入式开发的意义

以前很多裸机项目：

```cpp
STM32
ESP32
ARM Cortex-M
```

不得不写：

```cpp
自己的itoa
自己的atoi
自己的strlen
自己的memcpy
```

或者直接调用 C 库。

现在标准允许：

```cpp
std::to_chars
std::from_chars
std::char_traits
std::memcpy
std::strlen
std::errc
```

成为 Freestanding 的正式组成部分。

例如：

```cpp
char buf[16];

auto r =
    std::to_chars(
        buf,
        buf + sizeof(buf),
        sensor_value
    );
```

完全符合：

```text
无堆
无异常
无locale
Freestanding
```

的设计目标。

因此这项 C++26 改动的核心价值是：**把字符处理、数字转换和内存操作这些“纯基础设施”正式纳入 Freestanding 标准库，使嵌入式和系统级 C++ 能够使用现代标准库能力，而不需要完整 Hosted 运行时。**