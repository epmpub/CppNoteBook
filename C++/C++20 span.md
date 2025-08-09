

#  C++20 span

```C++
#include <array>
#include <span>
#include <vector>

size_t fn1(std::span<int> data) {
    size_t result = 0;
    for (size_t i = 0; i < data.size(); ++i)
        result += i;
    return result;
}

size_t fn2(std::span<int, 1024> data) {
    size_t result = 0;
    for (size_t i = 0; i < data.size(); ++i)
        result += i;
    return result;
}

int main() {
    std::array<int, 1024> data1;
    std::span arr1 = data1;
    // decltype(arr1) == std::span<int, 1024>

    static_assert(std::is_same_v<decltype(arr1), std::span<int, 1024>>);

    int data2[1024];
    std::span arr2 = data2;
    // delctype(arr2) == std::span<int, 1024>

    static_assert(std::is_same_v<decltype(arr2), std::span<int, 1024>>);

    std::vector<int> data3(16);
    fn1(data3); // OK
    // fn2(data3); // Wouldn't compile
}
```



这段代码展示了 C++20 中引入的 <span> 库的使用，std::span 是一种轻量级视图，用于表示连续内存块。它与 std::array、std::vector 和 C 风格数组配合使用，提供统一的接口。以下是对代码的详细解释：

------

**代码结构**

1. **头文件**：
   - <array>：提供 std::array。
   - <span>：提供 std::span。
   - <vector>：提供 std::vector。
2. **主要内容**：
   - 定义两个函数 fn1 和 fn2，分别接受动态和固定大小的 std::span。
   - 在 main 中展示 std::span 与不同容器类型的兼容性。

------

**代码逐步解释**

**1. 函数 fn1：动态大小 std::span**

cpp

```cpp
size_t fn1(std::span<int> data) {
    size_t result = 0;
    for (size_t i = 0; i < data.size(); ++i)
        result += i;
    return result;
}
```

- **std::span<int>**：
  - 表示动态大小的 int 序列视图。
  - 不拥有数据，仅提供对连续内存的引用。
- **功能**：
  - 计算索引之和：0 + 1 + ... + (size-1)。
  - 使用 data.size() 获取元素数量。
- **灵活性**：
  - 可接受任意大小的连续 int 序列。

------

**2. 函数 fn2：固定大小 std::span**

cpp

```cpp
size_t fn2(std::span<int, 1024> data) {
    size_t result = 0;
    for (size_t i = 0; i < data.size(); ++i)
        result += i;
    return result;
}
```

- **std::span<int, 1024>**：
  - 表示固定大小（1024 个元素）的 int 序列视图。
  - 编译期指定大小，运行时仍可访问 size()（固定为 1024）。
- **功能**：
  - 同 fn1，计算索引之和。
- **限制**：
  - 只接受正好 1024 个元素的序列。

------

**3. main 函数**

cpp

```cpp
std::array<int, 1024> data1;
std::span arr1 = data1;
// decltype(arr1) == std::span<int, 1024>
static_assert(std::is_same_v<decltype(arr1), std::span<int, 1024>>);
```

- **std::array<int, 1024> data1**：
  - 固定大小的数组，包含 1024 个 int。
- **std::span arr1 = data1**：
  - 从 data1 构造 std::span，自动推导为 std::span<int, 1024>。
  - std::span 提供对 data1 的视图。
- **static_assert**：
  - 验证 arr1 的类型是 std::span<int, 1024>。
  - std::is_same_v 是 C++17 的类型比较工具。

------

cpp

```cpp
int data2[1024];
std::span arr2 = data2;
// delctype(arr2) == std::span<int, 1024>
static_assert(std::is_same_v<decltype(arr2), std::span<int, 1024>>);
```

- **int data2[1024]**：
  - C 风格数组，固定大小 1024。
- **std::span arr2 = data2**：
  - 从 data2 构造 std::span，推导为 std::span<int, 1024>。
- **static_assert**：
  - 确认 arr2 类型为 std::span<int, 1024>。
- **注释错误**：
  - 代码注释中拼写错误：delctype 应为 decltype。

------

cpp

```cpp
std::vector<int> data3(16);
fn1(data3); // OK
// fn2(data3); // Wouldn't compile
```

- **std::vector<int> data3(16)**：
  - 动态大小向量，初始化为 16 个元素。
- **fn1(data3)**：
  - std::vector 提供连续内存，fn1 接受动态大小的 std::span<int>。
  - 隐式转换为 std::span<int>，大小为 16。
  - 返回：0 + 1 + ... + 15 = 120。
- **fn2(data3)**：
  - 被注释掉，因为无法编译。
  - fn2 要求 std::span<int, 1024>，但 data3 只有 16 个元素，大小不匹配。

------

**关键技术点**

1. **std::span**：
   - C++20 引入的轻量级视图，表示连续内存块。
   - 支持动态大小（std::span<T>）和固定大小（std::span<T, N>）。
   - 不拥有数据，类似指针+长度。
2. **类型推导**：
   - 从 std::array 和 C 风格数组构造时，std::span 自动推导固定大小。
   - 从 std::vector 构造时，生成动态大小的 std::span。
3. **兼容性**：
   - std::span 统一了不同连续容器的接口（array、vector、数组）。
   - 动态 std::span 更灵活，固定 std::span 提供编译期检查。
4. **函数参数**：
   - fn1 接受任意大小的序列。
   - fn2 限制为 1024 个元素，类型安全但不灵活。

------

**运行流程总结**

- **arr1**：从 std::array 创建固定大小 std::span<int, 1024>。
- **arr2**：从 C 风格数组创建固定大小 std::span<int, 1024>。
- **data3**：从 std::vector 传入 fn1，动态大小 std::span<int>。
- **fn2**：因大小不匹配无法使用 data3。

------

**可能的改进或注意事项**

1. **错误注释**：
   - 修正 delctype 为 decltype。
2. **实际使用**：
   - fn1 和 fn2 未使用 data 的值，仅累加索引，可改为操作 data[i]。
3. **输出结果**：
   - 当前代码无输出，可添加 std::cout << fn1(data3) << '\n';。

------

**总结**

- **std::span**：提供统一的连续内存视图，支持动态和固定大小。
- **用途**：增强代码的通用性，避免手动传递指针和长度。
- **类型安全**：固定大小 std::span 在编译期验证长度。

如果你有具体问题（例如 std::span 的其他用法），欢迎提问！