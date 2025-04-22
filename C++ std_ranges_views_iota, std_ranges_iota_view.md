

# C++ std::ranges::views::iota, std::ranges::iota_view

```C++
#include <algorithm>
#include <iostream>
#include <ranges>
 
struct Bound
{
    int bound;
    bool operator==(int x) const { return x == bound; }
};
 
int main()
{
    for (int i : std::ranges::iota_view{1, 10})
        std::cout << i << ' ';
    std::cout << '\n';
 
    for (int i : std::views::iota(1, 10))
        std::cout << i << ' ';
    std::cout << '\n';
 
    for (int i : std::views::iota(1, Bound{10}))
        std::cout << i << ' ';
    std::cout << '\n';
 
    for (int i : std::views::iota(1) | std::views::take(9))
        std::cout << i << ' ';
    std::cout << '\n';
 
    std::ranges::for_each(std::views::iota(1, 10),
                          [](int i){ std::cout << i << ' '; });
    std::cout << '\n';
}
```

