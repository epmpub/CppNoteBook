C++ 23 对println的增强;

range支持



```c
// #include <tuple>
// #include <iostream>

// void func(int x, double y, char z) {
//     std::cout << x << ", " << y << ", " << z << '\n';
// }

// int main() {
//     std::tuple<int, double, char> t {42, 3.14, 'A'};
//     std::apply(func, t); // 输出 42, 3.14, A
//     // 使用 lambda
//     std::apply([](int x, double y, char z) {
//         std::cout << x + y << ", " << z << '\n';
//     }, t); // 输出 45.14, A
//     return 0;
// }

// #include <print>

// struct Callable
// {
//     static auto operator()(int val){return val + 42;}
// };

// int use_func(auto f){
//     return f(10);
// }

// int main(){
//     std::print("{}\n\n",Callable::operator()(45));
//     std::print("{}\n\n",Callable()(45)); 

//     auto l = [](int v)  {
//      return v + 42; 
//     };

//     return use_func(l);


// }

// #include <cstdio>
// #include <filesystem>
// #include <print>
 
// int main()
// {
//     std::print("{2} {1}{0}!\n", 23, "C++", "Hello");  // overload (1)
 
//     const auto tmp{std::filesystem::temp_directory_path() / "test.txt"};
//     if (std::FILE* stream{std::fopen(tmp.c_str(), "w")})
//     {
//         std::print(stream, "File save path is:{}\n", tmp.string()); // overload (2)
//         std::fclose(stream);
//     }
// }

// #include <print>

// int main()
// {
//     std::println("{:n}",std::tuple{1,2,3,"Hello World"});
//     std::println("{:<20} {:<20}","ID","value");
//     std::println("{:<20} {:<20}",3.14,6.8);
//     std::println("{:<20} {:<20} ",3.14926,3.65);
//     std::println("{:<20} {:<20} ",3.149261314,8.95);
//     return 0;
// }


#include <print>
#include <utility>
#include <vector>
#include <ranges>
#include <array>
#include <map>
#include <string>
#include <chrono>

int main()
{
    std::println("{}", std::pair{42, 4.2});
    std::println("{:n}", std::tuple{1, 2, 3, "Hello World"});
    std::println("{:n}", std::vector{4, 5, 6, 7, 8});
    std::println("{}", std::array{9, 10, 11, 12} | std::views::drop(2));

    std::vector<std::pair<char, int>> v2{{'1', 1}, {'2', 2}};
    std::println("{::m}", v2);

    std::map<
        std::string,
        std::pair<
            int,
            std::map<
                std::chrono::time_point<std::chrono::system_clock>,
                std::vector<int>
            >
        >
    > data;

    data["Hello"] = {
        1,
        {
            {std::chrono::system_clock::now(), {1, 2, 3, 4}}
        }
    };

    std::println("{}",data);
}
```

这些语法（`{:n}`、`{::m}`）不是“随便加的 print 扩展”，而是建立在 **C++20 `std::format` + ranges/tuple formatter 扩展提案体系**上的一套“格式化语法组合机制”。

本质上它们来自三条标准化路线的叠加：

------

# 1. 核心基础：std::format（P0645 / C++20）

所有这些语法的根基是：

> `std::format` / `std::println` 使用统一的 format string grammar

格式基本形态：

```
{ [arg-id] [:format-spec] }
```

例如：

```cpp
"{:d}"   // decimal
"{:x}"   // hex
```

👉 这部分来自 C++20 `std::format`（P0645 + P2216 修订）

------

# 2. {:n} 的来源：Nested / Range formatting（P2286 + P2093体系）

## ✔ {:n} 并不是基础 std::format 的一部分

它来自一个“二级扩展体系”：

### 关键提案：

- P2093R14 — Format for Ranges
- P2286R8 — Formatting Ranges, Tuples, Optional, Variant
- P3107 — Printing with std::print / std::println

------

## ✔ {:n} 的语义

```cpp
{:n}
```

表示：

> “递归格式化 nested range / tuple / structured types”

### 它的效果：

| 类型                | 行为              |
| ------------------- | ----------------- |
| vector              | 展开元素          |
| tuple               | 展开成员          |
| pair                | 展开 first/second |
| nested vector/tuple | 递归展开          |

------

## ✔ 为什么叫 n？

官方语义：

```
n = nested
```

即：

> recursive formatting mode

------

## ✔ 举例

```cpp
std::println("{:n}", std::tuple{1,2,std::pair{3,4}});
```

输出类似：

```
(1, 2, (3, 4))
```

------

# 3. {::m} 的来源：range formatter + nested format spec propagation

这个是更“隐晦”的扩展语法。

------

## ✔ 结构拆解

```
{ ::m }
```

可以拆成：

| 部分  | 含义                                            |
| ----- | ----------------------------------------------- |
| `:`   | format spec start                               |
| `:m`  | inner format spec for elements                  |
| `::m` | apply inner spec recursively to nested elements |

------

## ✔ 它来源于哪个标准？

它来自组合机制：

### 关键标准点：

### (1) P2286 — tuple/range formatting

允许：

```
format-spec := nested-spec | element-spec
```

------

### (2) formatter propagation rule

当对象是 range 时：

> outer formatter → inner formatter applied to elements

------

## ✔ m 是什么？

`m` 并不是 C++ 核心保留字，而是：

> **user-defined / library-defined format spec for map-like printing**

在 libstdc++ / MSVC STL / fmtlib 中：

| spec | meaning                                 |
| ---- | --------------------------------------- |
| `m`  | map-like formatting (key/value aligned) |

------

## ✔ 所以 {::m} 实际意思是：

```text
outer {}:
    no format for container itself

first : :
    enable nested formatting

second m:
    format elements as "map style"
```

------

## ✔ 对 v2 的作用

```cpp
std::vector<std::pair<char,int>> v2{{'1',1},{'2',2}};
std::println("{::m}", v2);
```

等价于：

> “把 vector 当 range 递归展开，并且 pair 按 map-style 输出”

可能输出类似：

```
{1:1, 2:2}
```

（具体取决于 STL 实现）

------

# 4. 三层体系总结（最重要）

这些语法其实分三层：

------

## Layer 1：C++20 std::format grammar（核心）

```
{ arg [: format-spec] }
```

来源：

- P0645
- P2216

------

## Layer 2：range/tuple/variant formatting（结构扩展）

```
{:n}
```

来源：

- P2093R14
- P2286R8

作用：

- enable structured recursive formatting

------

## Layer 3：container-specific format specifiers

```
m, n, r, ...
```

来源：

- STL implementation / fmtlib extension layer
- 标准允许“formatter customization”

------

# 5. 一个关键误区（很重要）

这些东西：

```
:n
::m
```

❌ 不是 core language syntax
❌ 不是 compiler feature
❌ 不是 operator overload

✔ 是 **format string mini-language（运行时解析DSL）**

------

# 6. 本质模型（最关键理解）

`std::print` 实际做的是：

```
format string
   ↓
parse mini-DSL
   ↓
formatter<T>::format(...)
   ↓
output iterator / FILE*
```

------

# 7. 一句话总结

- `{:n}`：来自 **tuple/range recursive formatting（P2286）**
- `{::m}`：来自 **range formatter + map-style formatter customization（fmtlib/STL扩展层）**
- 本质：**std::format 的 DSL + STL formatter traits 系统组合结果**

------

如果你想进一步深入，我可以把这一块拆到更底层：

> `formatter<T>` specialization 是如何让 vector / tuple / pair “自动可打印”的，以及编译期 concept 是怎么约束 format string correctness 的（这是 C++20 format system 最核心的类型系统设计）。
