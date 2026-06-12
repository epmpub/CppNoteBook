Ordering of constraints involving fold expressions

在 C++26 中，**[P2963R3 提案（Ordering of constraints involving fold expressions）](https://cppreference.com/cpp/26)** 解决了一个困扰泛型编程多年的核心痛点：**允许编译器“看穿”折叠表达式（Fold Expressions），从而对包含参数包的约束（Concepts）进行正确的偏特化偏序（Subsumption / 包含关系）推导**。 [1, 2] 

这一改进使得涉及变长参数包的重载决议（Overload Resolution）变得更加智能和直观。 [1] 

------

## 历史痛点：折叠表达式是一堵“不透明的墙”

在 **C++20 和 C++23** 中，约束的包含关系（Subsumption）只能识别三种基础约束：**合取（`&&`）**、析取（`||`）和原子约束（Atomic Constraints）。 [1, 2] 

由于折叠表达式（如 `(Concept<Ts> && ...)`）在语法上是一个独立的整体表达式，C++20/23 只能将其当作一个**原子约束**来处理。这意味着**编译器无法窥探折叠表达式内部的模式**。 [1, 3] 

##  C++23 报错示例

假设我们有两个 Concept，其中 `Integral` 是 `Object` 的更严格子集（`Integral` 包含了 `Object`）：

```cpp
template<typename T> concept Object = std::is_object_v<T>;
template<typename T> concept Integral = Object<T> && std::is_integral_v<T>;

// 重载 A：要求包中所有类型都是 Object
template<typename... Ts> requires (Object<Ts> && ...)
void process(Ts... args);

// 重载 B：要求包中所有类型都是 Integral（更严格）
template<typename... Ts> requires (Integral<Ts> && ...)
void process(Ts... args);

int main() {
    // C++23 编译错误：call to 'process' is ambiguous（调用产生二义性）
    process(1, 2, 3); 
}
```

- **为什么 C++23 会失败？** 虽然我们知道 `Integral` 强于 `Object`，但编译器将 `(Object<Ts> && ...)` 和 `(Integral<Ts> && ...)` 视为两个互不相干的原子约束。它不知道这两个折叠表达式展开后的结构是一样的，因此无法判断谁“更严格”，最终抛出二义性错误。 [1] 

------

## C++26 的解决方案：引入“折叠展开约束”

C++26 引入了一种全新的约束类别：**折叠展开约束（Fold Expanded Constraints）**。
现在，约束规范化（Constraint Normalization）机器能够理解折叠表达式的结构，并提取出它的内部模式（Pattern）和操作符（`&&` 或 `||`）。 [1, 4] 

## 新的包含（Subsumption）规则

在 C++26 中，如果一个折叠约束 P 想要包含（Subsume）另一个折叠约束 Q，必须同时满足以下三个条件： [1, 4] 

1. 两个折叠表达式必须**展开相同的参数包**。
2. 两个折叠表达式必须**使用相同的折叠操作符**（即同为 `&&` 或同为 `||`）。
3. 在针对单个类型 T 时，**P 的内部模式必须包含 Q 的内部模式**。 [1, 4] 

对于上文的例子：

- 它们展开相同的包 `Ts`。
- 它们都使用 `&&` 操作符。
- 对于单类型 `T`，`Integral<T>` 包含了 `Object<T>`。 [1, 4] 

因此，**C++26 完美通过编译**，并会精准选择更严格的**重载 B**。 [1] 

------

## 两个重要的技术细节

## 1. 操作符必须严格匹配

若操作符不一致，即使逻辑上有包含关系，标准也**不认定**其具有包含性。例如： [1] 

- `(Integral<Ts> && ...)` **不会**包含 `(Integral<Ts> || ...)`。
  因为前者要求“全都是”，后者要求“至少有一个”，它们在逻辑上属于不同的声明，因此无法直接进行强弱比较。 [1] 

## 2. 左右折叠的语义归一化

为了防止开发者因为手写习惯不同（如写成左折叠 `(... && Pattern)` 或右折叠 `(Pattern && ...)`）导致推导失败，C++26 会在编译器内部对它们进行**归一化统一**（将其全部视为一类逻辑合取/析取），从而保证无论你怎么写折叠表达式，包含偏序推导都能正常工作。 [2] 

## 总结

C++26 的这一改进打通了 **Concept（概念）** 与 **Variadic Templates（变长模板）** 之间的最后一公里。它允许我们在编写高性能变长完美转发、元编程库或 tuple 复合操作时，直接写出符合直觉的重载代码，而不再需要退回到 C++17 时代用 `std::enable_if_t` 和复杂多路重载的丑陋打补丁方案。 [1] 

如果你在写类似 `std::variant` 或高阶元编程容器，是否遇到了某些参数包重载的二义性痛点？我可以帮你用 C++26 的新特性重构它们！

[1] [https://www.sandordargo.com](https://www.sandordargo.com/blog/2026/05/27/cpp26-constraints-ordering-fold-expressions)

[2] [https://www.open-std.org](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2963r3.pdf)

[3] [https://www.reddit.com](https://www.reddit.com/r/cpp/comments/kd77ee/fold_expressions_work_inside_constraints/)

[4] [https://app.daily.dev](https://app.daily.dev/posts/c-26-ordering-of-constraints-involving-fold-expressions-gliewnpif)