# *std::sort* ,std::ranges::sort

The *std::sort* is perhaps one of the most well-known algorithms.

The algorithm sorts elements (by default in non-descending order) and doesn’t maintain the relative order of equivalent elements.

C++17 standard added a parallel variant.
The C++20 standard added a range version and enabled constexpr evaluation for all but the parallel variant.

```c++
#include <algorithm>
#include <vector>
#include <execution>


std::vector<int> data{9,2,6,4,3,5,1,7,8};

// Default sort using operator<
std::sort(data.begin(), data.end());
// data == {1,2,3,4,5,6,7,8,9}

// Sort with custom comparator
std::sort(data.begin(), data.end(), std::greater<>{});
// data == {9,8,7,6,5,4,3,2,1}

// Same as above, but using a lambda
std::sort(data.begin(), data.end(), 
          [](int l, int r) { return l > r; });
// data == {9,8,7,6,5,4,3,2,1}

// C++17 parallel version
std::sort(std::execution::par_unseq, data.begin(), data.end());
// data == {1,2,3,4,5,6,7,8,9}

// C++20 range version
// Note: default comparator is std::ranges::less{}, not operator<
std::ranges::sort(data, std::ranges::greater{});
// data == {9,8,7,6,5,4,3,2,1}

struct Item {
    int id;
    std::string label;
};
std::vector<Item> words{{9, "bench"}, {2, "rain"}, {0, "reasonable"},
                        {5, "fool"}, {3, "waist"}, {7, "insure"}};

// Sort lexicographically by label.
std::ranges::sort(words, {}, &Item::label);
// std::ranges::sort(words, std::ranges::less{}, &Item::label);

// {9:"bench", 5:"fool", 7:"insure", 
//  2:"rain", 0:"reasonable", 3:"waist"}

// Sort by id, non-ascending
std::ranges::sort(words, std::ranges::greater{}, &Item::id);
// {9:"bench", 7:"insure", 5:"fool", 
//  3:"waist", 2:"rain", 0:"reasonable"}
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/Y9fWYdG9a)