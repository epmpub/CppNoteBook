# Higher-level synchronization objects

Most of the low-level synchronization primitives are quite error-prone and inconvenient to use, which is why the standard library also exposes some higher-level synchronization objects.

These abstractions can be built out of lower-level primitives. For efficiency, the sync objects in the standard library are usually implemented with help from the operating system’s kernel, which is able to reschedule the threads while they are blocked on acquiring a lock.

The following is an overview of the available synchronization objects:

- [`Arc`](http://127.0.0.1:52485/rust/doc.rust-lang.org/1.88.0/std/sync/struct.Arc.html): Atomically Reference-Counted pointer, which can be used in multithreaded environments to prolong the lifetime of some data until all the threads have finished using it.
- [`Barrier`](http://127.0.0.1:52485/rust/doc.rust-lang.org/1.88.0/std/sync/struct.Barrier.html): Ensures multiple threads will wait for each other to reach a point in the program, before continuing execution all together.
- [`Condvar`](http://127.0.0.1:52485/rust/doc.rust-lang.org/1.88.0/std/sync/struct.Condvar.html): Condition Variable, providing the ability to block a thread while waiting for an event to occur.
- [`mpsc`](http://127.0.0.1:52485/rust/doc.rust-lang.org/1.88.0/std/sync/mpsc/index.html): Multi-producer, single-consumer queues, used for message-based communication. Can provide a lightweight inter-thread synchronisation mechanism, at the cost of some extra memory.
- [`mpmc`](http://127.0.0.1:52485/rust/doc.rust-lang.org/1.88.0/std/sync/mpmc/index.html): Multi-producer, multi-consumer queues, used for message-based communication. Can provide a lightweight inter-thread synchronisation mechanism, at the cost of some extra memory.
- [`Mutex`](http://127.0.0.1:52485/rust/doc.rust-lang.org/1.88.0/std/sync/struct.Mutex.html): Mutual Exclusion mechanism, which ensures that at most one thread at a time is able to access some data.
- [`Once`](http://127.0.0.1:52485/rust/doc.rust-lang.org/1.88.0/std/sync/struct.Once.html): Used for a thread-safe, one-time global initialization routine. Mostly useful for implementing other types like `OnceLock`.
- [`OnceLock`](http://127.0.0.1:52485/rust/doc.rust-lang.org/1.88.0/std/sync/struct.OnceLock.html): Used for thread-safe, one-time initialization of a variable, with potentially different initializers based on the caller.
- [`LazyLock`](http://127.0.0.1:52485/rust/doc.rust-lang.org/1.88.0/std/sync/struct.LazyLock.html): Used for thread-safe, one-time initialization of a variable, using one nullary initializer function provided at creation.
- [`RwLock`](http://127.0.0.1:52485/rust/doc.rust-lang.org/1.88.0/std/sync/struct.RwLock.html): Provides a mutual exclusion mechanism which allows multiple readers at the same time, while allowing only one writer at a time. In some cases, this can be more efficient than a mutex.



- A **multiprocessor** system executing multiple hardware threads at the same time: In multi-threaded scenarios, you can use two kinds of primitives to deal with synchronization:
  - [memory fences](http://127.0.0.1:52485/rust/doc.rust-lang.org/1.88.0/std/sync/atomic/fn.fence.html) to ensure memory accesses are made visible to other CPUs in the right order.
  - [atomic operations](http://127.0.0.1:52485/rust/doc.rust-lang.org/1.88.0/std/sync/atomic/index.html) to ensure simultaneous access to the same memory location doesn’t lead to undefined behavior.