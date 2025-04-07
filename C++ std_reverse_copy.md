

# *std::reverse_copy* 

The *std::reverse_copy* is a simple algorithm that copies the source bidirectional range in reverse order into the output range.

Typically, *std::reverse_copy* offers a more natural solution than using *std::copy* with reverse iterators.

```C++
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> data{1,2,3,4,5,6,7,8,9};
    std::vector<int> out;
    
    std::reverse_copy(data.begin(), data.end(),
        std::back_inserter(out));
    // out == {9, 8, 7, 6, 5, 4, 3, 2, 1}

    for (auto v : out)
        std::cout << v << " ";
    std::cout << "\n";

    out.clear();
    // same as:
    std::copy(data.rbegin(), data.rend(), 
        std::back_inserter(out));
    // out == {9, 8, 7, 6, 5, 4, 3, 2, 1}
    
    for (auto v : out)
        std::cout << v << " ";
    std::cout << "\n";
}
```

