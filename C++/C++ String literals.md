# String literals 

String literals String literals in C++ are immutable character arrays (for compatibility with C).

This can be inconvenient when working with generic functions or when using auto.

Since C++14, the standard library provides support for standard string literals, that is, literals of type *std::string* and its variants.

```
#include <string>
using namespace std::string_literals;

auto s1 = "Hello World!"s;
// decltype(s1) == std::string

auto s2 = L"😀"s;
// decltype(s2) == std::wstring

// utf8 string literal since C++20
auto s3 = u8"🙃"s; 
// decltype(s3) == std::u8string

auto s4 = u"😜"s;
// decltype(s4) == std::u16string

auto s5 = U"🤔"s;
// decltype(s5) == std::u32string
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/ocn3szTKq)