现代 C++（C++11 及之后版本）提供了多种线程同步方法，旨在安全高效地协调多线程操作。以下是主要的线程同步工具及其应用场景：

---

### **1. 互斥量（Mutexes）**
用于保护共享数据的独占访问，防止数据竞争。

#### **(1) `std::mutex`**
- **基本互斥锁**，需手动管理锁定和解锁。
  ```cpp
  std::mutex mtx;
  mtx.lock();
  // 临界区操作
  mtx.unlock();
  ```

#### **(2) `std::lock_guard`**
- **RAII 封装**，自动释放锁（推荐基础用法）。
  ```cpp
  std::mutex mtx;
  {
      std::lock_guard<std::mutex> lock(mtx);
      // 临界区操作（离开作用域自动解锁）
  }
  ```

#### **(3) `std::unique_lock`**
- **更灵活的 RAII 封装**，支持延迟锁定和手动控制。
  ```cpp
  std::mutex mtx;
  std::unique_lock<std::mutex> lock(mtx, std::defer_lock);
  lock.lock(); // 可手动控制锁定时机
  ```

#### **(4) `std::scoped_lock`（C++17）**
- **同时锁定多个互斥量**，避免死锁。
  ```cpp
  std::mutex mtx1, mtx2;
  std::scoped_lock lock(mtx1, mtx2); // 自动按顺序锁定
  ```

---

### **2. 条件变量（Condition Variables）**
用于线程间有条件地等待或唤醒，实现复杂的同步逻辑。

#### **(1) `std::condition_variable`**
- **等待条件成立**，需配合互斥量使用。
  ```cpp
  std::mutex mtx;
  std::condition_variable cv;
  bool data_ready = false;
  
  // 等待线程
  std::unique_lock<std::mutex> lock(mtx);
  cv.wait(lock, []{ return data_ready; });
  
  // 通知线程
  {
      std::lock_guard<std::mutex> lock(mtx);
      data_ready = true;
  }
  cv.notify_one();
  ```

---

### **3. 原子操作（Atomic Operations）**
提供无锁线程安全操作，适合简单变量同步。

#### **(1) `std::atomic<T>`**
- **保证对变量的操作原子性**（无需锁）。
  ```cpp
  std::atomic<int> counter(0);
  counter.fetch_add(1, std::memory_order_relaxed);
  ```

#### **(2) 内存序（Memory Order）**
- 控制原子操作的内存可见性顺序：
  - `memory_order_relaxed`：无顺序保证。
  - `memory_order_acquire`/`release`：同步数据依赖。
  - `memory_order_seq_cst`（默认）：严格顺序。

---

### **4. 读写锁（Read-Write Locks）**
适用于读多写少的场景，提高并发性能。

#### **(1) `std::shared_mutex`（C++17）**
- **允许多个读线程或单个写线程**。
  ```cpp
  std::shared_mutex rw_mtx;
  
  // 读操作（共享锁）
  {
      std::shared_lock<std::shared_mutex> lock(rw_mtx);
      // 读取共享数据
  }
  
  // 写操作（独占锁）
  {
      std::unique_lock<std::shared_mutex> lock(rw_mtx);
      // 修改共享数据
  }
  ```

---

### **5. 一次性操作（One-Time Initialization）**
确保某操作仅执行一次（如单例初始化）。

#### **(1) `std::call_once`**
- **配合 `std::once_flag` 使用**。
  ```cpp
  std::once_flag flag;
  std::call_once(flag, []{ /* 仅执行一次 */ });
  ```

---

### **6. 栅栏（Barriers）与门闩（Latches）（C++20）**
协调多个线程在特定点同步。

#### **(1) `std::latch`**
- **一次性同步点**，等待所有线程到达后继续。
  ```cpp
  std::latch sync_point(3); // 等待 3 个线程
  void worker() {
      sync_point.arrive_and_wait(); // 到达并等待
  }
  ```

#### **(2) `std::barrier`**
- **可重复使用的同步点**，支持多阶段同步。
  ```cpp
  std::barrier sync_barrier(3); // 每阶段等待 3 个线程
  void worker() {
      sync_barrier.arrive_and_wait(); // 进入下一阶段
  }
  ```

---

### **7. 异步结果传递（Futures and Promises）**
用于线程间传递异步操作结果。

#### **(1) `std::future` 与 `std::promise`**
- **获取异步操作的返回值**。
  ```cpp
  std::promise<int> prom;
  std::future<int> fut = prom.get_future();
  
  std::thread t([&prom]{ prom.set_value(42); });
  int result = fut.get(); // 阻塞直到结果就绪
  t.join();
  ```

#### **(2) `std::async`**
- **简化异步任务启动**。
  ```cpp
  auto fut = std::async(std::launch::async, []{ return 42; });
  int result = fut.get();
  ```

---

### **总结：如何选择同步方法？**
| **场景**           | **推荐工具**                           |
| ------------------ | -------------------------------------- |
| 简单共享变量保护   | `std::atomic`                          |
| 复杂临界区操作     | `std::mutex` + `std::lock_guard`       |
| 读多写少的数据访问 | `std::shared_mutex`                    |
| 线程间条件等待     | `std::condition_variable`              |
| 一次性初始化       | `std::call_once`                       |
| 多线程阶段同步     | `std::latch` / `std::barrier`（C++20） |
| 异步结果传递       | `std::future` 和 `std::promise`        |

**注意事项**：
- 避免死锁：按固定顺序加锁，或使用 `std::scoped_lock`。
- 优先使用 RAII 类型（如 `lock_guard`），防止忘记解锁。
- 无锁编程（原子操作）需谨慎处理内存顺序。