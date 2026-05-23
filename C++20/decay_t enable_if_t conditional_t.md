三个重要的 Type Traits：

### 1. `std::decay_t<T>` —— 类型退化

**作用**：把类型“退化”成最普通的类型，模拟**函数参数传递时的类型转换**。

#### 它会做三件事：
- 去掉 `const`、`volatile`
- 去掉引用（`&`、`&&`）
- 数组 → 指针（`T[N]` → `T*`）
- 函数 → 函数指针

#### 示例：

```cpp
#include <type_traits>
#include <string>

template<typename T>
void show_decay()
{
    using Decayed = std::decay_t<T>;
    std::cout << typeid(T).name() << "  -->  " 
              << typeid(Decayed).name() << "\n";
}

int main()
{
    show_decay<int&>();                    // int&          -> int
    show_decay<const std::string&>();      // const string& -> string
    show_decay<int[10]>();                 // int[10]       -> int*
    show_decay<void(int)>();               // void(int)     -> void(*)(int)
    show_decay<const int[5]&>();           // const int[5]& -> int*
}
```

**常见用途**：
- 模板中需要“值类型”时
- 实现 `std::make_pair`、`std::vector::push_back` 等

---

### 2. `std::enable_if<Condition, T>` —— 条件启用（SFINAE 核心工具）

**作用**：在编译期根据条件选择性地**启用或禁用**某个模板。

#### 现代写法（C++14+）：

```cpp
template<typename T>
std::enable_if_t<std::is_integral_v<T>, T>   // 返回类型
add(T a, T b) { return a + b; }
```

#### 完整示例：

```cpp
#include <type_traits>

// 只能对整数类型使用
template<typename T>
std::enable_if_t<std::is_integral_v<T>, T> 
multiply(T a, T b) 
{
    return a * b;
}

// 只能对浮点数使用
template<typename T>
std::enable_if_t<std::is_floating_point_v<T>, T>
divide(T a, T b) 
{
    return a / b;
}

int main()
{
    multiply(3, 5);        // OK
    // multiply(3.5, 2.0); // 编译错误

    divide(3.5, 2.0);      // OK
    // divide(10, 2);     // 编译错误
}
```

**另一个常见写法**（放在模板参数里）：

```cpp
template<typename T, typename = std::enable_if_t<std::is_pointer_v<T>>>
void process(T ptr) { ... }
```

---

### 3. `std::conditional_t<Condition, TrueType, FalseType>` —— 条件选择类型

**作用**：根据编译期条件，在**两种类型之间选择一个**。

#### 示例：

```cpp
template<typename T>
using PointerOrValue = std::conditional_t<
    std::is_integral_v<T>,     // 条件
    T*,                        // 如果是整数 → 用指针
    T                          // 否则 → 用原类型
>;

int main()
{
    PointerOrValue<int> a;      // int*
    PointerOrValue<double> b;   // double
    PointerOrValue<std::string> c; // std::string
}
```

#### 更实用的例子（根据大小选择存储方式）：

```cpp
template<typename T>
struct OptimizedStorage
{
    using Type = std::conditional_t<
        sizeof(T) <= 8,
        T,           // 小对象：直接存储
        T*           // 大对象：存储指针
    >;
};
```

---

### 三者对比总结

| Type Trait             | 主要作用          | 典型使用场景                | C++ 版本 |
| ---------------------- | ----------------- | --------------------------- | -------- |
| `std::decay_t<T>`      | 类型退化          | 去除引用、const、数组转指针 | C++11    |
| `std::enable_if_t<>`   | 条件启用/禁用模板 | SFINAE、重载分辨、类型约束  | C++11    |
| `std::conditional_t<>` | 条件选择类型      | 根据条件在两种类型中二选一  | C++11    |

---

**现代 C++ 趋势**（C++20+）：
- `std::enable_if` 正在被 **Concepts** 大量替代（更清晰）
- 但 `decay_t` 和 `conditional_t` 依然非常常用

---

需要我给你**综合实战例子**（把三个一起用），还是分别深入某个的更多用法？