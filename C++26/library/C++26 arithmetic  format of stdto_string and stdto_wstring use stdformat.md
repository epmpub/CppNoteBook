## C++26 arithmetic  format of std::to_string and std::to_wstring use std::format

在 **C++26** 中，通过了提案 [P2587R3（"to_string or not to_string"）](https://www.sandordargo.com/blog/2025/07/09/cpp26-format-part-1)，标准库对 `std::to_string` 和 `std::to_wstring` 的底层实现进行了重大重构：**所有算术类型（整数、浮点数）的重载版本，现在全部基于 `std::format`（本质上最终调用 `std::to_chars`）来实现，彻底废弃了原先基于 C 语言 `sprintf` 的旧机制**。

------

🔍 重构带来的 3 大核心改进

1. 浮点数不再产生误导性的死板格式

在 C++26 之前，`std::to_string` 转换浮点数时硬编码使用 `sprintf` 的 `%f` 格式，默认保留 **6 位小数**。这导致极大或极小的数字会丢失精度变成 `0.000000`。
C++26 切换到 `std::format` 后，采用**最短且无损的默认表现形式**（等同于 `{}` 格式化）：

| 输入数据 (double) | C++26 之前（基于 `sprintf`）       | **C++26（基于 `std::format`）** | 说明                       |
| ----------------- | ---------------------------------- | ------------------------------- | -------------------------- |
| `0.42`            | `"0.420000"` ❌                     | **`"0.42"`** 🚀                  | 自动去除了末尾无意义的 0   |
| `-1e-7`           | `"-0.000000"` ❌                    | **`"-1e-07"`** 🚀                | 极小数字不再丢失精度归零   |
| `1e40`            | `"1000000000000000030378...000"` ❌ | **`"1e+40"`** 🚀                 | 极大数字自动使用科学计数法 |

2. 彻底摆脱全局本地化（Locale）的影响

- **过去**：`std::to_string` 依赖于系统的 C Locale（全局区域设置）。如果在欧洲某些国家运行（例如德国），由于其小数点習慣用逗号，`std::to_string(0.5)` 会变成 `"0,5"`，这极易导致网络协议或 JSON 解析崩溃。
- **C++26**：由于底层改用 `std::format` 和 `std::to_chars`，其默认行为是 **Locale-independent（无视本地化）** 的。无论在什么操作系统或国家地区运行，小数点永远是 `.`。 ![img](https://encrypted-tbn1.gstatic.com/faviconV2?url=https://en.cppreference.com&client=AIM&size=128&type=FAVICON&fallback_opts=TYPE,SIZE,URL)***\*cppreference.com\** \**+2\****
- 性能的大幅提升（零堆内存分配）

- **过去**：旧实现会先通过 `sprintf` 写入内部临时缓冲区，再拷贝构造 `std::string`。
- **C++26**：现在编译器可以直接计算出转换后所需的字符串大小，直接在 `std::string` 内部的缓冲区（利用 SSO 优化或提前 `reserve`）通过 `std::to_chars` 写入，**避免了多次内存拷贝和不必要的开销**。 ![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAwAAAAMCAYAAABWdVznAAAA60lEQVR4AZSQO2sCQRRGz67IJpIEXdIEA3l0IYG0WvhDtLBRrAVtBLESbLWx9i/Y2NgJYm/lA7ERRHEVn4ivdS2EHV0LLxwYZu6Zme/KehD9HmTuLEshXH8lYmB115WgbWTyfxqtpZ1U08XmIAmeIKz2EpmOk3Tbid+9pG1Ip7XZEARF1ol+zSiPH8h1X/h52lKZKOZ+BCHbdeGpvhH7nJH71SgOHBT+R7cFn7oi/j2lt7aRaKgE3As+Hne3haQRsjR0cGK+k7BJOmJkxC/NjYmcUZU9z/YDlyVkqHn7mAm9Ly77xReuTi02jgAAAP//8beTQgAAAAZJREFUAwDW8GRpNWaJpAAAAABJRU5ErkJggg==)***\*Stack Overflow\** \**+1\****

------

💻 等价行为对照表

在 C++26 的规范中，`std::to_string` 和 `std::to_wstring` 的逻辑直接等价于调用不带特定格式标签的 `std::format`：

```c
// 在 C++26 中，两边的行为完全等价：

std::string s1 = std::to_string(42);       // 等价于 -> std::format("{}", 42);
std::string s2 = std::to_string(0.123);    // 等价于 -> std::format("{}", 0.123);

std::wstring w1 = std::to_wstring(42);     // 等价于 -> std::format(L"{}", 42);
std::wstring w2 = std::to_wstring(0.123);  // 等价于 -> std::format(L"{}", 0.123);
```

请谨慎使用此类代码。

⚙️ 特征测试宏

标准库为了这一改变提供了一个专属的功能测试宏：

```c
#include <string>

#if defined(__cpp_lib_to_string) && __cpp_lib_to_string >= 202306L
    // 当前标准库已支持 C++26 基于 std::format 版本的 to_string
#endif
```

如果你希望在更老的编译器（如 C++20/C++23 模式）中提前获得这种干净、高效的浮点数转换，可以主动放弃 `to_string`，在代码中直接手动编写 **`std::format("{}", value)`** 予以替代。

