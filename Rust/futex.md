##  futex

（“快速用户空间互斥锁”的缩写）是一种 Linux 内核系统调用，旨在为用户空间应用程序提供高效同步，特别是对于共享内存的多线程程序或进程。它充当更高级别同步原语（如互斥体、信号量和条件变量）的低级构建块，通常用于 POSIX 线程 （pthreads） 等库。

A futex (short for "fast userspace mutex") is a Linux kernel system call designed to provide efficient synchronization for user-space applications, particularly for multithreaded programs or processes sharing memory. It acts as a low-level building block for higher-level synchronization primitives like mutexes, semaphores, and condition variables, commonly used in libraries such as POSIX threads (pthreads).

Futex 的主要特点
Key Features of Futexes

1. 用户空间效率 ：Futex 作针对无争议情况进行了优化，这意味着它们主要使用原子作在用户空间中执行，以最大限度地减少昂贵的内核系统调用。内核仅在发生争用时（例如，当一个线程需要阻塞或唤醒另一个线程时）才会参与 。
   Userspace Efficiency: Futex operations are optimized for the uncontended case, meaning they primarily execute in user space using atomic operations to minimize expensive kernel system calls. The kernel is only involved when contention occurs (e.g., when a thread needs to block or wake up another thread).
2. 结构 ：futex 是存储在共享内存中的 32 位整数（“futex 字”），可由多个线程或进程访问。使用原子指令来作此整数来管理同步。
   Structure: A futex is a 32-bit integer (a "futex word") stored in shared memory, accessible by multiple threads or processes. This integer is manipulated using atomic instructions to manage synchronization.
3. 核心业务 ：
   Core Operations:
   - FUTEX_WAIT：如果 futex 字的值与预期值匹配，则挂起线程，将其放入内核等待队列中，直到唤醒。这避免了忙碌循环并节省了 CPU 资源。
     FUTEX_WAIT: Suspends a thread if the futex word's value matches an expected value, placing it in a kernel wait queue until woken up. This avoids busy-looping and saves CPU resources.
   - FUTEX_WAKE：唤醒指定数量的等待 futex 的线程，允许它们继续。
     FUTEX_WAKE: Wakes up a specified number of threads waiting on the futex, allowing them to proceed.
   - 其他作（如 FUTEX_CMP_REQUEUE 或 FUTEX_WAKE_OP） 为特定用例提供高级功能。
     Other operations like FUTEX_CMP_REQUEUE or FUTEX_WAKE_OP provide advanced functionality for specific use cases.
4. 共享内存 ：Futex 可以使用 mmap 或 shmat  等机制在进程之间共享 ，也可以通过全局变量在单个进程的线程内共享。
   Shared Memory: Futexes can be shared between processes using mechanisms like mmap or shmat, or within threads of a single process via a global variable.
5. 性能 ：通过避免内核在无争议的情况下参与，futex 比旧的同步机制（如 System V 信号量）快得多，后者总是需要系统调用。
   Performance: By avoiding kernel involvement in uncontended cases, futexes are significantly faster than older synchronization mechanisms like System V semaphores, which always required system calls.

Futex 的工作原理
How Futexes Work

- 在无争议的情况下 ，线程尝试使用原子作（例如，比较和交换）获取锁。如果成功，则不需要内核交互，速度非常快。
  In the uncontended case, a thread attempts to acquire a lock using atomic operations (e.g., compare-and-swap). If successful, no kernel interaction is needed, making it extremely fast.
- 在争用的情况下 ，当锁已经被持有时，线程调用 futex（） 并 FUTEX_WAIT 进入睡眠状态，内核管理等待队列。释放锁时，另一个线程使用 FUTEX_WAKE 来通知等待的线程。
  In the contended case, when a lock is already held, the thread calls futex() with FUTEX_WAIT to sleep, and the kernel manages the wait queue. When the lock is released, another thread uses FUTEX_WAKE to notify waiting threads
- futex 字的值用于跟踪状态（例如，锁定或解锁），原子作确保无竞争更新。
  The futex word's value is used to track the state (e.g., locked or unlocked), and atomic operations ensure race-free updates.

示例用例
Example Use Case一个简单的互斥锁可以用一个 futex 实现：
A simple mutex can be implemented with a futex:

- 锁定 ：线程原子地检查 futex 字是否为 0（解锁）。如果是这样，它会将其设置为 1（锁定）。如果没有，它会调用 FUTEX_WAIT 进入睡眠状态，直到锁可用。
  Lock: A thread atomically checks if the futex word is 0 (unlocked). If so, it sets it to 1 (locked). If not, it calls FUTEX_WAIT to sleep until the lock is available.
- 解锁 ：线程将 futex 字设置为 0 并调用 FUTEX_WAKE 来唤醒等待的线程（如果有）。
  Unlock: The thread sets the futex word to 0 and calls FUTEX_WAKE to wake a waiting thread, if any.

为什么 Futex 很重要
Why Futexes Matter

- 可扩展性 ：Futex 是轻量级且可扩展的，在无竞争的情况下不保持内核状态，从而减少了开销。
  Scalability: Futexes are lightweight and scalable, with no kernel state maintained in the uncontended case, reducing overhead.
- 灵活性 ：它们支持从互斥体到条件变量的广泛同步原语，并广泛用于 NPTL（本机 POSIX 线程库）等库。
  Flexibility: They support a wide range of synchronization primitives, from mutexes to condition variables, and are used extensively in libraries like NPTL (Native POSIX Thread Library).
