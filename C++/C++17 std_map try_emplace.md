# std::map::try_emplace

`` 是 C++17 引入的一个成员函数，用于向 `std::map` 容器中有条件地插入元素。它的设计目的是提供一种更高效的方式来实现"如果键不存在则插入"的操作。

`try_emplace` 的主要特点：

1. 如果键已经存在于映射中，则不会执行任何构造，也不会修改现有值。
2. 如果键不存在，则会在适当的位置原地构造新元素，避免不必要的临时对象创建。

基本语法：

```cpp
template<class... Args>
pair<iterator, bool> try_emplace(const key_type& k, Args&&... args);

template<class... Args>
pair<iterator, bool> try_emplace(key_type&& k, Args&&... args);

template<class... Args>
iterator try_emplace(const_iterator hint, const key_type& k, Args&&... args);

template<class... Args>
iterator try_emplace(const_iterator hint, key_type&& k, Args&&... args);
```

与 `insert` 或 `emplace` 相比的优势：

- `try_emplace` 保证只有在需要时才会构造值对象
- 键参数不会被转发到值构造函数中，而是被单独处理
- 对于已经存在的键，不会创建任何临时对象

简单示例：

```cpp
#include <iostream>
#include <map>
#include <string>

int main() {
    std::map<int, std::string> m;
    
    // 键1不存在，插入成功
    auto [it1, inserted1] = m.try_emplace(1, "one");
    std::cout << "1: " << inserted1 << ", " << it1->second << std::endl;
    
    // 键1已存在，不会插入
    auto [it2, inserted2] = m.try_emplace(1, "another one");
    std::cout << "2: " << inserted2 << ", " << it2->second << std::endl;
    
    return 0;
}
```

输出：

```
1: 1, one
2: 0, one
```

这个函数特别适合用于那些构造成本较高的值类型，或者当你希望避免不必要构造时使用。



对于 `std::map` 中已经存在的键，你有多种方法可以更新其对应的值：

1. 使用下标操作符 `[]`：

```cpp
std::map<int, std::string> m;
m[1] = "one";
// 更新已存在键的值
m[1] = "updated one";
```

1. 使用 `at()` 方法：

```cpp
std::map<int, std::string> m;
m[1] = "one";
// 更新已存在键的值
m.at(1) = "updated one";
```

1. 使用 `find()` 方法查找迭代器后更新：

```cpp
std::map<int, std::string> m;
m[1] = "one";
// 查找并更新
auto it = m.find(1);
if (it != m.end()) {
    it->second = "updated one";
}
```

1. 使用 `insert_or_assign()` 方法（C++17引入）：

```cpp
std::map<int, std::string> m;
m[1] = "one";
// 不管键是否存在，都会赋新值
m.insert_or_assign(1, "updated one");
```

1. 先检查是否存在，然后有条件地更新：

```cpp
std::map<int, std::string> m;
m[1] = "one";
// 条件更新
if (m.count(1) > 0) {
    m[1] = "updated one";
}
```

不同方法的特点：

- `[]` 操作符简单直接，但如果键不存在会创建新元素
- `at()` 如果键不存在会抛出 `std::out_of_range` 异常
- `find()` 最灵活，可以进行更复杂的条件判断
- `insert_or_assign()` 专为"插入或更新"场景设计，返回操作结果信息
- `count()` 配合其他操作可实现明确的条件更新

通常，如果你确定键存在，`[]` 操作符是最简洁的方式。如果需要处理键可能不存在的情况，使用 `find()` 或 `insert_or_assign()` 会更合适。