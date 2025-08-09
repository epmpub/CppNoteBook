

# C++ >> 和 getline的区别

getline与 is >> s 的区别

- is >> s（>> 运算符）：
  - 读取以空白（空格、制表符、换行符）分隔的单词。
  - 跳过前导空白，停止于下一个空白。
  - 适合读取单个单词。
- std::getline(is, s)：
  - 读取整行，直到换行符（或指定分隔符）。
  - 保留行内的空格和制表符。
  - 适合读取整行文本

例如:

```C++
#include <iostream>
#include <vector>
#include <algorithm>
#include <string>
#include <sstream>

std::vector<std::string> read1(std::istream& is)   // 按照单词输出,以空格等作为分隔符
{
    std::vector<std::string> res;
    for (std::string s; is >> s;)
        res.push_back(s);
    return res;
}

std::vector<std::string> read2(std::istream& is)  // 按行输出,以 \n 回车 等作为分隔符
{
    std::vector<std::string> res;
    for (std::string s; std::getline(is, s);) 
        res.push_back(s);
    return res;
}

int main()
{
    auto v = read2(std::cin);
    for (auto& s : v)
    {
        std::cout << s << std::endl;
    }

    std::cout << std::endl;

    return 0;

}
```

