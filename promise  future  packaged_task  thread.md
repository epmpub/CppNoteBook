# promise / future / packaged_task / thread 

```c++
// promise_.cpp : This file contains the 'main' function. Program execution begins and ends there.
//

#include <iostream>
#include <thread>
#include <future>
#include "promise_.h"

void pro_future()
{
    std::promise<int> pro;
    std::future<int> r = pro.get_future();
    pro.set_value(1);
    std::cout << r.get() << "\n";
}

int  f(int a, int b) { return std::pow(a, b); }
void task_bind()
{
    std::packaged_task<int()> task(std::bind(f, 2, 4));
    std::future<int> result = task.get_future();
    task();
    std::cout << result.get() << "\n";
}

void task_lambda()
{
    std::packaged_task<int(int, int)> task1([](int a, int b) {
        return std::pow(a, b);
        });
    std::future<int> f = task1.get_future();
    task1(2, 5);

    std::cout << f.get() << "\n";
}

void task_thread()
{
    std::packaged_task<int(int, int)> task(f);
    std::future<int> result = task.get_future();

    std::thread task_td(std::move(task), 2, 10);
    task_td.join();
    std::cout << "task_thread:\t" << result.get() << "\n";
}

int main()
{
    task_lambda();
    task_bind();
    task_thread();

    return 0;

}


```

