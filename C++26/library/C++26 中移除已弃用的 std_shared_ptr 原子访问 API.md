**C++26 中移除已弃用的 `std::shared_ptr` 原子访问 API**

### 1. 什么是被移除的 API？

C++11 引入了以下**自由函数**（free functions），用于对 `std::shared_ptr` 进行原子操作：

```cpp
std::atomic_is_lock_free(const shared_ptr<T>* p);
std::atomic_load(const shared_ptr<T>* p);
std::atomic_load_explicit(const shared_ptr<T>* p, memory_order);
std::atomic_store(shared_ptr<T>* p, shared_ptr<T> r);
std::atomic_store_explicit(shared_ptr<T>* p, shared_ptr<T> r, memory_order);
std::atomic_exchange(shared_ptr<T>* p, shared_ptr<T> r);
std::atomic_exchange_explicit(...);
std::atomic_compare_exchange_weak(...);
std::atomic_compare_exchange_strong(...);
std::atomic_compare_exchange_weak_explicit(...);
std::atomic_compare_exchange_strong_explicit(...);
```

这些函数位于 `<memory>` 头文件中，属于 **C 风格** 的原子接口。

### 2. 历史

- **C++11**：引入这些函数。
- **C++20**（P0718R2）：**标记为 deprecated**（弃用），推荐使用 `std::atomic<std::shared_ptr<T>>`。
- **C++26**（P2869R4）：**正式移除**。

提案：**P2869R4** — *Remove Deprecated `shared_ptr` Atomic Access APIs from C++26*

### 3. 为什么弃用并最终移除？

这些旧 API **容易误用**，存在严重的安全隐患：

- 用户经常直接操作普通的 `shared_ptr<T>`，同时又通过原子函数访问，导致**数据竞争（data race）** 和未定义行为。
- 正确的用法要求：**只能** 通过原子函数访问该 `shared_ptr`，不能直接使用它的成员函数（`operator*`、`operator->` 等），否则 UB。
- 接口设计不直观、不类型安全。
- C++20 引入了更好的替代方案：`std::atomic<std::shared_ptr<T>>`，它是一个**类型安全的** 原子包装器，使用起来更清晰，也更难误用。

### 4. 推荐新写法（C++20 及以上）

**旧代码（已弃用，C++26 无法编译）：**

```cpp
std::shared_ptr<int> p = std::make_shared<int>(42);
std::shared_ptr<int> q = std::atomic_load(&p);        // 旧
std::atomic_store(&p, std::make_shared<int>(100));    // 旧
```

**新代码（推荐）：**

```cpp
#include <memory>
#include <atomic>

std::atomic<std::shared_ptr<int>> p{std::make_shared<int>(42)};

auto q = p.load();                                 // 推荐
p.store(std::make_shared<int>(100));

std::shared_ptr<int> r = std::make_shared<int>(200);
p.exchange(r);

p.compare_exchange_strong(r, std::make_shared<int>(300));
```

或者使用成员函数风格：

```cpp
p.load(std::memory_order_acquire);
p.store(value, std::memory_order_release);
```

### 5. 迁移影响

- **编译错误**：C++26 模式下使用旧的 `atomic_*` 函数会直接报错（不再是警告）。
- **修复方式** 非常简单：把 `shared_ptr<T>` 变量改为 `atomic<shared_ptr<T>>`。
- 大多数现代代码已经在 C++20 后迁移完成，实际影响有限。
- 这属于 C++26 “清理长期弃用功能” 的一部分（类似移除 `strstream`、旧的 `polymorphic_allocator::destroy` 相关弃用等）。
