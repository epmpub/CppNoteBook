Folly 的 Executor / 执行器策略

------

## 一、总入口：Executors 官方文档（必读）

核心文档：**folly/docs/Executors.md**（讲所有策略、线程池、队列、优先级）

https://github.com/facebook/folly/blob/main/folly/docs/Executors.md

覆盖内容：

- `CPUThreadPoolExecutor` / `IOThreadPoolExecutor` 设计与策略
- 队列策略：**LifoSem（默认 LIFO）、MPMC、优先级队列**
- 调度策略：**优先级（多队列抢占）、动态伸缩、保活**
- 拒绝策略、Observer、线程工厂

------

## 二、各执行器头文件文档（权威接口说明）

### 1. 基础执行器

- `Executor.h`（基类、优先级常量、接口）

  

  https://github.com/facebook/folly/blob/main/folly/Executor.h

### 2. 线程池核心

- `ThreadPoolExecutor.h`（动态伸缩、min/max 线程、保活）

  

  https://github.com/facebook/folly/blob/main/folly/executors/ThreadPoolExecutor.h

### 3. CPU 池

- `CPUThreadPoolExecutor.h`（CPU 密集、LifoSem 队列、亲和性）

  

  https://github.com/facebook/folly/blob/main/folly/executors/CPUThreadPoolExecutor.h

### 4. IO 池

- `IOThreadPoolExecutor.h`（I/O 密集、EventBase 绑定、网络 / 文件）

  

  https://github.com/facebook/folly/blob/main/folly/executors/IOThreadPoolExecutor.h

### 5. 全局执行器

- `GlobalExecutor.h/cpp`（全局单例、默认参数、FLAGS）

  

  https://github.com/facebook/folly/blob/main/folly/executors/GlobalExecutor.h

### 6. 特殊执行器

- `InlineExecutor.h`（同步执行）

  

  https://github.com/facebook/folly/blob/main/folly/executors/InlineExecutor.h

- `StrandExecutor.h`（串行化）

  

  https://github.com/facebook/folly/blob/main/folly/executors/StrandExecutor.h

------

## 三、队列与调度策略文档

- 优先级队列：`folly/executors/PriorityQueue.h`
- LifoSem 队列：`folly/executors/LifoSemMPMCQueue.h`
- 线程工厂：`folly/executors/ThreadFactory.h`

------

## 四、阅读顺序建议（快速吃透策略）

1. 先读 **Executors.md**（整体策略与选型）
2. 再看 **ThreadPoolExecutor.h**（伸缩 / 保活机制）
3. 最后看 **CPU/IOThreadPoolExecutor.h**（各自专属策略）

------

要不要我把这些链接整理成一份可直接复制的**书签列表**，并标注每个文档的核心关注点？