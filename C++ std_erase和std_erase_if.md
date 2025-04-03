# *std::erase*和*std::erase_if*

在 C++20 之前，根据元素的值或谓词从大多数容器中删除元素是一个**两步**过程。

C++20 添加了一组特定于容器的*std::erase*和*std::erase_if*函数重载。

这些函数将两步过程简化为**单个**调用，并且（对于值版本）支持异构比较。

```c++
#include <vector>
#include <map>
#include <algorithm>
#include <iostream>

int main() {
    std::vector<int> data{1,2,3,4,5};

    std::erase(data, 3);
    // same as:
    // auto it = std::remove(data.begin(), data.end(), 3);
    // data.erase(it, data.end());
    // data == {1, 2, 4, 5}

    std::cout << "data == ";
    for (auto v : data)
        std::cout << v << " ";
    std::cout << "\n";

    std::erase_if(data, [](int v) { return v % 2 == 0; });
    // same as:
    auto it = std::remove_if(data.begin(), data.end(), [](int v) { return v % 2 == 0; });
    data.erase(it, data.end());

    std::cout << "data == ";
    for (auto v : data)
        std::cout << v << " ";
    std::cout << "\n";

    // Additionally std::erase(_if) returns the number of erased elements:
    size_t cnt = std::erase_if(data, [](int v){ return v != 0; });
    // cnt == 2, data == {}

    std::cout << "cnt == " << cnt << "\n";

    std::cout << "data == ";
    for (auto v : data)
        std::cout << v << " ";
    std::cout << "\n";

    // Supported for:
    // std::vector, std::deque, std::list, std::forward_list, std::string

    // Heterogeneous comparison:
    std::vector<std::string> mixed{"hello","this","is","dog"};
    std::erase(mixed, "hello");
    // "hello" directly compared with each element without conversion
    // mixed == {"this", "is", "dog"}

    // unordered and associative containers only support std::erase_if
    std::map<int,int> map{{1,2},{2,3},{3,4}};
    std::erase_if(map, [](auto &e) { return e.first < 3; });
    // map == {3->4}

    std::cout << "map == ";
    for (auto &[k,v] : map)
        std::cout << k << "->" << v << " ";
    std::cout << "\n";
    // Note that map::erase() already supports heterogeneous erase-by-value
}
```

