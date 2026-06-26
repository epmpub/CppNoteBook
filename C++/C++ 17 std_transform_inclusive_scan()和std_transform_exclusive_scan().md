C++ 17 std::transform_inclusive_scan()`和`std::transform_exclusive_scan()

```C++
#include <numeric>
#include <algorithm>
#include <iostream>
#include <iterator>
#include <array>

int main()
{
    std::array coll{3, 1, 7, 0, 4, 1, 6, 3};

    auto twice = [] (int v) { return v*2; };

    std::cout << " source:                      ";
    std::copy(coll.begin(), coll.end(),
              std::ostream_iterator<int>(std::cout, " "));

    std::cout << "\n transform_inclusive_scan():      ";
    std::transform_inclusive_scan(coll.begin(), coll.end(),
                                  std::ostream_iterator<int>(std::cout, " "),
                                  std::plus{}, twice);

    std::cout << "\n transform_inclusive_scan():      ";
    std::transform_inclusive_scan(coll.begin(), coll.end(),
                                  std::ostream_iterator<int>(std::cout, " "),
                                  std::plus{}, twice, 100);

    std::cout << "\n transform_exclusive_scan():  ";
    std::transform_exclusive_scan(coll.begin(), coll.end(),
                                  std::ostream_iterator<int>(std::cout, " "),
                                  100, std::plus{}, twice); // 注意参数顺序
}
```



输出:

```C++
source:                      3 1 7 0 4 1 6 3
transform_inclusive_scan():      6 8 22 22 30 32 44 50
transform_inclusive_scan():      106 108 122 122 130 132 144 150
transform_exclusive_scan():  100 106 108 122 122 130 132 144
```

