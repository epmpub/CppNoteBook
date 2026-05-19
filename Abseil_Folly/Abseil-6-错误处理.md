

####  C++ 错误处理进化路线

## 1️⃣ Return Code（返回码时代）

```cpp
int divide(int a, int b, int& out) {
    if (b == 0) return -1;
    out = a / b;
    return 0;
}
```

### ✔ 特点

- 用返回值表示错误（0 成功，非 0 失败）
- 真正结果通过参数传出

### 👍 优点

- 简单、无额外语言机制
- 性能可控（无异常开销）

### 👎 缺点

- **可读性差**（成功路径被错误处理打断）
- **容易忘记检查返回值**
- 错误信息表达能力弱（只能用 error code）

### 📌 使用场景

- C 语言 / 内核 / 嵌入式（例如 Linux kernel）

------

## 2️⃣ Exception（异常机制）

```cpp
int divide(int a, int b) {
    if (b == 0) throw std::runtime_error("divide by zero");
    return a / b;
}
try {
    int r = divide(10, 0);
} catch (const std::exception& e) {
    std::cout << e.what();
}
```

### ✔ 特点

- **错误和正常逻辑分离**
- 用 `throw` / `try-catch` 处理异常

### 👍 优点

- 代码更干净（没有 if 检查污染主逻辑）
- 错误可以自动向上传播（stack unwinding）
- 表达能力强（异常类型 + message）

### 👎 缺点

- **性能不可预测**（尤其在高频路径）
- 控制流“隐式跳转”（不容易读懂执行路径）
- 在某些项目（游戏、低延迟系统）被禁用

### 📌 使用场景

- 应用层逻辑（GUI / 业务代码）
- 错误是“异常情况”（不是常态）

------

## 3️⃣ std::expected（现代 C++ / 函数式风格）

```cpp
std::expected<int, std::string> divide(int a, int b) {
    if (b == 0)
        return std::unexpected("divide by zero");
    return a / b;
}
auto result = divide(10, 0)
    .transform([](int x) { return x * 2; })
    .or_else([](const std::string& e) {
        std::cout << e;
        return std::unexpected(e);
    });
```

------

### ✔ 特点

- **显式返回：要么值，要么错误**
- 没有异常跳转（控制流可见）
- 支持函数式链式调用（monadic）

------

### 👍 优点

#### 1. 类型安全（最重要）

```cpp
std::expected<T, E>
```

👉 编译期强制你处理错误

------

#### 2. 无隐藏控制流

不像 exception：

```cpp
foo(); // ❗ 可能突然 throw
```

expected 是显式的：

```cpp
auto r = foo();
if (!r) ...
```

------

#### 3. 可组合（函数式）

```cpp
f().and_then(g).and_then(h)
```

👉 非常适合你现在写的这种 pipeline

------

#### 4. 性能稳定

- 没有异常栈展开
- 和返回值一样可预测

------

### 👎 缺点

- 比 exception **更啰嗦**
- 需要 discipline（团队统一风格）
- 生态还在成长（相比异常）

------

## 🆚 三者对比总结

| 特性       | Return Code | Exception | std::expected |
| ---------- | ----------- | --------- | ------------- |
| 可读性     | ❌ 差        | ✅ 好      | ✅ 好          |
| 性能       | ✅ 稳定      | ⚠ 不稳定  | ✅ 稳定        |
| 控制流     | ✅ 显式      | ❌ 隐式    | ✅ 显式        |
| 错误表达   | ❌ 弱        | ✅ 强      | ✅ 强          |
| 编译期约束 | ❌ 无        | ❌ 无      | ✅ 强          |
| 函数组合   | ❌ 差        | ❌ 差      | ✅ 强          |

------

# 🧠 核心哲学变化

### 🧱 C 风格（return code）

> “错误是一个数字”

------

### ⚡ C++ 异常

> “错误是一个事件（exception）”

------

### 🧬 modern C++（expected）

> “错误是一个值（value）”

------

# 🚀 什么时候用哪个？

## ✔ 用 return code

- 内核 / 驱动
- 极端性能控制
- C 接口

------

## ✔ 用 exception

- 应用层逻辑
- 错误是“罕见异常”
- 不想污染主流程

------

## ✔ 用 std::expected（推荐现代风格）

- 系统编程（但又想要安全）
- 高性能 + 可控错误处理
- 需要函数式组合（你现在这种写法）

------

# 🔥 一句话总结

> 👉 `expected` = **没有异常的 exception + 没有混乱的 return code**

------

如果你下一步想更深入，可以继续问我这个（很关键）：

👉 **为什么 C++ 把 `expected::map` 命名成 `transform`？**
👉 **`and_then` 本质上是不是 flatMap？**

这两个点一旦打通，你就真正理解“monad 在 C++ 里的落地”了。