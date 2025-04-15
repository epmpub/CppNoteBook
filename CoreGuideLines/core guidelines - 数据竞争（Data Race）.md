# core guidelines - 数据竞争（Data Race）

是的，这段代码存在潜在的 **数据竞争（Data Race）** 问题，主要源于多个线程对共享数据的非同步访问。以下是详细分析：



```C++
void f(fstream& fs, regex pattern)
{
    array<double, max> buf;
    int sz = read_vec(fs, buf, max);            // 从 fs 读取到 buf 中
    gsl::span<double> s {buf};
    // ...
    auto h1 = async([&] { sort(std::execution::par, s); });     // 产生一个进行排序的任务
    // ...
    auto h2 = async([&] { return find_all(buf, sz, pattern); });   // 产生一个查找匹配的任务
    // ...
}
```



**1. 数据竞争的具体位置**

#### **(1) `sort` 任务 (`h1`)**
```cpp
auto h1 = async([&] { sort(std::execution::par, s); });
```
- **问题**：`sort` 在并行执行（`std::execution::par`）时会修改 `s`（即 `buf` 的 `gsl::span` 视图），而 `buf` 是栈上的局部数组。
- **竞争条件**：如果 `h1` 的排序操作与其他任务（如 `h2`）或主线程同时访问 `buf`，会导致未定义行为。

#### **(2) `find_all` 任务 (`h2`)**
```cpp
auto h2 = async([&] { return find_all(buf, sz, pattern); });
```
- **问题**：`find_all` 读取 `buf` 和 `sz`，而 `sort` 可能同时修改 `buf` 的内容。
- **竞争条件**：读写冲突（`sort` 写，`find_all` 读）。

---

### **2. 根本原因**
- **共享数据**：`buf` 和 `sz` 被两个异步任务 **通过引用捕获**（`[&]`），而它们的生命周期依赖于函数 `f` 的栈帧。
- **缺乏同步**：没有使用互斥锁（`mutex`）或原子操作保护共享数据。
- **悬垂引用风险**：如果 `f` 函数在异步任务完成前返回，`buf` 和 `sz` 会变成悬垂引用（Dangling Reference）。

---

### **3. 修复方案**
#### **(1) 确保数据独立（推荐）**
为每个任务分配独占的数据副本，避免共享：
```cpp
void f(fstream& fs, regex pattern) {
    array<double, max> buf;
    int sz = read_vec(fs, buf, max);

    // 将数据拷贝到任务内部
    auto h1 = async([buf = array<double, max>(buf), sz] { 
        gsl::span<double> s{buf};
        sort(std::execution::par, s); // 操作副本
    });

    auto h2 = async([buf = vector<double>(buf.begin(), buf.begin() + sz), pattern] {
        return find_all(buf.data(), buf.size(), pattern); // 操作副本
    });

    // 等待任务完成（若需要结果）
    h1.wait();
    auto results = h2.get();
}
```

#### **(2) 显式同步（若必须共享）**
使用 `std::mutex` 保护共享数据：
```cpp
void f(fstream& fs, regex pattern) {
    array<double, max> buf;
    int sz = read_vec(fs, buf, max);
    gsl::span<double> s{buf};
    std::mutex mtx;

    auto h1 = async([&] { 
        lock_guard<mutex> lock(mtx);
        sort(std::execution::seq, s); // 必须改为串行排序
    });

    auto h2 = async([&] {
        lock_guard<mutex> lock(mtx);
        return find_all(buf, sz, pattern);
    });

    h1.wait();
    auto results = h2.get();
}
```
- **代价**：同步会削弱并行性能。

#### **(3) 延长共享数据生命周期**
通过 `shared_ptr` 管理数据，确保生命周期覆盖异步任务：
```cpp
void f(fstream& fs, regex pattern) {
    auto buf = make_shared<array<double, max>>();
    int sz = read_vec(fs, *buf, max);

    auto h1 = async([buf, sz] { 
        gsl::span<double> s{*buf};
        sort(std::execution::par, s);
    });

    auto h2 = async([buf, sz, pattern] {
        return find_all(buf->data(), sz, pattern);
    });

    h1.wait();
    auto results = h2.get();
}
```

---

### **4. 关键问题总结**
| **问题**             | **风险**                                                     |
| -------------------- | ------------------------------------------------------------ |
| **并发修改 `buf`**   | `sort` 和 `find_all` 同时访问 `buf`，导致数据竞争（未定义行为）。 |
| **悬垂引用**         | 若 `f` 提前返回，`buf` 和 `sz` 失效，异步任务访问已释放内存。 |
| **并行排序的副作用** | `std::execution::par` 可能加剧竞争（多线程并发修改同一数据）。 |

---

### **5. 最佳实践建议**
1. **避免共享可变数据**：优先为任务分配独立的数据副本。
2. **警惕引用捕获**：异步任务中慎用 `[&]`，确保捕获的数据生命周期足够长。
3. **明确同步需求**：若必须共享，使用 `mutex` 或原子操作。
4. **检查标准算法并行性**：`std::execution::par` 需确保数据访问的线程安全性。

通过以上修复，可以消除数据竞争和悬垂引用风险，确保代码的正确性和健壮性。