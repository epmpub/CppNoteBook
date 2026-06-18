



```c
#include <functional>
#include <iostream>
#include <vector>
#include <string>

int main() {
    int a = 10;
    std::string s = "hello";

    // 创建 reference_wrapper
    std::reference_wrapper<int> rw_int = std::ref(a);
    std::reference_wrapper<std::string> rw_str = std::ref(s);

    // 使用方式和普通引用类似
    std::cout << rw_int << '\n';           // 10
    std::cout << rw_str.get() << '\n';     // hello   (.get() 获取引用)

    rw_int.get() = 42;        // 修改原变量
    std::cout << a << '\n';                // 42

    // std::cref 用于 const 引用
    auto rw_const = std::cref(s);
}
```

