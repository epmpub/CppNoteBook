**`std::is_structural` / `std::meta::is_structural_type` 类型特征（C++26）** 解释。

这是 C++26 中与**结构化类型（Structural Types）**和**静态反射（Static Reflection）** 相关的特性，提案主要为 **P3856**（或相关子提案），特性测试宏 `__cpp_lib_is_structural`。

### 背景：什么是 Structural Type？

从 C++20 开始，**Non-Type Template Parameters (NTTP)** 支持**结构化类型**（Structural Types），允许类对象作为模板参数（前提是类型满足特定条件）：

- 标量类型（scalar）、数组、引用。
- 具有 `public` 非静态数据成员的**聚合类型**（aggregate），且所有成员类型本身也是 structural。
- 所有基类和成员必须是 structural。
- 没有虚函数、没有用户提供的构造函数/析构函数/赋值等（必须是“简单”的、可在编译期比较和哈希的）。

```cpp
struct Point { int x, y; };  // C++20+ 是 structural type，可作为 NTTP

template <Point P> void foo() { ... }
foo<Point{1, 2}>();
```

之前没有**标准方式**在编译期查询一个类型是否是 structural，导致元编程、泛型代码、反射等场景很不方便。

### C++26 新增的类型特征

1. **库层级**：`std::is_structural<T>`（或类似 `std::is_structural_type_v<T>`）
   - 一个普通的 `std::integral_constant<bool, ...>` 风格的 trait。
   - 直接检查 `T` 是否为 structural type。

2. **反射层级**（C++26 Reflection 核心）：`std::meta::is_structural_type`
   ```cpp
   namespace std::meta {
       consteval bool is_structural_type(std::meta::info r);  // r 必须代表一个类型
   }
   ```

   - 输入：一个 reflection 对象（`^^T` 或 `reflexpr(T)`）。
   - 返回：`true` 如果该类型是 structural，否则 `false`。

### 用法示例

```cpp
#include <type_traits>
#include <meta>  // 或 experimental/meta，视实现

struct Good { int a; double b; };           // structural
struct Bad { virtual void f(); int x; };    // non-structural（有虚函数）

static_assert(std::is_structural_v<Good>);
static_assert(!std::is_structural_v<Bad>);

// 使用反射（C++26）
constexpr auto t = ^^Good;
static_assert(std::meta::is_structural_type(t));

template<class T>
void process() {
    if constexpr (std::is_structural_v<T>) {
        // 可以安全作为 NTTP 使用，或进行其他编译期操作
    }
}
```

### 主要用途

- **元编程**：在模板约束、SFINAE/concepts 中精确检测是否能用作 NTTP。
- **静态反射**：结合 `std::meta::nonstatic_data_members_of` 等，构建更强大的泛型工具（例如自动生成 NTTP 友好包装器）。
- **库设计**：实现 `std::constant_wrapper`（C++26 新特性）或其他需要区分 structural 类型的行为。
- **编译期检查**：帮助开发者诊断“为什么这个类型不能作为模板参数”。

### 与 Reflection 的配合

C++26 强大的反射设施（`^^` 操作符、`std::meta` 命名空间）让这个 trait 特别强大。你可以查询任意类型的结构属性，包括是否 structural，从而实现高度通用的代码生成和检查。

这是一个“填补长期缺失的诊断/元编程工具”的典型 C++26 改进。它让结构化类型（structural types）这个重要概念从“只能隐式使用”变成了“可显式查询和操纵”。更多细节可查阅 cppreference C++26 页面和提案 P3856。