- 采用 ：在 Linux 内核 2.5.7 （2002） 中引入并在 2.6.x （2003） 中稳定，futex 现在是 Linux 同步的基石。其他作系统，如 Windows（  自 2012 年起通过 WaitOnAddress）和 Fuchsia（Zircon 内核），也采用了类似的机制。
  Adoption: Introduced in Linux kernel 2.5.7 (2002) and stabilized in 2.6.x (2003), futexes are now a cornerstone of Linux synchronization. Other operating systems, like Windows (via WaitOnAddress since 2012) and Fuchsia (Zircon kernel), have adopted similar mechanisms.

挑战
Challenges

- 复杂性 ：Futex 是低级的，需要仔细处理原子作和内存排序以避免竞争条件或错误。Ulrich Drepper 的论文“Futexes Are Tricky”是理解这些复杂性的关键参考。
  Complexity: Futexes are low-level and require careful handling of atomic operations and memory ordering to avoid race conditions or bugs. Ulrich Drepper’s paper “Futexes Are Tricky” is a key reference for understanding these complexities
- 不用于直接使用 ：应用程序开发人员很少直接使用 futex;他们依赖于抽象 futex 作的 pthreads 等库。
  Not for Direct Use: Application developers rarely use futexes directly; they rely on libraries like pthreads that abstract futex operations.![img](data:image/png;base64,/9j/4QC8RXhpZgAASUkqAAgAAAAGABIBAwABAAAAAQAAABoBBQABAAAAVgAAABsBBQABAAAAXgAAACgBAwABAAAAAgAAABMCAwABAAAAAQAAAGmHBAABAAAAZgAAAAAAAABIAAAAAQAAAEgAAAABAAAABgAAkAcABAAAADAyMTABkQcABAAAAAECAwAAoAcABAAAADAxMDABoAMAAQAAAP//AAACoAQAAQAAABAAAAADoAQAAQAAABAAAAAAAAAA/9sAQwAGBAUGBQQGBgUGBwcGCAoQCgoJCQoUDg8MEBcUGBgXFBYWGh0lHxobIxwWFiAsICMmJykqKRkfLTAtKDAlKCko/9sAQwEHBwcKCAoTCgoTKBoWGigoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgo/8AAEQgAEAAQAwEiAAIRAQMRAf/EABYAAQEBAAAAAAAAAAAAAAAAAAQCBf/EACkQAAEEAAUCBQUAAAAAAAAAAAECAwQFBhESEyEABxQVIiQxMlFSYXH/xAAUAQEAAAAAAAAAAAAAAAAAAAAG/8QAIREAAAUCBwAAAAAAAAAAAAAAAAECETEDEhMhQVFhkbH/2gAMAwEAAhEDEQA/AIwzQNSe2zltDw355bC3MUpykL0s7IVnpaWn4Vlyfy/nWRj2niVjNLIahvVU6dGU7KqnUue3KVlAWkr9WlelSgkklP3II6ZWTKuZ2uVRSLaLAni58aBIaeKS3sBHBbbVzmfj9dHv7SvYwLW4ejzEW0pmW5MMtCFpbjIUNOw3uJCiFEa1cJGeX1HkGlWWaRxPoHqwzptlBbS/Y//Z)
- 稳健性问题 ：如果进程在持有基于二耦合体的锁时崩溃，恢复可能具有挑战性，有时需要重新启动系统。稳健的熔炉通过通知服务员崩溃来解决这个问题，但它们增加了复杂性。
  Robustness Issues: If a process crashes while holding a futex-based lock, recovery can be challenging, sometimes requiring a system reboot. Robust futexes address this by notifying waiters of crashes, but they add complexity.

历史背景
Historical Context

- Futex 由 Hubertus Franke、Matthew Kirkwood、Ingo Molnár 和 Rusty Russell 于 2002 年开发，旨在克服 System V 信号量的性能限制。
  Developed by Hubertus Franke, Matthew Kirkwood, Ingo Molnár, and Rusty Russell in 2002, futexes were introduced to overcome the performance limitations of System V semaphores.
- 它们已经发展了优先级继承和强大的 futex 等功能，尽管核心 API 仍然专注于简单性和速度。
  They’ve evolved with features like priority inheritance and robust futexes, though the core API remains focused on simplicity and speed.

实用说明
Practical Notes

- API：futex（） 系统调用是通过 syscall（SYS_futex， ...） 因为 glibc 不提供包装器。它采用 futex 地址 （uaddr）、作 （futex_op） 和值 （val） 等参数 。
  API: The futex() system call is invoked via syscall(SYS_futex, ...) since glibc provides no wrapper. It takes parameters like the futex address (uaddr), operation (futex_op), and value (val).
- 文档 ：futex（2） 和 futex（7） 手册页是必不可少的资源，尽管它们可能很简洁。
  Documentation: The futex(2) and futex(7) man pages are essential resources, though they can be terse.
- 漏洞 ：过去的问题，例如 2014 年的拒绝服务漏洞和 2015 年的死锁错误，凸显了谨慎实施的必要性。
  Vulnerabilities: Past issues, like a 2014 denial-of-service vulnerability and a 2015 deadlock bug, highlight the need for careful implementation.

总之，futex 是一种强大的低级同步原语，可在用户空间中实现快速、可扩展的锁定，为许多现代线程库奠定了基础。它们很难直接使用，但对于 Linux 上的高性能并发编程至关重要。如需更深入的见解，请参阅 Linux 手册页或 Ulrich Drepper 的“Futexes Are Tricky”。
In summary, futexes are a powerful, low-level synchronization primitive that enables fast, scalable locking in user space, forming the foundation for many modern threading libraries. They’re tricky to use directly but are critical for high-performance concurrent programming on Linux. For deeper insights, consult the Linux man pages or Ulrich Drepper’s “Futexes Are Tricky.”





