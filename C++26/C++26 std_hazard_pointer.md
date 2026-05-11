## std::hazard_pointer（C++26）

 是 C++ 标准库中新增的安全内存回收（Safe Memory Reclamation, SMR） 机制，主要用于无锁（lock-free）并发数据结构中安全地延迟释放内存。什么是 Hazard Pointer？Hazard Pointer（危险指针） 是一种指针保护机制：

- 每个线程可以拥有一个或多个 Hazard Pointer（本质是一个线程私有的“公告牌”）。
- 当线程要访问一个共享对象（通常是动态分配的节点，如链表、哈希表节点）时，它会把该对象的地址写到自己的 Hazard Pointer 中，向其他线程“宣告”：“我正在使用这个对象，请不要删除它！”
- 当线程不再使用该对象时，把 Hazard Pointer 清空（置为 nullptr）。
- 当一个线程想要删除（回收） 一个对象时，它不能立即 delete，而是先检查所有 Hazard Pointer，看是否还有线程在保护这个地址。如果没有保护，才可以安全回收。

核心目的：

- 解决无锁数据结构中的ABA 问题和Use-After-Free（释放后使用） 问题。
- 提供确定性、无 GC 暂停的内存回收方式，性能开销远低于引用计数（尤其是高读场景）。

C++26 在 <hazard_pointer> 头文件中提供了标准实现：

- std::hazard_pointer
- std::hazard_ptr_obj_base（需要被保护的类型继承它）
- 域（domain）、退休列表（retired list）等支持。

与 Rust 相比，主要解决了什么问题？Rust 和 C++ 在并发内存管理上的哲学不同：

| 方面         | C++ Hazard Pointer (std::hazard_pointer)                | Rust 典型做法（Epoch + Crossbeam / haphazard）               |
| ------------ | ------------------------------------------------------- | ------------------------------------------------------------ |
| 保护粒度     | 细粒度（只保护具体指针/对象）                           | Epoch：粗粒度（整个 epoch 内所有对象） haphazard：细粒度（类似 HP） |
| 内存回收时机 | 延迟回收，只有确认无 Hazard Pointer 保护时              | Epoch：全局 epoch 推进后批量回收 HP：类似 C++                |
| 性能特点     | 读端开销较低（一次 atomic store），写/回收端检查所有 HP | Epoch 读端通常更快（无每指针 fence） HP 类似 C++             |
| 安全性保证   | 程序员手动保护，容易出错但灵活                          | 借用检查器 + Send/Sync 强制安全，编译期防止很多错误          |
| 易用性       | 需要显式 acquire / protect / retire                     | Epoch 更简单（epoch::pin()），haphazard 更接近 C++           |
| 适用场景     | 高性能无锁数据结构（栈、队列、哈希表等）                | 同，但更推荐 Epoch（大多数情况）                             |

Hazard Pointer 在 C++ 中主要解决的问题（Rust 角度对比）：

1. 无 GC 环境下安全回收内存
   C++ 没有自动垃圾回收，裸 delete 在并发下极易导致 Use-After-Free。Hazard Pointer 提供了一种手动但可靠的延迟回收方案。
2. ABA 问题
   经典 CAS 循环中，指针值“看起来一样”但实际已被回收并重新分配，导致错误行为。Hazard Pointer 通过保护具体地址来避免。
3. 比引用计数更好的性能
   原子引用计数（std::shared_ptr）在高并发读时会有严重的缓存一致性开销。Hazard Pointer 的读端开销更低。
4. 比 RCU 更灵活的持有时间
   RCU（Read-Copy-Update）要求临界区不能太长，而 Hazard Pointer 允许长时间持有保护。

Rust 的优势：

- Epoch-Based Reclamation (EBR) 是 Rust 社区最常用的方案（crossbeam::epoch），比 Hazard Pointer 更简单、读端更快（全局 epoch 推进，无需每指针公告）。
- Rust 的所有权系统 + Arc + 借用检查器天然减少了很多需要 Hazard Pointer 的场景。
- 如果需要细粒度保护，Rust 有 haphazard crate 实现 Hazard Pointer。

总结：

- Hazard Pointer 是 C++ 在无垃圾回收语言中，为高性能 lock-free 数据结构提供的实用、安全的内存回收工具。
- Rust 更倾向用 Epoch（性能更好、更简单），只在需要极致细粒度时才用 Hazard Pointer。
- C++26 引入它，极大降低了编写高性能并发数据结构的门槛（之前开发者常自己实现或用第三方库如 Folly）。