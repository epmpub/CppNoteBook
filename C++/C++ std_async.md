# C++ std::async的用法

```C++
// generate_n.cpp : This file contains the 'main' function. Program execution begins and ends there.

#include <iostream>
#include <vector>
#include <random>
#include <set>
#include <algorithm>
#include <future>
#include <print>

std::set<int> make_sorted_random(const size_t num_elems) /*noexcept*/
{
    std::set<int> retval;
    std::random_device rd;
    std::mt19937 gen(rd());
    std::uniform_int_distribution<uint32_t> dis(0, num_elems-1);

    std::generate_n(std::inserter(retval,retval.end()), num_elems, [&]() {return dis(gen); });
    return retval;
}

int main()
{
    auto f1 = std::async(std::launch::async, make_sorted_random, 1000000);
    auto f2 = std::async(std::launch::async, make_sorted_random, 1000000);
    auto f3 = std::async(std::launch::async, make_sorted_random, 1000000);
    auto f4 = std::async(std::launch::async, make_sorted_random, 1000000);

    std::println("f1: {} f2:{} f3:{} f4:{} ", f1.get().size(),f2.get().size(),f3.get().size(),f4.get().size());
    return 0;
}

```

