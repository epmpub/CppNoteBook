# C++ *std::make_heap*, *std::push_heap*, *std::pop_heap* and std::sort_heap

The set of heap algorithms: *std::make_heap*, *std::push_heap*, *std::pop_heap* and *std::sort_heap* can be used as a replacement for *std::priority_queue*  and *std::set* when it is desirable to keep the elements in a contiguous storage or when we require cheap element extraction.

std::priority_queue ??



For the benefits, we pay with a more error-prone interface.

```
#include <vector>
#include <algorithm>
#include <string>

std::vector<int> data{8,2,1,7,4,5,3,6,9};
// initialize max-heap
auto begin = data.begin(), end = data.end();
std::make_heap(begin, end);

// pop each element from the heap producing
// a sorted order in the vector
while (begin != end) {
    // pop_heap swaps the max element with 
    // the last element in range, maintaining the heap
    std::pop_heap(begin, end--);
    // iterate over 9,8,...
}
// data == {1, 2, 3, 4, 5, 6, 7, 8, 9}


std::vector<std::string> labels{"world","bye","fox","lazy","dog"};
std::make_heap(labels.begin(), labels.end());
// labels now in heap order

// extract element from heap, without removing it from the vector
std::pop_heap(labels.begin(), labels.end());
// in-place modify
labels.back()[0] = 'e';
// insert it back into the heap
std::push_heap(labels.begin(), labels.end());

std::sort_heap(labels.begin(), labels.end());
// labels == {"bye", "dog", "eorld", "fox", "lazy"}
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/rbdG63asn)