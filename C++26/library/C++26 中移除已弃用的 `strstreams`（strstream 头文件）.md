**C++26 中移除已弃用的 `strstreams`（<strstream> 头文件）**

### 1. 什么是 strstreams？

`<strstream>` 头文件提供了以下类（基于 `char*` 缓冲区）：

- `std::strstreambuf`
- `std::istrstream`
- `std::ostrstream`
- `std::strstream`

这些是 **非常古老** 的流类，从 C++98 时代就存在，主要用于在固定或动态的 `char*` 缓冲区上进行输入/输出操作。

### 2. 历史

- **C++98**：已被 **标记为 deprecated**（Annex D）。
- **C++11 ~ C++23**：继续保留（仅弃用），发出警告。
- **C++26**：**正式移除**（通过提案 **P2867R2**）。

这是 C++ 标准库中最古老的弃用功能之一，几乎存在了近 30 年。

### 3. 为什么现在移除？

主要原因是现代替代方案已经足够好且更安全：

| 旧功能                          | 推荐新替代方案                                    | 改进点                |
| ------------------------------- | ------------------------------------------------- | --------------------- |
| `std::ostrstream` / `strstream` | `std::ostringstream` + C++20 `str()` move         | 无拷贝取出字符串      |
| `std::istrstream`               | `std::istringstream` 或 `std::spanstream` (C++23) | 更安全                |
| `std::strstreambuf`             | `std::stringbuf` / `std::spanbuf` (C++23)         | 避免 `char*` 手动管理 |

**核心问题**（导致弃用的根本原因）：
- `ostrstream` 容易产生内存泄漏（需要手动调用 `freeze(false)` 释放内存）。
- 基于裸 `char*` 的接口不安全，容易出现缓冲区溢出、生命周期问题。
- `std::string` + `stringstream` 结合现代移动语义后，已经完全覆盖了其功能，且更安全。

C++23 引入的 `<spanstream>`（`std::spanstream` / `std::spanbuf`）进一步补齐了零拷贝场景的需求。

### 4. 迁移建议

**旧代码（C++26 前）**：
```cpp
#include <strstream>

char buf[100];
std::ostrstream os(buf, sizeof(buf));
os << "Hello " << 42 << std::ends;
std::cout << os.str();   // 需要小心 freeze
```

**推荐新代码**（C++20+）：
```cpp
#include <sstream>
#include <string>
#include <iostream>

std::ostringstream os;
os << "Hello " << 42;
std::string result = os.str();        // C++20 起高效 move
std::cout << result << '\n';
```

**需要零拷贝 / 在已有缓冲区上操作时（C++23+）**：
```cpp
#include <spanstream>
#include <span>

std::array<char, 100> buf{};
std::ospanstream os(std::span{buf});
os << "Hello " << 42 << std::ends;
```

### 5. 影响

- **编译错误**：C++26 模式下 `#include <strstream>` 将直接失败。
- 大部分遗留代码早已迁移，实际影响较小。
- 这是 C++26 “清理老旧弃用功能”的一部分（同时移除的还有 `<codecvt>` 等）。

**提案**：**P2867R2** — *Remove Deprecated strstreams From C++26*

需要我提供更多迁移示例（比如处理 `freeze()` 的老代码）吗？