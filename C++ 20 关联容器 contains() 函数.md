# C++20 关联容器 contains() 函数

在 C++20 中，标准库的容器并没有直接引入一个通用的 contains() 成员函数。

然而，C++20 确实为某些关联容器（如 std::set、std::map、std::unordered_set 和 std::unordered_map）添加了 contains() 成员函数，用于检查容器中是否包含某个键。这个功能提高了代码的可读性和效率，因为它比传统的 find() != end() 检查更直观且通常更高效。

让我详细解释 C++20 中与 contains() 相关的内容，并说明哪些容器支持它以及如何使用。

------

**支持 contains() 的容器**

C++20 为以下关联容器添加了 contains() 成员函数：

1. **std::set**
2. **std::multiset**
3. **std::unordered_set**
4. **std::unordered_multiset**
5. **std::map**
6. **std::multimap**
7. **std::unordered_map**
8. **std::unordered_multimap**

这些容器是基于键值存储的关联容器，contains() 用于检查某个键是否存在于容器中。

**不支持 contains() 的容器**

- 序列容器（如 std::vector、std::deque、std::list）没有 contains() 成员函数。
- 如果你需要在这些容器中检查元素是否存在，通常需要使用算法（如 std::find）或手动实现。

------

**用法和语法**

contains() 是一个成员函数，接受一个键作为参数，返回一个 bool 值：

- 返回 true，如果键存在于容器中。
- 返回 false，如果键不存在。

**原型**

对于支持的容器，contains() 的典型声明如下（以 std::set 为例）：

cpp

```cpp
bool contains(const Key& key) const;
```

- **Key**: 容器的键类型。
- **const**: 表示这是一个常量成员函数，不会修改容器。

------

**示例代码**

**1. std::set**

cpp

```cpp
#include <iostream>
#include <set>

int main() {
    std::set<int> s = {1, 2, 3, 4};

    std::cout << std::boolalpha; // 打印 true/false 而不是 1/0
    std::cout << "Contains 3: " << s.contains(3) << "\n"; // true
    std::cout << "Contains 5: " << s.contains(5) << "\n"; // false
}
```

- **输出**:

  ```text
  Contains 3: true
  Contains 5: false
  ```

**2. std::map**

cpp

```cpp
#include <iostream>
#include <map>
#include <string>

int main() {
    std::map<std::string, int> m = {{"apple", 1}, {"banana", 2}};

    std::cout << std::boolalpha;
    std::cout << "Contains 'apple': " << m.contains("apple") << "\n"; // true
    std::cout << "Contains 'orange': " << m.contains("orange") << "\n"; // false
}
```

- **输出**:

  ```text
  Contains 'apple': true
  Contains 'orange': false
  ```

**3. std::unordered_set**

cpp

```cpp
#include <iostream>
#include <unordered_set>

int main() {
    std::unordered_set<int> us = {10, 20, 30};

    std::cout << std::boolalpha;
    std::cout << "Contains 20: " << us.contains(20) << "\n"; // true
    std::cout << "Contains 40: " << us.contains(40) << "\n"; // false
}
```

- **输出**:

  ```text
  Contains 20: true
  Contains 40: false
  ```

------

**与传统方法的比较**

在 C++20 之前，检查键是否存在通常使用 find() 或 count()：

- **find()**:

  cpp

  ```cpp
  if (s.find(3) != s.end()) { /* 键存在 */ }
  ```

- **count()**（仅适用于关联容器）:

  cpp

  ```cpp
  if (s.count(3) > 0) { /* 键存在 */ }
  ```

**contains() 的优势**:

1. **可读性**: s.contains(3) 比 s.find(3) != s.end() 更直观。
2. **效率**: 对于某些实现，contains() 可能比 find() 更高效，因为它不需要返回迭代器，只需返回布尔值。
3. **一致性**: 为关联容器提供统一的接口。

------

**为什么序列容器没有 contains()?**

- 序列容器（如 std::vector）存储的是值，而不是键值对，没有“键”的概念。

- 检查元素是否存在需要线性搜索（除非容器已排序并使用二分查找），这与关联容器的 O(1) 或 O(log n) 查找复杂度不同。

- 如果需要类似功能，可以使用标准算法：

  cpp

  ```cpp
  #include <algorithm>
  #include <vector>
  
  int main() {
      std::vector<int> v = {1, 2, 3, 4};
      bool has_3 = std::ranges::contains(v, 3); // C++23 引入
      // 或 C++20 之前: bool has_3 = std::find(v.begin(), v.end(), 3) != v.end();
  }
  ```

  - 注意：std::ranges::contains 是 C++23 引入的，C++20 中没有直接等价物。

------

**实现自定义 contains()**

如果想为不支持 contains() 的容器（如 std::vector）添加类似功能，可以自己实现：

cpp

```cpp
#include <algorithm>
#include <vector>

template <typename Container, typename T>
bool contains(const Container& c, const T& value) {
    return std::find(c.begin(), c.end(), value) != c.end();
}

int main() {
    std::vector<int> v = {1, 2, 3, 4};
    std::cout << std::boolalpha;
    std::cout << "Contains 3: " << contains(v, 3) << "\n"; // true
    std::cout << "Contains 5: " << contains(v, 5) << "\n"; // false
}
```

------

**中文解释**

**功能**

- C++20 为关联容器（如 std::set、std::map 等）添加了 contains() 成员函数，用于检查某个键是否存在。
- 返回 true（存在）或 false（不存在）。

**支持的容器**

- 有序关联容器：std::set, std::map, std::multiset, std::multimap。
- 无序关联容器：std::unordered_set, std::unordered_map, std::unordered_multiset, std::unordered_multimap。

**用法**

- 示例：s.contains(3) 检查 std::set<int> s 中是否包含键 3。
- 比传统的 find() != end() 更简洁高效。

**限制**

- 序列容器（如 std::vector）没有 contains()，需要用算法（如 std::find）或自定义实现。

**意义**

- 提高代码可读性，统一关联容器的接口，减少样板代码。

------

**总结**

C++20 的 contains() 是关联容器的一个实用新增功能，适用于键值存储的场景。对于不支持的容器，可以通过标准算法或自定义函数实现类似功能。如果你有具体问题或需要更多示例，请告诉我！