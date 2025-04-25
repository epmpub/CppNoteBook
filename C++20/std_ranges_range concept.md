# std::ranges::range concept

The `range` concept defines the requirements of a type that allows iteration over its elements by providing an iterator and sentinel that denote the elements of the 

### Notes

A typical `range` class only needs to provide two functions:

1. A member function `begin()` whose return type models [`input_or_output_iterator`](http://127.0.0.1:58597/c__/en.cppreference.com/w/cpp/iterator/input_or_output_iterator.html).
2. A member function `end()` whose return type models [`sentinel_for`](http://127.0.0.1:58597/c__/en.cppreference.com/w/cpp/iterator/sentinel_for.html)`<It>`, where `It` is the return type of `begin()`.

Alternatively, they can be non-member functions, to be found by [argument-dependent lookup](http://127.0.0.1:58597/c__/en.cppreference.com/w/cpp/language/adl.html).

### Example

```c++
#include <ranges>
 
// A minimum range
struct SimpleRange
{
    int* begin();
    int* end();
};
static_assert(std::ranges::range<SimpleRange>);
 
// Not a range: no begin/end
struct NotRange
{
    int t {};
};
static_assert(!std::ranges::range<NotRange>);
 
// Not a range: begin does not return an input_or_output_iterator
struct NotRange2
{
    void* begin();
    int* end();
};
static_assert(!std::ranges::range<NotRange2>);
 
int main() {}
```