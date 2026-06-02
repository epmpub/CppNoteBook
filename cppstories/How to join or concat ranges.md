# [How to join or concat ranges, C++26](https://www.cppstories.com/2025/join_concat_ranges/)

Modern C++ continuously improves its range library to provide more expressive, flexible, and efficient ways to manipulate collections. Usually, when you wanted to concatenate or flatten ranges, you’d use raw loops or custom algorithms. With C++’s range adaptors, we now have an elegant and efficient way to process collections lazily without unnecessary allocations.

In this post, we will explore three powerful range adaptors introduced in different C++ standards:

- `std::ranges::concat_view` (C++26)
- `std::ranges::join_view` (C++20)
- `std::ranges::join_with_view` (C++23)

Let’s break down their differences, use cases, and examples.

> Updated in May 2026: added `from_range` construction options.

## `std::ranges::concat_view` (C++26) [ ](https://www.cppstories.com/2025/join_concat_ranges/#stdrangesconcat_view-c26) 

The `concat_view` allows you to concatenate multiple independent ranges into a single sequence. Unlike `join_view`, it does not require a range of ranges-it simply merges the given ranges sequentially while preserving their structure.

In short:

- Takes multiple independent ranges as arguments.
- Supports random access if all underlying ranges support it.
- Allows modification if underlying ranges are writable.
- Lazy evaluation: No additional memory allocations.

See the example:

```cpp
#include <print>
#include <ranges>
#include <vector>
#include <array>

int main() {
    std::vector<std::string> v1{"world", "hi"};
    std::vector<std::string> v2 { "abc", "xyz" };
    std::string arr[]{"one", "two", "three"};
    
    auto v1_rev = v1 | std::views::reverse;
    auto concat = std::views::concat(v1_rev, v2, arr);

    concat[0] = "hello"; // access and write

    for (auto& elem : concat)
        std::print("{} ", elem);
}
```

