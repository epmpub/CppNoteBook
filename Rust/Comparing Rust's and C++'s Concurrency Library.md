# Comparing Rust's and C++'s Concurrency Library

2022-08-16 2579 words 13 mins read

## Contents

The concurrency features that are included in the Rust standard library are quite similar to what was available in C++11: threads, atomics, mutexes, condition variables, and so on. In the past few years, however, C++ has gained quite a few new concurrency related features as part C++17 and C++20, with more proposals still coming in for future versions.

Let’s take some time to review C++ concurrency features, discuss what their Rust equivalent could look like, and what it’d take to get there.

## atomic_ref

[P0019R8](https://wg21.link/p0019r8) introduced [`std::atomic_ref`](https://en.cppreference.com/w/cpp/atomic/atomic_ref) to C++. It’s a type that allows you to use a non-atomic object as an atomic one. For example, you can create a `atomic_ref<int>` that references a regular `int`, allowing you the same functionality as if it were an `atomic<int>`.

While in C++ this needed a whole new type that duplicates most of the `atomic` interface, the equivalent Rust feature is a one-line function: `Atomic*::from_mut`. This function allows you to convert, for example, a `&mut u32` to a `&AtomicU32`, which is a form of aliasing that’s perfectly sound in Rust.





The C++ `atomic_ref` type comes with safety requirements that you need to uphold manually. As long as you’re using an `atomic_ref` to access an object, all access to that object must be through an `atomic_ref`. Accessing it directly when there’s still an `atomic_ref` results in undefined behavior.

In Rust, however, this is already fully taken care of by the borrow checker. The compiler understands that by borrowing the `u32` mutably, nothing is allowed to access that `u32` directly until that borrow ends. The lifetime of the `&mut u32` that goes into the `from_mut` function is preserved as part of the `&AtomicU32` you get out of it. You can make as many copies of that `&AtomicU32` as you want, but the original borrow only ends once all copies of that reference are gone.

The [`from_mut`](https://doc.rust-lang.org/stable/std/sync/atomic/struct.AtomicBool.html#method.from_mut) function is currently unstable, but perhaps it’s time we stabilize it.

## Generic atomic type

In C++, the `std::atomic` is generic: you can have a `atomic<int>`, but also an `atomic<MyOwnStruct>`. In Rust, on the other hand, we only have specific atomic types: `AtomicU32`, `AtomicBool`, `AtomicUsize`, etc.

C++’s atomic type supports objects of any size, regardless of what the platform supports. It automatically falls back to a lock-based implementation for objects of a size that are not supported by the platform’s native atomic operations. On the other hand, Rust only provides the types that are natively supported by the platform. If you’re compiling for a platform that does not have 64 bit atomics, `AtomicU64` does not exist.

This has advantages and disadvantages. It means Rust code using `AtomicU64` might fail to compile for certain platforms, but it also means no performance related surprises when some types silently fall back to a very different implementation. It also means we can assume a `AtomicU64` is represented exactly the same as an `u64` in memory, allowing for functions like `AtomicU64::from_mut`.

Having a generic `Atomic<T>` in Rust that works for types of any size can be tricky. Without specialization, we can’t make `Atomic<LargeThing>` include a `Mutex`, while not including it in `Atomic<SmallThing>`. What we could do, however, is to store the mutexes in a global `HashMap`, indexed by memory address. Then the `Atomic<T>` can be identical in size to a `T`, and use a `Mutex` from this global hash map when necessary.

This is exactly what the popular [`atomic` crate](https://docs.rs/atomic/) does.

A proposal for adding such a universal `Atomic<T>` type to the Rust standard library would need to discuss whether it should be usable in `no_std` programs. A regular `HashMap` requires allocation, which isn’t possible in `no_std` programs. A fixed size table could work for `no_std` programs, but might be undesirable for various reasons.

## Compare-exchange with padding

[P0528R3](https://wg21.link/p0528r3) changes how `compare_exchange` deals with padding. A compare exchange operation on a `atomic<TypeWithPadding>` used to compare the padding bits as well, but that turned out to be a bad idea. Nowadays, padding bits are no longer included in the comparison.

Since Rust currently only provides atomic types for integers, without any padding, this change is irrelevant for Rust.

However, a proposal for a `Atomic<T>` with a `compare_exchange` method would need to discuss how padding is handled, and should probably take input from this proposal.

## Compare-exchange memory ordering

In C++11, the [`compare_exchange`](https://en.cppreference.com/w/cpp/atomic/atomic/compare_exchange) functions required the success memory ordering to be at least as strong as the failure ordering. A `compare_exchange(…, …, memory_order_release, memory_order_acquire)` was not accepted. This requirement was copied verbatim to Rust’s [`compare_exchange`](https://doc.rust-lang.org/stable/std/sync/atomic/struct.AtomicU32.html#method.compare_exchange) functions.

[P0418R2](https://wg21.link/p0418r2) argued that this restriction should be lifted, which happened as part of C++17.

The same restriction is lifted as part of Rust 1.64, as part of [rust-lang/rust#98383](https://github.com/rust-lang/rust/pull/98383).

## constexpr Mutex constructor

C++’s `std::mutex` has a `constexpr` constructor, which means it can be constructed as part of constant evaulation at compile time. However, not all implementations actually provide this. For example, Microsoft’s implementation of `std::mutex` doesn’t include a `constexpr` constructor. So, relying on this is a bad idea for portable code.

Also, interestingly, C++’s `std::condition_variable` and `std::shared_mutex` don’t provide a `constexpr` constructor at all.

Rust’s original `Mutex` in Rust 1.0 did not include a `const fn new`. Combined with how Rust’s strict requirements for static initialization, this made the `Mutex` quite annoying to use in a `static` variable.

This has been resolved in [Rust 1.63.0](https://blog.rust-lang.org/2022/08/11/Rust-1.63.0.html) as part of [rust-lang/rust#93740](https://github.com/rust-lang/rust/issues/93740): *all* of `Mutex::new`, `RwLock::new` and `Condvar::new` are now `const` functions.

## Latches and barriers

[P1135R6](https://wg21.link/p1135r6) introduced, among other things, `std::latch` and `std::barrier` to C++20. Both are types that allow waiting for several threads to reach a certain point. A latch is basically just a counter that gets decremented by each thread and allows you to wait for it to reach zero. It can only be used once. A barrier is a more advanced version of this idea that can be reused, and accepts a “completion function” to be automatically executed when the counter reaches zero.

Rust has had a similar [`Barrier`](https://doc.rust-lang.org/stable/std/sync/struct.Barrier.html) type since 1.0. It was inspired by pthread (`pthread_barrier_t`) rather than C++.

Rust’s (and pthread’s) barrier is less flexible than what’s now included in C++. It only has a “decrement and wait” operation (called `wait`), and lacks the “only wait”, “only decrement”, and “decrement and drop” functions that C++’s [`std::barrier`](https://en.cppreference.com/w/cpp/thread/barrier) comes with.

On the other hand, unlike C++, Rust’s (and pthread’s) “decrement and wait” operation assigns one thread to be the group leader. This is a (perhaps more flexible) alternative to a completion function.

The missing operations on the Rust version could easily be added at any point. All we need is a good proposal for the names of these new methods. :)



- [x] ## Semaphore

That same [P1135R6](https://wg21.link/p1135r6) also added semaphores to C++20: [`std::counting_semaphore` and `std::binary_semaphore`](https://en.cppreference.com/w/cpp/thread/counting_semaphore).

Rust does not have a general semaphore type, although it does equip every single thread with what’s effectively a binary semaphore, through [`thread::park` and `unpark`](https://doc.rust-lang.org/stable/std/thread/fn.park.html).

A semaphore can be easily constructed manually using a `Mutex<u32>` and a `Condvar`, but most operating systems allow for a more efficient and smaller implementation using a single `AtomicU32`. For example, through [`futex()`](https://man7.org/linux/man-pages/man2/futex.2.html) on Linux and [`WaitOnAddress()`](https://docs.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-waitonaddress) on Windows. It depends on the operating system and its version which sizes of atomics can be used for these operations.

C++’s `counting_semaphore` is a template that takes an integer as argument to indicate how far we want to be able to count. For example, a `counting_semaphore<1000>` can count up to at least 1000, and will therefore be 16 bit or larger. The `binary_semaphore` type is just an alias for `counting_semaphore<1>`, and can be a single byte on some platforms.

In Rust, we’re probably not quite ready for this kind of generic type any time soon. Rust’s generics force a certain kind of consistency that puts some limitations on what we can do with constants as generic arguments.

We could have separate `Semaphore32`, `Semaphore64`, and so on, but that seems a bit overkill. Having `Semaphore<u32>`, `Semaphore<u64>` and perhaps even `Semaphore<bool>` could be possible, but is something we haven’t done before in the standard library. Our atomic types are simply `AtomicU32`, `AtomicU64`, and so on.

As mentioned above, for our atomic types, we only provide the ones that are natively supported by the platform you’re compiling for. If we were to apply the same philosophy to `Semaphore`, it wouldn’t exist on platforms that don’t have a `futex` or `WaitOnAddress` function, such as macOS. And if we had separate semaphore types for different sizes, some sizes wouldn’t exist on (some versions of) Linux and various BSDs.

If we want a standard semaphore type in Rust, we’d first need some input on whether we actually need semaphores of different sizes, and what form of flexibility and portability would be necessary to make them useful. Perhaps we should go with just a single 32-bit `Semaphore` type that’s always available (using a lock-based fallback), but any such proposal would have to include a detailed explanation of use cases and limitations.



## Atomic wait and notify

The remaining new features that [P1135R6](https://wg21.link/p1135r6) adds to C++20 are the atomic [`wait` and `notify` functions](https://en.cppreference.com/w/cpp/atomic/atomic/wait).

These functions effectively directly expose Linux’s [`futex()`](https://man7.org/linux/man-pages/man2/futex.2.html) and Windows’s [`WaitOnAddress()`](https://docs.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-waitonaddress) through a standard interface.

However, they are available on atomics of all sizes, on all platforms, regardless of what the operating system supports. Linux futexes are always 32 bit, but C++ allows for `atomic<uint64_t>::wait` just fine.

A way of doing this, is using something resembling a [“parking lot”](https://webkit.org/blog/6161/locking-in-webkit/): effectively a global `HashMap` that maps memory addresses to locks and queues. That means that a 32 bit wait operation on Linux could use the very fast futex based implementation, while the other sizes would use a very different implementation.

If we were to follow the philosophy of only providing the types and functions that are natively supported (like we do for the atomic types), we wouldn’t provide such a fallback implementation. That’d mean we only have `AtomicU32::wait` (and `AtomicI32::wait`) on Linux, while all atomic types would include this `wait` method on Windows.

A proposal for `Atomic*::wait` and `Atomic*::notify` in Rust would need to include a discussion on whether a fall back to a global table is desirable in Rust or not.





## jthread and stop_token

[P0660R10](https://wg21.link/p0660r10) adds [`std::jthread`](https://en.cppreference.com/w/cpp/thread/jthread) and [`std::stop_token`](https://en.cppreference.com/w/cpp/thread/stop_token) to C++20.

If we ignore the `stop_token` for a second, `jthread` is basically just a regular `std::thread` that automatically gets `join()`‘ed on destruction. This avoids accidentally detaching a thread and letting it run for longer than expected, which might happen with a regular `thread`. However, it also introduces a potential new pitfall: immediately destructing a `jthread` object will immediately join the thread, effectively removing any potential parallelism.

As of [Rust 1.63.0](https://blog.rust-lang.org/2022/08/11/Rust-1.63.0.html), we have [scoped threads](https://doc.rust-lang.org/stable/std/thread/fn.scope.html) ([rust-lang/rust#93203](https://github.com/rust-lang/rust/issues/93203)). Just like a `jthread`, a scoped thread is automatically joined. However, point before which they are joined is made explicit, and is a guarantee that can be relied upon for safety. The borrow checker even understands this guarantee, allowing you to safely borrow local variables in the scoped thread(s), as long as those variables outlive the scope.

In addition to automatically joining, a main feature of `jthreads` is their `stop_token` and corresponding `stop_source`. One can call `request_stop()` on a `stop_source` to make the corresponding `stop_requested()` method on `stop_token` return true. This can be used to nicely ask the thread to please stop, and is automatically done in the destructor of `jthread` before joining. It’s up to the code of the thread to actually check the token and stop if it was set.

So far, it almost looks like a plain `AtomicBool`.

Where things get very different is the [`stop_callback`](https://en.cppreference.com/w/cpp/thread/stop_callback) type. This type allows registering a callback, a “stop function”, to be registered with a stop token. Requesting a stop using the corresponding stop source will execute this function. Effectively, a thread can use this to let others know how to stop or cancel its work.

In Rust, we could easily add the `AtomicBool`-like functionality to the `Scope` object of `thread::scope`. A simple `is_finished(&self) -> bool` or `stop_requested(&self) -> bool` that indicates whether the main `scope` function is finished might suffice. Maybe combined with a `request_stop(&self)` method to request it from anywhere.

The `stop_callback` feature is more complicated, and any Rust equivalent would probably need a detailed proposal discussing its interface, use cases and limitations.



## Atomic floats

[P0020R6](https://wg21.link/p0020r6) adds support for atomic floating point addition and subtraction to C++20.

It’d be easy to add a `AtomicF32` or `AtomicF64` to Rust as well, but it seems that the only platforms that natively support atomic floating point operations are some GPUs that are not supported by Rust (yet?).

A proposal to add these types to Rust would have to present some compelling use cases.



## ！！ Atomic per byte memcpy

Currently, it’s not possible to efficiently implement [sequence locks](https://en.wikipedia.org/wiki/Seqlock) in Rust or C++ that abides by all the rules of the memory model.

[P1478R7](https://wg21.link/p1478r7) proposes to add `atomic_load_per_byte_memcpy` and `atomic_store_per_byte_memcpy` to a future version of C++ to solve this issue.

For Rust, I wrote a proposal to expose the functionality through a `AtomicPerByte<T>` type: [RFC 3301](https://github.com/rust-lang/rfcs/pull/3301).





## Atomic shared_ptr

[P0718R2](https://wg21.link/p0718r2) added specializations for `atomic<shared_ptr>` and `atomic<weak_ptr>` to C++20.

Reference counted pointers (`shared_ptr` in C++, `Arc` in Rust) are quite commonly used for concurrent lock-free data structures. The `atomic<shared_ptr>` specialization makes it easier to do this correctly, by handling the reference count properly.

In Rust, we could add equivalent `AtomicArc<T>` and `AtomicWeak<T>` types. (Although `AtomicArc` sounds a bit weird maybe, considering the `A` of `Arc` already stands for “atomic”. :) )

However, C++’s `shared_ptr<T>` is nullable, while in Rust that requires a `Option<Arc<T>>`. It’s not immediately clear whether `AtomicArc<T>` should be nullable, or whether should we also have a `AtomicOptionArc<T>`.

The popular [`arc-swap` crate](https://docs.rs/arc-swap/) already provides all these variants in Rust, but, as far as I know, there hasn’t been any proposal yet to add anything similar to the standard library.



## synchronized_value

[P0290R2](https://wg21.link/p0290r2) was not accepted, but proposed a type called `synchronized_value<T>` which combines a `mutex` with a `T`. Even though it wasn’t accepted at that time into C++, it’s an interesting proposal, because `synchronized_value<T>` is pretty much exactly what a `Mutex<T>` is in Rust.



In C++, a `std::mutex` does not contain the data it protects, nor does it even know what it is protecting at all. This means that it is the responsibility of the user to remember which data is protected and by which mutex, and ensure the right mutex is locked every time “protected” data is accessed.

Rust’s `Mutex<T>` design with a `MutexGuard` behaving like a (mutable) reference to `T` allows for much more safety, while still allowing for a `Mutex<()>` in cases where you need only a mutex, without any data directly attached to it. The proposal for `synchronized_value<T>` was an attempt at adding this pattern to C++, but used closures instead of a mutex guard, since C++ doesn’t track lifetimes.



## Conclusion

It seems to me that C++ can continue to be a source of inspiration for Rust, although we should take care not to copy-paste ideas directly. As we’ve seen with `Mutex<T>`, scoped threads, `Atomic*::from_mut` and others, things can often take a very different (often more ergonomic) shape in Rust while providing the same functionality.

Providing the exact same functionality as C++ shouldn’t be a primary goal. The goal should be to provide exactly what the Rust ecosystem needs from the language and standard library, which might be different than what C++ users need from their language.

If you have concurrency needs from the Rust standard library that we currently don’t fulfill, I’d love to hear from you, regardless of whether it’s something that’s already solved in another language or not.