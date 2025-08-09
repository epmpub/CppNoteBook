# std::endian

The *std::endian* from C++20 is an *enum* in the *<bit>* header that provides information about the native endianness of the architecture the program was compiled for.

```c++
#include <bit>
#include <iostream>

int main() {
    if constexpr (std::endian::native == std::endian::little) {
        std::cout << "This system is little-endian.\n";
    } else if constexpr (std::endian::native == std::endian::big) {
        std::cout << "This system is big-endian.\n";
    }
}
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/bozxszz1e)