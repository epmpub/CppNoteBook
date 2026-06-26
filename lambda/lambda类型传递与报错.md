



```cpp
#include <iostream>
#include <vector>
#include <string>

int main()
{   
    std::vector<int> vi { 0, 1, 2, 4, 5, 6 };
    std::vector<std::string> vs { "Hello", "World" };
    
    auto foo = []<typename T>(std::vector<T> const& vec) { 
        std::cout<< std::size(vec) << '\n';
        std::cout<< vec.capacity() << '\n';
    };
    foo(vi);
    foo(vs);
    foo(10);

    //
    // struct
    // {
    //  template<typename T>
    //    auto operator()(std::vector<T> const& t) { .. };
    // }
        
}
```

```cpp
NICE ERROR MESSAGE:
--------------------------------------------------------------
template argument deduction/substitution failed:
        •   mismatched types 'const std::vector<T>' and 'int'
```

