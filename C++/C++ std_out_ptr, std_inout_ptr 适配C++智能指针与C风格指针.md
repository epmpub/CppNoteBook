# std::out_ptr, std::inout_ptr 适配C++智能指针与C风格指针

// 注意要定义自己的删除器 ,用来回收资源;

// 知道内存是如何分配的和应该如何释放;

![](D:\0.products\myBook\std_out_ptr.png)



```c++
#include <cstdlib>
#include <cerrno>
#include <memory>
#include <stdexcept>

// C API
struct Handle {};
int create_handle(Handle** handle);
int recreate_handle(Handle** handle);
void free_handle(Handle* handle);
int create_handle_ex(int option_a, int option_b, void** handle);

int main() {
    {
    std::unique_ptr<Handle, 
        decltype([](Handle* h) { free_handle(h); })> handle;

    // std::out_ptr for functions that create a handle
    if (int err = create_handle(std::out_ptr(handle)); err != 0)
        throw std::runtime_error("couldn't create handle");

    // std::inout_ptr for functions that first destroy and then create a handle
    if (int err = recreate_handle(std::inout_ptr(handle)); err != 0)
        throw std::runtime_error("couldn't re-create handle");
    
    // void** arguments are also supported
    if (int err = create_handle_ex(1, 2, std::out_ptr(handle)); err != 0)
        throw std::runtime_error("couldn't create handle");
    } // handle freed

    // Also supports std::shared_ptr
    {
    std::shared_ptr<Handle> handle;
    
    if (int err = create_handle(std::out_ptr(handle, free_handle)); err != 0)
        throw std::runtime_error("couldn't create handle");
    } // handle freed
}


int create_handle(Handle** handle) {
    *handle = static_cast<Handle*>(malloc(sizeof(Handle)));
    if (*handle == nullptr) return errno;
    return 0;
}
int recreate_handle(Handle** handle) {
    free(*handle);
    *handle = static_cast<Handle*>(malloc(sizeof(Handle)));
    if (*handle == nullptr) return errno;
    return 0;
}
void free_handle(Handle* handle) {
    free(handle);
}
int create_handle_ex(int, int, void** handle) {
    *handle = malloc(sizeof(Handle));
    if (*handle == nullptr) return errno;
    return 0;
}
```

这段代码展示了 C++20 中 <memory> 头文件引入的 std::out_ptr 和 std::inout_ptr，用于将 C 风格的指针接口与 C++ 的智能指针（std::unique_ptr 和 std::shared_ptr）无缝集成。代码通过示例展示了如何管理 C API 返回的资源，并确保资源在作用域结束时正确释放。以下是逐步解释。

------

代码概览

- 定义了一个模拟的 C API，提供创建和释放 Handle 的函数。
- 使用 std::unique_ptr 和 std::shared_ptr 管理这些资源。
- 通过 std::out_ptr 和 std::inout_ptr，将 C API 的输出参数适配到智能指针。

------

关键组件

1. **头文件**

cpp

```cpp
#include <memory>
#include <stdexcept>
```

- <memory>：提供 std::unique_ptr、std::shared_ptr、std::out_ptr 和 std::inout_ptr。
- <stdexcept>：提供 std::runtime_error 用于异常抛出。
- **C API 定义**

cpp

```cpp
struct Handle {};
int create_handle(Handle** handle);
int recreate_handle(Handle** handle);
void free_handle(Handle* handle);
int create_handle_ex(int option_a, int option_b, void** handle);
```

- **Handle**：一个空结构体，模拟 C API 的资源句柄。
- **create_handle(Handle\**)**：创建句柄并通过双重指针返回，成功返回 0。
- **recreate_handle(Handle\**)**：销毁现有句柄并重新创建，返回 0 表示成功。
- **free_handle(Handle\*)**：释放句柄。
- **create_handle_ex(int, int, void\**)**：带选项创建句柄，使用 void** 返回。
- **std::unique_ptr 示例**

cpp

```cpp
std::unique_ptr<Handle, 
    decltype([](Handle* h) { free_handle(h); })> handle;
```

