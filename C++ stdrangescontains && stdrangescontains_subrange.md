

#  std::ranges::contains && std::ranges::contains_subrange

C++23 introduced two new algorithms in the ranges namespace.

\- std::ranges::contains
\- std::ranges::contains_subrange

These two algorithms do not introduce novel behavior; instead, they remove the need to translate the results of std::ranges::find and std::ranges::search.



```c++
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> data{1,2,3,4,5,6,7,8,9};
    
    bool r1 = std::ranges::contains(data, 5);
    // r1 == true
    // Same as:
    bool r2 = std::ranges::find(data, 5) != data.end();
    // r2 == true

    std::cout << std::boolalpha << "r1 == " << r1 << ", r2 == " << r2 << "\n";

    bool r3 = std::ranges::contains_subrange(data, auto({3,4,5}));
    // r3 == true
    // Same as:
    bool r4 = not std::ranges::search(data, auto({3,4,5})).empty();
    // r4 == true

    std::cout << std::boolalpha << "r3 == " << r3 << ", r4 == " << r4 << "\n";
}
```

