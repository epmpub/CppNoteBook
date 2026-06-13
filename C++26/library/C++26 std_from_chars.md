std::from_chars

```c
#include <charconv>
#include <string_view>
#include <iostream>

void demo_from_chars(std::string_view str) {
    int value = 0; // 用来接收转换后的数字结果

    // 参数：起始指针，结束指针，接收结果的变量
    // C++26 写法：直接在 if 中判断是否成功
    if (auto res = std::from_chars(str.data(), str.data() + str.size(), value)) {
        std::cout << "解析成功！数字为: " << value << '\n';
        std::cout << "读取了 " << (res.ptr - str.data()) << " 个字符。\n";
    } else {
        // 失败处理
        if (res.ec == std::errc::invalid_argument) {
            std::cout << "失败：输入根本不是一个数字！\n";
        } else if (res.ec == std::errc::result_out_of_range) {
            std::cout << "失败：数字太大了，超出了 int 的承受范围（溢出）！\n";
        }
    }
}

int main() {
    demo_from_chars("42");        // 成功，输出 42
    demo_from_chars("9999999999");// 失败，超出 int 范围 (溢出)
    demo_from_chars("0xff");     // 失败，不是数字
}

```

方法 B：配合 `std::from_chars`（如果要转16进制单字符）

如果你非要用 `std::from_chars` 转单字符，需要用它的指针形式：

```c++
char c = 'A';
int value = 0;
// 第4个参数指定 16 进制解析
if (std::from_chars(&c, &c + 1, value, 16)) {
    std::cout << value << '\n'; // 结果是 10
}
```