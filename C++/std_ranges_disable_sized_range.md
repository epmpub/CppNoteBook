# std::ranges::disable_sized_range 

```C++
class MyCont {
...
std::size_t size() const; // assume this is expensive, so that this is not a sized range
};
// opt out from concept std::ranges::sized_range:
constexpr bool std::ranges::disable_sized_range<MyCont> = true;
```



std::ranges::disable_sized_range 是 C++20 Ranges 库中的一个定制点（customization point），用于明确禁用某个类型满足 std::ranges::sized_range 概念的行为。它允许开发者为特定类型（如你的 MyCont 类）选择退出 sized_range 概念，即使该类型可能表面上看起来符合条件。

背景：std::ranges::sized_range 概念

std::ranges::sized_range 是一个 C++20 概念，定义了一个范围（range），其大小可以通过 std::ranges::size(R) 在常数时间复杂度（O(1)）内获取。满足该概念的范围通常具有以下特性：

1. 是一个 std::ranges::range（可以通过 begin() 和 end() 遍历）。
2. 提供一个高效的 size() 方法，或者可以通过 std::ranges::end(R) - std::ranges::begin(R) 快速计算大小。

例如，std::vector, std::array, std::string 都满足 std::ranges::sized_range，因为它们可以在 O(1) 时间返回大小。

问题：为什么需要禁用？

在你的代码中，MyCont 有一个 size() 方法，但你注释说它是“昂贵的”（expensive，可能不是 O(1) 复杂度）。如果 MyCont 满足 std::ranges::range 且有 size() 方法，C++ 可能会自动推断它满足 std::ranges::sized_range 概念。然而，由于 size() 不是高效的，假定 MyCont 是 sized_range 可能会导致性能问题，因为 Ranges 算法可能依赖 size() 来优化操作，而昂贵的 size() 会违背这种优化的初衷。

为了避免这种情况，你可以通过设置 std::ranges::disable_sized_range<MyCont> = true 来明确告诉编译器：**不要将 MyCont 视为 std::ranges::sized_range**。

std::ranges::disable_sized_range 的作用

std::ranges::disable_sized_range 是一个 constexpr 布尔变量模板，定义在 <ranges> 头文件中。它的作用是：

- 当 std::ranges::disable_sized_range<T> == true 时，类型 T 被明确排除在 std::ranges::sized_range 概念之外。
- 这是一个定制点，允许用户为特定类型定制 Ranges 库的行为。

在你的代码中：

cpp

```cpp
constexpr bool std::ranges::disable_sized_range<MyCont> = true;
```

这行代码告诉编译器，尽管 MyCont 可能有 size() 方法或满足其他 sized_range 的条件，仍然不要将其视为 std::ranges::sized_range。这可以防止 Ranges 算法错误地假设 MyCont::size() 是高效的。

使用场景

1. **性能优化**：当一个类型的 size() 方法计算成本高（如需要遍历整个容器）时，禁用 sized_range 可以避免算法误用 size()。
2. **语义正确性**：某些类型可能有 size() 方法，但其语义不符合 Ranges 库对 sized_range 的期望（例如，size() 返回的不是元素数量）。
3. **第三方库兼容性**：如果你无法修改某个类型的实现，但需要确保它不被视为 sized_range，可以用 disable_sized_range 来调整行为。

示例

假设你的 MyCont 定义如下：

cpp

```cpp
class MyCont {
public:
    auto begin() const { /* ... */ }
    auto end() const { /* ... */ }
    std::size_t size() const { /* Expensive operation, e.g., O(n) */ }
};
```

如果没有禁用 sized_range，某些 Ranges 算法可能调用 MyCont::size()，导致性能问题。通过添加：

cpp

```cpp
constexpr bool std::ranges::disable_sized_range<MyCont> = true;
```

你确保 MyCont 不会被视为 sized_range，算法将转而使用其他方式（例如通过迭代器遍历）来处理 MyCont。

注意事项

1. **特化位置**：std::ranges::disable_sized_range 必须在 std::ranges 命名空间中特化，通常在全局命名空间中声明。例如：

   cpp

   ```cpp
   namespace std::ranges {
       constexpr bool disable_sized_range<MyCont> = true;
   }
   ```

   你的代码中直接使用 constexpr bool std::ranges::disable_sized_range<MyCont> = true; 是正确的。

2. **仅影响 sized_range**：禁用 sized_range 不会影响 MyCont 是否满足其他概念（如 std::ranges::range 或 std::ranges::input_range）。

3. **谨慎使用**：仅在确实需要避免 sized_range 行为时使用此定制点。如果 size() 是高效的，通常不需要禁用。

总结

std::ranges::disable_sized_range 是一个 C++20 Ranges 库的定制点，用于明确禁止某个类型满足 std::ranges::sized_range 概念。在你的代码中，它用于防止 MyCont 被视为 sized_range，因为其 size() 方法计算成本高。这增强了代码的性能和语义正确性，避免了 Ranges 算法误用低效的 size() 方法。