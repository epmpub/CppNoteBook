std::async random

以下代码演示了

1 随机生成integra类型数据并插入set容器中;

2 使用async异步执行;

3 函数抛出异常处理

```C++
// generate_n.cpp : This file contains the 'main' function. Program execution begins and ends there.
//

#include <iostream>
#include <vector>
#include <random>
#include <set>
#include <algorithm>
#include <future>

std::set<int> make_sorted_random(const size_t num_elems) /*noexcept*/
{
    std::set<int> retval;
    std::random_device rd;
    std::mt19937 gen(rd());
    std::uniform_int_distribution<uint32_t> dis(0, num_elems-1);

    std::generate_n(std::inserter(retval,retval.end()), num_elems, [&]() {return dis(gen); });

    throw std::runtime_error("Hello world");

    return retval;
}

int main()
{
    auto f1 = std::async(std::launch::async, make_sorted_random, 1000000);
    auto f2 = std::async(std::launch::async, make_sorted_random, 1000000);

    try
    {
        f1.get();
    }
    catch (const std::exception& e)
    {
        std::cout << "f1 threw exception: " << e.what() << std::endl;
    }

    return 0;
}

```

