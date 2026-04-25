#### unique_lock vs lock_guard

> 👉 **`std::condition_variable::wait` 必须接收 `std::unique_lock<std::mutex>`，不是因为“习惯”，而是因为它需要在内部“解锁→睡眠→再加锁”。`lock_guard` 做不到这一点。**

------

## 🧠 wait 背后到底发生了什么？

这一行：

```cpp
cv.wait(lock);
```

等价于（逻辑上）：

```cpp
while (!condition) {
    lock.unlock();      // ① 释放锁
    睡眠等待通知;        // ② 阻塞线程（不占 CPU）
    lock.lock();        // ③ 被唤醒后重新加锁
}
```

👉 关键是这三步必须是**原子配合**的，否则就会出严重问题。

------

## ❌ 为什么 `lock_guard` 不行？

`std::lock_guard` 的特点：

```cpp
{
    std::lock_guard<std::mutex> lock(mtx);
    // 自动加锁
}
// 自动解锁（作用域结束）
```

👉 它有两个致命限制：

### 1️⃣ **不能手动 unlock**

```cpp
lock.unlock(); // ❌ 不存在这个接口
```

👉 而 `wait()` 必须在内部 unlock

------

### 2️⃣ 生命周期不可控

`lock_guard`：

- 构造时加锁
- 析构时解锁

👉 中间不能“临时释放再拿回来”

------

## ✅ 为什么 `unique_lock` 可以？

`std::unique_lock` 的设计就是为这种场景准备的：

```cpp
std::unique_lock<std::mutex> lock(mtx);

lock.unlock(); // ✔ 可以
lock.lock();   // ✔ 可以
```

👉 所以 `condition_variable` 可以安全地：

- 暂时释放锁
- 等待通知
- 再重新获取锁

------

## 🔥 更深层原因（非常关键）

如果不能“原子地释放锁 + 进入等待”，会发生经典 bug：

### 🚨 Lost Wakeup（丢失唤醒）

#### 错误流程：

1. consumer 检查条件（发现不满足）
2. **准备进入等待（还没睡）**
3. producer 修改数据 + notify
4. consumer 开始睡眠 ❗

👉 结果：

- 通知已经发过了
- consumer 却睡死了

------

### ✅ condition_variable 如何解决？

`wait()` 内部保证：

> 👉 **“释放锁 + 进入等待”是一个原子操作**

这正是 `unique_lock` 能配合实现的

------

## 📌 为什么 API 就强制用 unique_lock？

函数签名：

```cpp
void wait(std::unique_lock<std::mutex>& lock);
```

👉 标准库直接强制你：

> “必须给我一个可控的锁对象，否则我不工作”

这是一种**设计层面的防错机制**

------

## 💡 对比总结

| 特性                    | lock_guard | unique_lock |
| ----------------------- | ---------- | ----------- |
| 自动加锁/解锁           | ✅          | ✅           |
| 手动 unlock             | ❌          | ✅           |
| 手动 lock               | ❌          | ✅           |
| 可转移所有权            | ❌          | ✅           |
| 用于 condition_variable | ❌          | ✅           |

------

## 🧠 一句话本质

> 👉 `condition_variable` 需要“可控的锁”，而 `unique_lock` 提供控制能力，`lock_guard` 只是一个简单 RAII 封装

------

## 🚀 再帮你升一层（理解到这里就很强了）

你可以这样记：

```text
lock_guard   = “我只负责上锁，不参与调度”
unique_lock  = “我是可编程的锁”
condition_variable = “线程调度器”
```

------

如果你想更深入一个层级，我可以继续讲这个（非常关键）：

👉 **为什么 wait 必须写成 `while + predicate`，而不是 `if`？**

这和“虚假唤醒 + happens-before”直接相关，是并发编程的核心知识点。