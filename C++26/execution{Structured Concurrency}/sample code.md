#### sample code: 1

```c#
#include <stdexec/execution.hpp>
#include <exec/static_thread_pool.hpp>   // ← 必须包含这个
#include <cstdio>

namespace ex = stdexec;
namespace exec = exec;   // static_thread_pool 所在的命名空间

int main() {
    // 创建一个固定大小的线程池（推荐方式）
    exec::static_thread_pool pool(4);               // 4 个工作线程，可根据你的 CPU 调整
    ex::scheduler auto sched = pool.get_scheduler();

    auto fun = [](int i) { return i * i; };

    // 并行计算三个平方
    auto work = ex::when_all(
        ex::on(sched, ex::just(0) | ex::then(fun)),
        ex::on(sched, ex::just(1) | ex::then(fun)),
        ex::on(sched, ex::just(2) | ex::then(fun))
    );

    // 同步等待结果
    auto [i, j, k] = ex::sync_wait(std::move(work)).value();

    std::printf("%d %d %d\n", i, j, k);   // 输出: 0 1 4

    return 0;
}
```



cmake file:

```cmake
cmake_minimum_required(VERSION 4.2.3)
project(simple_code_26 VERSION 0.1.0 LANGUAGES C CXX)
set(CMAKE_CXX_STANDARD 26)
set(CMAKE_CXX_STANDARD_REQUIRED ON)


file(
  DOWNLOAD
  https://github.com/cpm-cmake/CPM.cmake/releases/download/v0.40.8/CPM.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/cmake/CPM.cmake
  EXPECTED_HASH SHA256=78ba32abdf798bc616bab7c73aac32a17bbd7b06ad9e26a6add69de8f3ae4791
)
include(${CMAKE_CURRENT_BINARY_DIR}/cmake/CPM.cmake)

set(STDEXEC_BUILD_EXAMPLES OFF CACHE BOOL "Build stdexec examples" FORCE)
set(STDEXEC_BUILD_TESTS OFF CACHE BOOL "Build stdexec tests" FORCE)



CPMAddPackage(
  NAME stdexec
  GITHUB_REPOSITORY NVIDIA/stdexec
  GIT_TAG nvhpc-25.09 # or a specific tag
)

add_executable(simple_code_26 main.cpp)

target_link_libraries(simple_code_26 PRIVATE STDEXEC::stdexec)



```

