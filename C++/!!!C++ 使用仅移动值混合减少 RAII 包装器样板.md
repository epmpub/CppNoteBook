# C++ 使用仅移动值混合减少 RAII 包装器样板

如果您与遗留或高度可移植的 API 进行交互，您会经常遇到由简单类型表示的各种类型的句柄。

在 C++ 中，我们希望将这些句柄包装在 RAII 包装器中，以确保我们不会泄漏资源或在它们消失后访问它们。

对于指针句柄，我们可以使用*std::unique_ptr*和最近添加的（C++23）*std::out_ptr*和*std::inout_ptr*。

但是，如果句柄不是指针或者由多个变量表示（例如句柄+大小），那么您将需要实现样板 RAII 包装器。

使用可重复使用的 Mixin 可以简化这一过程。



```C++
#include <utility>
#include <filesystem>
#include <span>
#include <cassert>
#include <fcntl.h>
#include <unistd.h>
#include <sys/mman.h>
#include <sys/stat.h>

enum class Style : bool { Implicit, Explicit };

template <typename T, T empty_value = T{}, 
          Style style = Style::Explicit, auto cleanup = [](T){}>
struct MoveOnly {
    static constexpr bool is_explicit = (style == Style::Explicit);

    // Default construction
    MoveOnly() : store_(empty_value) {}
    // Constructor from the base type
    explicit(is_explicit) MoveOnly(T value) 
        : store_(std::move(value)) {}

    // Move constructor
    MoveOnly(MoveOnly&& other)
        : store_(std::exchange(other.store_, empty_value)) {}
    // Move assignment
    MoveOnly& operator=(MoveOnly&& other) {
        store_ = std::exchange(other.store_, empty_value);
        return *this;
    }

    // Destructor
    ~MoveOnly() {
        cleanup(store_);
    }

    // Conversion to the base type
    [[nodiscard]] explicit(is_explicit) operator T() const {
        return store_;
    }

    // Explicit access to the stored value
    [[nodiscard]] T get() const { return store_; }
    void set(T value) { store_ = std::move(value); }
private:
    template <typename>
    friend struct ingest;
    T store_;
};

// Alternative to var.set(value) for explicit style
// ingest{var} = value;
template <typename T>
struct ingest {
    template <T v, Style s, auto c>
    ingest(MoveOnly<T,v,s,c>& in) : store_(in.store_) {}
    ingest& operator=(T value) {
        store_ = value;
        return *this;
    }
private:
    T& store_;
};

// Example use for a single value, used as a Mixin
struct UnixFile : MoveOnly<int, -1, Style::Implicit, 
                    [](int fd) { if (fd != -1) close(fd); }> {
    UnixFile(const std::filesystem::path& path)
        : MoveOnly(open(path.c_str(), O_RDONLY)) {
        if (*this == -1)
            throw std::runtime_error("Failed to open file.");
    }
};

// Example use for multiple values used as members
struct MemoryMappedFile {
    MemoryMappedFile(const std::filesystem::path& path)
        : file_(path) {
        struct stat sb;
        if (fstat(file_, &sb) == 1)
            throw std::system_error(errno, std::system_category(),
                                    "Failed to read file stats");
        sz_.set(sb.st_size); // Explicit interface

        // begin_.set(...) would be less readable
        ingest{begin_} = static_cast<char *>(
            mmap(nullptr, sz_.get(), PROT_READ, MAP_PRIVATE, file_, 0));
        if (begin_.get() == MAP_FAILED)
            throw std::system_error(errno, std::system_category(),
                                    "Failed to map file to memory");
    }

    // We still have to declare move constructor/assignment
    // disabled by providing a custom destructor
    MemoryMappedFile(MemoryMappedFile&&) = default;
    MemoryMappedFile& operator=(MemoryMappedFile&&) = default;

    ~MemoryMappedFile() {
        if (begin_.get() != nullptr)
            munmap(begin_.get(), sz_.get());
    }
    std::span<const char> file_content() {
        assert(begin_.get() != nullptr);
        return {begin_.get(), sz_.get()};
    }
private:
    UnixFile file_;
    MoveOnly<char *> begin_;
    MoveOnly<size_t> sz_;
};

#include <iostream>

int main() {
    UnixFile f("./example.cpp");
    auto moved = std::move(f);
    std::cout << "./example.cpp opened as file descriptor no. " << (int)moved << "\n";
    std::cout << "moved from state: " << (int)f << "\n";

    MemoryMappedFile mf("./example.cpp");
    auto mmap_moved = std::move(mf);

    std::cout << "first 20 bytes of ./example.cpp:\n";
    for (char c : mmap_moved.file_content().subspan(0, 20))
        std::cout << c;
    std::cout << '\n';
}
```

