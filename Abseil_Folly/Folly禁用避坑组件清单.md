# Folly 禁用 / 避坑组件清单

结合 Meta 官方文档、工业级踩坑经验、Linux 服务端落地实践，按**禁止使用、谨慎使用、特殊场景受限**分级，附原因 + 替代方案，直接落地避坑。

> 前提：Folly 主力适配 **Linux + C++17/20**，Windows/macOS 大量组件残缺 / 有 Bug。

------

## 一、 绝对禁止 / 生产直接禁用

### 1. `folly/detail/` 下所有内部头文件

- 坑点：

  完全无兼容保证，版本升级直接删改、ABI 爆炸、无文档、无兼容性承诺。

- 错误用法：

  ```
  #include <folly/detail/xxx.h>
  ```

- 替代：只用 `folly/` 一级公开头文件。

------

### 2. `folly::Singleton` 旧版单例

------

### 3. `folly::wangle` 捆绑老旧网络组件（不属于纯 Folly 但常混用）

------

### 4. 非可移植汇编 / 平台硬编码模块

`folly/hardware/` 部分 CPU 特权指令、裸汇编、特定 CPU 优化分支

- 坑点：跨 CPU 架构（ARM / 龙芯）直接崩、编译失败、性能反向退化。
- 禁用：自研跨平台底层不要强依赖。

------

## 二、🟠 谨慎使用（有严格使用门槛，乱用必炸）

### 5. `folly::AsyncSignalHandler` 信号捕获

- 坑点：

  信号处理上下文是

  裸临界区

  、不可分配内存、不可加锁、极易死锁 / 内存损坏。

- 限制：仅专业底层框架使用，业务代码禁止注册信号回调。

------

### 6. `folly::Fiber` 纤程 / 旧版用户态线程

- 坑点：
  1. 依赖 Boost.Context，栈切换黑盒
  2. 锁、TLS、第三方库兼容性极差
  3. 调试极难、死锁隐蔽
- 替代：**优先用 Folly 原生 C++20 Coro 协程**，不要用 Fiber。

------

### 7. `folly::AtomicHashMap` 无锁并发哈希

- 坑点：
  1. **固定容量、不可动态扩容**
  2. 删除逻辑复杂、内存泄漏风险
  3. 迭代器非安全、业务极易误用
- 适用：仅**只读 / 写少、提前固定容量**的全局统计场景。
- 替代：通用并发场景 → `F14FastMap + 细粒度锁`。

------

### 8. `folly::Jemalloc 强制绑定` 内存定制分配器

- 坑点：
  1. 替换全局 `malloc/free`，和系统库 / 其他三方库冲突
  2. 多进程、动态库场景内存交叉释放崩溃
- 建议：不全局替换，仅局部容器自定义分配器。

------

### 9. `folly::flat_map / flat_set`

- 坑点：
  1. 纯连续内存，**insert/erase 大范围元素移动**
  2. 迭代器、引用大面积失效
  3. 频繁增删场景性能比 `F14NodeMap` 还差
- 正确使用：**静态有序集合、极少修改、高频查询**才用。
- 替代：频繁增删有序结构 → 放弃 flat，用 F14 系列。

------

### 10. `folly::ThreadLocal` 裸 TLS

- 坑点：

  误用会出现：线程退出析构顺序错乱、残留野指针、跨线程访问非法。

- 强制约束：

  只存

  无依赖、轻量、自包含

  对象，不要存复杂业务对象。

------

### 11. `LifoSemMPMCQueue` 业务自定义队列

- 坑点：

  Folly 线程池默认自带优化，

  业务不要自己裸写 LifoSem 队列

  ；

  唤醒策略、内存模型门槛极高，手写容易并发 BUG。

------

## 三、 场景限制（特定环境才能用）

### 12. `IOThreadPoolExecutor + EventBase`

- 限制：
  1. 每个线程绑定唯一 EventBase，**不能跨线程乱投递任务**
  2. 事件循环嵌套会阻塞、死循环
- 禁忌：不要在 IO 池任务里跑重度 CPU 计算，会拖垮 IO 多路复用。

------

### 13. `folly::Dynamic` 动态弱类型对象

- 坑点：
  1. 类型隐式转换、隐藏 BUG
  2. 性能差、内存开销高
- 建议：业务结构化数据**优先强类型结构体**，仅临时简单 JSON 用。

------

### 14. `folly::Subprocess` 子进程管理

- 坑点：
  1. 僵尸进程处理依赖开发者手动回收
  2. 管道缓冲区满会阻塞主进程
- 约束：必须配置超时 + 异步回收，禁止同步阻塞调用。

------

### 15. 带 `_experimental` 后缀所有模块

- 坑点：

  实验性 API，随时删除、接口大改、无生产验证。

- 规则：只要文件名 / 命名空间带 `experimental`，一律不用。

------

## 四、✅ 绝对放心无脑用（安全标杆）

反向对照，以下是**工业级稳、随便用**的核心组件：

1. F14 全家桶：`F14FastMap/F14NodeMap/F14ValueMap`
2. `folly::IOBuf` 零拷贝缓冲区
3. `CPUThreadPoolExecutor / IOThreadPoolExecutor` 标准线程池
4. `folly::Future<T>` 异步链式编程
5. `folly::MicroSpinLock / SharedMutex` 高性能锁
6. `small_vector` 小对象栈容器
7. 字符串工具：`split/join/trim/stringPrintf`
8. `folly::Clock` 高精度时间工具

------

## 五、Folly 全局编译避坑（CMake 必加）

```
# 关闭部分危险实验特性
add_definitions(-DFOLLY_NO_EXPERIMENTAL)
# 禁用jemalloc全局绑定（避免内存冲突）
set(FOLLY_USE_JEMALLOC OFF CACHE BOOL "" FORCE)
# 禁用Fiber老旧纤程
set(FOLLY_USE_FIBERS OFF CACHE BOOL "" FORCE)
```

------

## 六、一句话总结口诀

1. **内部 detail、实验库、旧单例 → 全禁用**
2. **Fiber、AtomicHashMap、flat_map → 严控场景**
3. **信号处理、子进程、全局分配器 → 业务不碰**
4. **哈希 F14、IOBuf、标准 Executor、Future → 放心猛用**

需要我给你一份 **Folly 最小安全 CMake 模板（关闭所有危险组件、稳定编译）** 直接复制即用吗？