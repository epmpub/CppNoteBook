

# std::unordered_(multi)map, std::unordered_(multi)set std::hash

```C++
#include <unordered_map>
#include <unordered_set>
#include <cstdint>
#include <string>
#include <print>

// Custom keys need to provide a hash specialization.
struct Key {
    uint64_t id;
    std::string label;
    // Besides a hash specialization, we need to provide operator==
    friend bool operator==(const Key&, const Key&) = default;
};

// Specialization of hash for a custom type:
// 定义并哈希自定义键类型 Key


template <> struct std::hash<Key> {
    std::size_t operator()(const Key& key) const noexcept {
        std::size_t h1 = std::hash<uint64_t>{}(key.id);
        std::size_t h2 = std::hash<std::string>{}(key.label);
        return h1 ^ (h2 << 1); // or use boost::hash_combine
    }
};

int main() {
    std::unordered_map<uint64_t, std::string> data;

    // insert new element if key doesn't exist
    data.insert(std::make_pair(UINT64_C(0), std::string("dog")));
    // data == {{0, "dog"}}
    std::println("data == {}", data);
    
    // C++17: insert if key doesn't exist, 
    // update value if key already exists
    data.insert_or_assign(UINT64_C(1), std::string("cat"));
    // data == {{0, "dog"}, {1, "cat"}}
    std::println("data == {}", data);

    // C++17: if key doesn't exist, insert a new element,
    // constructing the value in-place from the arguments
    data.try_emplace(UINT64_C(1), "monkey"); // 1, 2 used for the value
    // data == {{0, "dog"}, {1, "cat"}}
    std::println("data == {}", data);

    auto it1 = data.find(0); // lookup by key
    // it1->first == 0, it1->second == "dog"
    std::println("it1->first == {}, it1->second == {}", it1->first, it1->second);

    auto it2 = data.find(4);
    // it2 == data.end()
    std::println("(it2 == data.end()) == {}", it2 == data.end());

    // iterate over elements in unspecified order
    for (auto& [key, value] : data) {
        std::println("key == {}, value == {}", key, value);
    }

    std::unordered_set<Key> set{{0, "label1"}, {0, "label2"},
                                {1, "label1"}, {1, "label2"}};
    bool check = set.contains({0, "label1"});
    // check == true
    std::println("check == {}", check);
}
```

这段代码展示了 C++ 中 std::unordered_map 和 std::unordered_set 的使用，包括自定义键类型的哈希特化、插入操作、查找和迭代等功能。以下是对代码的详细解释：

------

**代码结构**

1. **头文件**：
   - <unordered_map>：提供无序映射容器。
   - <unordered_set>：提供无序集合容器。
   - <cstdint>：定义整数类型（如 uint64_t）。
   - <string>：提供 std::string。
   - <print>：C++23 的格式化输出工具（std::println）。
2. **主要内容**：
   - 定义并哈希自定义键类型 Key。
   - 使用 std::unordered_map 进行键值对操作。
   - 使用 std::unordered_set 存储自定义键。

------

**自定义键类型和哈希特化**

cpp

```cpp
struct Key {
    uint64_t id;
    std::string label;
    friend bool operator==(const Key&, const Key&) = default;
};

template <> struct std::hash<Key> {
    std::size_t operator()(const Key& key) const noexcept {
        std::size_t h1 = std::hash<uint64_t>{}(key.id);
        std::size_t h2 = std::hash<std::string>{}(key.label);
        return h1 ^ (h2 << 1); // or use boost::hash_combine
    }
};
```

**逐步解释**

1. **Key 结构体**：
   - 定义一个自定义类型，包含 id（uint64_t）和 label（std::string）。
   - **operator==**：
     - 使用 = default 自动生成相等比较函数。
     - 必需的，因为 std::unordered_map 和 std::unordered_set 需要键支持相等性比较。
2. **std::hash<Key> 特化**：
   - std::unordered_map 和 std::unordered_set 使用哈希表，需要为自定义键提供哈希函数。
   - 特化 std::hash 模板：
     - h1：对 key.id 的哈希值。
     - h2：对 key.label 的哈希值。
     - 组合：h1 ^ (h2 << 1)，使用异或和左移操作混合两个哈希值。
     - noexcept：保证哈希函数不抛异常，符合标准要求。
   - 注释提到 boost::hash_combine 是一种更健壮的替代方案，但这里使用简单实现。

------

**主函数：std::unordered_map 操作**

cpp

