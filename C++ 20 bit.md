C++20 bit

这段代码展示了 C++20 中引入的 <bit> 头文件提供的一些位操作函数。这些函数用于处理整数的二进制表示，提供了高效的位计算工具。以下是对代码的逐步解释：

```c++
#include <bit>
#include <cstdint>
#include <bitset>
#include <iostream>

int main() {
    uint64_t x = 42u;
    // How many bits to represent a value?
    int bitcnt = std::bit_width(x);
    // bitcnt == 6
    std::cout << "bitcnt == " << bitcnt << "\n";

    // floor: largest power of two <= value
    // ceil: smallest power of two >= value
    uint64_t floor = std::bit_floor(x);
    uint64_t ceil = std::bit_ceil(x);
    // x     ==  0b101010
    // floor ==  0b100000
    // ceil  == 0b1000000
    std::cout << "    x == " << std::bitset<7>(x) << "\n";
    std::cout << "floor == " << std::bitset<7>(floor) << "\n";
    std::cout << " ceil == " << std::bitset<7>(ceil) << "\n";

    // Counting consecutive 0s/1s from left and right
    uint16_t y = 0b1110000000001111;
    int ones_left = std::countl_one(y);
    int ones_right = std::countr_one(y);
    // ones_left == 3, ones_right == 4
    std::cout << "ones_left == " << ones_left << ", ones_right == " << ones_right << "\n";

    uint16_t z = 0b0001111111110000;
    int zeros_left = std::countl_zero(z);
    int zeros_right = std::countr_zero(z);
    // zeros_left == 3, zeros_right == 4
    std::cout << "zeros_left == " << zeros_left << ", zeros_right == " << zeros_right << "\n";

    // Bitwise rotations
    uint16_t i = 0b1000100010001000;
    uint16_t a = std::rotl(i, 1);
    uint16_t b = std::rotl(i, 2);
    uint16_t c = std::rotr(i, 1);
    uint16_t d = std::rotr(i, 2);
    // a == 0b0001000100010001
    // b == 0b0010001000100010
    // c == 0b0100010001000100
    // d == 0b0010001000100010
    std::cout << "y == " << std::bitset<16>(y) << "\n";
    std::cout << "a == " << std::bitset<16>(a) << "\n";
    std::cout << "b == " << std::bitset<16>(b) << "\n";
    std::cout << "c == " << std::bitset<16>(c) << "\n";
    std::cout << "d == " << std::bitset<16>(d) << "\n";

    // Counting number of one-bits anywhere
    constexpr uint64_t FLAG_A = 1 << 0;
    constexpr uint64_t FLAG_B = 1 << 1;
    constexpr uint64_t FLAG_C = 1 << 2;

    bool exclusive = std::has_single_bit(FLAG_C);
    // exclusive == true
    int cnt = std::popcount(FLAG_A | FLAG_C);
    // cnt == 2
    std::cout << "exclusive == " << std::boolalpha << exclusive << "\n";
    std::cout << "cnt == " << cnt << "\n";
}
```



------

1. **std::bit_width**

cpp

```cpp
uint64_t x = 42u;
int bitcnt = std::bit_width(x);
// bitcnt == 6
```

- **功能**：计算表示一个无符号整数值所需的最小位数。
- **计算方式**：返回 ceil(log2(x + 1))，即从 0 开始表示 x 所需的最小位数。
- **例子**：
  - x = 42，二进制为 0b101010。
  - 从最高位 1 开始数，需要 6 位来表示（位编号从 1 开始）。
  - std::bit_width(42) 返回 6。
- **注意**：对于 x = 0，返回值为 0。

------

2. **std::bit_floor 和 std::bit_ceil**

cpp

```cpp
uint64_t floor = std::bit_floor(x);
uint64_t ceil = std::bit_ceil(x);
// x     == 0b101010 (42)
// floor == 0b100000 (32)
// ceil  == 0b1000000 (64)
```

- **std::bit_floor**：
  - **功能**：返回小于或等于 x 的最大 2 的幂。
  - **计算方式**：找到最高位的 1，并将其右边的位清零。
  - **例子**：x = 42 (0b101010)，最高位是第 5 位（从 0 开始计数），2^5 = 32 (0b100000)。
- **std::bit_ceil**：
  - **功能**：返回大于或等于 x 的最小 2 的幂。
  - **计算方式**：如果 x 不是 2 的幂，则向上取整到下一个 2 的幂。
  - **例子**：x = 42 (0b101010)，大于 42 的最小 2 的幂是 2^6 = 64 (0b1000000)。
