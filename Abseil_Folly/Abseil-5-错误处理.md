#### std::expected

> 👉 **`and_then` 的 lambda 仍然必须返回 `std::expected`，不能返回裸 `int`**

------

## ❌ 当前代码的问题

你现在写的是：

```cpp
.and_then([](int result) {
    if (result > 10) {
        return result * 2;   // ❌ int
    }
    return result;           // ❌ int
})
```

但 `and_then` 期望的是：

```cpp
F: T -> std::expected<U, E>
```

也就是说：

> 👉 返回类型必须是 `std::expected<int, std::string>`

------

## ✅ 正确写法（最小修改版）

```cpp
static std::expected<int, std::string> testMonad(int m, int n)
{
    return testExpected(m, n)
        .and_then([](int result) -> std::expected<int, std::string> {
            if (result > 10) {
                return result * 2;
            }
            return result;
        })
        .and_then([](int result) -> std::expected<int, std::string> {
            if (result < 50) {
                return result + 5;
            }
            return result;
        })
        .or_else([](const std::string& error) -> std::expected<int, std::string> {
            std::cerr << "Error: " << error << std::endl;
            return std::unexpected("Default error");
        });
}
```

------

## 💡 为什么“看起来能省略”但其实不行？

你可能会疑惑：

> C++23 不是支持 `expected` 的隐式构造吗？

确实有这个“看起来能用”的点，但：

- `and_then` 是**模板约束的**
- 它要求返回类型严格匹配 `expected`
- 编译器**不会帮你把 `int` 自动提升为 `expected`（在这个上下文中）**

👉 所以**必须显式返回 `expected`**

------

## 🧠 更优雅的写法（推荐）

如果你只是“变换值”，其实你用错 API 了：

👉 应该用 `.transform()` 而不是 `.and_then()`

### ✔ 正确语义：

- `transform`：`T -> U`
- `and_then`：`T -> expected<U, E>`

------

### ✅ 用 transform 改写（更地道）

```cpp
static std::expected<int, std::string> testMonad(int m, int n)
{
    return testExpected(m, n)
        .transform([](int result) {
            return (result > 10) ? result * 2 : result;
        })
        .transform([](int result) {
            return (result < 50) ? result + 5 : result;
        })
        .or_else([](const std::string& error) -> std::expected<int, std::string> {
            std::cerr << "Error: " << error << std::endl;
            return std::unexpected("Default error");
        });
}
```

------

## 🔥 总结（关键认知）

你现在已经踩到 `expected` 最核心的设计点了：

| 操作         | 用法                        |
| ------------ | --------------------------- |
| 值变换       | `transform`                 |
| 链式可能失败 | `and_then`（返回 expected） |

------

## 🚀 一句话结论

> 👉 你这段逻辑应该用 `transform`，而不是 `and_then`

------

