#  *std::is_sorted* and *std::is_sorted_until* 

The C++11 *std::is_sorted*, and *std::is_sorted_until* algorithms verify that the provided range is sorted in non-descending order (using the operator< or a comparator).

The *std::is_sorted* algorithm returns a Boolean; 

the *std::is_sorted_until* returns an iterator to the first out-of-order element. 

第一个非正常排序的元素的迭代器;

Both algorithms provide parallel (C++17), constexpr (C++20) and range versions (C++20).

```C++
#include <algorithm>
#include <vector>

std::vector<int> nonsort{1,2,3,4,5,2,7,8,9};

auto r1 = std::is_sorted(nonsort.begin(), nonsort.end());
// r1 == false

auto r2 = std::is_sorted_until(nonsort.begin(), nonsort.end());
// r2 == nonsort.begin()+5, *r2 == 2


std::vector<std::string> sort{"x","mn","xyz","ijkl","abcde"};

auto r3 = std::ranges::is_sorted(sort,
    [](const auto& l, const auto& r) {
        return l.length() < r.length();
    });
// r3 == true

// Note, that while technically the above is equivalent to
// std::ranges::is_sorted(sort, {}, &std::string::length);
// it would be undefined behaviour as we are not permitted to
// take address of standard functions (including members).

auto r4 = std::ranges::is_sorted_until(sort, 
    [](const auto& l, const auto& r) {
        return l.length() < r.length();
    });
// r4 == sort.end()
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/9Eq9G9EEh)