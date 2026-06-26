std::inplace_vector<T, N> 

C++26 新引入的容器（定义在 <inplace_vector> 头文件中），全称描述为 “dynamically-resizable vector with fixed capacity inplace storage” —— 具有固定容量的原地（in-place）动态可调整大小的向量。 

核心特点

- 模板参数：
  - T：元素类型。
  - N：编译期固定容量（capacity() 始终返回 N，常量）。
- 存储方式：
  - 元素完全存储在对象内部（in-place / embedded storage），类似 std::array<T, N>。
  - 不进行任何动态内存分配（heap allocation），非常适合嵌入式系统、实时系统、内核、游戏引擎等不允许或不希望分配内存的场景。
  - 元素连续存储（contiguous），支持指针算术、随机访问迭代器等。
- 动态大小：
  - size() 可以从 0 动态变化到最多 N。
  - 支持 push_back、pop_back、insert、erase、resize 等类似 std::vector 的操作（但容量永远不会超过 N）。
- 与现有容器的对比：

| 容器                      | 大小（size） | 容量（capacity） | 存储位置   | 内存分配 | 初始化行为            |
| ------------------------- | ------------ | ---------------- | ---------- | -------- | --------------------- |
| std::array<T, N>          | 固定 = N     | 固定 = N         | 对象内部   | 无       | 默认构造所有 N 个元素 |
| std::vector<T>            | 动态         | 动态（可增长）   | 堆（heap） | 有       | 只构造实际元素        |
| std::inplace_vector<T, N> | 动态         | 固定 = N         | 对象内部   | 无       | 只构造实际元素        |

inplace_vector 结合了 vector 的灵活性和 array 的零分配特性。 

stackoverflow.com

主要优势与适用场景

- 零分配：嵌入式、无堆环境、硬实时系统（避免分配不确定性）。
- 已知最大大小：编译期知道最坏情况占用空间。
- 更好的默认构造：不像 std::array，默认构造的 inplace_vector 不构造任何元素（size=0），更高效。
- 可作为 std::vector 的 drop-in 替代（在容量足够的情况下），接口非常相似。
- 性能：没有分配开销，缓存友好（contiguous）。

基本用法示例

cpp

```cpp
#include <inplace_vector>
#include <cassert>

int main() {
    std::inplace_vector<int, 10> v;  // 容量固定为10，可存放0~10个int

    v.push_back(1);
    v.push_back(2);
    v.resize(5, 42);                 // size=5

    assert(v.size() == 5);
    assert(v.capacity() == 10);      // 始终是10
    assert(v.max_size() == 10);

    // 超过容量会抛出 std::bad_alloc 或 std::length_error（视实现）
    // v.push_back(999); // 如果 size==10 会失败

    v.pop_back();
    v.clear();  // size=0，但容量仍为10
}
```

重要成员函数（类似 vector）

- size(), capacity()（常量）, empty(), max_size()
  size()、capacity()（常量）、empty()、max_size()
- push_back, emplace_back, pop_back
  push_back、emplace_back、pop_back
- insert, erase, clear
  插入、删除、清除
- resize（可带默认值）
- at(), operator[], front(), back(), data()
  at()、operator[]、front()、back()、data()
- 迭代器支持（begin()/end() 等）
- swap（高效）

注意：

- 超过容量时，push_back/insert 等操作会抛出异常（通常 std::bad_alloc）。
- T 需要满足一定的要求（如可移动/可复制，取决于操作）。
- 如果 T 是 trivially copyable，那么整个 inplace_vector
  inplace_vector 也可能可以被轻松复制（具体实现方式取决于相关代码）。 也可能是 trivially copyable（实现相关）。 reddit.com

设计背景（P0843）

该容器来自提案 P0843（经过多轮修订，已进入 C++26）。它填补了 std::array（固定大小）和 std::vector（动态堆分配）之间的空白。 

open-std.org

总结：当你知道最大元素数量，不想/不能进行堆分配，但又需要动态增删元素时，std::inplace_vector<T, N> 是最佳选择。它是 C++26 中非常实用且“现代”的容器之一。 更多细节可参考 cppreference：std::inplace_vector。
更多详细信息请参阅 cppreference：std::inplace_vector。