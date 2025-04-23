# std::stringstream

std::stringstream 是一个功能强大的类，广泛用于字符串的输入输出操作、格式化数据和类型转换等场景。以下是 std::stringstream 的常见用法，涵盖其多样化的应用场景：

------

1. **字符串构建与格式化**

std::stringstream 可以像 std::cout 一样使用流操作符（<<）将数据拼接成字符串，适合动态构建格式化的字符串。

cpp

```cpp
#include <sstream>
#include <string>
#include <iostream>

int main() {
    std::stringstream ss;
    int age = 30;
    std::string name = "Alice";

    ss << "Name: " << name << ", Age: " << age;
    std::string result = ss.str(); // 获取拼接后的字符串

    std::cout << result << std::endl; // 输出: Name: Alice, Age: 30
}
```

**用途**：

- 构建日志消息、配置文件内容或用户界面文本。
- 格式化复杂的数据结构（如表格、JSON 等）。

------

2. **字符串解析**

std::stringstream 支持输入流操作（>>），可以从字符串中提取数据，类似于 std::cin。

cpp

```cpp
#include <sstream>
#include <string>
#include <iostream>

int main() {
    std::string input = "42 3.14 Hello";
    std::stringstream ss(input); // 用字符串初始化流

    int number;
    double pi;
    std::string word;

    ss >> number >> pi >> word; // 按顺序提取数据

    std::cout << "Number: " << number << ", Pi: " << pi << ", Word: " << word << std::endl;
    // 输出: Number: 42, Pi: 3.14, Word: Hello
}
```

**用途**：

- 解析配置文件或用户输入。
- 处理以空格或特定分隔符分隔的数据。

------

3. **类型转换**

std::stringstream 常用于将字符串转换为其他类型（如整数、浮点数）或将其他类型转换为字符串。

示例 1：字符串转数字

cpp

```cpp
#include <sstream>
#include <string>
#include <iostream>

int main() {
    std::string str = "12345";
    std::stringstream ss(str);

    int number;
    ss >> number;

    std::cout << "Converted number: " << number + 5 << std::endl; // 输出: Converted number: 12350
}
```

示例 2：数字转字符串

cpp

```cpp
#include <sstream>
#include <string>
#include <iostream>

int main() {
    int number = 12345;
    std::stringstream ss;

    ss << number;
    std::string str = ss.str();

    std::cout << "Converted string: " << str << std::endl; // 输出: Converted string: 12345
}
```

**用途**：

- 将用户输入的字符串转换为数字（比 std::stoi 更灵活，可处理多种格式）。
- 将数字或其他类型转换为字符串以用于显示或存储。

------

4. **重用流**

std::stringstream 可以清空并重用，通过 clear() 和 str() 方法重置状态和内容。

cpp

```cpp
#include <sstream>
#include <iostream>

int main() {
    std::stringstream ss;

    // 第一次使用
    ss << "First message";
    std::cout << ss.str() << std::endl; // 输出: First message

    // 清空并重用
    ss.str(""); // 清空内容
    ss.clear(); // 清空流状态

    // 第二次使用
    ss << "Second message";
    std::cout << ss.str() << std::endl; // 输出: Second message
}
```

**用途**：

- 在循环中重复使用同一 stringstream 对象，节省内存。
- 动态生成多个字符串。

------

5. **处理复杂数据格式**

std::stringstream 可以结合格式化标志（如 std::hex、std::fixed）处理特定格式的数据。

cpp

```cpp
#include <sstream>
#include <iomanip>
#include <iostream>

int main() {
    std::stringstream ss;

    // 设置浮点数精度
    ss << std::fixed << std::setprecision(2) << 3.14159;
    std::cout << "Formatted float: " << ss.str() << std::endl; // 输出: Formatted float: 3.14

    // 清空并重用
    ss.str("");
    ss.clear();

    // 转换为十六进制
    ss << std::hex << 255;
    std::cout << "Hex: " << ss.str() << std::endl; // 输出: Hex: ff
}
```

**用途**：

- 生成特定格式的输出（如 JSON、XML、CSV）。
- 格式化科学计数法、十六进制或固定小数点数据。

------

6. **与文件流或其他流的统一接口**

std::stringstream 继承自 std::ostream 和 std::istream，可以作为函数参数，兼容其他流类型（如 std::ofstream、std::cout）。

cpp

```cpp
#include <sstream>
#include <fstream>
#include <iostream>

void write_output(std::ostream& os) {
    os << "Unified output!" << std::endl;
}

int main() {
    // 写入字符串流
    std::stringstream ss;
    write_output(ss);
    std::cout << "String stream: " << ss.str();

    // 写入文件
    std::ofstream file("output.txt");
    write_output(file);
    file.close();

    // 写入控制台
    write_output(std::cout);
}
```

**用途**：

- 编写通用函数，处理不同类型的输出目标（如文件、字符串、控制台）。
- 提高代码复用性。

------

7. **处理分隔符和复杂输入**

std::stringstream 可以结合 std::getline 或其他方法处理以特定分隔符分隔的字符串。

cpp

```cpp
#include <sstream>
#include <string>
#include <iostream>

int main() {
    std::string input = "apple,banana,orange";
    std::stringstream ss(input);

    std::string fruit;
    while (std::getline(ss, fruit, ',')) {
        std::cout << "Fruit: " << fruit << std::endl;
    }
    // 输出:
    // Fruit: apple
    // Fruit: banana
    // Fruit: orange
}
```

**用途**：

- 解析 CSV 文件内容。
- 处理以自定义分隔符分隔的数据。

------

8. **临时缓冲区**

std::stringstream 可以作为临时缓冲区，用于在处理数据时存储中间结果。

cpp

```cpp
#include <sstream>
#include <string>
#include <iostream>

int main() {
    std::stringstream ss;
    int sum = 0;

    // 模拟逐行处理数字
    ss << "10 20 30";
    int num;
    while (ss >> num) {
        sum += num;
    }

    std::cout << "Sum: " << sum << std::endl; // 输出: Sum: 60
}
```

**用途**：

- 在内存中处理临时数据，替代文件或数据库操作。
- 适合小规模数据处理或测试。

------

注意事项

1. **性能**：
   - std::stringstream 适合小规模数据处理。对于大规模数据，文件流或直接字符串操作可能更高效。
2. **状态管理**：
   - 使用 clear() 重置流状态（如错误标志），否则后续操作可能失败。
   - str("") 清空内容，但不影响流状态。
3. **线程安全**：
   - std::stringstream 本身不是线程安全的，多线程操作需要同步。
4. **替代方案**：
   - C++20 引入了 std::format 和 <format>，可以替代部分字符串格式化需求。
   - std::to_string 和 std::stoi 等函数适合简单的类型转换。

------

总结

std::stringstream 是一个多功能的工具，适用于：

- 字符串构建与格式化
- 字符串解析
- 类型转换
- 统一流接口
- 复杂数据处理
- 临时缓冲区

它在需要以流方式处理字符串或需要与文件/控制台流兼容的场景中尤其有用。通过结合格式化标志和流操作，std::stringstream 提供了灵活且强大的字符串处理能力。