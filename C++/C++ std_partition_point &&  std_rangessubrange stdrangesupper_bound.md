



# std::partition_point &&  std::ranges::subrange std::ranges::upper_bound

The std::lower_bound and std::upper_bound are arguably the two most practically useful algorithms in the standard library.

Both algorithms are binary searches, operating in O(logn) on sorted ranges.

The std::lower_bound returns an iterator to the first element not ordered before the provided value and std::upper_bound to the first element ordered after the provided value.



```c++
#include <algorithm>
#include <vector>
#include <print>

int main() {
    std::vector<int> data{1,2,3,4,5,5,5,6,7,8,9};

    // First element for which el < value == false
    auto lb = std::lower_bound(data.begin(), data.end(), 5);
    // *lb == 5

    std::println("*lb == {}", *lb);
    
    // same as:
    lb = std::partition_point(data.begin(), data.end(), [](int el) {
        return el < 5;
    });
    // *lb == 5

    std::println("*lb == {}", *lb);

    // First element for which value < el == true
    auto ub = std::ranges::upper_bound(data, 5); // range version
    // *ub == 6

    std::println("*ub == {}", *ub);

    // same as:
    ub = std::ranges::partition_point(data, [](int el) {
        return not (5 < el);
    });
    // *ub == 6

    std::println("*ub == {}", *ub);

    // [begin, lb) forms the subrange of elements lower than value
    auto lower = std::ranges::subrange(data.begin(), lb);
    // lower == {1, 2, 3, 4}

    std::println("lower == {}", lower);

    // [lb, ub) forms the subrange of elements equal to the value
    auto equal = std::ranges::subrange(lb, ub);
    // equal == {5, 5, 5}

    std::println("equal == {}", equal);

    // [ub, end) forms the subrange of elements higher than the value
    auto high = std::ranges::subrange(ub, data.end());
    // high == {6, 7, 8, 9}

    std::println("high == {}", high);

    std::vector<std::string> strs{"a","ab","fge","cdefgh"};

    // Find the first string that is at least 2 characters long
    auto el = std::ranges::lower_bound(strs, 2,
        std::less<>{},              // Custom comparator
        [](const std::string& el) { // Projection
            return el.length();
        });
    // *el == "ab"

    std::println("*el == {}", *el);
}
```

