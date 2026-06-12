std::to_chars()

```c++

#include <charconv>
#include <array>
#include <iostream>

void demo_to_chars(float value) {
    std::array<char, 2> buffer;

    //  C++26 现代写法：直接在 if 条件中隐式转换成 bool
    if (auto res = std::to_chars(buffer.data(), buffer.data() + buffer.size(), value)) {
        std::cout << "成功，转换后长度: " << (res.ptr - buffer.data()) << '\n';
    } else {
        std::cout << "失败：缓冲区太小\n"; // 对应 std::errc::value_too_large
    }
    for(const auto &i:buffer){
        std::cout << i << "\n";
    }
}

int main(){
    demo_to_chars(3.14);
}
```

