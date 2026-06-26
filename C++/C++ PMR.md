#### PMR（Polymorphic Memory Resource）

它真正解决的是：

> “大量动态内存分配的 allocator 开销与碎片问题”。

尤其适合：

```text
大量短生命周期小对象
```

场景。

------

# 一、先理解“小对象分配为什么慢”

普通：

```cpp
new/delete
malloc/free
```

的问题：

------

# 1. 系统 allocator 很重

需要：

- thread synchronization
- free list 管理
- heap metadata
- fragmentation control

即使：

```cpp
new int
```

也可能很贵。

------

# 2. 小对象最糟糕

因为：

```text
对象很小
allocator 开销反而占大头
```

例如：

```cpp
std::string
std::list node
std::map node
json node
AST node
```

大量 tiny allocation。

------

# 3. cache locality 很差

普通 heap：

```text
对象 scattered everywhere
```

导致：

- cache miss
- TLB miss

------

# 二、PMR 是什么？

C++17：

# `std::pmr`

本质：

# allocator 抽象层

核心：

```cpp
memory_resource
```

------

普通 allocator：

```cpp
template<class T>
allocator<T>
```

是：

# compile-time allocator

------

PMR：

```cpp
memory_resource*
```

是：

# runtime polymorphic allocator

------

# 三、PMR 真正的核心价值

重点：

# allocator 可以“按场景切换”

例如：

```text
arena allocator
pool allocator
monotonic allocator
stack allocator
shared arena
```

都能：

# 统一接入 STL。

------

# 四、最关键：monotonic_buffer_resource

这是：

# PMR 提升小对象性能的核心。

------

例如：

```cpp
std::pmr::monotonic_buffer_resource pool;
```

它本质：

# bump allocator（线性分配器）

即：

```text
ptr += size
```

几乎：

# 零成本分配。

------

# 五、为什么它特别适合小对象？

普通 allocator：

```text
malloc:
  查 free list
  对齐
  metadata
  lock
```

------

monotonic：

```text
allocate:
  current += n
```

就结束。

------

因此：

# 小对象速度极快。

------

# 六、典型收益场景

------

# 1. parser / compiler

例如：

```text
AST node
token
symbol
```

全部：

- 小对象
- 生命周期一致

非常适合：

# arena allocation

------

# 2. JSON/XML DOM

```text
大量 node
```

------

# 3. request-scoped server objects

例如：

```text
一次 HTTP request
  创建几千对象
```

请求结束：

# 整个 arena 一次释放。

------

# 4. ECS/game object temp memory

------

# 七、为什么它性能高？

因为：

# 不需要单独 free

重点：

```text
arena 整体释放
```

而不是：

```text
一个个 delete
```

这：

# 极大降低 allocator 成本。

------

# 八、一个典型例子

------

# 普通 string

```cpp
std::vector<std::string>
```

可能：

# 每个 string 单独 heap allocation。

------

# PMR

```cpp
std::pmr::vector<std::pmr::string>
```

全部：

# 从同一个 memory arena 分配。

于是：

- locality 更好
- malloc 更少
- cache 更友好

------

# 九、但 PMR 不是万能加速器

这是重点。

------

# PMR 不会：

# magically optimize everything

如果：

```text
对象生命周期复杂
```

例如：

- 长期对象
- 随机释放
- 多线程共享

monotonic allocator：

# 反而可能更糟。

------

# 十、PMR 最大限制

------

# monotonic_buffer_resource

本质：

# “只增不减”

即：

```text
allocate OK
deallocate ignored
```

所以：

# 内存不会回收

直到：

```text
整个 resource 析构
```

------

因此：

# 不适合长期动态系统。

------

# 十一、真正解决“小对象”的通常不是 PMR 本身

而是：

# pool allocator / arena allocator

PMR：

只是：

# 标准化接口。

------

例如：

```text
pmr::unsynchronized_pool_resource
```

才真正针对：

# 小对象池化。

------

# 十二、pool_resource 做了什么？

它：

# 按 size class 管理 free list

例如：

```text
8 bytes pool
16 bytes pool
32 bytes pool
```

于是：

小对象：

# 不再频繁 malloc/free。

------

# 十三、和 jemalloc/tcmalloc 的关系

现代 allocator：

- jemalloc
- mimalloc
- tcmalloc

其实已经：

# 极度优化小对象。

因此：

很多时候：

```text
换 allocator
```

收益比 PMR 更明显。

------

# 十四、PMR 真正强在哪？

重点：

# “allocator propagation”

例如：

```cpp
pmr::vector<pmr::string>
```

内部：

# allocator 自动传播。

整个对象图：

都能：

# 使用同一个 arena。

这是传统 allocator 很难做到的。

------

# 十五、什么时候 PMR 提升巨大？

------

# 生命周期一致

例如：

```text
create all
destroy all
```

------

# 高频小对象

例如：

```text
millions of tiny allocations
```

------

# 临时对象很多

例如：

- parsing
- serialization
- query execution

------

# 十六、什么时候收益不大？

------

# 大对象

因为：

```text
allocation cost 占比小
```

------

# allocation 不频繁

------

# allocator 已很优秀

例如：

```text
jemalloc
mimalloc
```

------

# 十七、现代高性能系统实际怎么做？

通常：

```text
arena allocator
    +
object pool
    +
custom allocator
```

PMR：

只是：

# 标准库统一接口层。

------

# 十八、一句话总结

C++17 PMR：

# 可以显著改善“小对象大量分配”场景的性能。

但真正提升性能的原因不是“PMR”三个字。

而是：

# arena/pool allocation 模型。

PMR 的核心价值是：

> 让 STL 容器和整个对象图能够优雅地共享高性能 allocator。

------

# 最适合 PMR 的场景

| 场景                | 效果   |
| ------------------- | ------ |
| parser/compiler     | 极好   |
| JSON DOM            | 极好   |
| request memory      | 极好   |
| ECS/game temp alloc | 很好   |
| 高频小对象          | 很好   |
| 长生命周期随机释放  | 一般   |
| 大对象              | 不明显 |

------

可以把它理解成：

```text
PMR
≠ faster malloc

PMR
= allocator architecture for modern C++
```