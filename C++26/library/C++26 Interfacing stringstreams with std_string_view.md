

C++26 “Interfacing stringstreams with std::string_view”（提案 P2495R3）为 std::basic_stringbuf、basic_istringstream、basic_ostringstream 和 basic_stringstream 增加了直接支持 std::string_view（以及任何可隐式转换为 string_view 的类型）的构造函数和 str() 成员函数。 

sandordargo.com

主要变化在 <sstream> 头文件中新增了以下重载（以 basic_stringstream 为例，其他类似）：

cpp

```cpp
template<class CharT, class Traits = std::char_traits<CharT>,
         class Allocator = std::allocator<CharT>>
class basic_stringstream : public basic_iostream<CharT, Traits> {
public:
    // ... 现有构造函数 ...

    // C++26 新增：
    template<class T>
    explicit basic_stringstream(const T& t,
        ios_base::openmode which = ios_base::in | ios_base::out);

    template<class T>
    basic_stringstream(const T& t, const Allocator& a);

    template<class T>
    basic_stringstream(const T& t, ios_base::openmode which, const Allocator& a);

    // str() 成员函数也新增
    template<class T>
    void str(const T& t);
};
```

约束：std::is_convertible_v<const T&, std::basic_string_view<CharT, Traits>> 为 true。动机与改进在 C++26 之前，如果你有一个 std::string_view sv，必须先转为 std::string 才能构造 stringstream：

cpp

```cpp
// C++23 及之前（需要临时 string，产生拷贝）
std::string_view sv = "hello world";
std::stringstream ss(std::string(sv));   // 多余拷贝
```

C++26 后可以直接使用（零拷贝）：

cpp

```cpp
#include <sstream>
#include <string_view>
#include <iostream>

int main() {
    std::string_view sv = "C++26 is great!";

    // 直接构造
    std::stringstream ss(sv);                    // C++26
    std::istringstream iss(sv);
    std::ostringstream oss(sv);

    // 也可以带模式和分配器
    std::stringstream ss2(sv, std::ios::in);

    // 重置内容
    std::string_view new_view = "new content";
    ss.str(new_view);   // C++26

    std::string result = ss.str();
    std::cout << result << '\n';
}
```

关键设计要点

- 零拷贝：string_view 只提供视图，不发生字符串数据的拷贝（仅在 stringbuf 内部存储时才复制内容）。

- 兼容性：纯加法特性，不破坏现有代码。

- 适用范围：任何可转换为 basic_string_view 的类型（包括用户自定义的 string-like 类型，只要满足 is_convertible_v）。

- 特性测试宏：

  cpp

  ```cpp
  #ifdef __cpp_lib_sstream_from_string_view
      // 值通常为 202306L 或类似
  #endif
  ```

相关改进（同属 string/string_view 系列）C++26 还包括：

- std::bitset 支持从 string_view 构造（P2697）。
- std::string 与 std::string_view 之间支持 operator+（P2591）。

这个特性让 stringstream 在现代 C++ 中使用更自然，尤其适合需要解析或格式化 string_view 的场景，避免了不必要的临时 std::string 对象。 

sandordargo.com

更多细节可参考：

- [提案 P2495R3](https://wg21.link/P2495)
- cppreference（待更新）或 Clang 19+ / GCC 15+ 实现。