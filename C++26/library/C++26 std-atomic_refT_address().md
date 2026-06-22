**`std::atomic_ref<T>::address()`** 是 C++26 新增的成员函数（通过提案 **P2835** 引入）。

### 作用
返回 `std::atomic_ref<T>` 当前引用的底层对象的**地址**（即构造 `atomic_ref` 时传入的对象的指针）。

```cpp
constexpr T* address() const noexcept;   // (since C++26)
```

- **返回值**：指向被引用的对象的 `T*` 指针（内部存储的 `ptr`）。
- **特性**：`constexpr`、`noexcept`。

### 为什么需要这个函数？

`std::atomic_ref<T>`（C++20 引入）允许对已存在的非 `std::atomic` 对象进行原子操作，但它**不提供**直接获取底层对象地址的方式。这导致了一些实际问题：

- **遗留 API 迁移**：很多老代码使用 `volatile T*` 来传递“可并发访问”的对象，需要知道对象地址来做性能优化（如 contention-aware 数据结构、锁表索引等）。迁移到 `atomic_ref` 后无法获取地址。
- **原子访问 + 非原子访问混合**：在某些场景下（如最后一个线程完成工作后），可以切换回普通指针访问以获得更高性能。
- **数据结构元素访问**：例如对外部数组/容器元素进行原子操作时，需要基于 base 指针计算偏移。

**示例**（来自提案）：

```cpp
int data = 42;
std::atomic_ref<int> ref{data};

int* ptr = ref.address();        // 得到 &data
assert(ptr == &data);

*ptr = 100;   // 注意：只有在没有其他 live atomic_ref 时才能安全非原子访问，否则 UB
```

另一个典型用例（数组）：

```cpp
extern int arr[100];

void process(std::atomic_ref<int> base, size_t i) {
    auto* p = base.address();           // 得到数组首地址
    std::atomic_ref<int> elem{*(p + i)}; // 对特定元素创建 atomic_ref
    elem.fetch_add(1);
}
```

### 注意事项（非常重要）

1. **生命周期与 UB**：
   - 只要存在 **live** 的 `std::atomic_ref` 指向该对象，就**禁止**通过返回的普通指针进行**非原子**访问（否则数据竞争 → 未定义行为）。
   - 必须等到所有相关 `atomic_ref` 销毁后，才能安全地非原子访问。

2. **const 语义**：`address()` 返回 `T*`（即使 `atomic_ref` 是 `const` 的），因为 `atomic_ref` 的 constness 是 shallow（浅 const）。

3. **Feature Test Macro**：
   ```cpp
   #if __cpp_lib_atomic_ref >= 202411L
   // 支持 address()
   #endif
   ```

### 总结

- **用途**：让 `atomic_ref` 更实用，尤其在需要“地址感知”（address-aware）或混合原子/非原子访问的场景。
- **安全性**：函数本身是安全的，但**使用返回的指针**时必须严格遵守 `atomic_ref` 的生命周期规则。
- 这是 C++26 对 `std::atomic_ref` 的重要补充，使其能更好地替代旧的 `volatile*` 模式。

更多细节可参考 cppreference：[`std::atomic_ref::address`](https://en.cppreference.com/cpp/atomic/atomic_ref/address)。