Allocator 最初确实是为标准容器设计的，但它并不仅仅是“容器的内存分配器”。从标准角度看，Allocator 是一种**内存资源管理策略（memory allocation policy）**，容器只是最主要的使用者。

### 最常见的用途：标准容器

例如：

```cpp
std::vector<int> v;
```

实际上等价于：

```cpp
std::vector<
    int,
    std::allocator<int>
> v;
```

其中：

```cpp
std::allocator<int>
```

负责：

```text
申请内存
释放内存
构造对象
销毁对象
```

例如当 vector 扩容时：

```cpp
v.push_back(1);
```

内部可能执行：

```cpp
allocate()
construct()
```

扩容后：

```cpp
destroy()
deallocate()
```

------

### Allocator 并不属于容器

Allocator 本身是一个独立概念：

```cpp
std::allocator<int> alloc;
```

可以直接使用：

```cpp
int* p = alloc.allocate(10);

alloc.deallocate(p, 10);
```

虽然实际开发几乎没人这样写。

例如：

```cpp
#include <memory>

int main()
{
    std::allocator<int> alloc;

    int* p = alloc.allocate(3);

    p[0] = 10;
    p[1] = 20;
    p[2] = 30;

    alloc.deallocate(p, 3);
}
```

------

### 为什么要有 Allocator？

因为容器作者不希望把内存管理写死。

假设：

```cpp
std::vector<int>
```

内部固定使用：

```cpp
new[]
delete[]
```

那么：

```text
内存池
共享内存
NUMA内存
GPU内存
持久化内存
```

都无法接入。

Allocator 提供了一层抽象：

```text
vector
   ↓
allocator_traits
   ↓
Allocator
   ↓
具体内存来源
```

------

### 一个简单自定义 Allocator

例如统计分配次数：

```cpp
template<class T>
struct CountingAllocator
{
    using value_type = T;

    T* allocate(std::size_t n)
    {
        std::cout << "allocate "
                  << n*sizeof(T)
                  << " bytes\n";

        return static_cast<T*>(
            ::operator new(n*sizeof(T))
        );
    }

    void deallocate(T* p, std::size_t)
    {
        std::cout << "deallocate\n";

        ::operator delete(p);
    }
};
```

使用：

```cpp
std::vector<
    int,
    CountingAllocator<int>
> vec;
```

输出：

```text
allocate 4 bytes
allocate 8 bytes
allocate 16 bytes
...
```

------

### C++17 之后：Allocator 的地位下降

很多人发现 Allocator 接口太复杂。

例如：

```cpp
rebind
propagate_on_container_move_assignment
is_always_equal
```

这些机制非常晦涩。

因此 C++17 引入了：

Polymorphic Memory Resources（PMR）

头文件：

```cpp
#include <memory_resource>
```

例如：

```cpp
std::pmr::vector<int> v;
```

底层不再直接依赖 Allocator 类型，而依赖：

```cpp
std::pmr::memory_resource
```

运行时切换内存来源：

```cpp
std::pmr::monotonic_buffer_resource
std::pmr::unsynchronized_pool_resource
std::pmr::synchronized_pool_resource
```

例如：

```cpp
std::byte buffer[1024];

std::pmr::monotonic_buffer_resource pool{
    buffer,
    sizeof(buffer)
};

std::pmr::vector<int> vec{&pool};
```

整个 vector 都从：

```text
buffer[1024]
```

中分配内存。

------

### Allocator 在标准库中的使用范围

不仅仅是：

```cpp
std::vector
```

还包括：

```cpp
std::list
std::deque
std::map
std::set
std::unordered_map
std::unordered_set
std::basic_string
```

例如：

```cpp
std::basic_string<
    char,
    std::char_traits<char>,
    MyAllocator<char>
>
```

也是合法的。

------

### Allocator 不负责什么？

Allocator 只负责：

```text
内存管理
```

不负责：

```text
对象逻辑
容器逻辑
扩容策略
哈希算法
红黑树算法
排序算法
```

例如：

```cpp
std::map
```

的红黑树旋转：

```text
Allocator 不参与
```

Allocator 仅负责：

```text
节点申请
节点释放
```

------

### C++26 的现状

今天标准库内部大致是：

```text
容器
 ↓
allocator_traits
 ↓
Allocator
 ↓
operator new
```

或者：

```text
pmr容器
 ↓
polymorphic_allocator
 ↓
memory_resource
 ↓
具体内存池
```

所以可以把 Allocator 理解为：

> Allocator 是 C++ 标准库定义的一套“可插拔内存管理接口”。它最主要的使用场景是容器，但本质上并不限于容器，而是用于将“对象存储管理”和“容器逻辑”解耦。PMR 则是对这一思想的现代化改进。