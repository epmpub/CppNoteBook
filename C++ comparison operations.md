# comparison operations

The function objects *std::equal_to*, *std::not_equal_to*, *std::greater*, *std::less*, *std::greater_equal*, *std::less_equal* and (C++20) *std::compare_three_way* implement the corresponding comparison operations.

These objects can serve as comparators for algorithms that accept them. This saves us the hassle of writing a lambda just to return the result of comparing the two arguments.

```C++
#include <functional>
#include <algorithm>
#include <vector>

struct A {};
struct B {};
bool operator<(A,B) { return false; }


std::vector<int> data{8,4,1,3,2,9,6,5,7};
// Explicit specialization version:
std::ranges::sort(data, std::greater<int>{});
// data == {9,8,7,6,5,4,3,2,1}

// Explicit specialization can cause issues with implicit conversions.
std::vector<double> fpoint{4.2,4.1,4.3,4.4,4.9,4.5};
std::ranges::sort(fpoint, std::greater<int>{});
// Unspecified order, but technically should be a no-op.
// Reason: all elements are equal when converted to int.

// C++14 <void> specialization with deduction
std::ranges::sort(fpoint, std::greater<>{});
// Arguments of the comparator will be deducated as double.
// fpoint == {4.9,4.5,4.4,4.3,4.2,4.1}

// C++20 ranges version, arguments are always deduced.
std::ranges::sort(data, std::ranges::greater{});

// When using with non-homogenous types, the behaviour differs.
int x = 4; double y = 4.2;
// Old style, explicit specified type with forced implicit conversion
bool cmp1 = std::less<int>{}(x,y);
bool cmp2 = std::less<double>{}(x,y);
// cmp1 == false, cmp2 == true

// C++14, both arguments are deduced, operator< potentially
// aplied to a heterogeneous type.
bool cmp3 = std::less<>{}(x,y); // <int,double>
// cmp3 == true

// C++20, both arguments are deduced, 
// but constrained with stricter semantics.
bool cmp4 = std::ranges::less{}(x,y);
// cmp4 == true

std::less<>{}(A{},B{}); // OK
// std::ranges::less{}(A{},B{}); // Will not compile
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/dsdq5dneb)