

#  std::atexit

```C++
#include <print>

void callback_a() {
    std::println(stderr, "callback A");
}

void callback_b() {
    std::println(stderr, "callback B");
}

struct Global {
    ~Global() { std::println(stderr, "~Global()"); }
};

Global global;

int main() {
    std::atexit(callback_a);
    std::atexit(callback_b);
    // Implicit call to std::exit(0);
    
    // Calling std::Exit(int); or std::quick_exit(int); 
    // will not invoke the callbacks
}
```

这段代码展示了 C++ 中使用 std::atexit 注册退出时调用的回调函数，以及全局对象的析构行为。代码通过示例展示了程序正常退出时的行为，并提到 std::exit 和 std::quick_exit 的区别。以下是逐步解释。

------

代码概览

- 定义两个回调函数 callback_a 和 callback_b，在程序退出时执行。
- 定义全局对象 Global，其析构函数在程序结束时调用。
- 使用 std::atexit 注册回调，观察退出时的执行顺序。

------

关键组件

1. **头文件**

cpp

```cpp
#include <print>
```

- <print>：C++23 的头文件，提供 std::println。
- 隐含依赖 <cstdlib>（提供 std::atexit、std::exit 等）。
- **回调函数**

cpp

```cpp
void callback_a() {
    std::println(stderr, "callback A");
}
void callback_b() {
    std::println(stderr, "callback B");
}
```

- **callback_a 和 callback_b**：
  - 简单的函数，输出到 stderr。
- **用途**：
  - 通过 std::atexit 注册，在程序退出时调用。
- **全局对象**

cpp

```cpp
struct Global {
    ~Global() { std::println(stderr, "~Global()"); }
};
Global global;
```

- **Global**：
  - 析构函数输出 ~Global()。
- **global**：
  - 全局对象，程序结束时自动析构。
- **析构时机**：
  - 在 main 返回后，或 std::exit 调用时。
- **main 函数**

cpp

```cpp
int main() {
    std::atexit(callback_a);
    std::atexit(callback_b);
}
```

- **std::atexit**：
  - 原型：int atexit(void (*)())。
  - 注册退出回调函数，按后进先出（LIFO）顺序执行。
  - callback_b 先注册，callback_a 后注册。
- **退出行为**：
  - main 返回隐式调用 std::exit(0)。
  - std::exit：
    - 调用 atexit 注册的函数。
    - 销毁全局对象。
- **注释说明**：
  - std::exit(int) 调用回调和析构。
  - std::quick_exit(int) 不调用回调或析构，仅快速终止。

------

执行顺序

1. **main 返回**：
   - 触发程序退出。
2. **全局对象析构**：
   - global 的析构函数执行，输出 ~Global()。
3. **atexit 回调**：
   - 按 LIFO 顺序调用：
     - callback_a（后注册，先执行）。
     - callback_b（先注册，后执行）。

------

输出

```text
callback A
callback B
~Global()
```

- **顺序解释**：
  - std::atexit 回调在全局对象析构之前执行。
  - 回调按注册的逆序调用。

------

为什么这样工作？

1. **std::atexit**：
   - 注册的函数存储在全局列表中，std::exit 调用时按逆序执行。
2. **全局对象**：
   - C++ 保证静态/全局对象的析构在程序退出时发生。
3. **std::exit vs std::quick_exit**：
   - std::exit：清理全局对象，调用 atexit 回调。
   - std::quick_exit：立即终止，不清理。

------

使用场景

- **资源清理**：
  - 注册回调释放资源。
- **调试**：
  - 跟踪程序退出流程。
- **特殊退出**：
  - 区分 exit 和 quick_exit 的需求。

------

总结

- std::atexit 注册 callback_a 和 callback_b，按逆序执行。
- 全局对象 global 在回调后析构。
- std::exit 调用回调和析构，std::quick_exit 不调用。
- 代码展示了退出时的清理顺序。