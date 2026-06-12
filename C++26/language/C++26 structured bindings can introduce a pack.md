structured bindings can introduce a pack



```c++
#include <tuple>
#include <iostream>

// 函数模板
template<typename... Ts>
void print_first_and_rest(const std::tuple<Ts...>& t) {
    auto [first, ...rest] = t;
    std::cout << first << '\n';
    ((std::cout << rest << ' '), ...);
    std::cout << '\n';
}

int main() {
    std::tuple t(10, 20, 30, 40);
    print_first_and_rest(t);           // 10  20 30 40
}
```

