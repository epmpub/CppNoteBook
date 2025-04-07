# C++  decltype(auto)

代码:

```c++
#include <type_traits>
#include <functional>

// Same as auto v = 1;, i.e. int
auto f1() { return 1; }
// Same as int x; auto v = x;, i.e. int
auto f2() { static int x = 1; return x; }
// Same as auto v = f2();, i.e. int
auto f3() { return f2(); }
// All three functions deduce int
// f2 (and transitively f3) results in a copy

int get_temp() { return 1; }
int& get_var() { static int x = 1; return x; }

// Same as decltype(get_temp()), i.e. int
decltype(auto) f4() { return get_temp(); }
// Same as decltype(get_var()), i.e. int&
decltype(auto) f5() { return get_var(); }

// Notably this is importantly when working with generic callables
decltype(auto) wrap(auto&& arg, auto projection) {
    return projection(std::forward<decltype(arg)>(arg));
}

int main() {
    static_assert(std::is_same_v<decltype(f1()), int>);
    static_assert(std::is_same_v<decltype(f2()), int>);
    static_assert(std::is_same_v<decltype(f3()), int>);


    static_assert(std::is_same_v<decltype(get_temp()), int>);
    static_assert(std::is_same_v<decltype(f4()), int>);
    static_assert(std::is_same_v<decltype(get_var()), int&>);
    static_assert(std::is_same_v<decltype(f5()), int&>);

    // Using std::identity: wrap(int{}, identity) 
    // int&& argument, forwarded and passed through as int&&
    static_assert(std::is_same_v<decltype(wrap(int{}, std::identity{})), int&&>);

    int x = 42;
    // Using std::identity{}: wrap(x, identity)
    // int& argument, forwarded and passed through as int&
    static_assert(std::is_same_v<decltype(wrap(x, std::identity{})), int&>);

    auto custom = [](int arg) { return arg; };
    // Using a custom projection that returns int: wrap(42, custom)
    // int&& argument, bound into int, returned as int
    static_assert(std::is_same_v<decltype(wrap(42, custom)), int>);
}
```



这段代码展示了 C++ 中 auto 和 decltype(auto) 在函数返回类型推导中的行为，以及它们在不同场景下的区别。代码还通过一个泛型函数 wrap 展示了如何结合通用引用和投影函数（如 std::identity）来保留或转换值的类型。以下是逐步解释。

------

代码概览

- 使用 auto 定义函数 f1、f2、f3，推导返回类型。
- 使用 decltype(auto) 定义函数 f4、f5，与原始函数的返回类型保持一致。
- 定义泛型函数 wrap，测试类型转发和投影的效果。
- 使用 static_assert 验证类型推导结果。

------

关键组件

1. **头文件**

cpp

```cpp
#include <type_traits>
#include <functional>
```

- <type_traits>：提供 std::is_same_v 用于类型检查。
- <functional>：提供 std::identity。
- **auto 返回类型**

cpp

```cpp
auto f1() { return 1; }
auto f2() { static int x = 1; return x; }
auto f3() { return f2(); }
```

- **f1()**：
  - 返回 1（prvalue），auto 推导为 int，结果是值类型。
- **f2()**：
  - 返回静态变量 x（lvalue），auto 推导为 int，返回副本。
- **f3()**：
  - 调用 f2()，返回其结果（int），auto 推导为 int。
- **验证**：
  - decltype(f1()) == int。
  - decltype(f2()) == int。
  - decltype(f3()) == int。
- **规则**：
  - auto 总是推导为值类型，移除引用和 cv 限定符。
- **decltype(auto) 返回类型**

cpp

```cpp
int get_temp() { return 1; }
int& get_var() { static int x = 1; return x; }
decltype(auto) f4() { return get_temp(); }
decltype(auto) f5() { return get_var(); }
```

- **get_temp()**：
  - 返回 1（prvalue），类型是 int。
- **get_var()**：
  - 返回静态变量的引用（lvalue），类型是 int&。
- **f4()**：
  - decltype(auto) 推导为 decltype(get_temp())，即 int。
- **f5()**：
  - decltype(auto) 推导为 decltype(get_var())，即 int&。
- **验证**：
  - decltype(f4()) == int。
  - decltype(f5()) == int&。
- **规则**：
  - decltype(auto) 保留原始表达式的类型（包括引用）。
- **泛型函数 wrap**

cpp

```cpp
decltype(auto) wrap(auto&& arg, auto projection) {
    return projection(std::forward<decltype(arg)>(arg));
}
```

- **参数**：
  - auto&& arg：通用引用，保留值类别。
  - auto projection：投影函数（如 std::identity 或 lambda）。
- **返回类型**：
  - decltype(auto) 推导为 projection 的返回类型。
- **转发**：
  - std::forward 保持 arg 的值类别（lvalue 或 rvalue）。

**测试用例**

cpp

```cpp
static_assert(std::is_same_v<decltype(wrap(int{}, std::identity{})), int&&>);
int x = 42;
static_assert(std::is_same_v<decltype(wrap(x, std::identity{})), int&>);
auto custom = [](int arg) { return arg; };
static_assert(std::is_same_v<decltype(wrap(42, custom)), int>);
```

- **wrap(int{}, std::identity{})**：
  - int{} 是 prvalue，auto&& 推导为 int&&。
  - std::identity 返回 int&&，decltype(auto) 保留为 int&&。
- **wrap(x, std::identity{})**：
  - x 是 lvalue，auto&& 推导为 int&。
  - std::identity 返回 int&，decltype(auto) 保留为 int&。
- **wrap(42, custom)**：
  - 42 是 prvalue，auto&& 推导为 int&&。
  - custom 接受 int，返回 int（值类型），decltype(auto) 推导为 int。

------

为什么这样工作？

1. **auto**：
   - 推导为值类型，总是返回副本，忽略引用和 cv。
2. **decltype(auto)**：
   - 保留原始表达式的类型（值、引用等），更精确。
3. **通用引用**：
   - auto&& 根据值类别推导：
     - lvalue → T&。
     - rvalue → T&&。
4. **投影**：
   - std::identity 保持输入类型，custom 强制值类型。

------

输出

- 无运行时输出，全部为编译期验证。

------

使用场景

- **auto**：
  - 简单场景，不需要引用。
- **decltype(auto)**：
  - 需要保留引用或精确类型的场景。
- **wrap**：
  - 泛型编程中转发值或应用投影。

------

总结

- f1、f2、f3 使用 auto，推导为 int。
- f4、f5 使用 decltype(auto)，分别推导为 int 和 int&。
- wrap 展示通用引用和投影的灵活性：
  - int&& → int&&（std::identity）。
  - int& → int&（std::identity）。
  - int&& → int（custom）。
- 代码突出 auto 和 decltype(auto) 的区别及应用。