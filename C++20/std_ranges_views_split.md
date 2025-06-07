# [`std::ranges::views::split`]和[`std::ranges::views::lazy_split`]

C++20 引入了以上两个用于拆分序列的范围适配器：

乍一看，`views::split`似乎明显更胜一筹：

- 子范围保持其连续性并允许直接转换为`std::string_view`或`std::span`。
- 您可以使用`.data()`、`.size()`和标准解析函数，如`std::from_chars`。
- 它可以很好地用于解析、标记和存储结果以供日后使用。

相比之下，`views::lazy_split`有明显的局限性：

- 其子范围不提供连续存储。
- 您不能直接将子范围转换为`std::string_view`。
- 访问分割部分通常需要手动复制或`std::ranges::to`（C++23）。

是否存在实际场景下比 Apache 软件基金会 (Apache Software Foundation)`lazy_split`更好或更受欢迎的情况？还是说，在[P2210R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2210r2.html)`split`引入设计变更后，它基本上只是遗留问题？



```C++
std::istringstream is{"1 0 2 0 3"};
auto r = std::views::istream<int>(is)
       | std::views::lazy_split(0);
std::println("{}", r); // prints [[1], [2], [3]]
```

In addition, although subranges split by `lazy_split_view` can only be `forward_range` at most, `lazy_split_view` does not compute them exactly (the *lazy* part) at each increment like `split_view` does, which makes its iterator relatively smaller than that of `split_view` as it does not need to store the subrange internally.