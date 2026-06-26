**Constexpr stable sorting** 是 C++26 标准库的一项改进：使稳定排序算法能够在常量求值（constant evaluation）期间执行。

简单来说：

```cpp
constexpr auto result = ...;
```

现在不仅可以在编译期排序，而且可以进行**稳定排序（stable sort）**。

------

## 1. 什么是 stable sort

稳定排序（stable sort）的定义：

> 如果两个元素比较结果相等，那么排序后仍保持原有相对顺序。

例如：

```cpp
struct Employee
{
    int age;
    const char* name;
};
```

原始数据：

```cpp
{
    {20, "Tom"},
    {20, "Jerry"},
    {30, "Alice"}
}
```

按 age 排序：

```cpp
Tom
Jerry
Alice
```

因为：

```cpp
20 == 20
```

所以：

```cpp
Tom
Jerry
```

顺序保持不变。

------

而普通：

```cpp
std::sort
```

不保证这一点。

可能得到：

```cpp
Jerry
Tom
Alice
```

------

## 2. C++20 已经支持 constexpr sort

C++20 开始：

```cpp
std::sort
```

已经可以 constexpr。

例如：

```cpp
constexpr auto test()
{
    std::array a{4,3,2,1};

    std::sort(a.begin(), a.end());

    return a;
}

static_assert(test()[0] == 1);
```

成立。

------

但：

```cpp
std::stable_sort
```

不行。

------

原因是实现方式。

------

## 3. 为什么 stable_sort 很难 constexpr

### 普通 sort

典型实现：

- quicksort
- introsort
- heapsort

主要依赖：

```cpp
swap
compare
```

即可。

------

### stable_sort

典型实现：

```text
merge sort
```

需要：

- 临时缓冲区
- 大量移动元素
- 动态内存管理

例如：

```cpp
buffer = new T[n];
```

然后：

```cpp
merge(...)
```

最后：

```cpp
delete[] buffer;
```

------

而早期 constexpr 环境中：

```cpp
new
delete
```

支持有限。

因此标准库无法保证：

```cpp
stable_sort
```

在常量求值期间工作。

------

## 4. C++26 的改进

C++26 要求：

```cpp
std::stable_sort
```

可 constexpr。

即：

```cpp
constexpr auto sorted()
{
    std::array a{
        4,3,1,2
    };

    std::stable_sort(
        a.begin(),
        a.end()
    );

    return a;
}
```

合法。

------

甚至：

```cpp
static_assert(sorted()[0] == 1);
```

成立。

------

## 5. 典型示例

### 编译期稳定排序

```cpp
#include <array>
#include <algorithm>

struct Item
{
    int key;
    char id;
};

constexpr auto make_sorted()
{
    std::array items{
        Item{2,'A'},
        Item{1,'B'},
        Item{2,'C'},
        Item{1,'D'}
    };

    std::stable_sort(
        items.begin(),
        items.end(),
        [](auto const& a, auto const& b)
        {
            return a.key < b.key;
        });

    return items;
}

constexpr auto result = make_sorted();

static_assert(result[0].id == 'B');
static_assert(result[1].id == 'D');
static_assert(result[2].id == 'A');
static_assert(result[3].id == 'C');
```

保持原顺序：

```text
B D A C
```

------

## 6. 影响的不只是 stable_sort

相关算法也一起变成 constexpr：

包括：

```cpp
std::stable_sort
std::stable_partition
```

以及依赖稳定归并思想的实现。

具体取决于最终标准库实现。

------

## 7. 实现层面怎么做到

标准没有规定实现。

主流库通常有两种方案：

### 方案1：constexpr merge sort

编译期直接执行归并排序：

```text
split
merge
split
merge
...
```

全部在常量求值器里完成。

------

### 方案2：constexpr temporary buffer

利用现代 constexpr 动态分配能力：

C++20 引入：

```cpp
constexpr new
```

后，可以实现：

```cpp
constexpr vector-like buffer
```

用于 merge。

------

libc++ 和 libstdc++ 当前实验实现基本走这个方向。

------

## 8. 为什么这个特性有价值

以前：

```cpp
constexpr lookup_table
```

只能使用：

```cpp
std::sort
```

------

如果需要：

- 多关键字排序
- 保留原始顺序
- 数据预处理

就比较麻烦。

------

现在：

```cpp
constexpr
```

环境下可以直接：

```cpp
std::stable_sort
```

构造复杂静态表。

例如：

- 编译期反射数据整理
- 编译期协议表生成
- 编译期状态机构造
- 编译期 DSL 元数据处理

------

## 9. 与 constexpr vector 的关系

近年来标准库一直在推动：

```cpp
constexpr
```

成为真正的编译期执行环境。

时间线大致：

| 标准  | 变化                      |
| ----- | ------------------------- |
| C++20 | constexpr sort            |
| C++20 | constexpr vector 基础能力 |
| C++23 | 更多 constexpr algorithms |
| C++26 | constexpr stable_sort     |

方向很明确：

> 让标准算法在编译期与运行期拥有尽可能一致的能力。

------

## 10. 总结

Constexpr stable sorting 的核心内容是：

```cpp
std::stable_sort
```

以及相关稳定排序算法变为 constexpr 可用。

其意义在于：

- 支持编译期稳定排序
- 保证相等元素顺序不变
- 允许在 `static_assert` 和 `constexpr` 环境中使用
- 进一步完善“编译期 STL”能力
- 为反射、元编程、代码生成等 C++26 特性提供基础设施

最直接的效果是：

```cpp
constexpr std::array data = ...;

std::stable_sort(
    data.begin(),
    data.end(),
    comp
);
```

终于成为标准支持的写法。