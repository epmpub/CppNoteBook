





# std::bind std::function

```C++
// bind.cpp : This file contains the 'main' function. Program execution begins and ends there.
//

#include <iostream>
#include <functional>
#include <string>
#include <type_traits>

template <typename T>
void print(T i, const std::string& s)
{
    std::cout << i << ' ' << s << '\n';
}


int main()
{
    int i = 5;
    const auto f = std::bind(&print<int>, std::placeholders::_2,std::placeholders::_1);
    f("hello",i,1,2);

    i = 6;
    f("world", i);

    std::function<void(const std::string&, int)> f2(f);
    //f2(i,"goodbye");

    f2("test",i);

}


```





example:

```C++
#include <functional>
#include <iostream>
 
struct Foo
{
    Foo(int num) : num_(num) {}
    void print_add(int i) const { std::cout << num_ + i << '\n'; }
    int num_;
};
 
void print_num(int i)
{
    std::cout << i << '\n';
}
 
struct PrintNum
{
    void operator()(int i) const
    {
        std::cout << i << '\n';
    }
};
 
int main()
{
    // store a free function
    std::function<void(int)> f_display = print_num;
    f_display(-9);
 
    // store a lambda
    std::function<void()> f_display_42 = []() { print_num(42); };
    f_display_42();
 
    // store the result of a call to std::bind
    std::function<void()> f_display_31337 = std::bind(print_num, 31337);
    f_display_31337();
 
    // store a call to a member function
    std::function<void(const Foo&, int)> f_add_display = &Foo::print_add;
    const Foo foo(314159);
    f_add_display(foo, 1);
    f_add_display(314159, 1);
 
    // store a call to a data member accessor
    std::function<int(Foo const&)> f_num = &Foo::num_;
    std::cout << "num_: " << f_num(foo) << '\n';
 
    // store a call to a member function and object
    using std::placeholders::_1;
    std::function<void(int)> f_add_display2 = std::bind(&Foo::print_add, foo, _1);
    f_add_display2(2);
 
    // store a call to a member function and object ptr
    std::function<void(int)> f_add_display3 = std::bind(&Foo::print_add, &foo, _1);
    f_add_display3(3);
 
    // store a call to a function object
    std::function<void(int)> f_display_obj = PrintNum();
    f_display_obj(18);
 
    auto factorial = [](int n)
    {
        // store a lambda object to emulate "recursive lambda"; aware of extra overhead
        std::function<int(int)> fac = [&](int n) { return (n < 2) ? 1 : n * fac(n - 1); };
        // note that "auto fac = [&](int n) {...};" does not work in recursive calls
        return fac(n);
    };
    for (int i{5}; i != 8; ++i)
        std::cout << i << "! = " << factorial(i) << ";  ";
    std::cout << '\n';
}
```

