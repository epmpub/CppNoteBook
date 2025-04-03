# std::invoke

在 C++ 中，std::invoke 是一个函数模板，定义在 <functional> 头文件中，最初在 C++17 引入。它提供了一种统一的方式来调用各种可调用对象（callable objects），包括普通函数、函数指针、成员函数指针、lambda 表达式以及其他具有 operator() 的对象。std::invoke 的设计目标是简化调用语义，使代码更通用和一致。

以下是对 std::invoke 的详细解释：

------

定义和基本概念

- **头文件**：<functional>

- **函数签名**：

  cpp

  ```cpp
  template<class F, class... Args>
  std::invoke_result_t<F, Args...> invoke(F&& f, Args&&... args);
  ```

  - F：可调用对象的类型。
  - Args：传递给可调用对象的参数类型。
  - 返回值：调用结果的类型，由 std::invoke_result_t 推导。

- **用途**：

  - 统一调用普通函数和成员函数。
  - 处理不同的调用语法（如 obj.memfun() vs (*ptr).*memfun）。
  - 在模板编程中提供一致的调用接口。

- **支持的可调用对象**：

  - 普通函数。
  - 函数指针。
  - 成员函数指针。
  - 仿函数（重载了 operator() 的对象）。
  - Lambda 表达式。

------

工作原理

std::invoke 根据可调用对象的类型执行不同的调用方式：

1. **普通函数或仿函数**：
   - 直接调用 f(args...)。
2. **成员函数指针**：
   - 如果第一个参数是对象，则调用 (obj.*f)(args...)。
   - 如果第一个参数是指针，则调用 (*ptr).*f(args...)。
3. **成员数据指针**：
   - 返回成员变量的值，如 obj.*f 或 (*ptr).*f。

所有参数都会通过 std::forward 完美转发，以保留值的类别（左值或右值）。

------

示例代码

1. 调用普通函数

cpp

```cpp
#include <iostream>
#include <functional>

void say_hello(const std::string& name) {
    std::cout << "Hello, " << name << "!\n";
}

int main() {
    std::invoke(say_hello, "Alice");
    return 0;
}
```

- **输出**：Hello, Alice!
- **说明**：直接调用普通函数，等价于 say_hello("Alice")。
- 调用成员函数

cpp

```cpp
#include <iostream>
#include <functional>

struct MyClass {
    void print(int x) const {
        std::cout << "Value: " << x << "\n";
    }
};

int main() {
    MyClass obj;
    std::invoke(&MyClass::print, obj, 42); // 调用成员函数
    return 0;
}
```

- **输出**：Value: 42
- **说明**：
  - &MyClass::print 是成员函数指针。
  - std::invoke 自动处理 obj.print(42) 的调用。
- 使用指针调用成员函数

cpp

```cpp
#include <iostream>
#include <functional>

struct MyClass {
    void print(int x) const {
        std::cout << "Value: " << x << "\n";
    }
};

int main() {
    MyClass obj;
    MyClass* ptr = &obj;
    std::invoke(&MyClass::print, ptr, 100); // 使用指针
    return 0;
}
```

- **输出**：Value: 100
- **说明**：std::invoke 解引用指针并调用成员函数。
- 访问成员变量

cpp

```cpp
#include <iostream>
#include <functional>

struct MyClass {
    int value = 42;
};

int main() {
    MyClass obj;
    int result = std::invoke(&MyClass::value, obj); // 访问成员变量
    std::cout << "Member value: " << result << "\n";
    return 0;
}
```

- **输出**：Member value: 42
- **说明**：对于成员数据指针，std::invoke 返回成员的值。
- 使用 Lambda

cpp

```cpp
#include <iostream>
#include <functional>

int main() {
    auto lambda = [](int x, int y) { return x + y; };
    int result = std::invoke(lambda, 3, 4);
    std::cout << "Result: " << result << "\n";
    return 0;
}
```

- **输出**：Result: 7
- **说明**：std::invoke 可以调用任何具有 operator() 的对象。

------

与直接调用的比较

| 场景             | 直接调用             | 使用 std::invoke                      |
| ---------------- | -------------------- | ------------------------------------- |
| 普通函数         | f(args...)           | std::invoke(f, args...)               |
| 成员函数（对象） | obj.memfun(args...)  | std::invoke(&C::memfun, obj, args...) |
| 成员函数（指针） | ptr->memfun(args...) | std::invoke(&C::memfun, ptr, args...) |
| 成员变量（对象） | obj.member           | std::invoke(&C::member, obj)          |
| Lambda/仿函数    | f(args...)           | std::invoke(f, args...)               |

- **优势**：
  - 统一调用语法，特别在模板编程中无需区分调用类型。
  - 支持成员函数和成员变量的通用访问。
  - 完美转发参数，避免不必要的拷贝。
- **劣势**：
  - 对于简单场景可能显得冗余。
  - 需要 C++17 支持。

------

相关工具

1. **std::invoke_result（C++17）**：
   - 类型特征，用于推导 std::invoke 的返回类型。
   - 示例：std::invoke_result_t<decltype(f), Args...>。
2. **std::is_invocable（C++17）**：
   - 检查是否可以调用某个可调用对象。
   - 示例：std::is_invocable_v<F, Args...>。

------

注意事项

- **性能**：std::invoke 是编译时展开的，通常无运行时开销。
- **异常**：如果调用无效（例如空函数指针），行为未定义。
- **线程安全**：std::invoke 本身不提供线程安全，依赖被调用对象的特性。

------

总结

std::invoke 是 C++17 引入的一个强大工具，它统一了各种可调用对象的调用方式，特别在泛型编程中非常有用。通过它，可以轻松处理普通函数、成员函数、成员变量和仿函数的调用，减少代码中的条件分支和冗余逻辑。对于需要高度抽象和灵活性的场景，std::invoke 是标准库中不可或缺的一部分。