#### **`DR11/20: cv-qualified types in std::atomic and std::atomic_ref`**

？？？volatile 究竟有什么作用？？？

 是 C++26 通过提案 **P3323R1** 解决的缺陷报告（Defect Report），对应 LWG 问题 #4069 和 #3508。

### 背景与问题
- C++20 引入 `std::atomic_ref<T>` 时，`std::atomic<T>` 和 `std::atomic_ref<T>` 对 **cv-qualified**（`const` / `volatile` 限定的）类型的支持不清晰。
- CWG 2094 让 `is_trivially_copyable_v<volatile T>`（对于整数、指针、浮点等类型）变为 `true`，导致一些实现（如 MSVC）允许 `std::atomic<volatile T>`，但特殊化（specializations）往往只针对 cv-unqualified 类型，造成不一致和不可用。
- `std::atomic_ref<volatile T>` 在与遗留 `volatile` 接口（共享内存、硬件等）交互时非常有用，但之前实现支持不完整。

### 主要变更（C++26）

#### 1. **`std::atomic<T>`**
- **明确禁止** cv-qualified 类型（`const T` 或 `volatile T`）。
- 现在要求 `same_as<T, remove_cv_t<T>>` 为 `true`，否则程序 ill-formed。
- **理由**：`volatile atomic<int>` 已经能满足大多数需求，没必要支持 `atomic<volatile int>`。

**效果**：`std::atomic<volatile int>`、`std::atomic<const int>` 等现在是 ill-formed（之前部分实现可能接受但行为不完整）。

#### 2. **`std::atomic_ref<T>`**（改进支持）
`atomic_ref` 对 cv-qualified `T` 有**条件支持**，这是提案重点改进的地方：

- **value_type** 定义为 `remove_cv_t<T>`。
- **const-qualified**（`atomic_ref<const T>`）：
  - 只允许 **只读** 操作：`load()`、`operator T()` 等。
  - 禁止修改操作（`store`、`exchange`、`fetch_add`、`compare_exchange`、`wait`/`notify` 等）——这些操作会加上 `Constraints: is_const_v<T> is false`。
- **volatile-qualified**（`atomic_ref<volatile T>`）：
  - 只允许 **lock-free** 的情况（`is_always_lock_free == true`，或运行时 `is_lock_free()` 返回 `true`）。
  - 如果不是 lock-free，则程序 ill-formed（`The program is ill-formed if is_always_lock_free is false and is_volatile_v<T> is true`）。
- 特殊化（integral、floating-point、pointer）现在支持 **possibly cv-qualified** 类型。

**构造**：`explicit atomic_ref(T& obj);` —— `T` 可以是 cv-qualified，引用的是带 cv 的对象。

### 为什么这样设计？
- `atomic_ref` 的目标是方便对**已有非 atomic 对象**（尤其是历史遗留的 `volatile` 对象）进行原子操作。
- `const` 只读场景合理（如多线程只读原子视图）。
- `volatile` 限制为 lock-free 是为了实现可靠性和性能（非 lock-free 通常需要 mutex，不适合 volatile 语义）。

### 示例

```cpp
// C++26
volatile int v = 42;
std::atomic_ref<volatile int> ar(v);  // OK（如果 lock-free）

ar.store(100);                        // OK
int x = ar.load();                    // OK

const int c = 10;
std::atomic_ref<const int> ar_const(c);
int y = ar_const.load();              // OK
// ar_const.store(20);                // 编译错误（const）
```

```cpp
// std::atomic 不允许
// std::atomic<volatile int> a;       // ill-formed in C++26
volatile std::atomic<int> va;         // 推荐方式
```

### 实际影响
- 修复了之前的不一致性。
- 提升了与遗留代码（尤其是使用 `volatile` 的共享内存/硬件代码）的互操作性。
- 编译器/标准库需更新实现（libc++、libstdc++ 等正在跟进）。

更多细节可参考：
- [P3323R1 提案](https://wg21.link/P3323R1)
- cppreference `std::atomic_ref`（已更新 DR 表格）。

这个 DR 让 `std::atomic` / `std::atomic_ref` 对 cv 限定的处理更加清晰、一致且实用。