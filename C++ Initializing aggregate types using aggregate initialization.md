#  Initializing aggregate types using aggregate initialization.

Aggregate types can be initialized using special aggregate initialization.

This initializes members in their declaration order.

Members that are not explicitly initialized and do not have a default member initializer are initialized using empty copy-list-initialization (i.e. T x={}).

```C++
#include <string>
#include <vector>

struct Data {
    int x;  
    double y;
    std::string label = "Hello World!"; // only permitted since C++14 
    std::vector<int> arr;
};

struct X {
    int a;
    int b;
};

struct Y {
    X x;
    X y;
};

// Initialization is done in declaration order:
Data a = {10, 2.3};
// a.x == 10, a.y == 2.3
// a.label == "Hello World!", a.arr == std::vector<int>{}

// Nested brackets can be omitted:
Y b = { 10, 20, 30 };
// b.x == {10, 20}, b.y == {30, int{} == 0}
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/c8bh3sr9E)