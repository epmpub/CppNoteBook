C++26 views::concat

```c#
#include <ranges>
#include <vector>
#include <array>
#include <list>
#include <iostream>

int main() {
    std::vector<int> v1 = {1, 2, 3};
    std::vector<int> v2 = {4, 5};
    std::array<int, 3> arr = {6, 7, 8};
    std::list<int> lst = {9, 10};

    // 拼接多个 range
    auto concatenated = std::views::concat(v1, v2, arr, lst);

    for (int x : concatenated) {
        std::cout << x << ' ';   // 输出: 1 2 3 4 5 6 7 8 9 10
    }

    // 支持随机访问（如果所有输入都支持）,lst不支持随机访问，所以 concatenated 也不支持随机访问
    // std::cout << concatenated[5] << '\n'; // 编译错误
    if (concatenated.size() > 5) {
        // concatenated[5] = 99;    // 报错；
    }
}

```

