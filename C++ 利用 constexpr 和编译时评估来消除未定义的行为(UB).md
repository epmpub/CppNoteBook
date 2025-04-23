

# 利用 constexpr 和编译时评估来消除未定义的行为(UB)

编译时常量表达式不允许调用未定义的行为。这包括在编译时评估的 constexpr 函数。

此属性可用于静态测试代码，确保代码不会调用未定义的行为。



```C++
#include <numeric>
#include <vector>

constexpr int midpoint(int a, int b) {
    return (a + b)/2; // can overflow, int overlow is UB
}

constexpr int generate() {
    std::vector<int> data = {1};
    auto it = data.begin();
    for (int i = 0; i < 10; i++)
        data.push_back(i); // invalidates it
    return *it; // accessing invalid iterator
}

constexpr int process() {
    int* buffer = new int[10];
    for (int i = 0; i < 10; i++)
        buffer[i] = i;
    int sum = 0;
    for (int i = 0; i < 10; i++)
        sum += i;
    return sum; // we memory leak buffer
}

constexpr int cnt_space(const char* str, size_t sz) {
    int cnt = 0;
    for (size_t i = 0; i < sz ; ++i) {
        if (str[i] == ' ') ++cnt; // out-of-bounds
    }
    return cnt;
}

int main() {
    constexpr int a = std::numeric_limits<int>::max();
    constexpr int b = a - 2;
    constexpr int c = a - 1;

    // Wouldn't compile: "overflow in constant expression"
    static_assert(midpoint(a, b) == c);
    // Wouldn't compile "use of storage after deallocation"
    static_assert(generate() == 1);
    // Wouldn't compile "storage has not been deallocated"
    static_assert(process() == 45);
    // Wouldn't compile "array subscript value '8' is outside 
    //                   the bounds of array type 'const char [8]'"
    static_assert(cnt_space("a b c d", 9) == 3);
}
```

这段代码展示了几个在 C++ 中使用 constexpr 函数时可能出现的问题，并通过 static_assert 验证这些问题会导致编译失败。以下是对每个函数及其问题的逐步解释：

------

1. midpoint 函数

cpp

```cpp
constexpr int midpoint(int a, int b) {
    return (a + b) / 2; // can overflow, int overflow is UB
}
```

- **用途**：计算两个整数的中间值。

- **问题**：当 a 和 b 非常大时（如接近 int 的最大值），a + b 可能会导致整数溢出。在 C++ 中，有符号整数溢出是未定义行为（UB）。

- **示例**：

  - a = std::numeric_limits<int>::max()（例如 2147483647）
  - b = a - 2（2147483645）
  - a + b = 4294967292，超出 int 最大值，触发溢出。

- **编译器反应**：在 constexpr 上下文中，编译器会检测到溢出并拒绝编译。

- **验证**：

  cpp

  ```cpp
  static_assert(midpoint(a, b) == c); // c = a - 1
  ```

  - 期望 midpoint(a, b) 等于 c，但因溢出而无法通过编译，错误信息类似于“overflow in constant expression”。

------

2. generate 函数

cpp

```cpp
constexpr int generate() {
    std::vector<int> data = {1};
    auto it = data.begin();
    for (int i = 0; i < 10; i++)
        data.push_back(i); // invalidates it
    return *it; // accessing invalid iterator
}
```

- **用途**：尝试在编译期生成一个向量并操作其迭代器。

- **问题**：

  - data.push_back(i) 会增加向量的大小，可能导致底层存储重新分配，从而使之前的迭代器 it 失效。
  - 在 constexpr 函数中，访问失效的迭代器是非法的。

- **限制**：

  - C++20 之前，std::vector 的动态操作（如 push_back）不支持 constexpr。
  - 即使在 C++20 中支持，访问失效迭代器仍会导致未定义行为。

- **编译器反应**：编译器检测到迭代器失效或动态分配的非法使用，报错类似于“use of storage after deallocation”。

- **验证**：

  cpp

  ```cpp
  static_assert(generate() == 1);
  ```

  - 期望返回初始值 1，但因迭代器失效无法编译。

------

3. process 函数

cpp

```cpp
constexpr int process() {
    int* buffer = new int[10];
    for (int i = 0; i < 10; i++)
        buffer[i] = i;
    int sum = 0;
    for (int i = 0; i < 10; i++)
        sum += i;
    return sum; // we memory leak buffer
}
```

- **用途**：分配动态内存，填充数据，计算总和并返回。

- **问题**：

  - 使用 new 分配了内存，但没有 delete[] 释放，导致内存泄漏。
  - 在 constexpr 上下文中，动态内存分配必须在函数结束前释放，否则是非法的。

- **限制**：

  - C++20 引入了对 constexpr 中动态内存分配的支持，但要求分配和释放必须配对。

- **编译器反应**：检测到未释放的内存，报错类似于“storage has not been deallocated”。

- **验证**：

  cpp

  ```cpp
  static_assert(process() == 45);
  ```

  - 期望返回 0+1+2+...+9 = 45，但因内存泄漏无法编译。

------

4. cnt_space 函数

cpp

```cpp
constexpr int cnt_space(const char* str, size_t sz) {
    int cnt = 0;
    for (size_t i = 0; i < sz ; ++i) {
        if (str[i] == ' ') ++cnt; // out-of-bounds
    }
    return cnt;
}
```

- **用途**：统计字符串中空格的数量。

- **问题**：

  - 如果 sz 超过字符串的实际长度，str[i] 会导致越界访问。
  - 在 constexpr 上下文中，越界访问是未定义行为，编译器会检测并拒绝。

- **示例**：

  - "a b c d" 长度为 8（包括末尾的空字符 \0），但传入 sz = 9，会导致越界。

- **编译器反应**：检测到数组越界，报错类似于“array subscript value '8' is outside the bounds of array type 'const char [8]'”。

- **验证**：

  cpp

  ```cpp
  static_assert(cnt_space("a b c d", 9) == 3);
  ```

  - 期望统计出 3 个空格，但因越界访问无法编译。

------

main 函数和 static_assert

cpp

```cpp
int main() {
    constexpr int a = std::numeric_limits<int>::max();
    constexpr int b = a - 2;
    constexpr int c = a - 1;

    static_assert(midpoint(a, b) == c);
    static_assert(generate() == 1);
    static_assert(process() == 45);
    static_assert(cnt_space("a b c d", 9) == 3);
}
```

- **用途**：使用 static_assert 测试上述函数在编译期的行为。
- **结果**：所有 static_assert 都会失败，因为每个函数都违反了 constexpr 的要求：
  1. midpoint：整数溢出。
  2. generate：迭代器失效。
  3. process：内存泄漏。
  4. cnt_space：数组越界。

------

总结

这段代码通过反面示例展示了 constexpr 函数的严格要求：

1. 不能有未定义行为（如溢出、越界）。
2. 动态内存分配必须正确管理（分配和释放配对）。
3. 迭代器使用必须有效。
4. 所有操作必须在编译期可计算。

这些 static_assert 故意设计为无法通过，以演示编译器如何检测和阻止这些问题。