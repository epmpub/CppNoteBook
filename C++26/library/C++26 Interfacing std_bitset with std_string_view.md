C++26 “Interfacing std::bitset with std::string_view”（提案 P2697R1）为 std::bitset<N> 增加了直接支持 std::basic_string_view 的构造函数，避免了之前必须先转为 std::string 才能构造所产生的不必要拷贝。 

sandordargo.com

主要新增内容在 <bitset> 头文件中，std::bitset<N> 新增了以下构造函数（C++26）：

cpp

```cpp
template<std::size_t N>
class bitset {
public:
    // C++26 新增的重载
    template<class CharT>
    explicit bitset(const std::basic_string_view<CharT>& str,
                    typename std::basic_string_view<CharT>::size_type pos = 0,
                    typename std::basic_string_view<CharT>::size_type n = std::basic_string_view<CharT>::npos,
                    CharT zero = CharT('0'),
                    CharT one = CharT('1'));

    // 类似地，也支持 const CharT* 的新重载（行为一致）
};
```

这个新构造函数的行为与原有的 std::string 版本完全一致，只是参数类型从 const std::basic_string<CharT>& 变成了 const std::basic_string_view<CharT>&。使用示例

cpp

```cpp
#include <bitset>
#include <string_view>
#include <iostream>

int main() {
    std::string_view sv = "101101001";   // 二进制字符串

    // C++26 之前（需要临时 string，产生拷贝）
    // std::bitset<9> b1(std::string(sv));

    // C++26 之后（零拷贝构造）
    std::bitset<9> b(sv);                    // 直接使用 string_view

    // 也可以指定起始位置、长度和字符映射
    std::bitset<4> b2(sv, 2, 4, '0', '1');  // 从位置2取4位

    std::cout << b << '\n';      // 输出: 101101001
    std::cout << b2 << '\n';     // 输出: 1101
    std::cout << b.to_ullong() << '\n';
}
```

动机与改进

- 避免不必要的拷贝：string_view 是轻量级的视图，直接传递无需复制底层字符序列。
- 一致性：与 C++26 中 std::stringstream、std::string 其他接口的 string_view 支持保持一致（同属 Michael Florian Hava 的系列提案）。
- 性能：在需要从字符串字面量、子串或视图构造 bitset 的场景下更高效，尤其当字符串较长时。
- 兼容性：纯加法特性，不影响现有 std::string 构造函数。

特性测试宏

cpp

```cpp
#ifdef __cpp_lib_bitset
    // C++26 值为 202306L
#endif
```

设计要点

- 只增加了构造函数，没有为 to_string() 添加 string_view 版本（因为 to_string 返回的是拥有数据的 std::string）。
- 支持自定义 zero 和 one 字符（默认 '0' 和 '1'）。
- 如果输入字符串中出现除 zero/one 以外的字符，会抛出 std::invalid_argument 异常。
- 长度检查：如果有效位数超过 N，同样会抛出异常。

这个特性属于 C++26 中一系列提升 string_view 易用性的小改进之一（其他包括 stringstream 支持 string_view），让现代 C++ 代码在处理只读字符串视图时更加自然和高效。 

sandordargo.com

更多细节可参考：

- 提案 [P2697R1](https://wg21.link/P2697)
- [cppreference: std::bitset::bitset](https://en.cppreference.com/cpp/utility/bitset/bitset)（已更新 C++26 版本）