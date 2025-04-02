C++ optional 和 unique_ptr

If we want to represent an optional value, we can use std::unique_ptr (C++11) and heap allocation (nullptr denoting absence). However, this incurs a runtime cost and can fail (and throw).

std::optional (C++17) will not allocate dynamic memory and only incurs the cost of storing a boolean.



```c++
#include <memory>
#include <optional>

struct Data{};

std::unique_ptr<Data> dynamic_return() {
    return std::make_unique<Data>();
}

std::optional<Data> no_allocation() {
    return std::make_optional<Data>();
}

int main() {
    auto x = dynamic_return();
    if (x != nullptr) { // or simply if(x)
        // process data
    }

    auto y = no_allocation();
    if (y.has_value()) { // or simply if(y)
        // process data
    }
}
```

