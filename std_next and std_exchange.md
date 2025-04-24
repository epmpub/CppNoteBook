# std::next and std::exchange



```C++
// next_and_exchange.cpp : This file contains the 'main' function. Program execution begins and ends there.
//

#include <iostream>
#include <utility>
#include <vector>
#include <algorithm>

int main()
{
    int i = 10;
    int j = 1;
    int retval = std::exchange(i,j);

    std::cout << "i: " << i << " j: " << j << " retval: " << retval << std::endl;

    std::vector<int> v = { 2,1, 2, 3, 4, 5 };

    std::cout << "is sorted ? " << std::is_sorted(std::next(v.begin()), v.end()) << std::endl;

    return 0;
}


```

 

std::exchange 要求第一个参数是可赋值的

问题分析

- std::exchange 的定义：

  cpp

  ```cpp
  template<typename T, typename U = T>
  T exchange(T& obj, U&& new_value);
  ```

  它将 obj 的值替换为 new_value，并返回 obj 的旧值。

  obj 必须是可赋值的，而 std::cout 和 std::cerr 是全局对象，无法被重新赋值。