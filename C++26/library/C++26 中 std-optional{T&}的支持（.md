C++26 中 std::optional<T&> 的支持（提案 P2988R12，已被采纳）解决了 C++17/20/23 中 std::optional 不能直接存放引用（reference）的问题。 

sandordargo.com

1. 为什么以前不支持？

- 引用不是对象（T& 不能作为值类型存储）。
- 赋值语义存在争议：是“写穿（assign-through）”还是“重绑定（rebind）”？
  - 如果是普通引用语义，opt = new_value; 应该修改被引用的对象。
  - 但这会导致赋值行为依赖 optional 是否 engaged（有值），非常容易出错。
- 之前只能用 std::reference_wrapper<T> 间接实现，或者直接用 T*。
- C++26 的设计（关键特性）

- 非拥有型（non-owning）：不像 std::optional<T> 那样存储/拥有值，它只是引用一个已存在的对象。
- 内部实现类似指针：本质上存一个 T*（或 nullptr 表示 empty）。
- 始终重绑定（Rebinding semantics）：赋值操作总是改变引用的目标，而不是修改被引用的对象（与 Boost.Optional 一致）。 sandordargo.com

cpp

```cpp
int a = 1, b = 2;
std::optional<int&> opt = a;   // opt 引用 a
opt = b;                       // 重绑定：现在 opt 引用 b（a 保持不变！）
*opt = 42;                     // 修改 b 的值
```

- 值语义 + 引用语义混合：提供 std::optional 的常规接口（has_value()、value()、operator*、operator-> 等），但行为像可空的引用。
- value_type 是 T&（不是 T）。
- 构造和使用示例

cpp

```cpp
#include <optional>

int x = 10;
std::optional<int&> opt1;           // empty
std::optional<int&> opt2 = x;       // 引用 x
std::optional<int&> opt3{std::ref(x)}; // 也可以用 std::ref，但不必要

if (opt2) {
    *opt2 = 42;                     // 修改 x
    std::cout << x;                 // 42
}

opt2 = std::nullopt;                // 置空
opt2 = x;                           // 重新绑定
```

不支持 make_optional<T&>（因为它本来就用于值构造），直接用构造函数或 {obj}。4. 与其他方式对比

| 方式                                     | 优点                                    | 缺点                            |
| ---------------------------------------- | --------------------------------------- | ------------------------------- |
| std::optional<T&> (C++26)                | 类型安全、清晰、自文档化、 monadic 操作 | -                               |
| T*                                       | 简单、性能好                            | 不自文档（可能指向堆/野指针等） |
| std::reference_wrapper<T>                | 老办法                                  | 需要手动处理 empty 状态         |
| std::optional<std::reference_wrapper<T>> | 以前的 workaround                       | 啰嗦、接口不自然                |

5. 其他新特性（C++26）

- std::optional 现在支持 range（begin()/end()），可以用于 range-based for 或 views。
- Monadic 操作（and_then、or_else、transform 等）对引用版本也自然支持。

总结std::optional<T&> 让 “可空的引用” 成为第一公民，极大提升了代码的表达力和安全性，尤其适合返回“可能存在也可能不存在的某个对象引用”的场景（工厂函数、查找、缓存等）。

它不是普通引用的简单包装，而是“可空的、可重绑定的引用”，行为更接近“智能的可空指针”，但语义更清晰。 

sandordargo.com

推荐阅读：

- Sandor Dargo 的博客（非常清晰的解释）
- Steve Downey 在 CppCon 2025 的演讲（提案作者）
- 提案纸：P2988R12

有具体使用场景想看例子吗？