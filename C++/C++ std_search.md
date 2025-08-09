# std::search

The std::search algorithm returns the first instance of a sub-sequence.

The C++17 variant supports both parallel execution and custom searchers. Custom searchers offer better average complexity (up to linear).

```C++
#include <string>
#include <algorithm>
#include <iostream>
#include <iomanip>
#include <functional>

int main() {
    std::string text = "the quick brown fox jumps over the lazy dog";
    std::string needle = "fox";

    // Find the first instance of a sub-sequence in text
    auto it = std::search(text.begin(), text.end(),
        needle.begin(), needle.end());
    // it != text.end()
    std::cout << std::boolalpha << (it != text.end()) << "\n";

    std::string_view word(it, it+needle.length());
    // word == "fox"
    std::cout << std::quoted(word) << "\n";

    // C++17 introduced searchers that offer better average complexity
    std::boyer_moore_horspool_searcher searcher(needle.begin(), needle.end());
    // Average linear complexity
    it = std::search(text.begin(), text.end(), searcher);
    // Same behaviour as default search
    // it != text.end()
    word = std::string_view(it, it+needle.length());
    // word == "fox"
    std::cout << std::quoted(word) << "\n";

    // Range version doesn't support searchers and returns a subrange
    auto [begin, end] = std::ranges::search(text, needle);
    word = std::string_view(begin,end);
    // word == "fox"
    std::cout << std::quoted(word) << "\n";
}
```

