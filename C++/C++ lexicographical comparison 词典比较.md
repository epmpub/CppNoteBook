# lexicographical comparison 词典比较

Standard C++ **containers** provide lexicographical comparison through the standard set of comparison operators (before C++20) and the three-way comparison operator (since C++20).

The support covers:

 *std::array*, *std::vector*, *std::deque*, *std::(forward_)list*, *std::string* (and variants), *std::(multi)set*, *std::(multi)map*, *std::stack* and *std::queue*.

```C++
#include <vector>
#include <string>
#include <cassert>
 
std::vector<int> data1{1, 3, 4, 5};
std::vector<int> data2{1, 3, 6, 7};
std::vector<int> data3{1, 3};

assert(data1 < data2);
assert(data3 < data1);

std::u8string str1 = u8"😀😄😁";
std::u8string str2 = u8"😁😁😁";
std::u8string str3 = u8"😁";

assert(str1 < str2);
assert(str3 < str2);
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/Tar5TTKna)