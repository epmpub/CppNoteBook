### **C++26 中移除 `std::wstring_convert`（以及相关组件）**

##### 位于 std::codecvt namespace

### 1. 什么是 `std::wstring_convert`？

`std::wstring_convert` 是 C++11 引入的**方便转换类**（位于 `<locale>` 头文件），用于在 `std::string`（窄字符/多字节）和 `std::wstring`（宽字符）之间进行转换，通常配合 `std::codecvt_utf8`、`std::codecvt_utf16` 等 facet 使用。

典型用法示例：

```cpp
#include <locale>
#include <codecvt>
#include <string>

std::wstring_convert<std::codecvt_utf8<wchar_t>> conv;
std::string narrow = conv.to_bytes(L"你好，世界");   // wstring → string
std::wstring wide  = conv.from_bytes("Hello");       // string → wstring
```

还有配套的 `std::wbuffer_convert`（用于流缓冲区转换）。

### 2. 历史

- **C++11**：引入。
- **C++17**：**标记为 deprecated**（弃用），同时整个 `<codecvt>` 头文件中的 Unicode 转换 facet 也被弃用。
- **C++26**：**正式移除**（通过提案 **P2872R3**）。

### 3. 为什么移除？

主要原因：

- **设计不完善（underspecified）**：存在多个未解决的 LWG 问题（LWG2478 等），行为在边缘情况（如错误处理、状态管理）下不明确。
- **维护成本高**：改进这些设施需要大量工作，委员会认为投入产出比不高。
- **更好的替代方案已存在**：
  - C++11 起 `std::codecvt` 本身未被弃用，但推荐使用平台特定 API 或第三方库。
  - 现代 C++ 倾向于使用 **UTF-8 everywhere**（`std::string` 存 UTF-8）。
  - C++20+ 的 `char8_t`、`std::u8string` 等提供了更好的 Unicode 支持。
- 这些功能**容易误用**，导致平台依赖和安全问题（尤其是 Windows 的 wchar_t 是 UTF-16）。

### 4. 推荐替代方案

#### **推荐方式（跨平台、现代）**

1. **UTF-8 Everywhere + C++23 `std::print` / `std::format`**（最推荐）：
   ```cpp
   #include <string>
   #include <print>   // C++23
   
   std::string u8str = "你好，世界";  // 直接用 UTF-8
   std::wstring wstr = /* 转换函数 */;
   ```

2. **Windows 平台**（最常用场景）：
   ```cpp
   #include <Windows.h>
   #include <string>
   
   std::wstring to_wstring(const std::string& str) {
       if (str.empty()) return {};
       int size = MultiByteToWideChar(CP_UTF8, 0, str.data(), (int)str.size(), nullptr, 0);
       std::wstring wstr(size, 0);
       MultiByteToWideChar(CP_UTF8, 0, str.data(), (int)str.size(), wstr.data(), size);
       return wstr;
   }
   
   std::string to_string(const std::wstring& wstr) {
       if (wstr.empty()) return {};
       int size = WideCharToMultiByte(CP_UTF8, 0, wstr.data(), (int)wstr.size(), nullptr, 0, nullptr, nullptr);
       std::string str(size, 0);
       WideCharToMultiByte(CP_UTF8, 0, wstr.data(), (int)wstr.size(), str.data(), size, nullptr, nullptr);
       return str;
   }
   ```

3. **第三方库**（推荐）：
   - **ICU**（功能最全）
   - **iconv**（POSIX）
   - **tinyutf8** / **utf8.h**（轻量级单头文件）

4. **C++17+ 手动实现简单转换**（使用 `std::mbstowcs` / `std::wcstombs`，但注意 locale 问题）。

### 5. 影响

- C++26 模式下编译会直接报错（不再是警告）。
- 大部分项目已在 C++17 后开始迁移。
- 这是 C++26 “清理长期弃用且维护不佳的功能”系列的一部分（类似 `strstream`、`shared_ptr` 原子旧 API 等）。

**提案**：**P2872R3** — *Remove `wstring_convert` From C++26*

需要我提供完整的跨平台转换示例、或针对特定平台的代码吗？