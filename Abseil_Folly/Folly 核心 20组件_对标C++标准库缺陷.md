# Folly 核心 20 组件 + 对标 C++ 标准库缺陷 速查表

适配服务端 / 高性能 C++ 开发，逐条写明：**组件名、解决 Std 痛点、核心优势、适用场景**，极简落地。

> 前置说明：Folly 最低建议 C++17/20，全为工业级线上验证，对标原生 STL 短板。

------

## 一、容器类（STL 重灾区）

### 1. F14 哈希全家桶

- 组件：`F14FastMap / F14NodeMap / F14ValueMap / F14VectorMap`
- Std 缺陷：`std::unordered_map` 拉链哈希、缓存失效、慢、内存碎片、rehash 开销大
- 核心优势：SIMD 桶检索、紧凑内存、2~8 倍性能、可控迭代器稳定性、低扩容开销
- 场景：高频 KV、内存敏感、大数据量缓存

### 2. folly::small_vector

- Std 缺陷：`std::vector` 小对象堆分配频繁、短生命周期开销高
- 核心优势：栈内预分配、小对象无堆内存、兼容 std 接口
- 场景：短数组、临时容器、协议解析

### 3. folly::flat_map / flat_set

- Std 缺陷：`std::map` 红黑树、极致慢、内存碎片化严重
- 核心优势：连续内存有序容器、二分查找、cache 友好、性能碾压树容器
- 场景：有序 KV、配置表、索引集合

### 4. folly::SparseArray

- Std 缺陷：原生无稀疏容器，空下标浪费大量内存
- 核心优势：稀疏下标存储、极低内存占用
- 场景：id 稀疏映射、游戏 / 服务端索引

------

## 二、内存 & 零拷贝（STL 完全缺失）

### 5. folly::IOBuf

- Std 缺陷：无原生零拷贝缓冲区，`std::string`/`vector<char>` 频繁内存拷贝
- 核心优势：链式缓冲区、引用计数、分片读写、零拷贝拼接 / 切割、网络标准组件
- 场景：网络 IO、协议编解码、RPC、缓存分片

### 6. folly::SharedMutex（读写锁）

- Std 缺陷：`std::shared_mutex` 实现低效、高并发排队严重
- 核心优势：用户态优化、读写分离极致性能、抢占策略优化
- 场景：读多写少配置、全局共享数据

### 7. folly::ThreadLocal

- Std 缺陷：`thread_local` 原生初始化坑多、析构顺序乱、性能一般
- 核心优势：安全生命周期、懒加载、高性能 TLS、跨平台稳定
- 场景：线程独立计数器、缓存、日志上下文

------

## 三、并发 / 异步 / 执行器（STL 生态空白）

### 8. 完整 Executor 线程池体系

- 组件：`CPUThreadPoolExecutor / IOThreadPoolExecutor / StrandExecutor`
- Std 缺陷：无内置线程池、`std::async` 不可控、无优先级、无动态扩缩容
- 核心优势：任务优先级、动态线程伸缩、EventBase 绑定、串行隔离 (Strand)、LIFO 调度
- 场景：业务池化、异步任务、IO 多路复用、协程调度

### 9. folly::Future / Promise

- Std 缺陷：`std::future` 无法链式调用、无回调、无超时、组合能力为 0
- 核心优势：链式 then / 延迟 / 超时 / 重试、异常统一捕获、协程无缝结合
- 场景：异步编排、多任务聚合、RPC 异步调用

### 10. folly::coro 协程库

- Std 缺陷：C++20 原生协程 API 原始、无调度器、无超时、上手极难
- 核心优势：封装完善、绑定 Executor、同步写法写异步、轻量化
- 场景：高并发连接、异步服务端、大量轻量任务

### 11. folly::LifoSem 信号量

- Std 缺陷：`std::semaphore` 公平 FIFO 唤醒、多线程缓存颠簸
- 核心优势：LIFO 后进先出唤醒、减少缓存失效、高吞吐低延迟
- 场景：线程池队列、高并发任务排队

------

## 四、字符串 & 基础工具（STL 功能残缺）

### 12. folly::string_view 增强 + 字符串工具集

- 组件：`split/join/trim/startsWith/endsWith/stringPrintf`
- Std 缺陷：C++17 string_view 工具匮乏、无便捷分割拼接、格式化繁琐
- 核心优势：零拷贝字符串操作、安全格式化、高频工具开箱即用
- 场景：日志、配置解析、文本处理

### 13. folly::Format

- Std 缺陷：`std::format` 编译器兼容差、老 C++ 版本不支持
- 核心优势：跨平台稳定、性能更强、兼容旧标准
- 场景：高性能日志、动态文本拼接

### 14. folly::Dynamic

- Std 缺陷：STL 无原生动态弱类型容器，json / 动态参数需手写
- 核心优势：类 JS 动态对象、自动类型转换、快速序列化
- 场景：简单 JSON、动态配置、弱类型参数

------

## 五、高性能原语 & 系统底层优化

### 15. folly::MicroSpinLock

- Std 缺陷：`std::mutex` 重量级、用户态自旋锁缺失
- 核心优势：短临界区自旋、无内核态切换、超高并发吞吐
- 场景：小粒度加锁、计数器、高频共享变量

### 16. folly::AtomicHashmap

- Std 缺陷：STL 无并发安全哈希表，手写加锁复杂度高
- 核心优势：无锁 / 细粒度锁、并发安全、高吞吐 KV
- 场景：全局统计、连接映射、热点缓存

### 17. folly::Clock 高精度时间

- Std 缺陷：`std::chrono` 接口冗余、易误用、时钟漂移问题
- 核心优势：封装单调时钟、高精度、时间戳快速转换、防漂移
- 场景：超时控制、埋点耗时、限流统计

------

## 六、工程化 & 稳定性工具

### 18. folly::Exception 增强异常

- Std 缺陷：原生异常无堆栈、错误信息简陋
- 核心优势：自动堆栈捕获、嵌套异常、错误码绑定
- 场景：服务端错误排查、崩溃定位

### 19. folly::RateLimiter 限流器

- Std 缺陷：STL 无限流组件，令牌桶 / 漏桶需自研
- 核心优势：高性能令牌桶、平滑限流、多策略
- 场景：接口限流、流量控制、防打爆

### 20. folly::Subprocess

- Std 缺陷：STL 无跨平台进程调用，system/popen 不安全、不可控
- 核心优势：安全子进程管理、管道捕获、超时控制、跨平台
- 场景：运维脚本调用、外部命令执行

------

# 极简选型口诀

1. **哈希必上 F14**，抛弃 unordered_map
2. **网络必用 IOBuf**，杜绝内存拷贝
3. **异步线程池直接 Executor**，不用手写 std::thread
4. **读多写少用 SharedMutex**，自旋锁护小临界区
5. **字符串处理全用 folly 工具**，少造轮子

------

# 补充 CMake 统一依赖

所有以上组件，仅需一行链接，无需额外子库：

cmake









```
target_link_libraries(xxx PRIVATE Folly::folly)
set(CMAKE_CXX_STANDARD 20)
```

需要我再补一份：**Folly 禁用 / 避坑组件清单（不要乱用的模块）** 吗？避免线上踩坑。
