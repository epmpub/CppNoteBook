C++26 “Native handles in file streams”（提案 P1759R6）为文件流（std::basic_filebuf、basic_ifstream、basic_ofstream、basic_fstream）增加了获取底层平台原生文件句柄（native handle）的标准化方式。 

open-std.org

1. 新增成员在 <fstream> 头文件中：

cpp

```cpp
// 对于 basic_filebuf
template<class CharT, class Traits = std::char_traits<CharT>>
class basic_filebuf : public basic_streambuf<CharT, Traits> {
public:
    using native_handle_type = /* implementation-defined */;  // C++26

    native_handle_type native_handle() const noexcept;  // C++26
    // ...
};

// 类似地，basic_ifstream / basic_ofstream / basic_fstream 也增加了：
using native_handle_type = typename basic_filebuf<CharT, Traits>::native_handle_type;

native_handle_type native_handle() const noexcept;
```

- native_handle_type：实现定义的类型，必须满足：
  - TriviallyCopyable
  - Semiregular（默认可构造、可复制、可销毁）
  - 常见平台：POSIX → int（文件描述符 fd）；Windows → HANDLE（即 void*）
- native_handle()：返回当前文件关联的原生句柄。
  - 前提：必须 is_open() == true，否则行为未定义（UB）。
  - 返回的句柄是非拥有（non-owning）的，只在流保持打开期间有效。
  - 函数是 const noexcept。
- 主要用途允许在继续使用 C++ 流接口的同时，调用平台特定的系统调用，而无需重新打开文件（避免竞争条件和缓冲同步问题）：

- 文件锁定（fcntl / LockFile）
- fstat 获取文件元数据（最后修改时间、inode 等）
- 矢量 I/O（scatter-gather）
- 非阻塞 I/O 设置
- 其他需要原生句柄的 API

示例（POSIX）：

cpp

```cpp
#include <fstream>
#include <fcntl.h>
#include <sys/stat.h>
#include <iostream>

int main() {
    std::ofstream ofs("test.txt");
    if (!ofs) return 1;

    // 获取原生 fd
    auto fd = ofs.native_handle();   // int on POSIX

    // 使用系统调用
    struct stat st;
    if (fstat(fd, &st) == 0) {
        std::cout << "File size: " << st.st_size << " bytes\n";
        std::cout << "Last modified: " << st.st_mtime << "\n";
    }

    ofs << "Hello, C++26!\n";  // 继续正常使用流
}
```

Windows 上 native_handle() 返回 HANDLE，可用于 GetFileInformationByHandle 等 API。3. 特性测试宏

cpp

```cpp
#ifdef __cpp_lib_fstream_native_handle
    // 值通常为 202306L
#endif
```

4. 设计要点与限制

- 只读句柄：只提供了获取句柄的功能，没有提供从原生句柄构造 fstream 的功能（出于所有权和实现复杂性考虑）。
- 非移植：句柄类型和语义是平台相关的，使用时需 #ifdef 或其他方式处理不同平台。
- 实现：各大库（如 libstdc++、libc++、MSVC）已较容易实现，因为它们内部通常持有 FILE*，再通过 fileno / _get_osfhandle 转换即可。
- 与现有 std::thread / std::mutex 的 native_handle 不同，此功能是强制要求的（而非实现定义是否存在）。

这个特性极大提升了 iostreams 在需要底层系统调用场景下的实用性，尤其适合需要高性能 I/O 或特定平台特性的代码，同时保留 RAII 的便利性。 

en.cppreference.com

更多细节可参考：

- [cppreference: basic_fstream::native_handle](https://en.cppreference.com/cpp/io/basic_fstream/native_handle)
- 提案 [P1759R6](https://wg21.link/P1759)