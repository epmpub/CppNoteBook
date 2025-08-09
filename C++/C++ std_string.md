

# *std::string*

The *std::string* is a container for storing null-terminated narrow character strings.

std *::string*是一个用于存储以空字符结尾的窄字符串的容器。

std::string provides very similar functionality to *std::vector* while maintaining the null-termination invariant on every operation.

Additionally, *std::string* also provides a couple of convenience methods (find, starts/ends_with, contains, substr).

std::string 提供与*std::vector*非常相似的功能，同时在每个操作上保持空终止不变。

此外，*std::string*还提供了一些便捷方法（find、starts/ends_with、contains、substr）。



```C++
#include <string>
#include <iostream>
#include <iomanip>

int main() {
    std::string str = "Hello World?";
    
    // Same interface as std::vector
    str.pop_back();
    str.push_back('!');
    // str == "Hello World!"

    std::cout << "str == " << std::quoted(str) << "\n";

    // Importantly, these operations maintain null-termination
    char c = str[str.size()]; // Unlike std::vector, this is OK
    // c == char{}, i.e. '\0'

    std::cout << "c == " << int(c) << "\n";
    
    // Convenience methods
    size_t pos = str.find_first_of(" ?");
    // pos == 5

    std::cout << "pos == " << pos << "\n";

    // Get a C-style string
    const char* cstring = str.c_str();
    // cstring == "Hello World!"

    std::cout << "cstring == " << std::quoted(cstring) << "\n";

    // Substring, offset and count
    auto world = str.substr(6, 5);
    // world == "World"

    std::cout << "world == " << std::quoted(world) << "\n";

    // Count can be omitted to take the remainder of the string
    auto WORLD = str.substr(6);
    // WORLD == "World!"

    std::cout << "WORLD == " << std::quoted(WORLD) << "\n";
}
```

