# [15 Different Ways to Filter Containers in Modern C++](https://www.cppstories.com/2021/filter-cpp-containers/)

Do you know how many ways we can implement a filter function in C++?

While the problem is relatively easy to understand - take a container, copy elements that match a predicate, and return a new container - it’s good to exercise with the C++ Standard Library and check a few ideas. We can also apply some Modern C++ techniques, including C++23.

Let’s start!

> The article was written in 2021 and recently updated in late 2025 to include additional techniques from C++23.
>
> Additionally, the text is also republished on the **ACCU website**:
>
> [ACCU Overload, 33: 15 Different Ways to Filter Containers in Modern C++](https://accu.org/journals/overload/33/190/filipek/).

## The Problem Statement [ ](https://www.cppstories.com/2021/filter-cpp-containers/#the-problem-statement) 

To be precise, by a *filter* I mean a function with the following interface:

```
auto Filter(const Container& cont, UnaryPredicate p) {}
```

It takes a container and a predicate, and creates an output container with elements (copies) that satisfy the predicate.

We can use it like the following:

```cpp
const std::vector<std::string> vec{ "Hello", "**txt", "World", "error", "warning", "C++", "****" };

auto filtered = Filter(vec, [](auto& elem) { return !elem.starts_with('*'); });
// filtered should have "Hello", "World", "error", "warning", "C++"
```

Writing such a function can be a good exercise with various options and algorithms in the Standard Library. What’s more, our function hides internal mechanisms like iterators, so it’s more like a range-based version.

Let’s start with the first option:

## Good old Raw Loops [ ](https://www.cppstories.com/2021/filter-cpp-containers/#good-old-raw-loops) 

While it’s good to avoid raw loops, they might help us to fully understand the problem. For our filtering problem, we can write the following code:

```cpp
// filter v1
template <typename T, typename Pred>
auto FilterRaw(const std::vector<T>& vec, Pred p) {
    std::vector<T> out;
    for (auto&& elem : vec)
        if (p(elem))
            out.push_back(elem);
    return out;
}
```

Simple yet very effective.

Please note some nice features of this straightforward implementation.

- The code uses `auto` return type deduction, so there’s no need to write the explicit type (although it could be just `std::vector<T>`).
- It returns the output vector by value, but the compiler will leverage the copy elision (named return value optimization - NRVO), or move semantics at worse.

Since we’re at raw loops, we can take a moment and appreciate range-based for loops that we get with C++11. Without this functionality, our code would look much worse:

```cpp
// filter v1 - old way
template <typename T, typename Pred>
std::vector<T> FilterRawOld(const std::vector<T>& vec, Pred p) {
  std::vector<T> out;
  for (typename std::vector<T>::const_iterator it = begin(vec); it != end(vec); ++it)
    if (p(*it))
      out.push_back(*it);
  return out;
}
```

And now let’s move to something better and see some of the existing `std::` algorithms that might help us with the implementation.

## Filter by `std::copy_if` [ ](https://www.cppstories.com/2021/filter-cpp-containers/#filter-by-stdcopy_if) 

`std::copy_if` is probably the most natural choice. We can leverage `back_inserter` to push matched elements into the output vector.

```cpp
// filter v2
template <typename T, typename Pred>
auto FilterCopyIf(const std::vector<T>& vec, Pred p) {
    std::vector<T> out;
    std::copy_if(begin(vec), end(vec), std::back_inserter(out), p);
    return out;
}
```

## `std::remove_copy_if` [ ](https://www.cppstories.com/2021/filter-cpp-containers/#stdremove_copy_if) 

We can also do the reverse:

```cpp
// filter v3
template <typename T, typename Pred>
auto FilterRemoveCopyIf(const std::vector<T>& vec, Pred p) {
    std::vector<T> out;
    std::remove_copy_if(begin(vec), end(vec), 
                        std::back_inserter(out), std::not_fn(p));
    return out;
}
```

Depending on the requirements, we can also use `remove_copy_if`, which copies elements that do not satisfy the predicate. For our implementation, I had to add `std::not_fn` to reverse the predicate.

One remark: `std::not_fn` has been available since C++17.

## The Famous Remove Erase Idiom [ ](https://www.cppstories.com/2021/filter-cpp-containers/#the-famous-remove-erase-idiom) 

One thing to remember: `remove_if` doesn’t remove elements; it only moves them to the end of the container. So we need to use `erase` to do the final work:

```cpp
// filter v4
template <typename T, typename Pred>
auto FilterRemoveErase(const std::vector<T>& vec, Pred p) {
    auto out = vec;
    out.erase(std::remove_if(begin(out), end(out), std::not_fn(p)), end(out));
    return out;
}
```

Here’s a minor inconvenience. Because we don’t want to modify the input container, we had to copy it first. This might cause some extra processing and is less efficient than using `back_inserter`.

## Adding Some C++20 [ ](https://www.cppstories.com/2021/filter-cpp-containers/#adding-some-c20) 

After seeing a few examples that can be implmented in C++11, we can leverage a convenient feature from C++20: `erase_if`:

```cpp
// filter v5
template <typename T, typename Pred>
auto FilterEraseIf(const std::vector<T>& vec, Pred p) {
    auto out = vec;
    std::erase_if(out, std::not_fn(p));
    return out;
}
```

This function is superior to the remove/erase iodiom, as you can just use a single function.

One minor thing, this approach copies all elements first. So it might be slower than the approach with `copy_if`.

## Adding Some C++20 Ranges [ ](https://www.cppstories.com/2021/filter-cpp-containers/#adding-some-c20-ranges) 

C++20 also brought us powerful ranges and range algorithms, and we can use them as follows:

```cpp
// filter v6
template <typename T, typename Pred>
auto FilterRangesCopyIf(const std::vector<T>& vec, Pred p) {
    std::vector<T> out;
    std::ranges::copy_if(vec, std::back_inserter(out), p);
    return out;
}
```

The code is super simple, and we might even say that our `Filter` function has no point here, since the Ranges interface is so easy to use in code directly.

## Making it More Generic [ ](https://www.cppstories.com/2021/filter-cpp-containers/#making-it-more-generic) 

So far, I showed you code that operates on `std::vector`. But how about other containers?

Let’s try and make our `Filter` function more generic. This is easy with `std::erase_if`, which has overloads for many standard containers:

```cpp
// filter v7
template <typename TCont, typename Pred>
auto FilterEraseIfGen(const TCont& cont, Pred p) {
    auto out = cont;
    std::erase_if(out, std::not_fn(p));
    return out;
}
```

And another version for ranges.

```cpp
// filter v8
template <typename TCont, typename Pred>
auto FilterRangesCopyIfGen(const TCont& vec, Pred p) {
    TCont out;
    std::ranges::copy_if(vec, std::back_inserter(out), p);
    return out;
}
```

Right now, it can work with other containers, not only with `std::vector`:

```cpp
std::set<std::string> mySet{ 
    "Hello", "**txt", "World", "error", "warning", "C++", "****" 
};
auto filtered = FilterEraseIfGen(mySet, [](auto& elem) { 
    return !elem.starts_with('*'); 
});
```

On the other hand, if you prefer not to copy all elements upfront, we might need more work.

### Generic “copy_if” Approach [ ](https://www.cppstories.com/2021/filter-cpp-containers/#generic-copy_if-approach) 

The main problem is that we cannot use `back_inserter` on associative containers, or on containers that don’t support the `push_back()` member function. In that case, we can fall back to the `std::inserter` adapter.

That’s why a possible solution is to detect if a given container supports `push_back` :

```cpp
// filter v9
template <typename T, typename = void>
struct has_push_back : std::false_type {};

template <typename T>
struct has_push_back<T,
  std::void_t<
    decltype(std::declval<T>().push_back(std::declval<typename T::value_type>()))
    >
  > : std::true_type {};

template <typename TCont, typename Pred>
auto FilterCopyIfGen(const TCont& cont, Pred p) {
    TCont out;
    if constexpr(has_push_back<TCont>::value)
        std::copy_if(begin(cont), end(cont), std::back_inserter(out), p);
    else
        std::copy_if(begin(cont), end(cont), std::inserter(out, out.begin()), p);

    return out;
}
```

Above, I used a technique available up to C++17 with `void_t` and SFINAE; read more here: [How To Detect Function Overloads in C++17, std::from_chars Example - C++ Stories](https://www.cppstories.com/2019/07/detect-overload-from-chars/).

But since C++20, we can leverage concepts and make the code much more straightforward.:

```cpp
template <typename T> 
concept has_push_back = requires(T container, typename T::value_type v) { 
    container.push_back(v);
};
```

And see more in [Simplify Code with if constexpr and Concepts in C++17/C++20 - C++ Stories](https://www.cppstories.com/2018/03/ifconstexpr/).

## More C++20 Concepts [ ](https://www.cppstories.com/2021/filter-cpp-containers/#more-c20-concepts) 

We can add more concepts and restrict other template parameters.

For example, if I write:

```cpp
auto filtered = FilterCopyIf(vec, [](auto& elem, int a) { 
    return !elem.starts_with('*'); 
});
```

In the above code I tried to use two arguments for the unary predicate. In Visual Studio I’m getting the following error message:

```text
C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Tools\MSVC\14.28.29333\include\algorithm(1713,13): error C2672: 'operator __surrogate_func': no matching overloaded function found
1>  C:\Users\Admin\Documents\GitHub\articles\filterElements\filters.cpp(38): message : see reference to function template instantiation '_OutIt std::copy_if<std::_Vector_const_iterator<std::_Vector_val<std::_Simple_types<_Ty>>>,std::back_insert_iterator<std::vector<_Ty,std::allocator<_Ty>>>,Pred>(_InIt,_InIt,_OutIt,_Pr)' being compiled
1>          with
```

Not very helpful… but then after a few lines, we have some clear reason:

```text
error C2780: 'auto main::<lambda_4>::operator ()(_T1 &,int) const': expects 2 arguments - 1 provided
```

We can experiment with concepts and restrict our predicate to be `std::predicate`, an existing concept from the Standard Library. In our case, we need a function that takes one argument and then returns a type convertible to `bool`.

```cpp
// filter v10
template <typename T, std::predicate<const T&> Pred>   // <<
auto FilterCopyIfConcepts(const std::vector<T>& vec, Pred p) {
    std::vector<T> out;
    std::copy_if(begin(vec), end(vec), std::back_inserter(out), p);
    return out;
}
```

And then the problematic code:

```cpp
auto filtered = FilterCopyIfConcepts(vec, [](auto& elem, int a) { 
    return !elem.starts_with('*'); 
});
```

This results in the following message:

```text
1>  filters.cpp(143,19): error C2672: 'FilterCopyIfConcepts': no matching overloaded function found
1>  filters.cpp(143,101): error C7602: 'FilterCopyIfConcepts': the associated constraints are not satisfied
```

It’s better, as we have messages about our top-level function rather than internals, but it would be great to see why and which constraint wasn’t satisfied.

## Making it Parallel? [ ](https://www.cppstories.com/2021/filter-cpp-containers/#making-it-parallel) 

Since C++17, we also have parallel algorithms, so why not add them to our list?

As it appears the parallel `std::copy_if` is not supported in Visual Studio, this problem is a bit more complicated. We’ll leave this topic for now and try to solve it next time.

For completeness, we can write the following naive code:

```cpp
// filter v11
std::mutex mut;
std::for_each(std::execution::par, begin(vec), end(vec),
    [&out, &mut, p](auto&& elem) {
        if (p(elem))
        {
            std::unique_lock lock(mut);
            out.push_back(elem);
        }
    });
```

This is, of course, a naive version, and will make the process serialized. The topic is quite advanced, so please have a look at my other text and experiment (`filter v12`): [Implementing Parallel copy_If in C++ - C++ Stories](https://www.cppstories.com/2021/par-copyif/).

## Direct filter support with `ranges::filter_view`, C++23 [ ](https://www.cppstories.com/2021/filter-cpp-containers/#direct-filter-support-with-rangesfilter_view-c23) 

In C++23, we got `std::ranges::filter_view` and `std::views::filter`. So the code is much simpler now:

```cpp
// filter v13
template <typename T, std::predicate<const T&> Pred>
auto FilterRangesFilter(const std::vector<T>& vec, Pred p) {
    std::vector<T> out;
    for (const auto& elem : vec | std::views::filter(p))
        out.push_back(elem);
    return out;
}
```

Play at [Compiler Explorer](https://godbolt.org/z/h8T4z14Ea)

## Adding `ranges::to`, C++23 [ ](https://www.cppstories.com/2021/filter-cpp-containers/#adding-rangesto-c23) 

What’s more, we can use `ranges::to` to automatically create a container.

```cpp
// filter v14
template <typename T, std::predicate<const T&> Pred>
auto FilterRangesFilterTo(const std::vector<T>& vec, Pred p) {
    return vec | std::views::filter(p) | std::ranges::to<std::vector>();
}
```

Additionally, `ranges::to` works with any container type and determines an appropriate way to populate it. So it works with more than just `std::vector`:

See [here @Compiler Explorer](https://godbolt.org/z/E5ss3vYTz)

Here’s an example:

```cpp
template <typename Cont, std::predicate<const typename C::value_type&> Pred>
auto FilterRangesFilterTo(const Cont& vec, Pred p) {
    return vec | std::views::filter(p) | std::ranges::to<Cont>();
}
```

## C++23: Lazy Filtering with `std::generator` [ ](https://www.cppstories.com/2021/filter-cpp-containers/#c23-lazy-filtering-with-stdgenerator) 

All previous versions of `Filter` in this article return a **materialised container** - a `std::vector`, `std::set`, or something similar. That’s often what we want, but sometimes it’s more efficient to:

- avoid allocating a separate container,
- process elements **on the fly** (e.g. streaming input, large ranges),
- or combine filtering with another lazy pipeline.

C++23 adds `std::generator`, a coroutine-based type that models a range. We can use it to express a **lazy filter**:

```cpp
template <typename T, std::predicate<const T&> Pred>
std::generator<const T&> FilterLazy(const std::vector<T>& vec, Pred p) {
    for (const auto& elem : vec) {
        if (p(elem))
            co_yield elem;
    }
}
```

Usage is straightforward:

```cpp
std::vector<std::string> vec{
    "Hello", "**txt", "World", "error", "warning", "C++", "****"
};

auto gen = FilterLazy(vec, [](const auto& s) {
    return !s.starts_with('*');
});

// Elements are produced lazily, on demand:
for (const auto& s : gen) {
    std::cout << s << '\n';
}
```

A few essential properties of this approach:

- Lazy evaluation - elements are filtered only when you iterate the generator.
- No intermediate container - no extra allocation by default.

## Summary [ ](https://www.cppstories.com/2021/filter-cpp-containers/#summary) 

In this article, I’ve shown at least 15 possible ways to filter elements from various containers. We started from code that worked on `std::vector`, and you’ve also seen multiple ways to make it more generic and applicable to other container types. For example, we used `std::erase_if` from C++20, concepts, and even a custom type trait. We also used the “holy grail” of C++23 with `ranges::filter` and `ranges::to`.