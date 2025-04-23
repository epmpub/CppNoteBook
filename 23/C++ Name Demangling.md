# C++ Name Demangling

嵌套的类型,用typeid打印出来,会很乱,使用了GNU cxxabi.h 文件提供的__cxa_demangle函数可以修复这个问题.

![](D:\0.products\myBook\23\Name demangling.png)



```C++
#include <cxxabi.h>
#include <vector>
#include <list>
#include <string>
#include <iostream>
#include <typeinfo>
#include <memory>
#include <utility>


struct MyClass
{
    std::vector<std::vector<std::list<std::pair<std::string,std::string>>>> m_data;
};

std::string demangle(const char* name)
{
    int status = -4;
    std::unique_ptr<char, void(*)(void*)> res{ abi::__cxa_demangle(name, NULL, NULL, &status), std::free };
    return (status == 0) ? res.get() : name;
}

int main()
{
    MyClass o;
    std::cout << demangle(typeid(&MyClass::m_data).name()) << '\n';
    return 0;
}

```

