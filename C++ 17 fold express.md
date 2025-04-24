

# C++ 17  fold expressions 

```C++

#include <iostream>
#include <functional>
#include <string>
#include <type_traits>
#include <tuple>
#include <iostream>

// Variadic template lambda function to handle multiple arguments
int main()
{
    const auto f = [](const std::string &arg1,auto &&...arg2 ) {
        std::cout << arg1;
        ((std::cout << " " << arg2), ...);
        std::cout << std::endl;
    };

    f("Hello", 2,3,4,5,6);

    return 0;
}

```



```C++
// bind.cpp : This file contains the 'main' function. Program execution begins and ends there.
//

#include <iostream>

// Variadic template lambda function to handle multiple arguments
int main()
{
    const auto f = [](auto &&... args) {
        return (args + ...); // Fold expression to sum all arguments
    };

    std::cout << f(1, 2, 3, 4) << std::endl;

    return 0;
}

```

