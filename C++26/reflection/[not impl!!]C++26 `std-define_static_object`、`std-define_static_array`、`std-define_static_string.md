**C++26 `std::define_static_object`、`std::define_static_array`、`std::define_static_string`** 是静态反射（`std::meta`）相关的重要辅助工具，定义在头文件 **`<meta>`** 中，由提案 **P3491R3** 引入。

### 核心作用

这些函数**将编译期（constexpr）计算的结果提升（promote）到静态存储期（static storage duration）**，使其可以在运行时安全使用，并且**返回值可以作为模板实参**（template argument）。

这是解决 C++ constexpr 内存分配“沙箱”限制的关键方案：`constexpr new` 分配的内存无法“逃出”常量求值上下文，而这些函数能把数据变成真正的全局静态对象。

### 1. `std::define_static_string`

```cpp
template <ranges::input_range R>
consteval const ranges::range_value_t<R>* 
define_static_string(R&& r);   // 仅支持 char / char8_t 等字符类型
```

- **功能**：将一个编译期字符串（或字符范围）提升为**静态存储的 null-terminated 字符串**。
- **返回值**：指向静态字符串首字符的 `const char*`（或对应字符类型指针）。
- **用途**：生成编译期计算的字符串常量表、错误消息、名称等。

**示例**：

```cpp
constexpr auto make_greeting(std::string_view name) {
    return std::define_static_string(std::format("Hello, {}!", name));
}

int main() {
    constexpr auto p = make_greeting("World");
    static_assert(std::is_same_v<decltype(p), const char*>);
    std::cout << p << '\n';   // Hello, World!
}
```

### 2. `std::define_static_array`

```cpp
template <ranges::input_range R>
consteval std::span<const ranges::range_value_t<R>> 
define_static_array(R&& r);
```

- **功能**：将任意输入范围（`std::vector`、`std::array`、生成器等）在编译期的数据提升为**静态存储的数组**。
- **返回值**：指向静态数组的 `std::span<const T>`。
- **关键**：底层数据具有静态存储期，span 本身是编译期构造的。

**示例**：

```cpp
constexpr auto make_lookup_table() {
    std::vector<int> v = {1, 4, 9, 16, 25};
    return std::define_static_array(v);   // 或直接用 ranges
}

int main() {
    constexpr auto table = make_lookup_table();
    std::cout << table[3] << '\n';   // 16
}
```

### 3. `std::define_static_object`

```cpp
template <class T>
consteval const std::remove_cvref_t<T>* 
define_static_object(T&& t);
```

- **功能**：将单个对象提升为**静态存储的对象**。
- **返回值**：指向静态对象的 `const T*`。
- 它是 `define_static_array` 的特例（单元素情况）。

**示例**：

```cpp
struct Config { int x; double y; };

constexpr auto get_config() {
    return std::define_static_object(Config{42, 3.14});
}

int main() {
    constexpr auto* cfg = get_config();
    std::cout << cfg->x << '\n';
}
```

### 共同特性

- **consteval**：只能在常量求值上下文中调用。
- **静态存储期**：返回的指针/span 指向真正的全局静态对象（与全局变量同寿命）。
- **可作为模板实参**：返回的值具有唯一身份，可用于模板元编程。
- **Structural 类型限制**：对于复杂类型，需要是 structural type（才能通过 `reflect_constant` 等机制处理）。
- **Feature Test Macro**：
  ```cpp
  #if __cpp_lib_define_static >= 202506L
  #include <meta>
  #endif
  ```

### 主要应用场景

- 编译期生成的查找表（LUT）、字符串表。
- 反射中需要运行时访问的元数据（枚举列表、成员列表等）。
- 将 `constexpr std::vector` / 计算结果“固化”为运行时可用的静态数据。
- 避免重复计算或临时 constexpr 分配的局限性。

这些工具是 C++26 **静态反射**生态的重要组成部分，极大提升了编译期计算结果向运行时的过渡能力。

更多细节可参考：
- 提案 [P3491R3](https://wg21.link/P3491)
- cppreference: `<meta>` 头文件下的三个函数

需要具体使用场景的完整示例吗？（比如反射 + define_static_array 生成枚举表）