- **注意**：x = 0 时，bit_floor 返回 0，bit_ceil 返回 1。

------

3. **std::countl_one 和 std::countr_one**

cpp

```cpp
uint16_t y = 0b1110000000001111;
int ones_left = std::countl_one(y);
int ones_right = std::countr_one(y);
// ones_left == 3, ones_right == 4
```

- **std::countl_one**：
  - **功能**：从左侧（高位）开始计数连续的 1 的个数。
  - **例子**：y = 0b1110000000001111，左侧有 3 个连续的 1，返回 3。
- **std::countr_one**：
  - **功能**：从右侧（低位）开始计数连续的 1 的个数。
  - **例子**：y = 0b1110000000001111，右侧有 4 个连续的 1，返回 4。
- **注意**：如果整个数都是 0 或 1，结果将是整个位宽（如 16 位类型返回 16）。

------

4. **std::countl_zero 和 std::countr_zero**

cpp

```cpp
uint16_t z = 0b0001111111110000;
int zeros_left = std::countl_zero(z);
int zeros_right = std::countr_zero(z);
// zeros_left == 3, zeros_right == 4
```

- **std::countl_zero**：
  - **功能**：从左侧（高位）开始计数连续的 0 的个数。
  - **例子**：z = 0b0001111111110000，左侧有 3 个连续的 0，返回 3。
- **std::countr_zero**：
  - **功能**：从右侧（低位）开始计数连续的 0 的个数。
  - **例子**：z = 0b0001111111110000，右侧有 4 个连续的 0，返回 4。
- **注意**：这些函数常用于检测前导零或尾随零的个数，与硬件指令（如 CLZ、CTZ）对应。

------

5. **std::rotl 和 std::rotr**

cpp

```cpp
uint16_t i = 0b1000100010001000;
uint16_t a = std::rotl(i, 1);
// a == 0b0001000100010001
uint16_t b = std::rotl(i, 2);
// b == 0b0010001000100010
uint16_t c = std::rotr(i, 1);
// c == 0b0100010001000100
uint16_t d = std::rotr(i, 2);
// d == 0b0010001000100010
```

- **std::rotl**（rotate left）：
  - **功能**：向左循环移位，移出的高位会回到低位。
  - **例子**：
    - i = 0b1000100010001000，左移 1 位后，最高位 1 移到最低位，结果为 0b0001000100010001。
    - 左移 2 位，结果为 0b0010001000100010。
- **std::rotr**（rotate right）：
  - **功能**：向右循环移位，移出的低位会回到高位。
  - **例子**：
    - i = 0b1000100010001000，右移 1 位后，最低位 0 移到最高位，结果为 0b0100010001000100。
    - 右移 2 位，结果为 0b0010001000100010。
- **注意**：移位次数会根据类型位宽取模（如 16 位类型，移 17 位等价于移 1 位）。

------

6. **std::has_single_bit 和 std::popcount**

cpp

```cpp
constexpr uint64_t FLAG_A = 1 << 0; // 0b001
constexpr uint64_t FLAG_B = 1 << 1; // 0b010
constexpr uint64_t FLAG_C = 1 << 2; // 0b100

bool exclusive = std::has_single_bit(FLAG_C);
// exclusive == true
int cnt = std::popcount(FLAG_A | FLAG_C);
// cnt == 2
```

- **std::has_single_bit**：
  - **功能**：检查一个数是否恰好是 2 的幂（即二进制中只有一个 1）。
  - **例子**：FLAG_C = 0b100，只有一个 1，返回 true。
  - **实现**：等价于 x != 0 && (x & (x - 1)) == 0。
- **std::popcount**：
  - **功能**：计算二进制表示中 1 的总数（population count）。
  - **例子**：FLAG_A | FLAG_C = 0b001 | 0b100 = 0b101，有两个 1，返回 2。
- **注意**：这些函数通常映射到硬件指令（如 POPCNT），非常高效。

------

总结

<bit> 头文件提供了一组强大的工具，用于处理整数的二进制表示：

- **bit_width**：计算表示值所需的位数。
- **bit_floor 和 bit_ceil**：找到上下界的 2 的幂。
- **countl_one 和 countr_one**：计数连续的 1。
- **countl_zero 和 countr_zero**：计数连续的 0。
- **rotl 和 rotr**：循环移位。
- **has_single_bit**：检查是否为 2 的幂。
- **popcount**：统计 1 的个数。

这些函数在底层编程、位操作优化和算法设计中非常有用，且通常由编译器优化为硬件指令，具有高性能。