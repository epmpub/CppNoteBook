# std::type_index *boost::type_index*

```c++
std::unordered_map<std::type_index, std::string> names

```



这段代码定义了一个 std::unordered_map，其中键是 std::type_index，值是 std::string。它可以用来存储类型信息（通过 std::type_index 表示）与字符串（例如类型名称）的映射关系。以下是对其详细解释。

------

代码

cpp

```cpp
std::unordered_map<std::type_index, std::string> names;
```

------

逐步解释

**1. std::unordered_map**

- **std::unordered_map** 是 C++ 标准库中的无序关联容器，定义在 <unordered_map> 头文件中。
- 特点：
  - 使用哈希表实现，键值对无序存储。
  - 平均时间复杂度为 O(1)（插入、查找、删除）。
- 模板参数：
  - Key: 键类型，这里是 std::type_index。
  - T: 值类型，这里是 std::string。
  - 默认提供哈希函数和比较器（std::hash 和 std::equal_to）。

**2. std::type_index**

- **std::type_index** 是 C++11 引入的类，定义在 <typeindex> 头文件中。
- 作用：
  - 包装 std::type_info 对象（由 typeid 运算符返回），提供类型标识的比较和哈希功能。
  - 用于运行时类型信息（RTTI）。
- 特点：
  - 可拷贝、可比较（==, < 等）。
  - 支持哈希（std::hash<std::type_index> 已定义）。
- 构造：
  - 通常通过 typeid(T) 获取，例如 std::type_index(typeid(int))。

**3. std::string**

- **std::string** 是标准字符串类，定义在 <string> 头文件中。
- 作用：
  - 存储键（类型）对应的值，例如类型名称或描述。

**4. names**

- **names** 是一个 std::unordered_map<std::type_index, std::string> 类型的变量。
- 初始状态：
  - 空映射，无键值对。
- 用途：
  - 可以将类型信息（通过 std::type_index 表示）映射到字符串。

------

示例用法

以下是一个完整的示例，展示如何使用这个映射：

cpp

```cpp
#include <unordered_map>
#include <typeindex>
#include <string>
#include <iostream>

int main() {
    std::unordered_map<std::type_index, std::string> names;

    // 插入类型和名称的映射
    names[std::type_index(typeid(int))] = "Integer";
    names[std::type_index(typeid(double))] = "Double";
    names[std::type_index(typeid(std::string))] = "String";

    // 查询类型名称
    std::cout << "int: " << names[std::type_index(typeid(int))] << "\n";
    std::cout << "double: " << names[std::type_index(typeid(double))] << "\n";
    std::cout << "string: " << names[std::type_index(typeid(std::string))] << "\n";

    // 检查未定义的类型
    auto it = names.find(std::type_index(typeid(char)));
    if (it == names.end()) {
        std::cout << "char: Not found\n";
    }

    return 0;
}
```

输出

```text
int: Integer
double: Double
string: String
char: Not found
```

------

工作原理

1. **std::type_index 的生成**：
   - typeid(T) 返回 std::type_info 对象，std::type_index 包装它。
   - 每个类型（包括基本类型和用户定义类型）有唯一的 type_info。
2. **哈希和比较**：
   - std::unordered_map 使用 std::hash<std::type_index> 计算键的哈希值。
   - 使用 == 比较 std::type_index 对象。
3. **存储**：
   - 键值对存储在哈希表中，键是 std::type_index，值是 std::string。

------

时间复杂度

- **插入/查找/删除**：
  - 平均：O(1)（哈希表特性）。
  - 最坏：O(n)（哈希冲突时，n 是元素数量）。
- **typeid**：
  - 通常 O(1)，依赖编译器实现。

------

使用场景

1. **类型注册**：

   - 将类型映射到人类可读的名称，用于调试或日志。

   cpp

   ```cpp
   names[std::type_index(typeid(MyClass))] = "MyClass";
   ```

2. **运行时类型处理**：

   - 在需要 RTTI 的场景中存储类型信息。

3. **工厂模式**：

   - 映射类型到描述或创建函数。

------

注意事项

1. **头文件**：
   - 需要 <unordered_map> 和 <typeindex>。
2. **typeid 限制**：
   - 需要启用 RTTI（编译器默认开启，若关闭则不可用）。
   - 对于多态类型，返回动态类型；非多态类型，返回静态类型。
3. **无序性**：
   - 键值对顺序不可预测。
4. **默认构造**：
   - names 默认构造为空。

------

与 std::map 的对比

| 特性       | std::unordered_map   | std::map     |
| ---------- | -------------------- | ------------ |
| 底层实现   | 哈希表               | 红黑树       |
| 键序       | 无序                 | 有序（按 <） |
| 查找复杂度 | 平均 O(1)，最坏 O(n) | O(log n)     |
| 适用场景   | 快速查找             | 需要排序     |

若需要类型按名称排序，可用 std::map<std::type_index, std::string>。

------

总结

std::unordered_map<std::type_index, std::string> names：

- 定义了一个无序映射，将类型（std::type_index）关联到字符串（std::string）。
- 使用 typeid 生成键，适合存储类型信息。
- 提供 O(1) 平均时间的访问效率。 它是处理类型元数据的一种强大工具，常见于 RTTI 和泛型编程场景。如果你有具体问题或想扩展用法，请告诉我！

------

## **使用boost::typeindex**

```c++
#include <boost/type_index.hpp>
#include <unordered_map>
#include <vector>
#include <string>
#include <iostream>

namespace bti = boost::typeindex;

struct A { virtual void fun() {} };
struct B : A {};

void inspector(A& a) {
    std::cout << bti::type_id_runtime(a).pretty_name() << "\n";
}


// Compile time type information:
int x = 20;
double y = 2.4;
std::cout << bti::type_id<decltype(x)>().pretty_name() << "\n";
// prints: "int"
std::cout << bti::type_id<decltype(y)>().pretty_name() << "\n";
// prints: "double"

// Runtime type information:
A a;
B b;
inspector(a);
// prints: "A"
inspector(b);
// prints: "B"

// Also supports cv qualifiers (typeid doesn't)
std::cout << bti::type_id_with_cvr<const int>().pretty_name() << "\n";
// prints: "int const"

// We also get demangling for complex nested types
using nested = std::unordered_map<A,std::vector<std::string>>;
std::cout << bti::type_id<nested>().pretty_name();
// prints (GCC): "std::unordered_map<A, std::vector<
//                  std::__cxx11::basic_string<char...
```

