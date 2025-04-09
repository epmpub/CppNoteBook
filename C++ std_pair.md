# std::pair

The *std::pair* is a simple heterogeneous container for storing two elements, conceptually similar to std::tuple.

*std::pair* supports the *std::tuple* interface: *std::get*, *std::tuple_size*, *std::tuple_element*.

If you care about performance and work with trivially copyable types, consider an aggregate instead because *std::pair* isn’t trivially copyable.



```C++

#include <utility>
#include <functional>

int main() {
    std::pair<int,double> v{7, 4.2};
    auto [a, b] = v; // deconstruct using structured binding
    // a == 7, b == 4.2

    // Pair interface:
    v.first = 0;
    v.second = 3.14;
    // v == {0, 3.14}

    // Tuple interface:
    std::get<0>(v) = 7;
    std::get<1>(v) = 4.2;
    // v == {7, 4.2}
    // std::tuple_size_v<decltype(v)> == 2
    // std::tuple_element_t<0, decltype(v)> == int
    // std::tuple_element_t<1, decltype(v)> == double

    static_assert(std::tuple_size_v<decltype(v)> == 2);
    static_assert(std::is_same_v<
        std::tuple_element_t<0, decltype(v)>, int>);
    static_assert(std::is_same_v<
        std::tuple_element_t<1, decltype(v)>, double>);

    // Not trivially copyable
    static_assert(not std::is_trivially_copyable_v<decltype(v)>);

    // Potential alternative
    struct simple_pair {
        int first;
        double second;
    } w{7, 42};
    static_assert(std::is_trivially_copyable_v<simple_pair>);

    // std::pair can be constructed using std::make_pair
    // the type is deduced using std::decay, 
    // except for std::reference_wrapper
    int arr[3] = {0,1,2};
    std::pair p1 = std::make_pair(arr, arr);
    // decltype(p1) == std::pair<int*,int*>

    static_assert(std::is_same_v<
        decltype(p1), std::pair<int*,int*>>);

    int x{7};
    std::pair p2 = std::make_pair(x, std::reference_wrapper(x));
    // decltype(p2) == std::pair<int, int&>

    static_assert(std::is_same_v<
        decltype(p2), std::pair<int,int&>>);

    // Note that CTAD guides do not have special handling 
    // for std::reference_wrapper
    std::pair p3{x, std::reference_wrapper(x)};
    // decltype(p3) == std::pair<int, std::reference_wrapper<int>>

    static_assert(std::is_same_v<
        decltype(p3), std::pair<int,std::reference_wrapper<int>>>);
}
```

