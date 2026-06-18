### C++26 Removing deprecated typedef std::allocator::is_always_equal 

从 C++11 开始，Allocator 模型变得越来越复杂。

在 C++17 中，标准引入了：

```cpp
std::allocator_traits<Alloc>::is_always_equal
```

用于告诉容器：

```text
任意两个 Alloc 对象是否总是等价
```

例如：

```cpp
std::allocator<int> a1;
std::allocator<int> a2;
```

它们实际上没有状态（stateless）：

```cpp
a1 == a2
```

永远成立。

因此：

```cpp
std::allocator_traits<std::allocator<int>>::is_always_equal::value
```

为：

```cpp
true
```

------

### C++17 的设计

当时标准库同时提供了两种访问方式：

```cpp
std::allocator<T>::is_always_equal
```

和

```cpp
std::allocator_traits<std::allocator<T>>::is_always_equal
```

例如：

```cpp
using A = std::allocator<int>;

static_assert(A::is_always_equal::value);
```

以及：

```cpp
static_assert(
    std::allocator_traits<A>::is_always_equal::value
);
```

两者效果相同。

------

### 问题

第一种实际上是冗余的。

因为标准早已规定：

```cpp
allocator_traits
```

才是 Allocator 属性的统一访问入口。

现代代码应该写：

```cpp
std::allocator_traits<Alloc>::xxx
```

而不是：

```cpp
Alloc::xxx
```

例如：

```cpp
using pointer =
    std::allocator_traits<Alloc>::pointer;
```

而不是：

```cpp
Alloc::pointer
```

------

### C++20 开始弃用

从 C++20 起：

```cpp
std::allocator<T>::is_always_equal
```

被标记为：

```cpp
[[deprecated]]
```

即：

```text
Deprecated in C++20
```

标准建议改用：

```cpp
std::allocator_traits
```

接口。

------

### C++26 正式移除

C++26 删除了：

```cpp
template<class T>
class allocator
{
public:
    using is_always_equal = true_type; // 删除
};
```

因此下面代码在 C++26 中不再保证可用：

```cpp
using A = std::allocator<int>;

A::is_always_equal::value;
```

可能编译失败。

------

### 正确写法

应该改成：

```cpp
using A = std::allocator<int>;

static_assert(
    std::allocator_traits<A>::is_always_equal::value
);
```

或者 C++17 起：

```cpp
static_assert(
    std::allocator_traits<A>::is_always_equal::value
);
```

------

### 为什么可以删除？

因为对于标准 allocator：

```cpp
std::allocator<T>
```

标准已经规定：

```cpp
std::allocator_traits<std::allocator<T>>::is_always_equal
```

始终为：

```cpp
std::true_type
```

即：

```cpp
static_assert(
    std::allocator_traits<
        std::allocator<int>
    >::is_always_equal::value
);
```

永远成立。

删除成员 typedef 不会影响功能。

------

### 实际影响

绝大多数代码：

```cpp
std::vector<int>
std::map<int,int>
std::string
```

完全不会受到影响。

受影响的主要是：

#### 老代码

```cpp
template<class Alloc>
void foo()
{
    if constexpr (Alloc::is_always_equal::value)
    {
        ...
    }
}
```

需要改为：

```cpp
template<class Alloc>
void foo()
{
    if constexpr (
        std::allocator_traits<Alloc>
            ::is_always_equal::value)
    {
        ...
    }
}
```

------

### 一个完整示例

旧代码（C++20 弃用，C++26 删除）：

```cpp
#include <memory>

int main()
{
    using Alloc = std::allocator<int>;

    static_assert(
        Alloc::is_always_equal::value
    );
}
```

新代码：

```cpp
#include <memory>

int main()
{
    using Alloc = std::allocator<int>;

    static_assert(
        std::allocator_traits<Alloc>
            ::is_always_equal::value
    );
}
```

------

### 与其他删除项的关系

C++26 同时删除了一批从 C++17/C++20 就已经废弃的 Allocator 成员：

```cpp
allocator<T>::pointer
allocator<T>::const_pointer
allocator<T>::reference
allocator<T>::const_reference
allocator<T>::rebind
allocator<T>::is_always_equal
```

统一迁移到：

```cpp
std::allocator_traits
```

体系。

例如：

```cpp
std::allocator_traits<Alloc>::pointer
std::allocator_traits<Alloc>::rebind_alloc<U>
std::allocator_traits<Alloc>::is_always_equal
```

------

总结：

| 项目                                            | C++20               | C++26        |
| ----------------------------------------------- | ------------------- | ------------ |
| `std::allocator<T>::is_always_equal`            | Deprecated          | Removed      |
| `std::allocator_traits<Alloc>::is_always_equal` | 推荐使用            | 唯一标准方式 |
| 功能变化                                        | 无                  | 无           |
| ABI影响                                         | 基本无              | 基本无       |
| 需要修改代码                                    | 少量旧Allocator代码 | 是           |

本质上，这是 C++26 对 Allocator 体系的一次历史包袱清理：**所有 Allocator 属性统一通过 `std::allocator_traits` 访问，不再直接从 `std::allocator` 获取。**