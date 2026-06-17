这是 C++26 Freestanding 的一个小改进，来源于 **P2013R5 Freestanding Language: Optional `::operator new`**。

先说背景。

### 什么是 Freestanding？

C++ 标准定义了两种实现环境：

1. Hosted（完整环境）

例如：

- Linux
- Windows
- macOS

拥有：

```cpp
main()
文件系统
线程库
异常
堆内存
```

等完整运行时。

1. Freestanding（独立环境）

例如：

- MCU
- RTOS
- Kernel
- Bootloader
- Embedded

这类环境可能没有：

```cpp
main
文件系统
动态内存
异常
```

标准对其要求更少。

------

### C++23之前的问题

标准要求：

```cpp
new T
```

最终调用：

```cpp
::operator new(std::size_t)
```

例如：

```cpp
auto p = new int(42);
```

等价于：

```cpp
void* mem = ::operator new(sizeof(int));
new(mem) int(42);
```

因此语言层面的：

```cpp
new-expression
```

依赖于：

```cpp
::operator new
```

存在。

------

但很多 Freestanding 环境根本没有堆：

```cpp
malloc
free
operator new
operator delete
```

都不存在。

例如：

```cpp
STM32
裸机ARM
内核代码
```

常见配置：

```cpp
-fno-exceptions
-fno-rtti
-nostdlib
```

根本不希望提供：

```cpp
::operator new
```

------

### 标准的矛盾

一方面：

```cpp
new-expression
```

属于核心语言（Core Language）。

另一方面：

```cpp
::operator new
```

属于运行时支持。

结果导致：

> 即使程序从未使用 `new`，Freestanding 实现理论上仍然要提供全局 `::operator new`。

这是一个不合理要求。

------

### P2013R5 的修改

C++26 改为：

> Freestanding 实现可以不提供全局 `::operator new` 和 `::operator delete`。

即：

```cpp
::operator new
::operator new[]
::operator delete
::operator delete[]
```

变成：

```text
optional
```

而非：

```text
required
```

------

### 对用户代码的影响

以前：

```cpp
int main()
{
    int x = 42;
}
```

理论上 Freestanding 实现仍需提供：

```cpp
::operator new
```

即使根本不用。

C++26 后：

```cpp
int main()
{
    int x = 42;
}
```

实现完全可以不提供：

```cpp
::operator new
```

因为程序没有使用它。

------

### 如果用了 new 呢？

例如：

```cpp
auto p = new int;
```

编译器会尝试生成：

```cpp
::operator new(sizeof(int))
```

如果平台没有提供：

```cpp
::operator new
```

那么：

```text
链接失败
```

或者：

```text
实现定义行为
```

这是允许的。

因为 Freestanding 环境已经声明：

```text
我不支持动态分配
```

------

### 嵌入式场景

以前很多嵌入式项目不得不写：

```cpp
void* operator new(std::size_t)
{
    while(true);
}
```

或者：

```cpp
void* operator new(std::size_t)
{
    return nullptr;
}
```

仅仅为了满足标准要求。

C++26 后不需要了。

如果整个工程禁止：

```cpp
new
delete
```

那么：

```cpp
::operator new
::operator delete
```

都可以不存在。

------

### Placement New 不受影响

注意：

```cpp
new(buffer) T(...)
```

依赖的是：

```cpp
void* operator new(std::size_t, void*)
```

即 Placement New。

它位于：

```cpp
<new>
```

中的：

```cpp
inline constexpr
```

版本。

例如：

```cpp
alignas(T) char storage[sizeof(T)];

T* p = new(storage) T();
```

这种用法在 Freestanding 中仍然非常重要。

因为嵌入式程序经常采用：

```cpp
静态内存池
arena
固定缓冲区
```

而不是堆。

------

### 为什么叫 Optional `::operator new`

这里的关键是：

```cpp
::operator new
```

而不是：

```cpp
new-expression
```

C++26 并没有删除：

```cpp
new T
```

语言特性。

只是说：

```cpp
::operator new
```

对于 Freestanding：

```text
不再是必须提供的运行时组件
```

------

### 实际意义

对于桌面开发：

```cpp
Windows
Linux
macOS
```

几乎没有影响。

因为 Hosted 实现仍然必须提供：

```cpp
::operator new
::operator delete
```

真正受益的是：

- Embedded C++
- RTOS
- Bare Metal
- Hypervisor
- Kernel
- Bootloader

这些环境终于可以做到：

```cpp
C++语言
+
无堆内存
+
符合标准
```

而不需要为了满足标准要求额外提供假的：

```cpp
::operator new
::operator delete
```

实现。

一句话概括：**C++26 将 Freestanding 环境中的全局 `::operator new/delete` 从“强制要求”降为“可选支持”，使无堆（heapless）的嵌入式和系统级 C++ 实现能够更自然地符合标准。**