在 C++ 中虽然没有原生的 Monad 类型系统，但可以通过标准库组件或自定义结构模拟 Monad 的核心思想。以下是几个实际场景中类似 Monad 的用法，它们体现了 Monad 处理副作用、实现链式操作的核心特性：

### 1. `std::optional`（Maybe Monad 类似物）

`std::optional` 用于表示 “可能存在或不存在的值”，避免空指针问题，类似函数式编程中的 **Maybe Monad**。它封装了 “值存在”（`has_value()` 为 `true`）或 “值不存在”（`has_value()` 为 `false`）的上下文，并支持链式操作。

**示例：安全的链式计算**

```cpp
#include <optional>
#include <iostream>

// 安全的整数除法（避免除零）
std::optional<int> safe_divide(int a, int b) {
    if (b == 0) return std::nullopt; // 无值（类似 Nothing）
    return a / b; // 有值（类似 Just a/b）
}

int main() {
    // 链式操作：先除以 2，再除以 3
    auto result = safe_divide(12, 2)
        .and_then([](int x) { return safe_divide(x, 3); }); 
        // and_then 类似 Monad 的 >>= 操作符

    if (result) {
        std::cout << "结果: " << *result; // 输出：结果: 2
    } else {
        std::cout << "计算失败";
    }
    return 0;
}
```

- **关键特性**：`and_then` 确保当中间步骤返回 `nullopt` 时，后续操作会直接短路（不再执行），避免无效计算。

### 2. `std::expected`（C++23，Either Monad 类似物）

`std::expected`（C++23 引入）用于表示 “成功结果” 或 “错误信息”，类似 **Either Monad**。它封装了操作的两种可能状态（`value` 或 `error`），并支持链式处理。

**示例：带错误信息的链式操作**

```cpp
#include <expected>
#include <string>
#include <iostream>

// 解析字符串为整数（失败时返回错误信息）
std::expected<int, std::string> parse_int(const std::string& s) {
    try {
        return std::stoi(s); // 成功：返回值
    } catch (...) {
        return std::unexpected("解析失败"); // 失败：返回错误
    }
}

// 安全加法（避免溢出，简化示例）
std::expected<int, std::string> safe_add(int a, int b) {
    if (a > 0 && b > INT_MAX - a) {
        return std::unexpected("溢出");
    }
    return a + b;
}

int main() {
    // 链式操作：解析字符串 -> 加 10 -> 输出结果
    auto process = parse_int("123")
        .and_then([](int x) { return safe_add(x, 10); });

    if (process.has_value()) {
        std::cout << "结果: " << process.value(); // 输出：结果: 133
    } else {
        std::cout << "错误: " << process.error();
    }
    return 0;
}
```

- **关键特性**：`and_then` 自动传递成功值，若中间步骤出错则直接返回错误，避免嵌套的错误判断。

### 3. 范围视图（Range Views，类似 List Monad）

C++20 引入的范围视图（如 `std::views::transform`、`std::views::filter`）支持链式操作，对序列元素进行转换或过滤，类似 **List Monad** 的 “容器 + 操作链” 思想。

**示例：链式处理序列**

```cpp
#include <vector>
#include <ranges>
#include <iostream>

int main() {
    std::vector<int> nums = {1, 2, 3, 4, 5};

    // 链式操作：过滤偶数 -> 乘以 10 -> 输出
    auto result = nums | std::views::filter([](int x) { return x % 2 == 0; })
                       | std::views::transform([](int x) { return x * 10; });

    for (int x : result) {
        std::cout << x << " "; // 输出：20 40
    }
    return 0;
}
```

- **关键特性**：每个操作返回新的视图（不实际执行计算），最终迭代时才按链执行，实现 “惰性计算” 和无副作用的序列处理。

### 4. 自定义状态 Monad（模拟状态传递）

在需要维护状态的场景（如计数器、配置），可自定义 Monad 结构封装状态，确保状态在链式操作中安全传递，避免全局变量。

**示例：带计数状态的操作链**

cpp



运行









```cpp
#include <iostream>
#include <functional>

// 封装值和状态（计数）的 Monad 结构
template <typename T>
struct StateMonad {
    using State = int; // 状态类型：计数器
    std::function<std::pair<T, State>(State)> run; // 状态转换函数

    // 链式操作：将当前值传递给下一个函数
    template <typename U>
    StateMonad<U> and_then(std::function<StateMonad<U>(T)> f) const {
        return StateMonad<U>{
            [this, f](State s) {
                auto [val, new_s] = run(s); // 执行当前操作，获取值和新状态
                return f(val).run(new_s);   // 传递新状态给下一个操作
            }
        };
    }
};

// 创建初始 Monad（包装值并初始化状态）
template <typename T>
StateMonad<T> return_(T val) {
    return StateMonad<T>{
        [val](StateMonad<T>::State s) {
            return std::make_pair(val, s); // 不修改状态
        }
    };
}

int main() {
    // 示例操作：给值加 10，并将计数器加 1
    auto add10 = [](int x) {
        return StateMonad<int>{
            [x](int count) {
                return std::make_pair(x + 10, count + 1); // 状态+1
            }
        };
    };

    // 链式操作：初始值 5 -> 加 10 -> 再加 10
    auto chain = return_(5).and_then(add10).and_then(add10);
    
    // 运行链（初始状态为 0）
    auto [result, count] = chain.run(0);
    
    std::cout << "结果: " << result << ", 操作次数: " << count; 
    // 输出：结果: 25, 操作次数: 2
    return 0;
}
```

- **关键特性**：状态通过 `and_then` 隐式传递，避免手动管理状态变量，确保操作的可组合性。

### 总结

C++ 中的这些结构虽不严格符合数学上 Monad 的定义（缺乏统一的类型约束），但核心思想一致：

1. 封装上下文（值存在 / 不存在、成功 / 错误、序列、状态等）；
2. 提供链式操作接口（`and_then`、`|` 等），自动处理上下文传递；
3. 隔离副作用（如错误、状态变化），使代码更可读、可维护。

在实际开发中，`std::optional`、`std::expected` 和范围视图是最常用的 “类 Monad” 工具，能有效简化错误处理和数据处理流程。