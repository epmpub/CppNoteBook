# std::string to integer and floating point

Functions for converting std::string to integer and floating point types:

 std::stoi, std::stol, std::stoll, std::stoul, std::stoul, std::stof, std::stod, std::stold.

C++11 added functions for converting std::string into integer and floating point types.

Signed integers: *std::stoi*, *std::stol*, *std::stoll*.

Unsigned integers: *std::stoul*, *std::stoull*.

Floating point types: *std::stof*, *std::stod*, *std::stold*.

For *std::string_view* or string literals, prefer the *std::from_chars* function (to an excessive *std::string* temporary).

```c++
#include <string>

// Normal use:
std::string negative = "-200";
auto v1 = stoi(negative);
// decltype(v1) == int, v1 == -200

auto v2 = stol(negative);
// decltype(v2) == long, v2 == -200

auto v3 = stoll(negative);
// decltype(v3) == long long, v3 == -200

std::string positive = "42";
auto v4 = std::stoul(positive);
// decltype(v4) == unsigned long, v4 == 42

auto v5 = std::stoull(positive);
// decltype(v5) == unsigned long long, v5 == 42

std::string pi = "3.14";
auto v6 = std::stof(pi);
// decltype(v6) == float, v6 ~= 3.14

auto v7 = std::stod(pi);
// decltype(v7) == double, v7 ~= 3.14

auto v8 = std::stold(pi);
// decltype(v8) == long double, v8 ~= 3.14

// Optional, length of parsed text, and base for integer types:
std::string complex = "  169AQ";
size_t offset = 0;
auto v9 = std::stol(complex, &offset, 10);
// v9 == 169, decltype(v9) == long
// std::string_view(complex.begin()+offset, complex.end()) == "AQ"

offset = 0;
auto v10 = std::stol(complex, &offset, 16);
// v10 == 5786 (0x169A), decltype(v10) == long
// std::string_view(complex.begin()+offset, complex.end()) == "Q"

// Note that these functions require std::string, not std::string_view.
// For that, prefer std::from_chars.
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/serncE6qK)