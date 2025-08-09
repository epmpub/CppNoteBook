# std::print  std::println

The C++23 *std::print* and *std::println* are the counterparts to *std::format* that format to a stdio FILE descriptor instead of producing a *std::string*.

Both *std::print* and *std::println* by default output to the standard output (*stdout*).

```C++
# define _CRT_SECURE_NO_WARNINGS
#include <print>
#include <format>
#include <ranges>

int main() {
    // Outputs through stdio to stdout
    std::print("{} {}!\n", "Hello", "World");
    // Same as above
    std::println("{} {}!", "Hello", "World");

    // The FILE descriptor can be specified as the first argument
    std::FILE* f = std::fopen("out.txt", "w");
    // Output will be written to "out.txt"

    std::println(f, " output is : {1} {0}!", "Hello", "World");
    // out.txt:  output is : World Hello!
    
    std::fclose(f);

    // Standard error output
    std::println(stderr, "{} {}!", "Hello", "World");

    // Same formatting options as std::format
    std::println("The first five multiples of 17 : {:n}",
        std::views::iota(0, 5) |
        std::views::transform([](int v) { return v * 17; }));
    // prints:
    // The first ten multiples of 17 : 0, 17, 34, 51, 68
}
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/Y9dvx6xr3)