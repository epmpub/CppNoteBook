

# std::views::chunk 

The C++23 std::views::chunk produces a view of sub-ranges spanning the provided number of elements each.

If the source range cannot be divided without a remainder, the last element in the range will be a shorter subrange that contains the remainder of the elements.

The view works for input ranges and will provide additional features based on the type of the underlying range.

```c++
#include <vector>
#include <ranges>
#include <iostream>
#include <utility>

void print_range(auto&& rng);

int main() {
    std::vector<int> data{1,2,3,4,5,6,7,8,9};

    // Iterate over chunks of size 4 (and one chunk of size 1)
    for (const auto& chunk : data | std::views::chunk(4)) {
        print_range(chunk);
    }
    std::cout << '\n';

    auto view = data | std::views::chunk(4);
    auto second = view[1]; // OK
    // operator[] provided if source range is random access
    // second == {5,6,7,8}

    print_range(second);
    std::cout << '\n';

    auto last = view.back(); // OK
    // back() provided if source range is bidirectional
    // last == {9}

    print_range(last);
    std::cout << '\n';

    std::vector<int> cube(27,0);
    // Can be used to iterate multi-dimensional data stored as 1D
    for (const auto &slice : cube | std::views::chunk(3*3)) {
        for (const auto &row : slice | std::views::chunk(3)) {
            print_range(row);
        }
        std::cout << '\n';
    }
}

void print_range(auto&& rng) {
    std::cout << "{";
    std::string delim;
    for (int v : rng)
        std::cout << std::exchange(delim,", ") << v;
    std::cout << "}\n";
}
```

