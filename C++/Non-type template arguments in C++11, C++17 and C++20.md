

# Non-type template arguments in C++11, C++17 and C++20

Besides types, templates can also be parametrized using non-type arguments from a limited set of structural types.

In C++11, the selection was limited to integral types, enumerations and pointers/references to global objects.

C++17 introduced a syntactic change, allowing auto to be used instead of a concrete type.

Finally, C++20 added support for floating-point types and literal class types (that only have public members and are recursively structural).

The argument is available in the template scope as a constant object. A prvalue for non-class types (except for lvalue reference); for class types, the argument is considered a static storage duration object.



```C++
#include <cstdint>
#include <print>
#include <array>

// C++11
// integral, pointer, enumeration, or lvalue reference
template <typename T, std::size_t size>
struct MyArray {
    T data_[size];
    T* begin() { return data_; }
    T* end() { return data_ + size; }
};

struct Info { int a; double b; };
static constexpr Info global1{42, 3.14};
static constexpr Info global2{7, 1.61};

template <const Info &ref>
struct Printer {
    void print() {
        std::println("ref.a == {}, ref.b == {}", ref.a, ref.b);
    }
};

// C++17
// placeholder
template <auto value>
struct Generic {
    void print_info() {
        if constexpr (std::is_same_v<decltype(value), int>) {
            std::println("int with value : {}", value);
        } else if constexpr (std::is_same_v<decltype(value), const int*>) {
            std::println("pointer to int with value : {}", *value);
        }
    }
};

static constexpr int global3 = 42;

// C++20
// literal types, CTAD
template <auto fun, std::array arr>
struct Caller {
    auto call(auto&&... extra_args) {
        return fun(arr, std::forward<decltype(extra_args)>(extra_args)...);
    }
};

int main() {
    MyArray<int,3> r1{{1,2,3}};
    // decltype(r1.data_) == int[3]
    // r1.data_ == {1,2,3}

    static_assert(std::is_same_v<decltype(r1.data_), int[3]>);

    for (auto v : r1)
        std::print("{} ", v);
    std::println("");

    Printer<global1> r2;
    r2.print();
    // prints "ref.a == 42, ref.b == 3.14"

    Printer<global2> r3;
    r3.print();
    // prints "ref.a == 7, ref.b == 1.61"

    Generic<42> r4;
    r4.print_info();
    // prints "int with value : 42"

    Generic<&global3> r5;
    r5.print_info(); 
    // prints "pointer to int with value : 42"

    constexpr auto fn = [](const std::array<int,3> &arr, int x) {
        return arr[0] + arr[1] + arr[2] + x;
    };
    Caller<fn, std::array{1,2,3}> r6;
    int sum = r6.call(42);
    // sum == 48

    std::println("sum == {}", sum);
}
```

我来详细解释这段 C++ 代码，它展示了从 C++11 到 C++20 的模板编程特性，包括自定义容器、非类型模板参数和编译期计算。

------

代码结构

使用的头文件：

- <cstdint>：提供 std::size_t。
- <print>：提供 std::print 和 std::println（C++23 特性）。
- <array>：提供 std::array。

代码包含三个主要模板类：

1. MyArray（C++11）：自定义数组容器。
2. Printer 和 Generic（C++17）：使用非类型模板参数。
3. Caller（C++20）：结合函数指针和编译期数组。

------

1. MyArray - 自定义数组容器（C++11）

定义

cpp

```cpp
template <typename T, std::size_t size>
struct MyArray {
    T data_[size];
    T* begin() { return data_; }
    T* end() { return data_ + size; }
};
```

解释

- **功能**：封装固定大小的数组，提供迭代器支持。

- **成员**：

  - data_：底层数组，类型为 T[size]。
  - begin() 和 end()：返回指向数组首尾的指针，支持范围 for 循环。

- **示例**：

  cpp

  ```cpp
  MyArray<int,3> r1{{1,2,3}};
  ```

  - 创建一个包含 {1,2,3} 的 MyArray<int,3>。
  - decltype(r1.data_) == int[3]。
  - 使用范围 for 循环输出：1 2 3。

------

2. Printer - 使用常量引用作为模板参数（C++11）

定义

cpp

```cpp
struct Info { int a; double b; };
static constexpr Info global1{42, 3.14};
static constexpr Info global2{7, 1.61};

template <const Info &ref>
struct Printer {
    void print() {
        std::println("ref.a == {}, ref.b == {}", ref.a, ref.b);
    }
};
```

