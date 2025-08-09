# **tcmalloc** 和 **jemalloc**

**tcmalloc** 和 **jemalloc**是两种高性能的内存分配器，设计用于替代标准 C 库中的 malloc（如 glibc 的 malloc），以提高内存分配效率，尤其是在多线程和高负载场景下。它们常用于优化 C++ 应用程序的内存管理，与标准库分配器（如 std::allocator）相比，提供更高效的内存分配和释放机制。

以下是对 **tcmalloc** 和 **jemalloc** 的详细比较

------

1. **tcmalloc 和 jemalloc 简介**

**tcmalloc（Thread-Caching Malloc）**

- **开发背景**：由 Google 开发，集成在 gperftools 中，专为多线程应用程序优化。
- **核心设计**：
  - **线程本地缓存（ThreadCache）**：每个线程有独立的缓存，用于快速分配小对象，减少锁争用。
  - **分层架构**：包括前端（每个核心的缓存）、中端（管理内存片段）和后端（从系统分配大块内存）。
  - **大页支持（Huge Page Awareness）**：优化大内存分配，减少 TLB（Translation Lookaside Buffer）开销。
  - **内存释放**：较新版本支持将内存返还给操作系统，但速度较慢，可通过参数调优。
- **优点**：
  - 在多线程场景下分配小对象时性能优异，延迟低。
  - 针对高并发工作负载（如数据库、Web 服务器）优化。
  - 支持大页（huge pages），在内存密集型应用中减少 TLB 缓存缺失。
- **缺点**：
  - 内存占用较高，尤其在某些工作负载下，驻留内存（RSS）可能显著增加（可能比 glibc malloc 高 2-3 倍）。
  - 对于动态线程创建和销毁的场景，性能可能不如 jemalloc。
  - 内存碎片管理不如 jemalloc 高效，尤其在长期运行的应用程序中。

**jemalloc**

- **开发背景**：最初为 FreeBSD 开发，后被 Facebook 等公司广泛采用（如在 Firefox 和 Redis 中），专注于内存效率和低碎片。
- **核心设计**：
  - **多竞技场（Multiple Arenas）**：将内存分配划分为多个竞技场（arenas），每个线程绑定到一个竞技场，减少锁争用。
  - **细粒度大小分类**：内存分配粒度更精细，减少内存浪费。
  - **高效碎片管理**：通过积极的内存合并和释放，减少长期运行时的内存碎片。
  - **内存返还**：比 tcmalloc 更积极地将未使用的内存返还给操作系统。
- **优点**：
  - 内存使用效率高，驻留内存（RSS）和虚拟内存（VSZ）通常比 tcmalloc 低。
  - 更适合长期运行的应用程序（如数据库），内存碎片较少。
  - 在高并发和多线程场景下表现出色，尤其在动态线程变化较小的场景。
  - 提供调试工具，便于分析内存使用问题。
- **缺点**：
  - 在某些小对象分配密集的场景下，性能可能略逊于 tcmalloc。
  - 对于大页支持和分配速度的优化不如 tcmalloc 激进。

------

2. **与 C++ 标准库分配器的关系**

C++ 标准库分配器（如 std::allocator）是容器（如 std::vector）的内存管理接口，默认使用 new 和 delete，而这些操作通常依赖底层的 malloc 实现（在 Linux 上通常是 glibc 的 malloc）。tcmalloc 和 jemalloc 可以替换 glibc 的 malloc，从而提高标准库分配器的性能。以下是它们与标准库分配器的关联和改进：

