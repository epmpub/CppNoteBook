#### **C++26 `std::inplace_vector` 的 `constexpr` 支持（尤其是非平凡类型）**

### 1. 什么是 `std::inplace_vector`？

```cpp
#include <inplace_vector>   // 或 <vector>（部分实现可能在此）

template<typename T, size_t N>
class inplace_vector;
```

- **固定容量**、**原地存储**（in-place / embedded storage）的动态大小 `vector`。
- 所有元素都存储在对象内部（类似 `std::array<T, N>` + `std::vector` 的接口）。
- **零堆分配**（除非 `T` 本身会分配内存）。
- 非常适合嵌入式、实时系统、高性能、以及 `constexpr` 场景。

### 2. `constexpr` 支持演进

| 阶段                     | 支持情况                               | 备注     |
| ------------------------ | -------------------------------------- | -------- |
| 早期提案                 | 仅 **Trivial** 类型                    | 实现简单 |
| C++26 初始               | **Trivially Copyable** + 默认/平凡析构 | 限制较严 |
| **最终 C++26** (P3074R7) | **非平凡类型（Non-trivial）也支持**    | 重大改进 |

**关键提案**：
- **P0843R14**：引入 `std::inplace_vector`
- **P3074R7**（Trivial Unions）：解决底层存储问题，让**非平凡类型**也能在 `constexpr` 中正确构造/析构。

**特性测试宏**（非常重要）：

```cpp
#if __cpp_lib_constexpr_inplace_vector >= 202502L
// C++26 中支持非平凡类型的 constexpr inplace_vector
#endif
```

### 3. 示例（非平凡类型）

```cpp
#include <inplace_vector>
#include <string>
#include <cassert>

struct NonTrivial {
    std::string s;
    NonTrivial(std::string str) : s(std::move(str)) {}
    ~NonTrivial() { /* 可以有副作用 */ }
};

constexpr void test() {
    std::inplace_vector<NonTrivial, 10> v;

    v.emplace_back("hello");
    v.emplace_back("constexpr");
    v.push_back(NonTrivial{"world"});

    assert(v.size() == 3);
    assert(v[0].s == "hello");
    assert(v.back().s == "world");

    v.pop_back();
    v.clear();        // 必须正确调用析构函数
}

int main() {
    constexpr auto result = [] {
        test();
        return 42;
    }();
    static_assert(result == 42);
}
```

### 4. 注意事项（非常重要）

1. **存储实现**：
   - 内部通常使用 `union` + `std::start_lifetime_as`（或等价技巧）来处理未初始化存储。
   - P3074 让这种模式在 `constexpr` 中对非平凡类型合法。

2. **限制**：
   - `T` 必须是 `constexpr` 可构造和析构的（即支持常量求值）。
   - `N` 过大会导致编译期内存/时间消耗显著增加。
   - 某些极端情况下（如非常复杂的析构函数），编译器支持可能仍有差异。

3. **与 `std::vector`、`std::array` 对比**：

   | 容器                  | 存储位置 | 容量     | constexpr（非平凡） | 堆分配 |
   | --------------------- | -------- | -------- | ------------------- | ------ |
   | `std::array<T,N>`     | 原地     | 固定     | 优秀                | 否     |
   | `std::inplace_vector` | 原地     | 固定上限 | **C++26 良好**      | 否     |
   | `std::vector<T>`      | 堆       | 动态     | 有限                | 是     |

### 5. 编译要求

```bash
# GCC 16.1+ / Clang 19+ 等
g++ -std=c++26 -O2 -march=native test.cpp -o test
```

**总结**：

C++26 通过 `P3074`（trivial unions 等语言机制）解决了长期痛点，让 **`std::inplace_vector<T, N>` 对非平凡类型也具备完整的 `constexpr` 支持**。这是 C++26 constexpr 容器扩展中非常实用的一部分，特别适合需要在编译期构建固定上限动态容器且元素有非平凡构造/析构的场景。

需要完整示例（带 `resize`、`insert`、`constexpr` 静态表等）还是具体某个操作的用法？随时说！