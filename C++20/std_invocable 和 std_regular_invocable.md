# std::invocable 和 std::regular_invocable

std::invocable 和 std::regular_invocable 是 C++20 引入的两个标准库概念（concepts），位于 <concepts> 头文件中。它们用于约束一个可调用对象（callable，如函数、函数对象、lambda 等）是否可以以特定参数类型调用，并对调用行为施加不同程度的语义要求。以下是对两者的详细解释和对比。

------

1. std::invocable

定义

std::invocable<F, Args...> 是一个概念，约束一个可调用对象 F 是否可以以参数类型 Args... 进行调用。它只关心调用是否合法（即语法上有效），而不对调用的语义或返回值做额外要求。

要求

一个类型 F 满足 std::invocable<F, Args...>，如果：

- F 是一个可调用对象（如函数指针、函数对象、lambda、成员函数指针等）。
- 表达式 std::invoke(f, args...) 是合法的，其中 f 是 F 类型的值，args... 是 Args... 类型的值。
- std::invoke 是 C++17 引入的工具，用于统一调用各种可调用对象（包括普通函数、成员函数、函数对象等）。

语义

- **只关心调用合法性**：std::invocable 不要求调用有特定返回值，也不要求调用是“纯函数”或具有特定副作用。
- **灵活性**：适用于任何可调用的场景，包括可能有副作用或不确定的行为。

示例

cpp

```cpp
#include <concepts>
#include <functional>

void func(int) {}

struct Functor {
    void operator()(int) const {}
};

int main() {
    auto lambda = [](int) {};

    static_assert(std::invocable<decltype(func), int>);    // 函数指针
    static_assert(std::invocable<decltype(lambda), int>);  // lambda
    static_assert(std::invocable<Functor, int>);           // 函数对象
    static_assert(std::invocable<decltype(&Functor::operator()), Functor, int>); // 成员函数
}
```

使用场景

- 用于泛型编程中，确保一个可调用对象可以接受特定参数。
- 适用于需要调用但不关心调用结果或副作用的场景，如事件处理、回调函数等。

------

2. std::regular_invocable

定义

std::regular_invocable<F, Args...> 是一个更严格的概念，是 std::invocable 的子概念。它不仅要求调用合法，还要求调用行为满足“正则”（regular）语义，即调用是纯函数式的，具有一致性和可预测性。

要求

一个类型 F 满足 std::regular_invocable<F, Args...>，如果：

1. 满足 std::invocable<F, Args...>（即调用语法合法）。
2. 调用行为是“正则的”，具体包括：
   - **确定性**：相同的输入（f 和 args...）总是产生相同的结果（包括返回值和副作用）。
   - **无意外副作用**：调用不会修改不相关的状态（例如，不修改全局变量，除非这是调用者的明确意图）。
   - **语义等价性**：如果 f 是值语义的（value-semantic），多次调用等价的 f 实例应产生相同结果。

语义

- **纯函数倾向**：std::regular_invocable 倾向于描述纯函数（pure functions）或行为类似纯函数的可调用对象。
- **更严格的约束**：它要求调用是可预测的、稳定的，适合在需要强一致性的算法中使用。

示例

cpp

```cpp
#include <concepts>

int pure_func(int x) { return x + 1; } // 纯函数，满足 regular_invocable

struct NonRegular {
    int state = 0;
    int operator()(int x) { return state++ + x; } // 有副作用，不满足 regular_invocable
};

int main() {
    auto pure_lambda = [](int x) { return x + 1; }; // 纯函数
    auto non_pure_lambda = [state = 0](int x) mutable { return state++ + x; }; // 有副作用

    static_assert(std::regular_invocable<decltype(pure_func), int>);
    static_assert(std::regular_invocable<decltype(pure_lambda), int>);
    static_assert(std::invocable<decltype(non_pure_lambda), int>);
    static_assert(!std::regular_invocable<decltype(non_pure_lambda), int>); // 有副作用
    static_assert(!std::regular_invocable<NonRegular, int>); // 有副作用
}
```

使用场景

- 用于需要确定性和无副作用的场景，如 Ranges 算法（std::ranges::transform, std::ranges::sort 等），这些算法假设可调用对象的行为是可预测的。
- 适合在数学或函数式编程风格的代码中，确保调用行为符合严格的语义要求。

------

关键区别

| 特性     | std::invocable                        | std::regular_invocable                      |
| -------- | ------------------------------------- | ------------------------------------------- |
| 定义     | 约束调用语法合法                      | 约束调用语法合法且行为正则（纯函数式）      |
| 要求     | std::invoke(f, args...) 合法          | 满足 invocable 且调用是确定性、无意外副作用 |
| 副作用   | 允许任意副作用                        | 不允许意外副作用（倾向于纯函数）            |
| 确定性   | 不要求相同输入产生相同结果            | 要求相同输入产生相同结果                    |
| 严格性   | 较宽松                                | 更严格（是 invocable 的子概念）             |
| 典型用途 | 通用回调、事件处理                    | Ranges 算法、函数式编程                     |
| 示例     | 任何可调用对象（如有副作用的 lambda） | 纯函数、std::plus<>、无状态 lambda          |

------

你的代码上下文

虽然你的原始代码中没有直接使用 std::invocable 或 std::regular_invocable，但这些概念可能与 Ranges 库中的算法或函数模板相关。例如，你的 add 函数使用了 std::ranges::input_range 来约束输入范围。如果你在处理 Ranges 算法（如 std::ranges::for_each 或 std::ranges::transform），可能会需要约束一个可调用对象：

cpp

```cpp
template<typename Coll, std::ranges::input_range T, std::regular_invocable<typename Coll::value_type> F>
void process(Coll& coll, const T& val, F func) {
    for (auto&& v : val) {
        coll.push_back(func(v)); // func 必须是纯函数
    }
}
```

在这里，std::regular_invocable 确保 func 是确定性的，适合 Ranges 算法的语义。而如果只需要调用合法性（允许副作用），可以用 std::invocable。

------

示例：区分 invocable 和 regular_invocable

cpp

```cpp
#include <concepts>
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> vec = {1, 2, 3};
    
    // 纯函数：满足 regular_invocable
    auto pure = [](int x) { return x * 2; };
    std::ranges::transform(vec, vec.begin(), pure);
    for (int x : vec) std::cout << x << " "; // 输出: 2 4 6
    std::cout << "\n";

    // 非纯函数：满足 invocable，但不满足 regular_invocable
    int state = 0;
    auto non_pure = [&state](int x) mutable { return x + state++; };
    std::ranges::transform(vec, vec.begin(), non_pure);
    for (int x : vec) std::cout << x << " "; // 输出: 2 5 8（受 state 影响）
}
```

------

总结

- **std::invocable**：约束一个可调用对象可以以特定参数调用，只关心语法合法性，允许任意副作用。
- **std::regular_invocable**：更严格，要求调用是正则的（确定性、无意外副作用），适合函数式编程和 Ranges 算法。
- 在 Ranges 相关代码中，std::regular_invocable 常用于需要纯函数行为的场景，而 std::invocable 适用于更通用的回调场景。