See at [@Compiler Explorer](https://godbolt.org/z/z1r7WE16j)

The output:

```text
hello world abc xyz one two three 
```

The example below shows how to concatenate three ranges, v1 is reversed and then combined with v2 and arr. Notice that we can also access the value at position 0 and update it.

And a bit more complex example:

```cpp
#include <print>
#include <ranges>
#include <vector>
#include <list>
#include <string>

struct Transaction {
    std::string type;
    double amount;
};

int main() {
    std::vector<Transaction> bank_transactions{
       {"Deposit", 100.0}, 
       {"Withdraw", 50.0}, 
       {"Deposit", 200.0}
    };
    std::list<Transaction> credit_card_transactions{
        {"Charge", 75.0}, {"Payment", 50.0}
    };
    
    auto filtered_bank = bank_transactions 
        | std::views::filter([](const Transaction& t) {
        return t.amount >= 100.0;
    });
    
    auto filtered_credit = credit_card_transactions 
        | std::views::filter([](const Transaction& t) {
        return t.amount > 60.0;
    });
    
    auto all_transactions = std::views::concat(filtered_bank, filtered_credit);
    
    for (const auto& t : all_transactions)
        std::println("{} - {}$", t.type, t.amount);
}
```

Run [@Compiler Explorer](https://godbolt.org/z/a9b8YEqr8)

## `std::ranges::join_view` (C++20) [ ](https://www.cppstories.com/2025/join_concat_ranges/#stdrangesjoin_view-c20) 

The `join_view` is designed for flattening a **single range of ranges** into a single sequence. It removes the structural boundaries between nested ranges.

- Works on a single range of ranges (e.g., `std::vector<std::vector<int>>`).
- Does not support `operator[]` (no random access).
- Eliminates boundaries between sub-ranges.
- Lazy evaluation, avoiding memory copies.

A simple example:

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<std::vector<int>> nested{{1, 2}, {3, 4, 5}, {6, 7}};
    auto joined = std::views::join(nested);
    
    for (int i : joined) 
        std::println(i);
}
```

Run [@Compiler Explorer](https://godbolt.org/z/1E1ohqe6Y)

The output:

```text
1 2 3 4 5 6 7
```

Of course, we can have different nested ranges… and this can be handy for string processing:

```cpp
#include <print>
#include <ranges>
#include <vector>
#include <string>
#include <map>

int main() {
    std::vector<std::string> words { "Hello", "World", "Coding" };

    // regular:
    std::map<char, int> freq;
    for (auto &w : words)
        for (auto &c : w)
            freq[::tolower(c)]++;

    // join:
    std::map<char, int> freq2;
    for (auto &c : words | std::views::join)
        freq2[::tolower(c)]++;

    for (auto& [key, val] : freq2)
        std::println("{} -> {}", key, val);
}
```

Run [@Compiler Explorer](https://godbolt.org/z/vcE3Kj6ch)

As you can see, thanks to `views_join` we can save one nested loop and iterate through a single range of characters.

## `std::ranges::join_with_view` (C++23) [ ](https://www.cppstories.com/2025/join_concat_ranges/#stdrangesjoin_with_view-c23) 

The `join_with_view` works like `join_view`, but it allows inserting a **delimiter** between flattened sub-ranges.

- Works on a single range of ranges.
- Allows specifying a delimiter (single element or a range).
- Does not support random access.
- Useful for formatting strings or separating collections.

See the example below:

```cpp
#include <iostream>
#include <ranges>
#include <vector>
#include <string>
#include <string_view>

std::string to_uppercase(std::string_view word) {
    std::string result(word);
    for (char& c : result) 
        c = std::toupper(static_cast<unsigned char>(c));
    return result;
}

int main() {
    std::vector<std::string_view> words{
        "The", "C++", "ranges", "library"
    };

    auto words_up = words | std::views::transform(to_uppercase);
    auto joined = std::views::join_with(words_up, std::string_view(" "));

    for (auto c : joined) 
        std::cout << c;
}
```

See at [Compiler Explorer](https://godbolt.org/z/njWhb3Gf3)

Here’s the expected output:

```text
THE C++ RANGES LIBRARY
```

## Creating new containers from the view [ ](https://www.cppstories.com/2025/join_concat_ranges/#creating-new-containers-from-the-view) 

I think I overlooked one important aspect of our experiments.

It’s good that we can print flattened or concatenated ranges… but what if you want to create another container from that view?

A simple example could be as follows:

```cpp
std::vector<int> make_flat(const std::vector<std::vector<int>> &nested) {
 // ??
}

auto new_vector = make_flat(vec_vec);
```

We have at least four options here:

- Before C++23, we can push data to the new container in some loop,
- Or pass `begin()` and `end()` iterators,
- For C++23, we can use `ranges::to()`,
- Or, also for C++23, create container directly from a range/view.

See the example here:

```cpp
#include <print>
#include <ranges>
#include <string>
#include <string_view>
#include <vector>

std::vector<int> make_flat_loop(const std::vector<std::vector<int>>& nested) {
    std::vector<int> out;
    for (int value : nested | std::views::join)
        out.push_back(value);
    return out;
}

std::vector<int> make_flat_iters(const std::vector<std::vector<int>>& nested) {
    auto joined = nested | std::views::join;
    return { joined.begin(), joined.end() };
}

std::vector<int> make_flat_to(const std::vector<std::vector<int>>& nested) {
    return nested
        | std::views::join
        | std::ranges::to<std::vector<int>>();
}

std::vector<int> make_flat_from_range(const std::vector<std::vector<int>>& nested) {
    auto joined = nested | std::views::join;
    return std::vector<int>(std::from_range, joined);
}

int main() {
    std::vector<std::vector<int>> nested{{1, 2}, {3, 4, 5}, {6, 7}};
    auto flat1 = make_flat_loop(nested);
    auto flat2 = make_flat_iters(nested);
    auto flat3 = make_flat_to(nested);
    auto flat4 = make_flat_from_range(nested);

    for (auto&& flat : {flat1, flat2, flat3, flat4}) {
        for (int value : flat)
            std::print("{} ", value);
        std::println("");
    }

    std::vector<std::string> words{"C++", "ranges", "are", "powerful"};
    auto sentence = std::string(
        std::from_range,
        std::views::join_with(words, std::string_view(" "))
    );

    std::println("{}", sentence);
}
```

See [@Compiler Explorer](https://godbolt.org/z/MrTGWafn7)

The output:

```text
1 2 3 4 5 6 7
1 2 3 4 5 6 7
1 2 3 4 5 6 7
1 2 3 4 5 6 7
C++ ranges are powerful
```

The C++23 changes came from the following proposal: [P1206R7 - Conversions from ranges to containers](https://wg21.link/P1206).

## Comparing `concat_view`, `join_view`, and `join_with_view` [ ](https://www.cppstories.com/2025/join_concat_ranges/#comparing-concat_view-join_view-and-join_with_view) 

|                   Feature                   |       `concat_view` ✅        | `join_view` ✅ | `join_with_view` ✅ |
| :-----------------------------------------: | :--------------------------: | :-----------: | :----------------: |
|  **Works on multiple independent ranges?**  |              ✅               |       ❌       |         ❌          |
|         **Flattens nested ranges?**         |              ❌               |       ✅       |         ✅          |
| **Supports separators between sub-ranges?** |              ❌               |       ❌       |         ✅          |
|         **Random access support?**          | ✅ (if all inputs support it) |       ❌       |         ❌          |

## Summary [ ](https://www.cppstories.com/2025/join_concat_ranges/#summary) 

C++’s range adaptors provide efficient ways to manipulate collections without unnecessary copying. Here’s a quick summary of when to use each view:

- Use `concat_view` when merging multiple independent ranges.
- Use `join_view` when flattening a range of ranges.
- Use `join_with_view` when flattening a range of ranges but needing a separator between elements.

## References [ ](https://www.cppstories.com/2025/join_concat_ranges/#references) 

Books:

- [C++23 Best Practices](https://amzn.to/3YWRGEL) - by Jason Turner
- [Modern C++ Programming Cookbook](http://amzn.to/2qKTyis) - by Marius Bancila
- [Programming: Principles and Practice Using C++](https://amzn.to/40yGQG4) - by Bjarne Stroustrup

Other:

- [std::ranges::join_view - cppreference.com](https://en.cppreference.com/w/cpp/ranges/join_view)
- [std::ranges::join_with_view - cppreference.com](https://en.cppreference.com/w/cpp/ranges/join_with_view)
- [std::ranges::concat_view - cppreference.com](https://en.cppreference.com/w/cpp/ranges/concat_view)

#### Back to you

- Do you use ranges?
- What are your most useful views ald algorithms on ranges?

Share your comments below

I