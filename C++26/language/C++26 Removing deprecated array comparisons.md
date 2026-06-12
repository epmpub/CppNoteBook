## Removing deprecated array comparisons

在 C++26 中，**[P2865R5 提案（Removing deprecated array comparisons）](https://github.com/cplusplus/papers/issues/1796)** 迈出了代码安全清理的坚实一步：**正式将两个数组（Array）之间进行直接比较（如 `==` 或 `!=`）的行为，从 C++20 标记的“弃用（Deprecated）”状态彻底升级为“编译错误（ill-formed / Removed）”**。

这顺应了 C++26 强化类型安全、消灭隐式逻辑陷阱的核心主旨。

------

## 极具欺骗性的“指针隐式退化”

在 C++20 之前，允许对两个原生数组直接进行 `==` 比较。这种设计非常危险，因为它**不是**在比较两个数组里的内容是否相等，而是触发了 C 语言遗留的**指针隐式退化（Pointer Decay）**。

##  旧标准的荒谬行为

```cpp
int a[] = {1, 2, 3};
int b[] = {1, 2, 3};

// C++17 及更早：允许编译！但结果永远为 false！
// 内部本质在比较：&a[0] == &b[0] （两个不同的内存地址）
if (a == b) { 
    // 永远无法到达这里
}
```

这种语法具有极强的误导性，许多初学者（甚至是资深开发者在粗心时）会误以为这行代码会像 `std::vector` 或 `std::string` 一样去按元素逐个比对。

为了阻止这种低级 Bug，**C++20 正式宣布弃用（Deprecated）这一行为**，编译时会发出警告。而到了 **C++26，它被彻底禁止**。

------

## C++26 的彻底改变

在 C++26 编译器下，直接比较两个原生数组将**直接导致编译失败**。

## C++26 无法通过编译的代码

```cpp
void compare_arrays() {
    int arr1[3] = {1, 2, 3};
    int arr2[3] = {1, 2, 3};

    // C++26 编译错误！error: comparison between two arrays is ill-formed
    if (arr1 == arr2) { /* ... */ } 
}
```

##  现代 C++ 的正确替代方案

如果你在 C++26 代码中需要比较两个数组，必须采用以下明确、类型安全的现代替代方案：

## 方案一：升级为 `std::array`（最推荐）

原生数组是 C 语言的遗留物。在现代 C++ 中，应该用 `std::array` 替换它们。`std::array` 完美重载了 `==` 运算符，能够符合直觉地进行**内容比对**：

```cpp
#include <array>

std::array<int, 3> arr1 = {1, 2, 3};
std::array<int, 3> arr2 = {1, 2, 3};

// C++26 完美通过，结果为 true
if (arr1 == arr2) { 
    // 内容完全一致！
}
```

## 方案二：使用 `std::equal` 算法（针对无法停用原生数组的场景）

如果你必须处理标准原生数组（例如维护底层硬件驱动或老旧标准库接口），请使用 `<algorithm>` 头文件中的 `std::equal`：

```cpp
#include <algorithm>

int arr1[3] = {1, 2, 3};
int arr2[3] = {1, 2, 3};

// 安全地按值比对两个原生数组的内容
if (std::equal(std::begin(arr1), std::end(arr1), std::begin(arr2))) {
    // 内容完全一致！
}
```

## 方案三：使用 C++20 范围库 `std::ranges::equal`（更现代的写法）

利用 Ranges 库，你可以直接传入整个原生数组，代码同样非常精简：

```cpp
#include <algorithm>

int arr1[3] = {1, 2, 3};
int arr2[3] = {1, 2, 3};

// C++20/26 推荐：直接传递原生数组进行内容比较
if (std::ranges::equal(arr1, arr2)) {
    // 内容完全一致！
}
```

##  总结

C++26 通过移除数组直接比较，再次向开发者表明了态度：**任何可能引发歧义和隐藏 UB/Bug 的旧语法，都在逐步退出历史舞台**。

你负责的代码库里是否有大量老旧的原生数组（C-style array）需要向现代 C++ 标准迁移？我们可以聊聊如何借助 **C++20/26 的 `std::span`** 或者自动化重构工具安全地升级它们！