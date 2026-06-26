这三个都是 Allocator Traits 体系中的重要成员，主要用于解决**容器复制、移动、交换以及不同类型分配器之间的转换**问题。

先看一个典型 Allocator：

```cpp
template<class T>
struct MyAllocator
{
    using value_type = T;
};
```

实际上标准库更关心的是：

```cpp
std::allocator_traits<MyAllocator<T>>
```

中的各种属性。

------

## 1. rebind

### 解决什么问题？

容器存储的类型和实际分配的类型可能不同。

例如：

```cpp
std::list<int>
```

表面上存储：

```cpp
int
```

实际上分配的是：

```cpp
struct ListNode
{
    ListNode* prev;
    ListNode* next;
    int value;
};
```

因此：

```text
Allocator<int>
        ↓
Allocator<ListNode>
```

需要一种转换机制。

------

### C++03 时代

Allocator 必须提供：

```cpp
template<class U>
struct rebind
{
    using other = MyAllocator<U>;
};
```

例如：

```cpp
template<class T>
struct MyAllocator
{
    using value_type = T;

    template<class U>
    struct rebind
    {
        using other = MyAllocator<U>;
    };
};
```

使用：

```cpp
using NodeAlloc =
    MyAllocator<int>::rebind<ListNode>::other;
```

得到：

```cpp
MyAllocator<ListNode>
```

------

### C++11之后

标准库统一通过：

```cpp
std::allocator_traits<Alloc>::rebind_alloc<U>
```

实现：

```cpp
using NodeAlloc =
    std::allocator_traits<
        MyAllocator<int>
    >::rebind_alloc<ListNode>;
```

因此 C++26 删除了：

```cpp
allocator::rebind
```

这个历史遗留接口。

------

## 2. is_always_equal

### 解决什么问题？

判断两个 Allocator 是否永远等价。

例如：

```cpp
std::allocator<int> a1;
std::allocator<int> a2;
```

实际上：

```cpp
a1 == a2
```

永远成立。

因为：

```cpp
std::allocator
```

没有状态。

------

### Stateless Allocator

```cpp
template<class T>
struct MyAllocator
{
    using value_type = T;
};
```

任何实例都一样：

```cpp
MyAllocator<int> a1;
MyAllocator<int> a2;
```

此时：

```cpp
using is_always_equal =
    std::true_type;
```

------

### Stateful Allocator

例如：

```cpp
template<class T>
struct PoolAllocator
{
    MemoryPool* pool;
};
```

这里：

```cpp
PoolAllocator<int> a1{pool1};
PoolAllocator<int> a2{pool2};
```

可能：

```cpp
a1 != a2
```

因此：

```cpp
using is_always_equal =
    std::false_type;
```

------

### 为什么容器关心这个？

例如：

```cpp
std::vector<int, Alloc> v1;
std::vector<int, Alloc> v2;
```

执行：

```cpp
v1.swap(v2);
```

如果：

```cpp
is_always_equal == true
```

容器可直接交换内部指针：

```text
O(1)
```

如果：

```cpp
is_always_equal == false
```

可能需要额外检查 allocator 是否兼容。

------

## 3. propagate_on_container_move_assignment

简称：

```cpp
POCMA
```

这是 Allocator 模型里最难理解的一个。

------

### 问题背景

考虑：

```cpp
std::vector<int, Alloc> a;
std::vector<int, Alloc> b;
```

执行：

```cpp
a = std::move(b);
```

问题：

```text
b 的 allocator
是否应该一起移动给 a？
```

------

### 情况1

Allocator 无状态：

```cpp
std::allocator<int>
```

无所谓。

------

### 情况2

Allocator 有状态：

```cpp
PoolAllocator<int>
```

例如：

```cpp
a.pool = pool1;
b.pool = pool2;
```

执行：

```cpp
a = std::move(b);
```

有两种策略。

------

### 策略A：传播 allocator

```cpp
using propagate_on_container_move_assignment =
    std::true_type;
```

结果：

```text
a.pool = pool2
```

同时：

```text
a 接管 b 的内存
```

只需交换指针：

```text
O(1)
```

示意：

```text
Before

a -> pool1
b -> pool2

After

a -> pool2
b -> empty
```

------

### 策略B：不传播 allocator

```cpp
using propagate_on_container_move_assignment =
    std::false_type;
```

此时：

```text
a.pool 仍然是 pool1
```

而：

```text
b.pool 是 pool2
```

由于 pool2 分配的内存不能交给 pool1 管理：

```text
不能直接偷走指针
```

容器必须：

```text
重新分配
逐元素移动
释放旧数据
```

复杂度变成：

```text
O(n)
```

------

### 标准库内部伪代码

大致类似：

```cpp
if constexpr (
    allocator_traits<Alloc>::
    propagate_on_container_move_assignment::value
)
{
    allocator_ = rhs.allocator_;
    steal_storage(rhs);
}
else
{
    move_elements(rhs);
}
```

------

## 三者关系总结

| 属性                                     | 作用                                 | 典型场景           |
| ---------------------------------------- | ------------------------------------ | ------------------ |
| `rebind_alloc<U>`                        | Allocator 转 Allocator               | list/map 节点分配  |
| `is_always_equal`                        | 任意 allocator 是否总相等            | swap、move 优化    |
| `propagate_on_container_move_assignment` | move assignment 时是否传播 allocator | `a = std::move(b)` |

对于现代 C++ 开发者：

- `rebind` 主要是容器实现者关心。
- `is_always_equal` 影响容器能否 O(1) 移动和交换。
- `propagate_on_container_move_assignment` 决定移动赋值到底是“偷指针”还是“逐元素移动”。

这三个属性共同决定了标准容器在复制、移动、交换时能否保持 **常数时间复杂度**。