这段代码展示了 C++ 中如何实现仅移动（move-only）类型的资源管理类，利用模板和 RAII（Resource Acquisition Is Initialization）管理如文件描述符和内存映射等系统资源。代码定义了一个通用的 MoveOnly 模板类，并通过特化和组合将其应用于 Unix 文件操作和内存映射文件管理。以下是对代码的详细解释：

------

**代码结构**

1. **头文件**：
   - <utility>：提供 std::exchange 和 std::move。
   - <filesystem>：提供 std::filesystem::path。
   - <span>：提供 std::span。
   - <cassert>：提供 assert。
   - <fcntl.h>、<unistd.h>、<sys/mman.h>、<sys/stat.h>：Unix 系统调用相关头文件。
2. **主要内容**：
   - MoveOnly：通用仅移动类型模板。
   - ingest：辅助类，用于显式风格的赋值。
   - UnixFile：管理文件描述符。
   - MemoryMappedFile：管理内存映射文件。

------

**代码逐步解释**

**1. MoveOnly 模板**

cpp

```cpp
enum class Style : bool { Implicit, Explicit };

template <typename T, T empty_value = T{}, 
          Style style = Style::Explicit, auto cleanup = [](T){}>
struct MoveOnly {
    static constexpr bool is_explicit = (style == Style::Explicit);

    MoveOnly() : store_(empty_value) {}
    explicit(is_explicit) MoveOnly(T value) : store_(std::move(value)) {}
    MoveOnly(MoveOnly&& other) : store_(std::exchange(other.store_, empty_value)) {}
    MoveOnly& operator=(MoveOnly&& other) {
        store_ = std::exchange(other.store_, empty_value);
        return *this;
    }
    ~MoveOnly() { cleanup(store_); }
    [[nodiscard]] explicit(is_explicit) operator T() const { return store_; }
    [[nodiscard]] T get() const { return store_; }
    void set(T value) { store_ = std::move(value); }
private:
    T store_;
};
```

- **Style**：
  - 枚举控制类型转换是隐式（Implicit）还是显式（Explicit）。
- **模板参数**：
  - T：存储类型。
  - empty_value：默认值（例如文件描述符的 -1）。
  - style：控制转换风格，默认 Explicit。
  - cleanup：清理函数（如关闭文件），默认空操作。
- **构造函数**：
  - 默认构造：初始化为 empty_value。
  - 值构造：根据 is_explicit 使用 explicit，接受 T 并移动。
  - 移动构造：使用 std::exchange 转移资源，旧对象置为 empty_value。
- **移动赋值**：
  - 类似移动构造，转移资源。
- **析构函数**：
  - 调用 cleanup 释放资源。
- **访问**：
  - operator T()：根据 style 提供类型转换。
  - get() 和 set()：显式访问和修改值。
- **特性**：
  - 仅移动（无拷贝构造函数），确保资源独占。

------

**2. ingest 辅助类**

cpp

```cpp
template <typename T>
struct ingest {
    template <T v, Style s, auto c>
    ingest(MoveOnly<T,v,s,c>& in) : store_(in.store_) {}
    ingest& operator=(T value) {
        store_ = value;
        return *this;
    }
private:
    T& store_;
};
```

