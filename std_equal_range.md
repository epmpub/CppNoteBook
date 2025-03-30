# *std::equal_range* 

The *std::equal_range* algorithm is a binary search algorithm (*2\*logn* comparisons) that operates on sorted ranges.

It returns the lower and upper bounds for the given value, effectively denoting the range of elements that equal the value.

Same as *std::lower_bound* and *std::upper_bound*, *std::equal_range* still provides *O(logn)* number of comparisons on non-random-access ranges.

```c++
#include <algorithm>
#include <ranges>
#include <vector>

std::vector<int> data{1, 2, 3, 4, 4, 4, 6, 7, 8, 9, 10};

{
auto [low, upp] = std::equal_range(data.begin(), data.end(), 4);
// [low, upp) == {4, 4, 4}
// *low == 4, *upp == 6
}

{
auto [low, upp] = std::equal_range(data.begin(), data.end(), 5);
// low == upp (empty range)
// *low == 6, *upp = 6
}

// Keep in mind that when using the range version with a projection,
// it is the projected range that must be sorted
std::vector<std::string> labels{"x", "aa", "xyz", "abc", "defg"};

{
// find the range of strings of length 3
auto [low, upp] = std::ranges::equal_range(labels, 3, 
    std::less<>{}, // comparator
    // project the strings to their length
    [](auto& s) { return s.length(); });
// [low, upp) == {"xyz", "abc"}
// *low == "xyz", *upp == "defg"
}
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/zzE93Tn5b)