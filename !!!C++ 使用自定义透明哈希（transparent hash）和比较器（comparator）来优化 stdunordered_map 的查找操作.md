# 使用自定义透明哈希（transparent hash）和比较器（comparator）来优化 std::unordered_map 的查找操作

```C++
#include <unordered_map>
#include <string>
#include <cstdint>
#include <iostream>

struct string_hash {
    // required to denote a transparent hash
    using is_transparent = void;
    // Hash operations required to be consistent: 
    // a == b => hash(a) == hash(b)
    size_t operator()(const char *txt) const {
        return std::hash<std::string_view>{}(txt);
    }
    size_t operator()(std::string_view txt) const {
        return std::hash<std::string_view>{}(txt);
    }
    size_t operator()(const std::string &txt) const {
        return std::hash<std::string>{}(txt);
    }
};

// Custom key type:
struct Wrapped {
    int64_t value;
    static auto make(int64_t v) { return Wrapped{v}; }
private:
    // Private constructor to demonstrate that this isn't a conversion
    Wrapped(int64_t v) : value(v) {};
};

// Custom transparent hash:
struct wrapped_hash {
    using is_transparent = void;
    size_t operator()(int64_t value) const {
        return std::hash<int64_t>{}(value);
    }
    size_t operator()(const Wrapped& wrapped) const {
        return std::hash<int64_t>{}(wrapped.value);
    }
};

// Custom comparator:
struct wrapped_cmp {
    using is_transparent = void;
    bool operator()(int64_t left, const Wrapped& right) const {
        return left == right.value; 
    }
    bool operator()(const Wrapped& left, const Wrapped& right) const {
        return left.value == right.value; 
    }
};

int main() {
    // Unordered map with a transparent hasher and std::equal_to
    // (since std::string already provides operator==)
    std::unordered_map<std::string, std::string, 
                    string_hash, std::equal_to<>> string_map;
    string_map.insert_or_assign("a", "label_a");
    string_map.insert_or_assign("b", "label_b");

    // No conversion and therefore no allocation here:
    auto i = string_map.find("long_key_that_requires_allocation");
    // i == string_map.end()

    std::cout << std::boolalpha << "(i == string_map.end()) == " << (i == string_map.end()) << "\n";

    // Unordered map with both the hasher and comparator customized
    std::unordered_map<Wrapped, std::string,
                    wrapped_hash, wrapped_cmp> data;
    data.insert_or_assign(Wrapped::make(10), "Hello World!");
    data.insert_or_assign(Wrapped::make(5), "Goodbye!");

    // No conversion, lookup directly using int64_t
    auto j = data.find(5z);
    // j->first == Wrapped{5}, j->second == "Goodbye!"

    std::cout << "j->first.value == " << j->first.value << ", j->second == " << j->second << "\n";
}
```

这段代码展示了 C++ 中如何使用自定义透明哈希（transparent hash）和比较器（comparator）来优化 std::unordered_map 的查找操作，避免不必要的类型转换和内存分配。代码通过两个示例（字符串映射和自定义类型映射）展示了透明操作的优势。以下是逐步解释。

------

代码概览

- 定义透明哈希 string_hash，支持多种字符串类型查找。
- 定义自定义类型 Wrapped，并为其实现透明哈希 wrapped_hash 和比较器 wrapped_cmp。
- 在 main 中使用 std::unordered_map 展示透明查找的效果。

------

关键组件

1. **头文件**

cpp

```cpp
#include <unordered_map>
#include <string>
#include <cstdint>
#include <iostream>
```

- <unordered_map>：提供 std::unordered_map。
- <string>：提供 std::string 和 std::string_view。
- <cstdint>：提供 int64_t。
- <iostream>：用于输出。
- **string_hash 透明哈希**

cpp

```cpp
struct string_hash {
    using is_transparent = void;
    size_t operator()(const char *txt) const {
        return std::hash<std::string_view>{}(txt);
    }
    size_t operator()(std::string_view txt) const {
        return std::hash<std::string_view>{}(txt);
    }
    size_t operator()(const std::string &txt) const {
        return std::hash<std::string>{}(txt);
    }
};
```

- **is_transparent**：
  - 标记为透明哈希，支持异构键查找（heterogeneous lookup）。
- **哈希函数**：
  - 支持 const char*、std::string_view 和 std::string，都委托给标准哈希。
  - 保证一致性：相同内容的键产生相同哈希值。
- **目的**：
  - 允许直接用非 std::string 类型查找，避免构造临时对象。
- **Wrapped 类型**

