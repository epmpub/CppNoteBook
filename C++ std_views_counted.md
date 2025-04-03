# *std::views::counted* 

The *std::views::counted* is the C++20 range equivalent to the *std::counted_iterator*, producing a range from an iterator and the number of elements.

Similar to *std::views::take*, this view also handles contiguous and random-access-sized ranges efficiently.

For contiguous ranges, it produces a *std::span*; for random-access-sized ranges, it resolves the end iterator.



```c++
#include <ranges>
#include <algorithm>
#include <vector>
#include <iostream>
#include <queue>

int main() {
    std::vector<int> data{1,2,3,4,5};

    auto c = std::views::counted(data.begin(), 3);
    // decltype(c) == std::span<int>
    // c == {1,2,3}

    static_assert(std::is_same_v<decltype(c), std::span<int>>);
    for (auto v : c)
        std::cout << v << " ";
    std::cout << "\n";

    std::deque<int> q{1,2,3,4,5};
    auto random = std::views::counted(q.begin(), 3);
    // random == {1,2,3}
    // *random.end() == 4, O(1) operation

    std::cout << "*random.end() == " << *random.end() << "\n";
    for (auto v : random)
        std::cout << v << " ";
    std::cout << "\n";

    // More involed example:

    // Preallocate space for three elements.
    std::vector<int> top_three(3);  
    // Produce the lowest three elements in top_three.
    std::ranges::partial_sort_copy( 
        // From upto 5 integers read from the standard input.
        std::views::counted( 
            std::istream_iterator<int>(std::cin), 5),
        top_three
    );
    // For input: 13 97 42 7 666 1
    // top_three == {7, 13, 42}

    for (auto v : top_three)
        std::cout << v << " ";
    std::cout << "\n";
}
```

