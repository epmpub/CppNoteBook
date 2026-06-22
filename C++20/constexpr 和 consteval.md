`constexpr` 和 `consteval` 都与编译期计算（compile-time evaluation）有关，但它们的约束强度不同。

最简单的理解：

- `constexpr`：**可以在编译期求值，也可以在运行期执行**
- `consteval`：**必须在编译期求值，绝不能在运行期执行**

例如：

```cpp
constexpr int square(int x)
{
    return x * x;
}
```

调用：

```cpp
constexpr int a = square(5); // 编译期

int n = 5;
int b = square(n);           // 运行期
```

两种都合法。

因此：

```cpp
constexpr
```

表示：

> "这个函数有能力在编译期执行。"

而不是：

> "必须在编译期执行。"

------

## consteval

```cpp
consteval int square(int x)
{
    return x * x;
}
```

调用：

```cpp
constexpr int a = square(5);
```

合法。

但是：

```cpp
int n = 5;
int b = square(n);
```

错误：

```text
call to consteval function is not a constant expression
```

因为：

```cpp
square(n)
```

无法在编译期得到结果。

`consteval` 要求：

```cpp
所有调用必须产生常量表达式
```

因此又称：

```text
Immediate Function
```

（立即函数）

这是 C++20 引入的概念。

------

## 对比示例

### constexpr

```cpp
constexpr int add(int a, int b)
{
    return a + b;
}

int main()
{
    constexpr int x = add(1, 2);

    int a = 1;
    int b = 2;

    int y = add(a, b);

    return y;
}
```

全部合法。

------

### consteval

```cpp
consteval int add(int a, int b)
{
    return a + b;
}

int main()
{
    constexpr int x = add(1, 2);

    int a = 1;
    int b = 2;

    int y = add(a, b); // 错误
}
```

编译失败。

------

## 为什么需要 consteval

假设你正在生成协议 ID：

```cpp
constexpr uint32_t make_id(std::string_view name)
{
    ...
}
```

别人这样写：

```cpp
std::string s;

auto id = make_id(s);
```

结果运行时计算。

而你的设计要求：

```text
协议 ID 必须在编译期生成
```

这时：

```cpp
consteval uint32_t make_id(...)
```

即可强制约束。

------

## 编译器视角

对于：

```cpp
constexpr int f(int x)
{
    return x + 1;
}
```

编译器允许：

```text
Constant Evaluation
       或
Runtime Evaluation
```

二选一。

对于：

```cpp
consteval int f(int x)
{
    return x + 1;
}
```

编译器只允许：

```text
Constant Evaluation
```

否则报错。

------

## constexpr 变量和 consteval 函数

很多人容易混淆：

```cpp
constexpr int x = 42;
```

和

```cpp
consteval int f()
{
    return 42;
}
```

不是同一个概念。

### constexpr 变量

表示：

```cpp
x 本身是编译期常量
```

------

### consteval 函数

表示：

```cpp
f() 的每次调用都必须在编译期执行
```

------

## consteval 能否替代 constexpr？

不能。

例如：

```cpp
std::array<int, square(5)> arr;
```

这里：

```cpp
square
```

使用 `constexpr` 即可。

如果：

```cpp
int x = square(n);
```

你还希望支持运行期调用。

这时必须使用：

```cpp
constexpr
```

而不能使用：

```cpp
consteval
```

------

## 与 if consteval 的关系（C++23）

C++23 引入：

```cpp
if consteval
```

用于检测：

```cpp
当前是否处于编译期求值
```

例如：

```cpp
constexpr int foo(int x)
{
    if consteval
    {
        return x * 2;
    }
    else
    {
        return x * 3;
    }
}
```

调用：

```cpp
constexpr int a = foo(10);
```

结果：

```cpp
20
```

而：

```cpp
int b = foo(10);
```

结果：

```cpp
30
```

这让 `constexpr` 函数能够根据求值上下文选择不同实现。

------

## C++ 标准中的关系

可以认为约束强度如下：

```text
普通函数
    ↓
constexpr
    ↓
consteval
```

| 特性             | 普通函数 | constexpr | consteval |
| ---------------- | -------- | --------- | --------- |
| 运行期调用       | ✔        | ✔         | ✘         |
| 编译期调用       | ✘        | ✔         | ✔         |
| 必须编译期执行   | ✘        | ✘         | ✔         |
| 可作为常量表达式 | ✘        | ✔         | ✔         |
| C++引入          | C++98    | C++11     | C++20     |

从设计角度看：

- `constexpr` = “允许编译期计算”
- `consteval` = “强制编译期计算”

因此在现代 C++（C++20/23/26）中：

- 数学函数、算法工具、容器辅助函数通常用 `constexpr`
- 编译期反射辅助工具、类型生成器、哈希生成器、格式检查器等元编程设施通常用 `consteval`。