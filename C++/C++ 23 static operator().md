C++ 23 static operator()

C++23 addressed a long-standing problem with function objects.

Because the call operator() was not allowed to be a static method, 

function objects couldn't be converted to function pointers, and each call to function object incurred the performance penalty of passing the "this" pointer.

With C++23, the operator() is allowed to be static, removing both issues.

C++ 23 后 ,operator()允许为static; 允许像这样的函数指针:

 int (*p3)(int&&) = newstyle_fn::operator();

------



```c++
#include <utility>

namespace mynamespace {
struct oldstyle_fn {
    // normal method (incurs the penalty of passing this pointer)
    // both in the function and on the call site
    auto operator()(auto&& v) const {
        return std::forward<decltype(v)>(v);
    }
};

struct newstyle_fn {
    // static method
    static auto operator()(auto&& v) {
        return std::forward<decltype(v)>(v);
    }
};

constexpr inline auto oldstyle = oldstyle_fn{};
constexpr inline auto newstyle = newstyle_fn{};
}

int main() {
    // will pass both &oldstyle and 10
    int x = mynamespace::oldstyle(10);
    // will pass only 10
    int y = mynamespace::newstyle(10);
 

    {
    using namespace mynamespace;
    
    // Wouldn't compile
    // cannot convert a member function to a function pointer
    // int (*p1)(int&&) = oldstyle_fn::operator();

    // OK: member function -> member function pointer
    int (oldstyle_fn::*p2)(int&&) const = &oldstyle_fn::operator();
    (oldstyle.*p2)(10); // OK

    // OK: static member function -> function pointer
    int (*p3)(int&&) = newstyle_fn::operator();
    p3(10); // OK
    }
}
```



