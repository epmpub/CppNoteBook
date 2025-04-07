# Ordered containers: std::(multi_)map, std::(multi_)set

Containers that maintain the elements ordered based on strict-weak ordering: *std::(multi_)map*, *std::(multi_)set*.

The ordered containers *std::(multi_)set* and *std::(multi_)map* are node-based containers that offer *log(n)* operation complexity for lookup, insertion and removal.

As with other node-based containers, we pay for the reference and iterator stability with performance.

Due to a relatively low constant overhead, ordered containers can sometimes outperform unordered containers.

```C++
#include <cstdint>
#include <string>
#include <map>
#include <set>

struct Blob { 
    int a;
    int b;
};

std::map<uint64_t,Blob> data;

// insert new element if key doesn't exist
data.insert(std::make_pair(0z, Blob{10,20}));
// insert if key doesn't exist, update value if key already exists
data.insert_or_assign(1z, Blob{1, 2});
// if key doesn't exist, insert a new element, constructing
// the value in-place from the arguments
data.try_emplace(1z, 1, 2); // 1, 2 used for the value

auto it1 = data.find(0); // lookup by key
// it1->first == 0, it1->second == {10, 20}
auto it2 = data.find(4);
// it2 == data.end()

// iterate over elements in strict-weak ordering
for (auto& [key, value] : data) {}


struct Key {
    uint64_t id;
    std::string label;
    auto operator<=>(const Key&) const = default;
};

// The key type has to support strict-weak ordering
std::set<Key> set{{0, "label1"}, {0, "label2"},
                  {1, "label1"}, {1, "label2"}};

bool check = set.contains({0, "label1"});
// check == true
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/na93K6dse)