```cpp
int main() {
    std::unordered_map<uint64_t, std::string> data;

    // insert new element if key doesn't exist
    data.insert(std::make_pair(UINT64_C(0), std::string("dog")));
    std::println("data == {}", data);
```

**1. 插入元素**

- **data.insert**：
  - 使用 std::make_pair 创建键值对 {0, "dog"}。
  - UINT64_C(0) 是 C 风格宏，确保 0 为 uint64_t 类型。
  - 如果键 0 不存在，则插入；如果存在，则忽略。
- **结果**：data == {{0, "dog"}}。
- **std::println**：格式化输出 data，显示键值对。

------

cpp

```cpp
data.insert_or_assign(UINT64_C(1), std::string("cat"));
std::println("data == {}", data);
```

**2. 插入或赋值（C++17）**

- **data.insert_or_assign**：
  - C++17 引入。
  - 如果键 1 不存在，则插入 {1, "cat"}；如果存在，则覆盖值为 "cat"。
- **结果**：data == {{0, "dog"}, {1, "cat"}}。

------

cpp

```cpp
data.try_emplace(UINT64_C(1), "monkey");
std::println("data == {}", data);
```

**3. 尝试插入（C++17）**

- **data.try_emplace**：
  - C++17 引入。
  - 如果键 1 不存在，则原地构造值 "monkey" 并插入；如果存在，则不修改。
  - 与 insert 的区别：避免不必要的对象拷贝或移动，直接构造。
- **结果**：键 1 已存在（值为 "cat"），插入失败，data 不变，仍为 {{0, "dog"}, {1, "cat"}}。

------

cpp

```cpp
auto it1 = data.find(0);
std::println("it1->first == {}, it1->second == {}", it1->first, it1->second);

auto it2 = data.find(4);
std::println("(it2 == data.end()) == {}", it2 == data.end());
```

**4. 查找元素**

- **data.find(0)**：
  - 查找键 0，返回迭代器 it1。
  - it1->first == 0，it1->second == "dog"。
- **data.find(4)**：
  - 查找不存在的键 4，返回 data.end()。
  - it2 == data.end() 为 true。

------

cpp

```cpp
for (auto& [key, value] : data) {
    std::println("key == {}, value == {}", key, value);
}
```

**5. 迭代**

- **结构化绑定（C++17）**：

  - auto& [key, value] 解构每个键值对。

- **无序性**：

  - std::unordered_map 不保证顺序，输出可能是：

    ```text
    key == 0, value == dog
    key == 1, value == cat
    ```

    或相反顺序。

------

**std::unordered_set 操作**

cpp

```cpp
std::unordered_set<Key> set{{0, "label1"}, {0, "label2"},
                            {1, "label1"}, {1, "label2"}};
bool check = set.contains({0, "label1"});
std::println("check == {}", check);
```

**逐步解释**

1. **std::unordered_set<Key>**：
   - 使用自定义 Key 类型，依赖 operator== 和 std::hash<Key>。
   - 初始化列表：
     - {0, "label1"}
     - {0, "label2"}
     - {1, "label1"}
     - {1, "label2"}
   - 集合中存储 4 个唯一元素（Key 的相等性由 id 和 label 共同决定）。
2. **set.contains**：
   - C++20 引入，检查 {0, "label1"} 是否存在。
   - 返回 true，因为集合中包含此键。
3. **输出**：check == true。

------

**关键技术点**

1. **自定义哈希**：
   - std::unordered_map 和 std::unordered_set 需要键类型提供哈希和相等性支持。
   - 这里通过特化 std::hash<Key> 实现。
2. **插入方法**：
   - insert：仅插入，不覆盖。
   - insert_or_assign：插入或覆盖。
   - try_emplace：高效插入，避免拷贝。
3. **无序容器特性**：
   - 平均 O(1) 的查找和插入时间。
   - 不保证元素顺序。
4. **C++17/20/23 特性**：
   - insert_or_assign 和 try_emplace（C++17）。
   - contains（C++20）。
   - std::println（C++23）。

------

**输出总结**

```text
data == {0: dog}
data == {0: dog, 1: cat}
data == {0: dog, 1: cat}
it1->first == 0, it1->second == dog
(it2 == data.end()) == true
key == 0, value == dog
key == 1, value == cat
check == true
```

------

**总结**

- **自定义键**：展示了如何为 std::unordered_map/set 定义哈希和相等性。
- **操作**：展示了插入、查找和迭代的多种方式。
- **现代特性**：利用了 C++17/20/23 的新功能，代码简洁高效。

如果你有具体问题（例如哈希函数的优化），欢迎进一步提问！