解释

- **功能**：将全局常量对象作为模板参数，打印其内容。

- **要求**：模板参数必须是编译期常量引用（const Info&），因此使用 static constexpr 定义 global1 和 global2。

- **示例**：

  cpp

  ```cpp
  Printer<global1> r2;
  r2.print();  // 输出: ref.a == 42, ref.b == 3.14
  Printer<global2> r3;
  r3.print();  // 输出: ref.a == 7, ref.b == 1.61
  ```

- **特点**：模板实例化时绑定到特定全局对象，编译期确定值。

------

3. Generic - 使用 auto 占位符（C++17）

定义

cpp

```cpp
template <auto value>
struct Generic {
    void print_info() {
        if constexpr (std::is_same_v<decltype(value), int>) {
            std::println("int with value : {}", value);
        } else if constexpr (std::is_same_v<decltype(value), const int*>) {
            std::println("pointer to int with value : {}", *value);
        }
    }
};
static constexpr int global3 = 42;
```

解释

- **功能**：使用 C++17 的 auto 模板参数接受任意类型的编译期常量。

- **实现**：

  - if constexpr 在编译期根据 value 类型选择分支。
  - 支持 int（直接值）和 const int*（指针）。

- **示例**：

  cpp

  ```cpp
  Generic<42> r4;
  r4.print_info();  // 输出: int with value : 42
  Generic<&global3> r5;
  r5.print_info();  // 输出: pointer to int with value : 42
  ```

- **特点**：泛化非类型模板参数，增加灵活性。

------

4. Caller - 使用函数和数组作为模板参数（C++20）

定义

cpp

```cpp
template <auto fun, std::array arr>
struct Caller {
    auto call(auto&&... extra_args) {
        return fun(arr, std::forward<decltype(extra_args)>(extra_args)...);
    }
};
```

解释

- **功能**：将函数指针和 std::array 作为模板参数，调用函数并传递数组及额外参数。

- **参数**：

  - fun：编译期函数指针（通过 auto 指定）。
  - arr：编译期 std::array 对象（C++20 支持类类型作为非类型模板参数）。
  - extra_args：运行时参数，使用完美转发。

- **示例**：

  cpp

  ```cpp
  constexpr auto fn = [](const std::array<int,3> &arr, int x) {
      return arr[0] + arr[1] + arr[2] + x;
  };
  Caller<fn, std::array{1,2,3}> r6;
  int sum = r6.call(42);  // sum == 48
  ```

  - fn 是一个 lambda，计算数组元素和额外参数的和。
  - std::array{1,2,3} 在编译期存储。
  - r6.call(42) 调用 fn，返回 1 + 2 + 3 + 42 == 48。

- **特点**：利用 C++20 的非类型模板参数扩展，支持复杂类型。

------

主函数输出

cpp

```cpp
int main() {
    MyArray<int,3> r1{{1,2,3}};
    for (auto v : r1) std::print("{} ", v); std::println("");  // 1 2 3

    Printer<global1> r2; r2.print();  // ref.a == 42, ref.b == 3.14
    Printer<global2> r3; r3.print();  // ref.a == 7, ref.b == 1.61

    Generic<42> r4; r4.print_info();  // int with value : 42
    Generic<&global3> r5; r5.print_info();  // pointer to int with value : 42

    Caller<fn, std::array{1,2,3}> r6;
    int sum = r6.call(42);  // sum == 48
    std::println("sum == {}", sum);
}
```

输出

```text
1 2 3
ref.a == 42, ref.b == 3.14
ref.a == 7, ref.b == 1.61
int with value : 42
pointer to int with value : 42
sum == 48
```

------

关键概念

1. **C++11 - 模板基础**：
   - MyArray 展示自定义容器和迭代器支持。
   - Printer 使用常量引用作为模板参数。
2. **C++17 - auto 模板参数**：
   - Generic 利用 auto 接受任意编译期值，结合 if constexpr 处理类型。
3. **C++20 - 扩展非类型模板参数**：
   - Caller 支持函数指针和 std::array 作为模板参数，结合完美转发。
4. **编译期计算**：
   - 所有模板参数（如 global1、fn、std::array{1,2,3}）在编译期确定。

这段代码展示了现代 C++ 模板编程的演进，从简单容器到复杂的编译期操作。如果有具体部分需要深入解释，请告诉我！