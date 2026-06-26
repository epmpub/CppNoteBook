constexpr
consteval
constinit

`constexpr` 和 `consteval` 都是 C++ 编译期计算相关的关键字，但它们的约束力度完全不同。

可以简单理解为：

| 关键字      | 含义                                     |
| ----------- | ---------------------------------------- |
| `constexpr` | **可以在编译期执行，也可以在运行期执行** |
| `consteval` | **必须在编译期执行**                     |

------

## 1. constexpr：尽量编译期

### constexpr函数

```cpp
constexpr int square(int x)
{
    return x * x;
}
```

调用时：

```cpp
constexpr int a = square(5);  // 编译期计算

int n = 5;
int b = square(n);            // 运行期计算
```

这里同一个函数：

- 参数是常量表达式 → 编译期执行
- 参数是运行时变量 → 运行期执行

因此：

```cpp
constexpr
```

表示：

> 这个函数有能力在编译期求值。

但编译器不保证一定在编译期执行。

------

## 2. consteval：强制编译期

### consteval函数

```cpp
consteval int square(int x)
{
    return x * x;
}
```

合法：

```cpp
constexpr int a = square(5);
```

非法：

```cpp
int n = 5;
int b = square(n);
```

编译错误：

```text
error: call to consteval function is not a constant expression
```

因为：

```cpp
consteval
```

表示：

> 这个函数只能在编译期调用。

------

## 3. 对比示例

### constexpr

```cpp
constexpr int add(int a, int b)
{
    return a + b;
}

int main()
{
    int x = 1;

    auto r1 = add(1, 2); // 编译期
    auto r2 = add(x, 2); // 运行期
}
```

编译通过。

------

### consteval

```cpp
consteval int add(int a, int b)
{
    return a + b;
}

int main()
{
    int x = 1;

    auto r1 = add(1, 2); // OK
    auto r2 = add(x, 2); // ERROR
}
```

编译失败。

------

## 4. 为什么需要 consteval

有些函数你根本不希望运行时执行。

例如：

```cpp
consteval auto MakeArraySize(int n)
{
    return n * 1024;
}
```

使用：

```cpp
std::array<int, MakeArraySize(10)> arr;
```

如果有人写：

```cpp
int n = 10;

std::array<int, MakeArraySize(n)> arr;
```

直接编译失败。

这样可以保证：

- 模板参数
- 元编程计算
- 编译期配置生成

全部在编译阶段完成。

------

## 5. 与 constinit 的区别

C++20同时引入了：

```cpp
constexpr
consteval
constinit
```

很多人容易混淆。

### constexpr

对象必须是常量表达式

```cpp
constexpr int x = 10;
```

------

### constinit

对象必须静态初始化，但不一定是常量

```cpp
constinit int x = 10;
```

之后可以修改：

```cpp
x = 20;
```

合法。

------

### consteval

作用于函数：

```cpp
consteval int f()
{
    return 42;
}
```

强制编译期执行。

------

## 6. 典型使用场景

### constexpr

适合：

```cpp
constexpr std::string_view version = "1.0";
constexpr int fib(int n);
```

既希望编译期计算，也允许运行期计算。

------

### consteval

适合：

- 编译期反射辅助工具
- 编译期字符串处理
- 编译期哈希
- 模板元编程
- DSL生成器

例如：

```cpp
consteval unsigned hash(char const* s)
{
    unsigned h = 0;

    while (*s)
        h = h * 131 + *s++;

    return h;
}
constexpr auto id = hash("Player");
```

保证哈希值一定在编译期生成。

------

## 7. C++23中的进一步增强

C++23允许在 `constexpr` 函数中使用更多语言特性：

- `try/catch`
- 动态内存分配 (`new/delete`)
- 更多标准库容器支持

因此：

```cpp
constexpr
```

越来越接近：

```cpp
"普通函数 + 可编译期执行"
```

而：

```cpp
consteval
```

始终表示：

```cpp
"只能编译期执行"
```

核心区别可以浓缩成一句话：

```cpp
constexpr -> 可以编译期，也可以运行期
consteval -> 必须编译期，禁止运行期
```

从设计角度看，`consteval` 相当于给 `constexpr` 再加上一层强制约束：

```text
consteval ⊂ constexpr
```

即每个 `consteval` 函数本质上都具有 `constexpr` 能力，但反过来并不成立。