# *std::next* and *std::prev*

*std::next* and *std::prev*  are C++11 iterator utilities that return the succeeding or preceding iterator.

If the provided iterator models random access,

 the operation will be constant, even if a custom distance is specified.

```c++
#include <vector>
#include <list>
#include <iterator>

std::vector<int> data{1, 2, 3, 4, 5, 6, 7};

// std::prev requires bidirectional iterator
auto it1 = std::prev(data.end());
// *it1 == 7

// distance can be customized
auto it2 = std::next(data.begin(), 3);
// *it2 == 4

std::list<int> lst{1, 2, 3, 4, 5, 6, 7};

// for non-random-access iterators the operation is linear
auto it3 = std::prev(lst.end(), 4);
// *it3 == 4
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/bKa9aff3h)