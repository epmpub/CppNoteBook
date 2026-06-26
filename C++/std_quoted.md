`std::quoted` 是 C++14 在 `<iomanip>` 中引入的一个 I/O 操纵器（I/O manipulator），用于：

> **在输入/输出字符串时自动添加、解析引号，并正确处理转义字符。**

你的代码：

```cpp
std::quoted(s)
```

出现在：

```cpp
std::cout << "Res::Res(" << std::quoted(s) << ");\n";
```

假设：

```cpp
s = "Hello, world!"
```

输出为：

```text
Res::Res("Hello, world!");
```

注意字符串两边自动加了双引号。

如果不用：

```cpp
std::cout << s;
```

输出：

```text
Hello, world!
```

------

## 最简单例子

```cpp
#include <iomanip>
#include <iostream>
#include <string>

int main()
{
    std::string s = "abc";

    std::cout << s << '\n';
    std::cout << std::quoted(s) << '\n';
}
```

输出：

```text
abc
"abc"
```

------

## 为什么有用

考虑字符串中本身包含引号：

```cpp
std::string s = R"(He said "Hello")";
```

普通输出：

```cpp
std::cout << s;
```

结果：

```text
He said "Hello"
```

------

使用：

```cpp
std::cout << std::quoted(s);
```

结果：

```text
"He said \"Hello\""
```

注意：

```text
内部引号被转义
```

因此输出可以安全地重新解析。

------

## 输入时更有价值

例如：

```cpp
std::string s;

std::cin >> std::quoted(s);
```

输入：

```text
"Hello world"
```

结果：

```cpp
s == "Hello world"
```

注意：

```text
空格被保留
外层引号被去掉
```

如果不用：

```cpp
std::cin >> s;
```

输入：

```text
Hello world
```

得到：

```cpp
s == "Hello"
```

因为空格会终止读取。

------

## 你的代码中的作用

构造函数：

```cpp
Res(std::string arg)
    : s{std::move(arg)}
{
    std::cout
        << "Res::Res("
        << std::quoted(s)
        << ");\n";
}
```

如果：

```cpp
Res{"Hello, world!"}
```

输出：

```text
Res::Res("Hello, world!");
```

而不是：

```text
Res::Res(Hello, world!);
```

这样更像调试日志中的构造表达式。

------

## operator<< 中的作用

这里：

```cpp
friend std::ostream&
operator<<(std::ostream& os,
           Res const& r)
{
    return os
        << "Res { s = "
        << std::quoted(r.s)
        << "; }";
}
```

假设：

```cpp
r.s = "Hello"
```

输出：

```text
Res { s = "Hello"; }
```

而不是：

```text
Res { s = Hello; }
```

更容易看出：

```text
这是字符串
不是变量名
不是关键字
```

------

## 默认行为

`std::quoted` 默认使用：

```cpp
'"'   // 引号字符
'\\'  // 转义字符
```

即：

```cpp
std::quoted(str)
```

等价于：

```cpp
std::quoted(str, '"', '\\')
```

------

## 自定义引号

例如：

```cpp
std::cout << std::quoted(s, '\'', '\\');
```

输出：

```text
'Hello'
```

使用单引号。