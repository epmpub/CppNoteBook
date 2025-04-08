# *std::ranges::starts_with*和*std::ranges::ends_with*

两种 C++23 范围算法*std::ranges::starts_with*和*std::ranges::ends_with*实现范围的前缀和后缀检查。

*std::ranges::starts_with*可以在任何范围上操作，*std::ranges::ends_with*至少需要一个前向范围。



```C++
#include <algorithm>
#include <vector>
#include <forward_list>
#include <print>

int main() {
    std::vector<int> haystack{1,2,3,4,5,6,7,8,9};
    std::forward_list<int> prefix{1,2,3};
    std::forward_list<int> suffix{7,8,9};

    bool pref = std::ranges::starts_with(haystack, prefix);
    bool suff = std::ranges::ends_with(haystack, suffix);
    // pref == true, suff == true

    std::println("pref == {}, suff == {}", pref, suff);

    bool nomatch = std::ranges::starts_with(prefix, suffix);
    // nomatch == false

    std::println("nomatch == {}", nomatch);

    // With projections and custom comparator
    bool proj = std::ranges::starts_with(haystack, suffix, 
        [](int l, int r) { return std::abs(l-r) <= 2; }, // custom comparator
        [](int l) { return l + 2; }, // project to {3,4,5...}
        [](int r) { return r - 2; }); // project to {5,6,7}
    // proj == true

    std::println("proj == {}", proj);
}
```

