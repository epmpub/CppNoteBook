

# std::string(_view)::starts_with, std::string(_view)::ends_with

C++20 added prefix and suffix checking methods: *starts_with* and *ends_with* to both *std::string* and *std::string_view*.



```c++
#include <string>
#include <string_view>
#include <iostream>

int main() {
    std::string str("the quick brown fox jumps over the lazy dog");
    bool t1 = str.starts_with("the quick"); // const char* overload
    // t1 == true
    bool t2 = str.ends_with('g'); // char overload
    // t2 == true

    std::cout << std::boolalpha << "t1 == " << t1 << "\n";
    std::cout << std::boolalpha << "t2 == " << t2 << "\n";

    std::string_view needle = "lazy dog";
    bool t3 = str.ends_with(needle); // string_view overload
    // t3 == true

    std::cout << std::boolalpha << "t3 == " << t3 << "\n";

    std::string_view haystack = "you are a lazy cat";
    // both starts_with and ends_with also available for string_view
    bool t4 = haystack.ends_with(needle);
    // t4 == false

    std::cout << std::boolalpha << "t4 == " << t4 << "\n";
}
```

