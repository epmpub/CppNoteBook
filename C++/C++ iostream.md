# iostream

```C++
#include <fstream>
#include <print>
#include <vector>
#include <cstring>
#include <sstream>

int main() {
    // example.cpp is this file
    std::ifstream in("./example.cpp", in.binary);
    
    // by-character operations

    int c = in.get(); // Read one character
    // static_cast<char>(c) == '#'

    std::println("c == {}", static_cast<char>(c));

    // Seeking, tellg() returns current position
    auto pos = in.tellg();
    // Seek to zero bytes from end (in.beg for begining, in.cur for current position)
    in.seekg(0, in.end);

    // If we try to read past the last character, we get std::char_traits<char>::eof()
    c = in.get();
    // c == std::char_traits<char>::eof()

    std::println("c == {}, std::char_traits<char>::eof() == {}", c, std::char_traits<char>::eof());

    // Clear the eof flag and seek back 
    // to one after the first character
    in.clear();
    // Seek to the provided postion
    in.seekg(pos);
    
    c = in.peek(); // Peek at the next character (do not advance the position)
    // c == 'f'
    // same eof semantics as get()

    std::println("c == {}", static_cast<char>(c));

    in.unget(); // "unread" the last read character
    // implemented by adjusting buffer positions, and if that
    // isn't possible, the stream specific unget will be invoked,
    // which might not be supported

    c = in.get();
    // c == '#'

    std::println("c == {}", static_cast<char>(c));

    in.putback('!'); // "unread" with overwite
    // same semantics as unget(), in this case we can do it, 
    // since the buffer is mutable (the file isn't)

    std::vector<char> buff(64);

    // Read until the delimiter, at most the specified number of characters
    in.getline(buff.data(), 63, '\n');
    // buff.data() == "!include <fstream>"

    std::println("buff.data() == {}", buff.data());

    // Read until the delimiter at most the specified number of characters, 
    // without storing the read characters
    in.ignore(63, '\n');
    // skips over #include <print>

    buff = std::vector<char>(64);

    in.read(buff.data(), 17); // Read at most specified number of characters
    // buff.data() == "#include <vector>"

    std::println("buff.data() == {}", buff.data());

    buff = std::vector<char>(64);

    in.readsome(buff.data(), 1); // Same as read,
    // but will only read already available data in the buffer
    // buff[0] == '\n' (if in buffer)

    std::println("static_cast<int>(buff[0]) == {}", static_cast<int>(buff[0]));

    std::stringstream both;
   
    const char* str = "Is this a good day";
    both.write(str, strlen(str)); // Write the specified number of characters
    // from the pointed memory to the stream

    both.put('\n'); // Write a single character
    
    // Seek to absolute position
    both.seekp(0);
    const char* fix = "This is";
    both.write(fix, strlen(fix));

    // Seek the read position
    both.seekg(0, both.beg);

    buff = std::vector<char>(64);
    both.getline(buff.data(), 63, '\n');
    // buff.data() == "This is a good day"

    std::println("buff.data() == {}", buff.data());
}
```

我来详细解释这段 C++ 代码，它展示了如何使用 std::ifstream（输入文件流）和 std::stringstream（字符串流）进行文件和字符串的读写操作。

------

代码结构

使用的头文件：

- <fstream>：提供文件流类（如 std::ifstream）。
- <print>：提供 std::println（C++23 特性）。
- <vector>：提供 std::vector 用于缓冲区。
- <cstring>：提供 strlen。
- <sstream>：提供 std::stringstream。

代码主要分为两部分：

1. 使用 std::ifstream 读取当前源文件（example.cpp）。
2. 使用 std::stringstream 进行字符串流的读写操作。

------

1. 文件流操作（std::ifstream）

打开文件

cpp

```cpp
std::ifstream in("./example.cpp", in.binary);
```

- 打开当前源文件 example.cpp，以二进制模式读取（避免文本模式下的换行符转换）。

------

逐字符操作

1. **in.get() - 读取单个字符**

   cpp

   ```cpp
   int c = in.get();  // c == '#'（文件开头）
   ```

   - 读取并移除流的第一个字符，返回其整数值。
   - 输出：c == #。

2. **in.tellg() 和 in.seekg() - 获取和设置位置**

   cpp

   ```cpp
   auto pos = in.tellg();  // pos == 1（读了一个字符后）
   in.seekg(0, in.end);    // 移到文件末尾
   ```

   - tellg() 返回当前位置（从文件开头算起的字节偏移）。
   - seekg(offset, direction) 将读取位置移到指定位置（in.end 表示文件末尾）。

