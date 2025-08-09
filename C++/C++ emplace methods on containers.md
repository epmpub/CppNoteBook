# emplace methods on containers 

The C++11 emplace methods on containers that can construct elements in place.

In C++11, all containers received emplace variants of their typical insert/push methods.

The emplace variants can construct the element in place, saving a move or copy.

```C++
#include <vector>
#include <string>

std::vector<std::string> vec;
{
    std::string s("Hello World!");
    vec.push_back(s); // Copy
    vec.push_back(std::move(s)); // Move
}
{
    std::string s("Hello World!");
    vec.emplace_back(s); // Copy (same as push_back)
    vec.emplace_back(std::move(s)); // Move (same as push_back)
    // In-place construction, no move or copy:
    vec.emplace_back("Hello World!");
    // Note the difference, this is still a move:
    vec.emplace_back(std::string{"Hello World!"});
}
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/hz3c8jdMa)