- **std::unique_ptr**：
  - 使用自定义删除器（lambda），调用 free_handle 释放资源。
  - decltype 推导 lambda 类型。
- **初始状态**：
  - handle 默认初始化为 nullptr。

使用 std::out_ptr

cpp

```cpp
if (int err = create_handle(std::out_ptr(handle)); err != 0)
    throw std::runtime_error("couldn't create handle");
```

- **std::out_ptr(handle)**：
  - 将 handle（unique_ptr）适配为 C API 所需的 Handle**。
  - 内部获取 handle 的原始指针地址，传递给 create_handle。
  - create_handle 填充新句柄后，std::out_ptr 自动将其设置到 handle。
- **错误检查**：
  - err 存储返回值，非 0 表示失败，抛出异常。
- **效果**：
  - 成功时，handle 持有新创建的资源。

使用 std::inout_ptr

cpp

```cpp
if (int err = recreate_handle(std::inout_ptr(handle)); err != 0)
    throw std::runtime_error("couldn't re-create handle");
```

- **std::inout_ptr(handle)**：
  - 用于“销毁并重建”场景。
  - 释放当前 handle 的资源（调用 free_handle），然后将 handle 置为 nullptr。
  - 传递 Handle** 给 recreate_handle，接收新句柄并更新 handle。
- **错误检查**：
  - 同上，失败时抛出异常。
- **效果**：
  - 旧资源被释放，新资源被管理。

使用 void**

cpp

```cpp
if (int err = create_handle_ex(1, 2, std::out_ptr(handle)); err != 0)
    throw std::runtime_error("couldn't create handle");
```

- **std::out_ptr(handle)**：
  - 适配 void** 参数，底层仍转换为 Handle**。
  - create_handle_ex 创建资源，std::out_ptr 更新 handle。
- **效果**：
  - 支持通用指针类型（如 void**），保持智能指针管理。

作用域结束

cpp

```cpp
} // handle freed
```

- **handle 析构**：
  - std::unique_ptr 自动调用删除器（free_handle），释放资源。
- **std::shared_ptr 示例**

cpp

```cpp
std::shared_ptr<Handle> handle;

if (int err = create_handle(std::out_ptr(handle, free_handle));
    err != 0)
    throw std::runtime_error("couldn't create handle");
} // handle freed
```

- **std::shared_ptr**：
  - 默认初始化为 nullptr。
- **std::out_ptr(handle, free_handle)**：
  - 显式指定删除器 free_handle（shared_ptr 需要）。
  - 将 handle 适配为 Handle**，传递给 create_handle。
  - 创建成功后，handle 持有资源，引用计数为 1。
- **析构**：
  - 作用域结束，引用计数减为 0，调用 free_handle 释放资源。

------

为什么这样工作？

1. **std::out_ptr**：
   - 提供临时指针适配器，允许 C API 修改智能指针管理的资源。
   - 自动将新指针设置到智能指针，避免手动管理。
2. **std::inout_ptr**：
   - 在 std::out_ptr 基础上，先释放旧资源，确保重建安全。
3. **智能指针**：
   - unique_ptr 和 shared_ptr 确保资源自动释放，配合 C API 的释放函数。
4. **类型兼容**：
   - 支持 Handle** 和 void**，适配多种 C API 风格。

------

输出

- 无运行时输出，代码展示编译期和异常行为。
- 异常抛出时，显示对应错误消息（如 "couldn't create handle"）。

------

使用场景

- **C/C++ 混合编程**：
  - 将 C API 返回的资源封装为智能指针，避免泄漏。
- **资源管理**：
  - 确保动态分配的句柄（如文件、网络连接）在异常情况下也能释放。
- **现代化改造**：
  - 将遗留 C 代码与现代 C++ 智能指针集成。

------

总结

- std::out_ptr 和 std::inout_ptr 桥接了 C API 和 C++ 智能指针。
- unique_ptr 示例展示了基本创建和重建。
- shared_ptr 示例展示了共享所有权支持。
- 代码利用 C++20 新特性，简化了与 C 风格接口的交互，同时保持资源安全。