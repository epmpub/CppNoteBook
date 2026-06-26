# std::ranges::input_range concepts

```C++
// add() for multiple values:
template<typename Coll, std::ranges::input_range T>
    requires ConvertsWithoutNarrowing<std::ranges::range_value_t<T>,
typename Coll::value_type>
void add(Coll& coll, const T& val)
{
    if constexpr (SupportsPushBack<Coll>) {
        coll.insert(coll.end(),
            std::ranges::begin(val), std::ranges::end(val));
    }
    else {
        coll.insert(std::ranges::begin(val), std::ranges::end(val));
    }
}


```



std::ranges::input_range 是 C++20 引入的 Ranges 库中的一个概念（concept），用于约束一个类型是否是一个输入范围（input range）。它描述了一个可以通过迭代器进行单次遍历（single-pass traversal）的范围，允许读取其元素。

详细解释：

std::ranges::input_range 要求一个类型 T 满足以下条件：

1. **是范围（range）**：T 必须满足 std::ranges::range 概念，即可以通过 std::ranges::begin(T) 和 std::ranges::end(T) 获取迭代器和哨兵（sentinel），以遍历范围。
2. **输入迭代器**：范围的迭代器类型必须至少满足 std::input_iterator 的要求。这意味着迭代器支持单次遍历（只保证一次读取有效），可以解引用以访问元素（*it），并支持递增（++it）。

代码中的作用：

在你的代码中：

cpp

```cpp
template<typename Coll, std::ranges::input_range T>
```

std::ranges::input_range T 约束模板参数 T 必须是一个输入范围。这确保 T 是一个可以通过迭代器遍历的序列（如 std::vector, std::list, std::string, 或其他容器/视图），并且可以从中提取元素（通过 std::ranges::begin(val) 和 std::ranges::end(val)）。

示例：

以下类型满足 std::ranges::input_range：

- 标准容器：std::vector<int>, std::list<double>, std::string
- Ranges 视图：std::ranges::iota_view, std::ranges::filter_view
- 某些临时范围：如 std::views::iota(0, 10)

但不满足的类型包括：

- 非范围类型：int, std::pair<int, int>
- 不支持输入迭代器的类型（如某些只支持输出的流）

在你的代码中的具体用途：

你的 add 函数使用 std::ranges::input_range 来确保参数 val 是一个可以迭代的范围，从而允许通过 std::ranges::begin(val) 和 std::ranges::end(val) 安全地访问其元素，并将这些元素插入到目标容器 coll 中。

总结：

std::ranges::input_range 是一个 C++20 概念，用于约束类型是一个支持单次遍历的输入范围。它的作用是增强代码的类型安全性，确保函数接收的参数是一个可迭代的序列，适用于 Ranges 库的现代 C++ 编程风格。