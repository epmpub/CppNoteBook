# std::not_fn

The function wrapper that returns the negation of the wrapped callable: std::not_fn.

*std::not_fn* is a C++17 utility from the functional header that creates a simple forwarding call wrapper that returns the negation of the wrapped callable.

```c++
#include <algorithm>
#include <vector>
#include <functional>

std::vector<int> data{1, 2, 3, 4, 5, 6};
auto is_even = [](int v) { return v % 2 == 0; };

std::vector<int> out1;
std::copy_if(data.begin(), data.end(),
    std::back_inserter(out1),
    is_even);
// out1 == {2, 4, 6}

std::vector<int> out2;
std::copy_if(data.begin(), data.end(),
    std::back_inserter(out2), 
    std::not_fn(is_even));
// out2 == {1, 3, 5}
// Note: same as std::remove_copy_if with is_even
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/7vr4rPr1a)