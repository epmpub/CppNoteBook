# std::midpoint

计算两个算术类型或指针之间的中点值可能看起来微不足道；

然而，当值接近数值限制或无序时，简单的实现很容易遇到未定义的行为。

C++20 引入了*std::midpoint*，它提供了一个安全的实现。

```c++
#include <numeric>

 auto m1 = std:: midpoint (std::numeric_limits< int >:: max (), 
    std::numeric_limits< int >:: max () -2 ); 
// m1 == std::numeric_limits<int>::max()-1 

int data[]={ 5 , 9 , 2 , 3 , 1 , 8 , 4 , 6 , 7 }; 
auto m2 = std:: midpoint (data, data+ 9 ); 
// *m2 == 1 

auto m3 = std:: midpoint ( 3.2 , 7.6 ); 
// m3 ~= 5.4
```

[在 Compiler Explorer 中打开示例。](https://compiler-explorer.com/z/jW8qG519s)