- **替换默认分配器**：

  - 可以通过链接 tcmalloc 或 jemalloc 库（例如，使用 LD_PRELOAD 或在编译时指定）替换 glibc 的 malloc，使 std::allocator 使用更高效的分配器。

  - 示例：

    bash

    ```bash
    LD_PRELOAD=/path/to/libtcmalloc.so ./my_program
    ```

    或在 MySQL 中启动时预加载：

    bash

    ~~~bash
    LD_PRELOAD=/path/to/libjemalloc.so mysqld
    ```[](https://www.managedserver.eu/Improve-mysql-and-mariadb-performance-with-memory-allocators-like-jemalloc-and-tcmalloc/)
    ~~~

- **性能提升**：

  - tcmalloc 和 jemalloc 通过线程本地缓存或多竞技场设计减少锁争用，显著提高多线程场景下 std::allocator 的分配效率。
  - 对于频繁分配小对象的容器（如 std::vector 或 std::string），tcmalloc 可将性能提升近一倍，而 jemalloc 在内存效率上更优。

- **自定义分配器**：

  - 如果需要更精细的控制，可以为 C++ 容器编写自定义分配器，调用 tcmalloc 或 jemalloc 的 API（如 tc_malloc 或 je_malloc），但这需要修改代码，而直接替换 malloc 更简单。

- **内存安全**：

  - 与您之前的 Rust vs. C++ 内存安全问题相关，tcmalloc 和 jemalloc 不会像 Rust 的借用检查器那样提供编译期内存安全保证。它们仍然可能导致未定义行为（如双重释放、越界访问），但通过更好的内存管理和调试工具（如 jemalloc 的分析功能），可以减少此类错误的影响。

------

3. **tcmalloc vs. jemalloc 性能比较**

以下是基于最新信息（截至 2025 年 5 月）的性能比较，结合您的问题背景和提供的搜索结果：

**多线程性能**

- **tcmalloc**：
  - 在小对象分配密集的多线程场景下表现出色，因其每个核心的缓存设计减少了锁争用。例如，在一个多线程工具中，tcmalloc 将性能提升近两倍。
  - 在高并发数据库（如 MySQL）测试中，tcmalloc 在 8-32 vCPU 场景下可实现 5000-12500 TPS（每秒事务数），优于 glibc malloc 的 3500-4000 TPS。
  - 但在分配较大对象时，可能因 pageheap_lock 锁争用成为瓶颈。
- **jemalloc**：
  - 通过多竞技场设计减少锁争用，适合线程数量较稳定或长期运行的场景。
  - 在相同数据库测试中，jemalloc 的吞吐量与 tcmalloc 相当（5000-12500 TPS），但在高线程数（如 2048-4096 线程）下更稳定，QPS 下降较少。
  - 在某些微基准测试中，jemalloc 在热点访问（hot-points）场景下比 tcmalloc 快，但在大范围只读查询中可能逊于 tcmalloc。

**内存使用效率**

- **tcmalloc**：
  - 内存占用较高，尤其在长期运行的应用程序中，驻留内存（RSS）可能比 glibc malloc 高 2-3 倍。例如，一个工具的 RSS 从 glibc 的基线增加到 2-3 倍。
  - 在 MySQL 测试中，tcmalloc 的虚拟内存（VSZ）增长较小（+514544K），但 RSS 增长较多（+456028K），比 jemalloc 高约 240MB。
  - 通过大页支持，tcmalloc 在内存密集型应用中可减少 TLB 开销，但碎片管理不如 jemalloc。
- **jemalloc**：
  - 内存效率更高，RSS 和 VSZ 通常低于 tcmalloc。例如，在 MySQL 测试中，jemalloc 的 RSS 增长为 +216084K，优于 tcmalloc 的 +456028K。
  - 在 RocksDB 测试中，jemalloc 显著减少了 RSS（比 glibc malloc 低约 4 GiB），适合内存敏感型应用。
  - 长期运行（如 Hyperledger Besu 两个月测试）显示，jemalloc 内存使用稳定在 4.8 GiB，而 glibc malloc 达到 10 GiB。

**内存碎片**

- **tcmalloc**：
  - 碎片管理较弱，尤其在分配和释放模式不规则时，可能导致内存浪费。例如，glibc malloc 假设逆序释放，而 tcmalloc 通过中端设计改善了这一点，但仍不如 jemalloc。
  - 在数据库（如 RocksDB）中，tcmalloc 对频繁分配/释放的容忍度优于 glibc，但不如 jemalloc。
- **jemalloc**：
  - 碎片管理是其强项，通过细粒度大小分类和积极的内存合并，显著减少碎片。例如，在 MySQL 和 MariaDB 中，jemalloc 将执行时间减半，并降低内存使用。
  - 适合需要高效内存管理的场景，如数据库和长期运行的服务。

**特定应用场景**

- **数据库（MySQL, MariaDB, RocksDB）**：
  - jemalloc 在内存碎片和长期稳定性方面更优，推荐用于内存敏感型数据库。
  - tcmalloc 在高并发、低延迟场景下表现更好，适合多线程工作负载。
  - 例如，Cloudflare 在 RocksDB 上使用 tcmalloc 后，内存使用量降低近三倍。
- **Web 服务器和 Ruby 应用**：
  - 在 Ruby on Rails 测试中，jemalloc 提供约 11% 的吞吐量提升，性能更稳定；tcmalloc 提升 4-9%，但变异较大。
- **动态线程场景**：
  - jemalloc 更适合线程数量稳定的场景，而 tcmalloc 在线程频繁创建/销毁时可能受限于锁争用。

**最新基准测试（2025 年）**

- 在 MySQL/MariaDB 测试（4-32 vCPU）中：
  - 4 vCPU：所有分配器性能相近（约 2500 TPS）。
  - 8 vCPU：jemalloc 和 tcmalloc 达到 5000 TPS，glibc malloc 降至 3500 TPS。
  - 32 vCPU：jemalloc 和 tcmalloc 达到 12500 TPS，glibc malloc 仅 3100 TPS，差距高达 4 倍。
- 在 RocksDB 测试中，jemalloc 和 tcmalloc 的 QPS 相似，但在某些微基准（如大范围只读查询）中，tcmalloc 表现更好，而在热点访问场景中，jemalloc 更优。
- X 平台上的用户反馈显示，实际应用性能因工作负载而异。例如，一位用户报告其应用使用 tcmalloc 时运行时间从 2m0s 缩短到 1m52s，而 jemalloc 为 2m21s，但 mimalloc 表现较差（7m21s）。

------

4. **如何选择 tcmalloc 或 jemalloc？**

**选择 tcmalloc 的场景**

- **高并发低延迟**：如果您的 C++ 应用（如 Web 服务器、数据库）涉及大量小对象分配和多线程操作，tcmalloc 的线程本地缓存和大页支持能显著降低延迟。
- **内存非首要约束**：如果内存使用不是主要瓶颈（如有充足 RAM 或短期运行应用），tcmalloc 的性能优势更突出。
- **大页优化**：在内存密集型应用中，tcmalloc 的大页支持可减少 TLB 缺失。
- **示例**：MySQL 在多线程工作负载下，tcmalloc 减少了分配延迟，提升了吞吐量。

**选择 jemalloc 的场景**

- **内存效率优先**：如果内存使用是关键（如长期运行的服务器、嵌入式系统），jemalloc 的低碎片和高内存返还效率更适合。
- **长期运行稳定性**：在数据库（如 RocksDB、MySQL）或需要稳定内存使用的应用中，jemalloc 表现更佳。
- **调试需求**：jemalloc 提供高级调试工具，便于分析内存问题。
- **示例**：Hyperledger Besu 使用 jemalloc 后，内存使用从 9 GiB 降至 4.8 GiB，长期稳定。

**建议**

- **测试和基准**：由于性能依赖于具体工作负载，建议在目标系统上使用典型工作负载进行基准测试。例如，使用 sysbench 测试 MySQL，或在 C++ 应用中测量 RSS 和吞吐量。
- **混合考虑**：如果内存和性能都很重要，可以先尝试 jemalloc（内存效率高），若延迟成为瓶颈，再切换到 tcmalloc。
- **替代方案**：考虑其他分配器如 **mimalloc**（微软开发，号称在某些基准测试中优于 tcmalloc 和 jemalloc，但可能不稳定）。

------

5. **与 Rust 的关联**

您之前提到 Rust 的内存安全性。Rust 标准库默认使用系统分配器（Linux 上为 glibc malloc），但也可以通过全局分配器切换到 jemalloc 或 tcmalloc。例如，Rust 曾尝试将 jemalloc 作为默认分配器（2013 年），但因维护成本放弃。 在 Rust 中：

- 使用 jemalloc 或 tcmalloc 需要夜间版（nightly）Rust 和 Allocator 特性，配置方式如下：

  rust

  ```rust
  #[global_allocator]
  static ALLOC: jemalloc::Jemalloc = jemalloc::Jemalloc;
  ```

- Rust 的内存安全（通过所有权和借用检查器）在编译期防止了许多 C++ 中常见的错误（如悬垂指针）。

- tcmalloc 和 jemalloc 仅优化分配性能，不会提供这种安全保证，因此在 C++ 中仍需小心管理内存。

------

6. **总结**

- **tcmalloc** 适合高并发、低延迟场景，擅长小对象分配和多线程工作负载，但内存占用较高，碎片管理较弱。
- **jemalloc** 适合内存敏感和长期运行的应用，内存效率高，碎片少，稳定性强，但在某些小对象分配场景下性能可能略逊。
- **与标准库分配器**：两者可通过替换 malloc 提升 std::allocator 的性能，jemalloc 在内存效率上更适合大多数 C++ 容器场景。
- **选择建议**：根据工作负载测试两者，优先考虑 jemalloc（内存效率），若需极致延迟则选 tcmalloc。长期运行或内存受限场景推荐 jemalloc，高并发低延迟场景推荐 tcmalloc。

如果您有具体的 C++ 代码、应用场景（如容器使用模式、线程模型）或需要基准测试指导，请提供更多细节，我可以进一步定制建议或提供代码示例！