cpp

```cpp
struct Wrapped {
    int64_t value;
    static auto make(int64_t v) { return Wrapped{v}; }
private:
    Wrapped(int64_t v) : value(v) {};
};
```

- **特性**：
  - 封装 int64_t，私有构造函数防止直接构造。
  - make 静态方法创建实例。
- **目的**：
  - 模拟需要封装的键类型，阻止隐式转换。
- **wrapped_hash 透明哈希**

cpp

```cpp
struct wrapped_hash {
    using is_transparent = void;
    size_t operator()(int64_t value) const {
        return std::hash<int64_t>{}(value);
    }
    size_t operator()(const Wrapped& wrapped) const {
        return std::hash<int64_t>{}(wrapped.value);
    }
};
```

- **is_transparent**：
  - 支持透明查找。
- **哈希函数**：
  - int64_t 和 Wrapped 都基于 value 计算哈希。
- **一致性**：
  - wrapped_hash(5) == wrapped_hash(Wrapped{5})。
- **wrapped_cmp 透明比较器**

cpp

```cpp
struct wrapped_cmp {
    using is_transparent = void;
    bool operator()(int64_t left, const Wrapped& right) const {
        return left == right.value; 
    }
    bool operator()(const Wrapped& left, const Wrapped& right) const {
        return left.value == right.value; 
    }
};
```

- **is_transparent**：
  - 支持异构比较。
- **比较函数**：
  - 支持 int64_t 与 Wrapped、以及 Wrapped 与 Wrapped 的比较。
- **一致性**：
  - 基于 value 的相等性。
- **main 函数**

**字符串映射**

cpp

```cpp
std::unordered_map<std::string, std::string, 
                string_hash, std::equal_to<>> string_map;
string_map.insert_or_assign("a", "label_a");
string_map.insert_or_assign("b", "label_b");
auto i = string_map.find("long_key_that_requires_allocation");
std::cout << std::boolalpha << "(i == string_map.end()) == " << (i == string_map.end()) << "\n";
```

- **string_map**：

  - 键为 std::string，值也为 std::string。
  - 使用 string_hash 和 std::equal_to<>（透明比较器）。

- **插入**：

  - 添加键值对 {"a", "label_a"} 和 {"b", "label_b"}。

- **find**：

  - 用 const char* 查找 "long_key_that_requires_allocation"。
  - 透明哈希和比较器避免构造 std::string，直接使用原始类型。
  - 未找到，返回 end()。

- **输出**：

  ```text
  (i == string_map.end()) == true
  ```

**自定义类型映射**

cpp

```cpp
std::unordered_map<Wrapped, std::string,
                wrapped_hash, wrapped_cmp> data;
data.insert_or_assign(Wrapped::make(10), "Hello World!");
data.insert_or_assign(Wrapped::make(5), "Goodbye!");
auto j = data.find(5z);
std::cout << "j->first.value == " << j->first.value << ", j->second == " << j->second << "\n";
```

- **data**：

  - 键为 Wrapped，值 为 std::string。
  - 使用 wrapped_hash 和 wrapped_cmp。

- **插入**：

  - 添加 {Wrapped{10}, "Hello World!"} 和 {Wrapped{5}, "Goodbye!"}。

- **find**：

  - 用 int64_t（5z）查找。
  - 透明哈希和比较器直接比较 5 和 Wrapped::value，无需构造 Wrapped。
  - 找到 Wrapped{5}。

- **输出**：

  ```text
  j->first.value == 5, j->second == Goodbye!
  ```

------

为什么这样工作？

1. **透明哈希和比较器**：
   - is_transparent 允许容器接受与键类型不同的查找类型。
   - 避免构造临时对象（如 std::string 或 Wrapped）。
2. **一致性**：
   - 哈希和比较器基于相同的值（string_view 或 int64_t），确保查找正确。
3. **性能**：
   - 减少内存分配和类型转换开销。

------

输出

```text
(i == string_map.end()) == true
j->first.value == 5, j->second == Goodbye!
```

------

使用场景

- **性能优化**：
  - 避免不必要的字符串或对象构造。
- **灵活查找**：
  - 使用原始类型（如 const char* 或 int64_t）直接查询。
- **自定义键**：
  - 为封装类型提供透明支持。

------

总结

- string_hash 实现透明字符串查找，避免 std::string 构造。
- wrapped_hash 和 wrapped_cmp 支持 Wrapped 和 int64_t 的直接比较。
- 代码展示了透明哈希和比较器在 unordered_map 中的高效应用。
- 