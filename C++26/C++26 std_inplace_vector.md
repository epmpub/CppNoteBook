## std::inplace_vector与 std::vector<T> 

std::inplace_vector<T, N>（C++26）与 std::vector<T> 的主要区别如下：1. 容量（Capacity）

1. 容量（容量）

- std::vector<T>：容量动态可变，可以无限增长（受内存限制）。capacity() 是运行时值，会根据需要自动扩容。
- std::inplace_vector<T, N>：容量在编译期固定为 N。capacity() 是 static constexpr，永远不会改变，也不会进行动态内存分配（除非 T 自身会分配）。
- 内存存储位置

- std::vector：元素存储在堆上（通过分配器，通常是 std::allocator）。
- std::inplace_vector：元素原地（inplace）存储在对象内部（类似于 std::array<T, N>），通常在栈上或嵌入到其他对象中。整个容器的大小 ≈ sizeof(T) * N + 少量元数据。
- API 相似性与行为差异两者接口非常接近（push_back、pop_back、insert、erase、resize 等都支持），但有以下关键差异：

| 特性          | std::vector<T> STD：：vector<T>     | std::inplace_vector<T, N> std：：inplace_vector<T，N>       |
| ------------- | ----------------------------------- | ----------------------------------------------------------- |
| 最大大小      | 动态（内存允许）                    | 固定为 N（超过会抛异常或 UB）                               |
| 内存分配      | 会动态分配/释放                     | 完全无分配（inplace）                                       |
| 移动/交换     | O(1)，通常不失效迭代器              | O(size)，会使所有迭代器失效（类似 array）                   |
| 移动后状态    | 通常变为空                          | 有效但未指定状态（若 T 是 trivially copyable 则行为更明确） |
| 默认构造/析构 | 对 trivial 类型 O(1)                | 可能 O(N)（取决于实现）                                     |
| 大小          | sizeof(vector) 很小（通常 24 字节） | 较大（≈ T*N）                                               |

4. 与 std::array<T, N> 的对比（常见疑问）

- std::array：容量固定，大小固定（始终有 N 个元素），构造时会默认初始化所有元素。
- std::inplace_vector：容量固定，但大小可变（size() 从 0 到 N），元素仅在插入时才构造，更灵活。
- 适用场景推荐使用 inplace_vector 的情况：

- 已知最大元素数量（编译期已知）。
- 禁止或不希望动态内存分配（嵌入式系统、实时系统、游戏引擎热点路径、内核/驱动）。
- 希望栈上/嵌入式分配，避免 heap 分配的开销和碎片。
- 需要 vector 风格的动态大小 API，但容量有上限。

继续使用 std::vector 的情况：

- 元素数量不确定或可能非常大。
- 希望最小化对象体积（vector 本身很小）。
- 需要频繁移动/交换容器本身（性能更好）。

总结：
inplace_vector 是 “固定容量 + 无堆分配 + vector-like API” 的混合体，特别适合性能敏感 + 内存受限的 DOP（面向数据编程）、ECS、嵌入式等场景。它不是替换 vector，而是补充一个重要的“无分配”选项。 

en.cppreference.com

需要具体代码示例（如初始化、性能对比）吗？