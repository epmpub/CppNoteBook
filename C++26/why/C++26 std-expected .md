相等运算符的约束（Constraining equality operators） 是对 C++23 std::expected 的一个重要改进，由提案 P3379R0 引入。 

open-std.org

背景与动机C++23 引入 std::expected<T, E> 时，其相等运算符（operator==）使用 Mandates（硬性要求）来检查比较表达式是否合法。如果类型不支持比较，程序会直接 ill-formed（编译错误），但这种检查方式不够“友好”，容易导致意外的模板实例化失败或 SFINAE 不佳。

C++26 借鉴了 std::optional、std::pair、std::tuple、std::variant 等类型的类似改进（参考 P2944R3 和 LWG4072），将 std::expected 的比较运算符改为使用 Constraints（约束），使其能更好地参与重载决议（SFINAE-friendly）。 



主要变化

- Mandates → Constraints：原来的硬性要求改为参与重载决议的约束。
- 新增特性测试宏：__cpp_lib_constrained_equality（值为 202411L）。

主要重载的变化（简化描述）

1. 两个 expected 比较：

   cpp

   ```cpp
   template<class T2, class E2> requires (!is_void_v<T2>)
   friend constexpr bool operator==(const expected& x, const expected<T2, E2>& y);
   ```

   - C++26 前：Mandates *x == *y 和 x.error() == y.error() 必须合法。
   - C++26：只有当这些表达式 well-formed 且结果可转换为 bool 时，该重载才参与重载决议。

2. 与 T 值比较：

   cpp

   ```cpp
   template<class T2> friend constexpr bool operator==(const expected& x, const T2& v);
   ```

   - 增加约束：T2 不是 expected 的特化，且 *x == v 必须合法。

3. 与 unexpected<E2> 比较：

   - 约束 x.error() == e.error() 必须合法。

void 偏特化（expected<void, E>）也有对应约束调整。 

en.cppreference.com

好处

- 更好的 SFINAE / Concepts 行为：当 T 或 E 不支持 == 时，重载不会被选中，而是尝试其他候选（或编译失败时给出更清晰错误）。
- 一致性：与 std::optional 等其他标准库类型行为对齐。
- 模板代码更健壮：在泛型代码中使用 expected 时，约束失败更易处理，不会导致整个模板立即 ill-formed。

示例

cpp

```cpp
std::expected<int, std::string> e1 = 42;
std::expected<long, std::string> e2 = 42;

if (e1 == e2) { ... }  // C++26 中约束检查更友好

// 如果 value_type 不支持 ==，相关重载不会参与决议
```

参考

- 提案：[P3379R0](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p3379r0.html)
- cppreference：[operator==(std::expected)](https://en.cppreference.com/cpp/utility/expected/operator_cmp)
- 特性测试宏：__cpp_lib_constrained_equality

这是 C++26 对标准库一致性和可用性的一次小但实用的打磨，尤其对泛型编程和模板元编程友好。