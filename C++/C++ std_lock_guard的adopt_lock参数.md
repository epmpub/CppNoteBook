# std::lock_guard` 的 `adopt_lock参数

在 C++ 中，`std::lock_guard` 的 `adopt_lock` 参数用于 **接管已经锁定的互斥量**，而不是尝试重新加锁。它的作用是指示 `lock_guard` 对象 **管理一个已经被当前线程锁定的互斥量**，并在析构时自动释放锁。

---

## **`adopt_lock` 的作用**
1. **不尝试加锁**：
   - 通常 `std::lock_guard` 会在构造时自动调用 `mutex.lock()`。
   - 但如果使用 `adopt_lock`，`lock_guard` **不会再次加锁**，而是假设互斥量已经被当前线程锁定。

2. **在析构时自动解锁**：
   - 即使使用 `adopt_lock`，`lock_guard` 仍然会在析构时调用 `mutex.unlock()`，确保锁被释放。

3. **用于手动加锁后的 RAII 管理**：
   - 适用于先调用 `mutex.lock()`，再用 `lock_guard` 管理锁释放的情况。

---

## **使用场景**
`adopt_lock` 主要用于以下情况：
- 当需要 **手动加锁**（如使用 `std::lock()` 同时锁定多个互斥量以避免死锁）。
- 之后再用 `lock_guard` 管理锁的生命周期，确保异常安全。

### **示例代码**
```cpp
#include <mutex>
#include <thread>

std::mutex m1, m2;

void safe_operation() {
    // 手动锁定多个互斥量（避免死锁）
    std::lock(m1, m2);

    // 用 lock_guard 接管已锁定的互斥量（adopt_lock 表示不重复加锁）
    std::lock_guard<std::mutex> lck1(m1, std::adopt_lock);
    std::lock_guard<std::mutex> lck2(m2, std::adopt_lock);

    // 临界区操作
    // ... 

    // lck1 和 lck2 析构时自动解锁 m1 和 m2
}
```

---

## **对比普通 `lock_guard`**
| 行为               | `std::lock_guard`（默认）   | `std::lock_guard` + `adopt_lock` |
| ------------------ | --------------------------- | -------------------------------- |
| **构造时是否加锁** | ✅ 是（调用 `mutex.lock()`） | ❌ 否（假设已锁定）               |
| **析构时是否解锁** | ✅ 是                        | ✅ 是                             |
| **适用场景**       | 直接管理单个锁              | 接管手动锁定的互斥量             |

---

## **为什么需要 `adopt_lock`？**
1. **避免重复加锁**：
   - 如果互斥量已经锁定，再次调用 `lock()` 会导致未定义行为（通常是死锁）。
   - `adopt_lock` 告诉 `lock_guard` 不要重复加锁。

2. **配合 `std::lock()` 使用**：
   - `std::lock(m1, m2)` 可以安全地同时锁定多个互斥量（避免死锁）。
   - 之后用 `lock_guard + adopt_lock` 管理释放，确保异常安全。

---

## **总结**
- **`adopt_lock`** 表示 `lock_guard` 接管一个 **已经锁定** 的互斥量。
- 它 **不尝试加锁**，但会 **在析构时解锁**。
- 主要用途：
  - 配合 `std::lock()` 实现多互斥量安全锁定。
  - 将手动锁转换为 RAII 管理，确保异常安全。

### **替代方案（C++17+）**
C++17 引入了 `std::scoped_lock`，可以更简洁地锁定多个互斥量：
```cpp
std::scoped_lock lck(m1, m2);  // 自动锁定，无需手动调用 std::lock()
```
但在需要手动控制加锁顺序时，`lock_guard + adopt_lock` 仍然有用。