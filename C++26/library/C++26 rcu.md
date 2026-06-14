##  <rcu> Read-Copy Update

C++26 方向中提到的 `<rcu>`（Read-Copy Update）并不是一个“已完全定型的单一特性”，而是一个正在标准库层面探索的并发内存管理机制抽象，目标是把 RCU 这种在 Linux kernel 中非常成熟的并发模式，引入 C++ 标准库语义体系中。

下面按“是什么 → 解决什么 → C++26 设计形态 → 使用方式 → 与现有方案对比”说明。

------

## 1. RCU 是什么（核心思想）

RCU（Read-Copy Update）是一种**高并发读优化同步机制**，核心设计哲学是：

> 读操作完全无锁；写操作通过“复制 + 延迟回收”完成。

典型模型：

- 读线程：几乎不加锁，直接访问共享数据
- 写线程：
  1. 复制数据
  2. 修改副本
  3. 原子替换指针
  4. 等待所有旧读者退出“临界读区”
  5. 回收旧数据

------

## 2. 为什么需要 `<rcu>`（C++26 动机）

C++ 现有并发模型：

| 机制                | 问题               |
| ------------------- | ------------------ |
| mutex               | 读多写少场景性能差 |
| shared_mutex        | 写竞争仍昂贵       |
| atomic + shared_ptr | 引用计数开销大     |
| lock-free CAS       | 编程复杂，ABA问题  |

RCU 目标解决的是：

> **极端读多写少场景（read-mostly workload）下的零锁读路径**

典型场景：

- 配置热更新
- 路由表 / DNS cache
- feature flag system
- in-memory lookup tables
- telemetry / metrics snapshot

------

## 3. C++26 `<rcu>` 的抽象模型（预期设计）

目前标准化方向（P-staged proposals）通常围绕三部分：

### 3.1 RCU 读侧 API（read-side critical section）

类似：

```cpp
std::rcu::read_lock_guard g;
```

或者：

```cpp
std::rcu::read_section([&] {
    // safe read of shared object
});
```

特点：

- 无 mutex
- 通常只是 thread-local state + memory barrier
- 极低开销（接近 no-op）

------

### 3.2 发布新版本（publish/update）

写线程使用：

```cpp
std::rcu::update(shared_ptr<Data> new_data);
```

或者：

```cpp
std::rcu::synchronize([&] {
    global_ptr = new_version;
});
```

核心语义：

- 原子替换指针（release semantics）
- 不影响正在读取旧版本的线程

------

### 3.3 延迟回收（grace period）

RCU 的关键：

> 旧数据不能立即 delete

必须等待：

- 所有“进入 read-side critical section 的线程”退出

类似 API：

```cpp
std::rcu::retire(old_ptr);
std::rcu::synchronize();
```

------

## 4. C++26 RCU 的内存模型语义（重点）

RCU 依赖三件事：

### 4.1 acquire/release ordering

写入：

```cpp
store_release(ptr)
```

读取：

```cpp
load_acquire(ptr)
```

------

### 4.2 “grace period”语义

保证：

> 在 grace period 结束前，任何 reader 都不会访问已释放对象

这是 C++ RCU 的核心保证。

------

### 4.3 与 `std::atomic` 的关系

RCU 本质上是：

- atomic pointer + epoch-based reclamation + thread quiescent states

------

## 5. 一个典型 RCU 使用模型（C++26 风格）

假设共享配置：

```cpp
struct Config {
    int value;
};
```

### 5.1 读路径（极快）

```cpp
void reader() {
    std::rcu::read_guard g;

    Config* cfg = std::rcu::load(config_ptr);
    int v = cfg->value;
}
```

特点：

- 无锁
- 无 malloc
- 只读指针

------

### 5.2 写路径（copy + swap）

```cpp
void writer() {
    auto new_cfg = std::make_unique<Config>(*std::rcu::load(config_ptr));
    new_cfg->value += 1;

    std::rcu::update(config_ptr, new_cfg.release());
}
```

------

### 5.3 回收旧版本

```cpp
std::rcu::retire(old_ptr);
```

系统在安全时机释放。

------

## 6. 与现有 C++ 并发方案对比

### 6.1 mutex vs RCU

| 特性     | mutex | RCU      |
| -------- | ----- | -------- |
| 读性能   | 慢    | 极快     |
| 写成本   | 中等  | 高       |
| 适用场景 | 通用  | 读多写少 |

------

### 6.2 shared_ptr vs RCU

shared_ptr：

- 原子引用计数（写放大）
- cache contention

RCU：

- 不依赖 refcount
- 只依赖“时间窗口回收”

------

### 6.3 lock-free CAS vs RCU

CAS：

- 需要处理 ABA
- 逻辑复杂

RCU：

- 避免 ABA（通过版本生命周期）
- 更偏“结构替换模型”

------

## 7. RCU 在 C++ 标准化中的难点

### 7.1 thread quiescent state 识别

必须判断：

- 线程什么时候“不再访问旧数据”

C++ runtime 需要参与调度或插桩。

------

### 7.2 回收延迟不可预测

内存释放可能延迟：

- real-time 系统风险
- latency tail risk

------

### 7.3 与 executor / coroutine 交互

RCU 和：

- `std::execution`
- coroutine suspension

之间的内存一致性模型复杂。

------

## 8. 总结

C++26 `<rcu>` 的目标不是“新增一个容器或类”，而是引入一种：

> **标准化的 read-mostly 并发内存管理模型**

核心收益：

- 读路径几乎无成本
- 写路径采用 copy-on-update
- 安全延迟回收
- 替代部分 shared_mutex / shared_ptr 设计

------

如果你需要，我可以继续展开更工程化的部分，例如：

- Linux RCU vs C++ RCU 语义差异
- epoch-based reclamation 在标准库中的可能实现
- liburcu / folly / concurrency kit 的映射关系
- 或者 C++26 RCU 和 hazard pointer 的对比（更底层）