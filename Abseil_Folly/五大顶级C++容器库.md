# Top 5 C++ Containers Libraries

#  五大顶级 C++容器库

C++ containers are essential for managing collections of data with different requirements for organization, access, and modification. They allow developers to manage groups of objects efficiently, without implementing data structures from scratch.
C++ 容器对于管理具有不同组织、访问和修改要求的数据集合至关重要。它们使开发者能够高效管理对象组，而无需从零实现数据结构。



The video below provides an introduction to the basics of C++ containers.
下面的视频介绍了 C++ 容器的基础知识。

### 1. **Boost Containers** 

- **Description**: [Boost](https://www.boost.org/) is one of the most extensive and widely used libraries in the C++ community, providing many container types that extend STL’s functionality.
  **描述** ：[Boost](https://www.boost.org/) 是 C++ 社区中最广泛且最广泛使用的库之一，提供了多种容器类型，扩展了 STL 的功能。
- **Notable Containers**: `boost::unordered_map`, `boost::multi_index_container`, and `boost::circular_buffer`.
  **著名容器** ：`boost：：unordered_map`、`boost：multi_index_container` 和 `boost：：circular_buffer`。
- **Strengths**: Known for being highly optimized and reliable, Boost provides additional containers, like the multi-index container, that allow multiple ways to index data, which STL lacks.
  **优势** ：Boost 以高度优化和可靠性著称，提供了额外的容器，如多索引容器，允许多种方式索引数据，而 STL 则缺乏这些。
- **Use Cases**: When you need containers with specialized behaviors (e.g., circular buffers or multi-indexed data structures) that are also compatible with STL.
  **使用场景** ：当你需要具备特殊行为（如循环缓冲区或多索引数据结构）且兼容 STL 的容器时。

### 3. **Abseil Containers (from Google)** 3. **绳降容器（来自谷歌）**

- **Description**: [Abseil](https://github.com/abseil/abseil-cpp) provides optimized versions of standard containers as part of Google’s suite of C++ libraries.
  **描述** ：[Abseil](https://github.com/abseil/abseil-cpp) 作为 Google C++ 库套件的一部分，提供了标准容器的优化版本。
- **Notable Containers**: `absl::flat_hash_map`, `absl::flat_hash_set`, and `absl::InlinedVector`.
  **著名容器** ：`absl：：flat_hash_map`、`absl：：flat_hash_set` 和 `absl：：InlinedVector`。
- **Strengths**: Abseil containers are designed for performance, particularly in reducing memory fragmentation and improving cache locality. `absl::flat_hash_map` is known for its efficiency compared to `std::unordered_map`.
  **优点** ：绳降容器设计注重性能，特别是减少内存碎片和改善缓存局部性。`ABSL：：flat_hash_map` 相较于 `STD：：unordered_map` 以效率著称。
- **Use Cases**: Ideal for performance-critical applications where memory and speed are priorities, Abseil containers are optimized for scenarios common in Google-scale applications.
  **使用场景** ：对于性能要求高、内存和速度为优先的应用，绳降容器针对谷歌规模应用中常见场景进行了优化。

### 4. **Intel Threading Building Blocks (TBB) Containers** 



- **Description**: Part of [Intel’s TBB library](https://oneapi-src.github.io/oneTBB/main/tbb_userguide/Containers.html), these containers are designed with multi-threading in mind.
  **描述** ：作为[英特尔 TBB 库](https://oneapi-src.github.io/oneTBB/main/tbb_userguide/Containers.html)的一部分，这些容器设计时考虑了多线程。
- **Notable Containers**: `tbb::concurrent_vector`, `tbb::concurrent_hash_map`, and `tbb::concurrent_queue`.
  **值得一提的容器** ：` 待定：concurrent_vector`、` 待续：：concurrent_hash_map` 和`待定：concurrent_queue`。
- **Strengths**: Provides thread-safe versions of common containers, optimized for high-performance, concurrent access without the need for external synchronization.
  **优势：** 提供常见容器的线程安全版本，优化为高性能并发访问，无需外部同步。
- **Use Cases**: Best suited for multi-threaded environments where multiple threads need to safely access and modify shared data without explicit locking.
  **使用场景** ：最适合多线程环境，需要多线程安全访问和修改共享数据且不显式锁定。

### 5. **Eastl (Electronic Arts Standard Template Library)** 

- **Description**: [EA’s own version of the STL](https://github.com/electronicarts/EASTL), designed specifically for performance in gaming environments.
  **描述** ：[EA 自家版本的 STL，](https://github.com/electronicarts/EASTL) 专为游戏环境性能设计。
- **Notable Containers**: `eastl::vector`, `eastl::fixed_vector`, and `eastl::hash_map`.
  **著名容器** ：`eastl：：vector`、`eastl：：fixed_vector` 和 `eastl：：hash_map`。
- **Strengths**: EA’s containers are optimized for speed, often outperforming the STL in real-time applications. Many containers in EASTL are designed with fixed-size allocations to avoid dynamic memory overhead.
  **优势：**EA 的容器优化速度，常常在实时应用中优于 STL。许多 EASTL 容器采用固定大小分配，以避免动态内存开销。
- **Use Cases**: Primarily in game development or other real-time applications where perfoSSSrmance is critical, and memory allocations need to be minimized.
  **使用场景** ：主要用于游戏开发或其他实时应用，性能至关重要且需要最小化内存分配。

Each of these libraries provides container options tailored to specific application needs, whether for high-performance single-threaded applications, multi-threaded systems, or resource-constrained environments like gaming.
这些库都提供针对特定应用需求的容器选项，无论是高性能单线程应用、多线程系统，还是像游戏这样资源受限的环境。