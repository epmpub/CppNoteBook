

# 使用concept对函数参数类型做限制



But at the same time, when mixed with concepts, we can mitigate implicit conversions while only accepting a specific integral type.

```C++
#include <concepts>

template <typename T>
concept IsInt = std::same_as<int, T>;

void function(const IsInt auto&) {}

int main() {
    function(0); // OK
    // function(0u); // will fail to compile, deduced type unsigned
}
```