3. **读取末尾后的行为**

   cpp

   ```cpp
   c = in.get();  // c == EOF
   ```

   - 读取超出文件末尾，返回 std::char_traits<char>::eof()（通常是 -1）。
   - 输出：c == -1, std::char_traits<char>::eof() == -1。

4. **in.clear() 和 in.seekg(pos) - 重置状态并回退**

   cpp

   ```cpp
   in.clear();       // 清除 EOF 标志
   in.seekg(pos);    // 回到位置 1
   ```

5. **in.peek() - 查看而不移动**

   cpp

   ```cpp
   c = in.peek();  // c == 'i'（#后的字符）
   ```

   - 查看下一个字符但不前进流位置。
   - 输出：c == i。

6. **in.unget() - 回退已读字符**

   cpp

   ```cpp
   in.unget();  // 回退到 '#' 前
   c = in.get();  // c == '#'
   ```

   - 将上一次读取的字符放回流中（依赖缓冲区实现，可能不总是支持）。
   - 输出：c == #。

7. **in.putback() - 回退并覆盖**

   cpp

   ```cpp
   in.putback('!');  // 将 '!' 放回当前位置
   ```

   - 类似 unget，但可以用指定字符覆盖（这里将 '#' 替换为 '!'）。

------

缓冲区操作

1. **in.getline() - 读取一行**

   cpp

   ```cpp
   std::vector<char> buff(64);
   in.getline(buff.data(), 63, '\n');  // 读取到换行符
   ```

   - 从流中读取最多 63 个字符，或直到遇到 '\n'，存储到 buff。
   - 结果：buff.data() == "!include <fstream>"（第一行被修改为以 '!' 开头）。
   - 输出：buff.data() == !include <fstream>。

2. **in.ignore() - 跳过字符**

   cpp

   ```cpp
   in.ignore(63, '\n');  // 跳过下一行
   ```

   - 跳过最多 63 个字符或直到遇到 '\n'，不存储数据。
   - 这里跳过了 #include <print>。

3. **in.read() - 读取固定长度**

   cpp

   ```cpp
   buff = std::vector<char>(64);
   in.read(buff.data(), 17);  // 读取 17 个字符
   ```

   - 读取指定字节数（17），存储到 buff。
   - 结果：buff.data() == "#include <vector>"。
   - 输出：buff.data() == #include <vector>。

4. **in.readsome() - 读取缓冲区可用数据**

   cpp

   ```cpp
   buff = std::vector<char>(64);
   in.readsome(buff.data(), 1);  // 读取 1 个可用字符
   ```

   - 只读取缓冲区中已有的数据（不等待输入），这里读取一个换行符。
   - 输出：static_cast<int>(buff[0]) == 10（'\n' 的 ASCII 值）。

------

2. 字符串流操作（std::stringstream）

定义和写入

cpp

```cpp
std::stringstream both;
const char* str = "Is this a good day";
both.write(str, strlen(str));  // 写入字符串
both.put('\n');                // 添加换行符
```

- 创建一个字符串流 both，写入字符串并追加换行符。

修改流内容

cpp

```cpp
both.seekp(0);  // 将写位置移到开头
const char* fix = "This is";
both.write(fix, strlen(fix));  // 覆盖开头部分
```

- seekp(offset) 设置写位置。
- 覆盖原始字符串的前 7 个字符，结果变为 "This is a good day\n"。

读取流内容

cpp

```cpp
both.seekg(0, both.beg);  // 将读位置移到开头
buff = std::vector<char>(64);
both.getline(buff.data(), 63, '\n');
```

- seekg(offset, direction) 设置读位置。
- 读取一行到 buff，结果为 "This is a good day"。
- 输出：buff.data() == This is a good day。

------

输出总结

```text
c == #
c == -1, std::char_traits<char>::eof() == -1
c == i
c == #
buff.data() == !include <fstream>
buff.data() == #include <vector>
static_cast<int>(buff[0]) == 10
buff.data() == This is a good day
```

------

关键概念

1. **文件流操作**：
   - get、peek、unget、putback 用于逐字符处理。
   - getline、read、ignore、readsome 用于批量读取。
   - seekg、tellg 控制读取位置，clear 重置状态。
2. **字符串流操作**：
   - write、put 用于写入。
   - seekp 控制写位置，seekg 控制读位置。
3. **缓冲区管理**：
   - 使用 std::vector<char> 作为缓冲区，注意长度限制和终止符。

这段代码展示了 C++ 中流操作的灵活性和强大功能，适用于文件读取和字符串处理。如果有具体问题或需要进一步解释，请告诉我！