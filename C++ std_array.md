# *std::array*

The *std::array* is a container that represents fixed-sized arrays. Besides the range interface,

 *std::array* also avoids the implicit decay into a pointer (e.g. *int[3]* into *int**).

避免隐式转换

On top of that, *std::array* doesn’t have explicit constructors, 

allowing it to maintain the trivially copyable property of the underlying data.

```C++
#include <array>
#include <algorithm>
#include <functional>

void fun(int,int,int,int,int) {}
struct A { A(int,int,int,int,int) {} };


std::array<int,5> data{1,2,3,4,5};

// Models contiguous range
std::ranges::sort(data, std::greater<>{});
// data == {5,4,3,2,1}

// Models tupple-like (C++11)
int x = std::get<2>(data);
// x == 4
size_t sz = std::tuple_size<decltype(data)>{};
// sz == 5

// Since C++17:
auto [a,b,c,d,e] = data;
// a == 5, b == 4, c == 3, d == 2, e == 1

// Since C++23:
// Construct A with the elements of data
A m = std::make_from_tuple<A>(data);
// Call fun with the elements of data
std::apply(fun, data);
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/eYbjoq76o)