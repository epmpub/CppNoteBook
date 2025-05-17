std::inclusive_scan 和 std::exclusive_scan

```C++
#include <numeric>
#include <iostream>
#include <iterator>
#include <array>

int main()
{
    std::array coll{ 1,2,3,4 };

    std::cout << " inclusive_scan():   ";
    std::inclusive_scan(coll.begin(), coll.end(),
        std::ostream_iterator<int>(std::cout, " "));

    std::cout << "\n exclusive_scan(): ";
    std::exclusive_scan(coll.begin(), coll.end(),
        std::ostream_iterator<int>(std::cout, " "),
        0);     // 必须的

    std::cout << "\n exclusive_scan(): ";
    std::exclusive_scan(coll.begin(), coll.end(),
        std::ostream_iterator<int>(std::cout, " "),
        100);   // 必须的

    std::cout << "\n inclusive_scan():     ";
    std::inclusive_scan(coll.begin(), coll.end(),
        std::ostream_iterator<int>(std::cout, " "),
        std::plus{}, 100);

    std::cout << "\n exclusive_scan(): ";
    std::exclusive_scan(coll.begin(), coll.end(),
        std::ostream_iterator<int>(std::cout, " "),
        100, std::plus{});  // 注意：参数顺序不同
}
```



`inclusive_scan()`和`exclusive_scan()`的区别在于:

`exclusive_scan()`的结果以初始值开始并**排除输入的最后一个元素。**
注意两种形式函数参数和初始值参数的顺序不同。

另外`exclusive_scan()`中初始值参数是必须的。



输出:

```C++
 inclusive_scan():   1 3 6 10
 exclusive_scan(): 0 1 3 6
 exclusive_scan(): 100 101 103 106
 inclusive_scan():     101 103 106 110
 exclusive_scan(): 100 101 103 106
C:\Users\sheng\source\repos\transform_reduce\x64\Debug\transform_reduce.exe (process 19844) exited with code 0 (0x0).
Press any key to close this window . . .
```

