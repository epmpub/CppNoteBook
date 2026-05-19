#### absl::Cord

absl::Cord 是 Abseil 库中一个非常实用的字符串类，专门设计用于高效处理大型字符串（特别是经常进行拼接、切割、前置/后置操作的场景）。

什么是 absl::Cord？与 std::string 的区别

- std::string：连续内存存储，修改（尤其是中间插入/删除）或频繁拼接时可能触发多次内存重分配，性能较差。
- absl::Cord（Cord = Concatenated Object Representation）：内部使用树状结构（rope-like）存储字符串，由多个小块（chunks）组成。
  - 优点：拼接（Append/Prepend）非常高效，几乎是 O(1) 或 O(log N)，不会复制已有数据。
  - 支持共享子串（copy-on-write 语义）。
  - 适合处理大日志、协议缓冲、大文件内容、频繁拼接的场景（如构建 JSON、日志消息等）。
  - 缺点：随机访问（如 operator[]）比 std::string 稍慢；内存占用略高。

什么时候用 Cord？

- 需要频繁 += 或拼接很多小字符串时。
- 字符串总大小可能达到 MB 甚至 GB 级别。
- 需要高效的前置（Prepend）操作（std::string 前置很慢）。

基本使用1. 头文件和构造

cpp

```cpp
#include "absl/strings/cord.h"

absl::Cord cord;                    // 空 Cord
absl::Cord cord2("hello world");    // 从 const char* 构造
absl::Cord cord3(std::string("abc"));
absl::Cord cord4(absl::string_view("view"));
```

2. 最常用的操作：Append 和 Prepend（高效拼接）

cpp

```cpp
absl::Cord cord;

cord.Append("Hello");                    // 后追加字符串
cord.Append(absl::string_view(" "));
cord.Append(std::string("World"));
cord.Append(another_cord);               // 可以追加另一个 Cord

cord.Prepend("Prefix: ");                // 前置（这是 Cord 的强项）

```

3. 获取内容

cpp

```cpp
std::string str = std::string(cord);           // 转为 std::string（会复制内容）
std::string str2;
absl::CopyCordToString(cord, &str2);           // 更高效的转换方式

absl::string_view view = cord;                 // 隐式转为 string_view（推荐）

// 打印
std::cout << cord << std::endl;                // 支持直接输出
```

4. 其他常用操作

cpp

```cpp
size_t len = cord.size();                      // 获取长度
bool empty = cord.empty();

cord.RemovePrefix(5);     // 移除前缀 N 个字符
cord.RemoveSuffix(3);     // 移除后缀 N 个字符

absl::Cord sub = cord.Subcord(10, 20);         // 提取子串（高效，不复制数据）

// 遍历字符（推荐方式）
for (char c : cord.Chars()) {
    // 处理每个字符
}

// 或使用迭代器
for (auto it = cord.char_begin(); it != cord.char_end(); ++it) {
    char c = *it;
}
```

5. 完整示例

cpp

```cpp
#include "absl/strings/cord.h"
#include <iostream>

int main() {
    absl::Cord cord;

    cord.Append("This is ");
    cord.Append("a large ");
    cord.Prepend("Header: ");           // 前置操作

    for (int i = 0; i < 100; ++i) {
        cord.Append(" part" + std::to_string(i));   // 频繁拼接
    }

    std::cout << "Length: " << cord.size() << std::endl;
    std::cout << "Content: " << std::string(cord).substr(0, 100) << "..." << std::endl;

    // 提取子串（高效）
    absl::Cord sub = cord.Subcord(8, 20);
    std::cout << "Sub: " << sub << std::endl;

    return 0;
}
```

高级用法提示

- 与 absl::StrCat / StrAppend 配合：absl::StrAppend(&cord, "text"); 不支持直接用于 Cord，但你可以先用 StrCat 生成 std::string 再 Append，或直接用 Append。
- 性能最佳实践：
  - 大量小字符串拼接 → 用 Cord。
  - 最终需要连续内存时 → 最后再转为 std::string。
  - 避免频繁随机访问单个字符（用 Chars() 迭代器较好）。
- 线程安全：Cord 是线程安全的（const 操作安全，修改时需注意）。