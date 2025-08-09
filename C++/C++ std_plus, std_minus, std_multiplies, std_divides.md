# The *std::plus*, *std::minus*, *std::multiplies*, *std::divides*

 *std::modulus* and *std::negate* are function objects from the *<functional>* header that wrap the corresponding arithmetic operators.

Since C++14, these objects also provide a void specialization that relies on type deduction for both arguments.

```c++
#include <vector>
#include <algorithm>

auto int_plus = std::plus<int>{};
auto v = int_plus(4.2, 3.9);
// decltype(v) == int, v == 7 (4+3)

auto deduced_plus = std::plus<>{};
auto w = deduced_plus(4, 3.9);
// decltype(w) == double, w ~= 7.9

// Useful when working with algorithms:
std::vector<int> data{1,2,3,4,5};
std::vector<int> out;
std::ranges::transform(data, std::back_inserter(out),
                       std::negate<>{});
// out == {-1,-2,-3,-4,-5}

auto product = std::ranges::fold_left(data, 1, 
                                      std::multiplies<>{});
// product == 120
```

