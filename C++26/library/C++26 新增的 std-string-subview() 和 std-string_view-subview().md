C++26 新增的 std::string::subview() 和 std::string_view::subview()（提案 P3044R2）。1. 为什么需要 subview()？

- std::string::substr() 返回 std::string（拥有数据，会复制）。
- std::string_view::substr() 返回 std::string_view（非拥有，零成本）。
- 以前想从 std::string 得到一个 string_view 子串，必须显式转换：std::string_view(s).substr(...) 或 std::string_view(s.data() + pos, len)，不够方便且不够自文档。

subview() 解决了这个问题：从 string 或 string_view 直接得到 string_view 子视图，语义清晰、零拷贝。2. 接口

cpp

```cpp
// std::basic_string
constexpr std::basic_string_view<CharT, Traits>
    subview(size_type pos = 0, size_type count = npos) const;

// std::basic_string_view
constexpr std::basic_string_view<CharT, Traits>
    subview(size_type pos = 0, size_type count = npos) const;
```

- 行为与 substr() 几乎完全一致：
  - 返回 [pos, pos + rlen) 的视图，其中 rlen = std::min(count, size() - pos)。
  - 如果 pos > size()，抛出 std::out_of_range。
  - 复杂度：常数时间（零拷贝）。
- 使用示例

cpp

```cpp
#include <string>
#include <string_view>
#include <cassert>
#include <iostream>

int main() {
    const std::string s = "Life is life!";

    // 从 string 直接得到 string_view 子视图
    std::string_view v1 = s.subview(5);          // "is life!"
    std::string_view v2 = s.subview(5, 2);       // "is"
    std::string_view v3 = s.subview(5, 100);     // "is life!"（截断到末尾）

    assert(v1 == "is life!");
    assert(v2 == "is");

    // string_view 自身也有 subview（作为 substr 的别名）
    std::string_view sv = "Hello cruel world!";
    auto sub = sv.subview(6, 5);   // "cruel"

    // 链式调用也很自然
    std::string_view subsub = s.subview(5).subview(0, 2);  // "is"
}
```

与旧方式对比：

cpp

```cpp
// C++23 及之前
auto old_way = std::string_view(s).substr(5);     // 必须显式转换

// C++26
auto new_way = s.subview(5);                      // 更清晰
```

4. subview vs substr

| 特性       | substr()                     | subview() (C++26)              |
| ---------- | ---------------------------- | ------------------------------ |
| 返回类型   | string（复制） / string_view | 始终 string_view               |
| 性能       | 有复制开销（string 版）      | 始终零拷贝                     |
| 适用场景   | 需要拥有数据的子串           | 需要只读视图（推荐大多数情况） |
| 异常行为   | 同上                         | 同上                           |
| 链式友好度 | 一般                         | 优秀（返回 view 可继续调用）   |

5. 设计细节

- string::subview() 等价于 std::string_view(*this).subview(pos, count)。
- string_view::subview() 是 substr() 的同义函数（alternate spelling），方便泛型代码统一处理 string 和 string_view。
- 特性测试宏：__cpp_lib_string_subview（202506L）。

总结subview() 是 C++26 中一个小而美的改进，让“从字符串获取子视图”这个极其常见的操作变得自然、直观、零成本。强烈推荐在新代码中使用 subview() 代替手动构造 string_view 或 substr() 后转换。它延续了 span::subspan、mdspan 等“view”家族的命名风格，体现了现代 C++ “视图优先”的设计思想。