- **作用**：
  - 为显式风格的 MoveOnly 提供赋值接口。
- **构造**：
  - 接受 MoveOnly 引用，绑定到其 store_。
- **operator=**：
  - 将新值赋给 store_。
- **用法**：
  - ingest{var} = value 替代 var.set(value)。

------

**3. UnixFile 类**

cpp

```cpp
struct UnixFile : MoveOnly<int, -1, Style::Implicit, 
                    [](int fd) { if (fd != -1) close(fd); }> {
    UnixFile(const std::filesystem::path& path)
        : MoveOnly(open(path.c_str(), O_RDONLY)) {
        if (*this == -1)
            throw std::runtime_error("Failed to open file.");
    }
};
```

- **UnixFile**：
  - 继承 MoveOnly，管理文件描述符。
  - T = int，empty_value = -1，style = Implicit，cleanup = close。
- **构造**：
  - 调用 open 打开文件，传入 MoveOnly 构造函数。
  - 检查文件描述符是否为 -1，失败则抛异常。
- **特性**：
  - 隐式转换为 int，析构时关闭文件。

------

**4. MemoryMappedFile 类**

cpp

```cpp
struct MemoryMappedFile {
    MemoryMappedFile(const std::filesystem::path& path)
        : file_(path) {
        struct stat sb;
        if (fstat(file_, &sb) == 1)
            throw std::system_error(errno, std::system_category(), "...");
        sz_.set(sb.st_size);
        ingest{begin_} = static_cast<char *>(mmap(nullptr, sz_.get(), ...));
        if (begin_.get() == MAP_FAILED)
            throw std::system_error(errno, std::system_category(), "...");
    }
    MemoryMappedFile(MemoryMappedFile&&) = default;
    MemoryMappedFile& operator=(MemoryMappedFile&&) = default;
    ~MemoryMappedFile() {
        if (begin_.get() != nullptr)
            munmap(begin_.get(), sz_.get());
    }
    std::span<const char> file_content() {
        assert(begin_.get() != nullptr);
        return {begin_.get(), sz_.get()};
    }
private:
    UnixFile file_;
    MoveOnly<char *> begin_;
    MoveOnly<size_t> sz_;
};
```

- **成员**：
  - file_：UnixFile，管理文件描述符。
  - begin_：MoveOnly<char *>，管理内存映射地址。
  - sz_：MoveOnly<size_t>，管理文件大小。
- **构造**：
  - 打开文件（file_）。
  - 获取文件大小（fstat），存入 sz_。
  - 映射文件（mmap），结果存入 begin_。
  - 失败时抛异常。
- **移动**：
  - 默认移动构造和赋值，确保资源转移。
- **析构**：
  - 释放内存映射（munmap）。
- **file_content()**：
  - 返回文件内容的 std::span，提供只读视图。

------

**关键技术点**

1. **MoveOnly**：
   - RAII 封装资源，防止拷贝，确保移动语义。
2. **Style**：
   - Implicit：允许隐式转换（如 UnixFile）。
   - Explicit：强制显式访问（如 begin_ 和 sz_）。
3. **ingest**：
   - 提供赋值语法糖，增强可读性。
4. **系统调用**：
   - open、fstat、mmap、close、munmap 用于文件和内存管理。
5. **std::span**：
   - 提供轻量视图访问映射内容。

------

**可能的改进或注意事项**

1. **错误处理**：
   - fstat 检查应为 == -1，当前代码错误。
2. **资源检查**：
   - 可添加更多状态验证。
3. **接口扩展**：
   - 可添加读写支持。

------

**总结**

- **MoveOnly**：通用仅移动类型，支持自定义清理。
- **UnixFile**：管理文件描述符。
- **MemoryMappedFile**：组合管理文件和内存映射。
- **用途**：安全的资源管理，避免泄漏。

如果你有具体问题（例如如何扩展功能），欢迎提问！