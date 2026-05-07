> **TBB = CPU 并行调度器（任务图 + work-stealing）**
> **Go scheduler = 语言级协程调度器（M:N + 抢占 + I/O 友好）**
> **Folly executor = 执行器抽象（队列 + 线程池策略，可选是否 work-stealing）**



------

#  一、调度对象：你到底在调什么？

## 1️⃣ oneTBB

- 调度单位：**task（小函数 / 闭包）**
- 特点：**短小、CPU-bound**
- 生命周期：很短（微任务）

👉 典型：

```cpp
tbb::parallel_for(...)
```

------

## 2️⃣ Go runtime（goroutine 调度器）

- 调度单位：**goroutine（协程）**
- 特点：
  - 可以阻塞（I/O）
  - 生命周期很长
- 是“轻量线程”

👉 典型：

```go
go func() { ... }()
```

------

## 3️⃣ Facebook Folly Executor

- 调度单位：**callable（任务）**
- 但它只是一个**接口抽象**：

```cpp
executor->add(fn);
```

👉 它不定义：

- 是否 work-stealing
- 是否支持协程
- 是否 CPU-bound

------

# 二、调度模型（核心差异）

## 1️⃣ TBB：Work-Stealing（为 CPU 饱和设计）

结构：

```
每个线程 → 一个 deque（双端队列）
```

规则：

- 自己线程：从 **队头 pop**
- 别的线程：从 **队尾 steal**

👉 好处：

- 减少锁竞争
- 保持 cache locality
- 自动负载均衡

👉 特点：

- ❗ **完全针对 CPU 密集任务**
- ❗ 不考虑 I/O blocking

------

## 2️⃣ Go scheduler：M:N + Work-Stealing + 抢占

结构：

```
M (OS线程)
P (processor)
G (goroutine)
```

👉 核心：

- M:N 调度（多个 G 映射到多个 M）
- 每个 P 有本地 run queue
- 也有 work stealing

但关键区别在这里：

### ✅ Go 支持“阻塞感知”

当 goroutine 阻塞：

- 不占用线程
- scheduler 会切换其他 G

👉 这是 TBB 做不到的

------

### ✅ Go 有抢占（preemption）

- 长时间运行的 goroutine 会被打断
- 防止“饿死”

👉 TBB 是 cooperative（不抢占）

##  3️⃣ Folly executor：策略可插拔

Folly 设计是：

> ❗ executor 只是接口，调度策略由实现决定

常见实现：

### 👉 CPUThreadPoolExecutor

- 线程池
- 任务队列
- ❌ 默认不是 work-stealing（偏 FIFO）

------

### 👉 IOThreadPoolExecutor

- 用 epoll / IOCP
- 事件驱动

------

### 👉 InlineExecutor

- 直接执行（无调度）

------

👉 重点：

> Folly 不强制一种调度模型
> 它是“调度策略容器”

------

#  三、核心差异对比

| 维度               | TBB      | Go           | Folly          |
| ------------------ | -------- | ------------ | -------------- |
| 抽象层             | 库       | 语言 runtime | 库             |
| 调度单位           | task     | goroutine    | callable       |
| 是否 work-stealing | ✅ 强依赖 | ✅            | ❓（看实现）    |
| 是否支持阻塞       | ❌        | ✅            | ❓              |
| 是否抢占           | ❌        | ✅            | ❌              |
| 目标               | CPU 并行 | 通用并发     | 可组合执行框架 |

------

# 四、本质差异（最关键）

## 1. 是否“理解阻塞”

- TBB：❌ 完全不理解（假设任务不阻塞）
- Go：✅ runtime 感知（syscall / network）
- Folly：❓ 取决于 executor

👉 这决定了：

- TBB 不适合 I/O
- Go 非常适合 I/O

------

## 🎯 2. 是否“强绑定策略”

- TBB：✔ 强绑定 work-stealing
- Go：✔ 内置 scheduler（不可替换）
- Folly：❌ 完全解耦（可自定义）

------

## 🎯 3. 是否“语言级支持”

- Go：✔ 编译器 + runtime 支持
- TBB / Folly：❌ 只是库

👉 这带来：

- Go 可以做：
  - 抢占
  - 栈扩展
  - 阻塞感知

#  五、直觉类比（很好记）

| 系统           | 类比                      |
| -------------- | ------------------------- |
| TBB            | 高速流水线（CPU 工厂）    |
| Go             | 操作系统调度器（微型 OS） |
| Folly executor | 插座（你自己选电器）      |

------

# 六、什么时候用哪个？

## 👉 用 TBB

- 数值计算
- 图像处理
- 编译器
- CPU-bound

------

## 👉 用 Go

- 网络服务
- 高并发 I/O
- 微服务

------

## 👉 用 Folly

- C++ 服务端
- 需要组合：
  - Future
  - Executor
  - IO

------

# 七、一个很深的洞察

> **TBB 解决的是“CPU 利用率问题”**
> **Go 解决的是“并发建模问题”**
> **Folly 解决的是“执行解耦问题”**

------

# 总结一句话

> TBB = work-stealing 做到极致
> Go = 调度系统做到语言级
> Folly = 调度抽象做到可组合

------

