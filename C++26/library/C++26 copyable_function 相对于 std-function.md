

```cpp
std::copyable_function<void()> cf{F{}};
```

注意模板参数是：

```cpp
void()
```

不是：

```cpp
void() &&
```

根据标准，`copyable_function<R(Args...)>` 的 `operator()` 的 cv/ref 限定符完全来自模板参数。对于 `void()`，调用等价于：

```cpp
invoke(f, ...)
```

其中内部保存的 callable `f` 被当作左值使用。 ([en.cppreference.com](https://en.cppreference.com/w/cpp/utility/functional/copyable_function/operator().html?utm_source=chatgpt.com))

标准明确规定：

- `copyable_function<void()>` → 内部调用 `f`
- `copyable_function<void() &>` → 内部调用 `f`
- `copyable_function<void() &&>` → 内部调用 `std::move(f)` ([en.cppreference.com](https://en.cppreference.com/w/cpp/utility/functional/copyable_function/operator().html?utm_source=chatgpt.com))

因此：

```cpp
std::copyable_function<void()> cf{F{}};

cf();            // F::operator()() &
std::move(cf)(); // 仍然 F::operator()() &
```

完全符合标准。

------

很多人容易误解：

```cpp
std::move(cf)();
```

中的 `cf` 已经是 xvalue 了。

确实如此。

但是这只影响：

```cpp
copyable_function::operator()
```

这个成员函数本身的重载选择。

而你的类型是：

```cpp
copyable_function<void()>
```

它只有：

```cpp
R operator()(Args...)
```

没有 `&&` 限定版本。

进入 `operator()` 以后，标准要求内部 target 仍然按左值调用。 ([en.cppreference.com](https://en.cppreference.com/w/cpp/utility/functional/copyable_function/operator().html?utm_source=chatgpt.com))

------

验证方法：

```cpp
#include <functional>
#include <cstdio>

struct F {
    void operator()() &  { puts("lvalue"); }
    void operator()() && { puts("rvalue"); }
};

int main()
{
    std::copyable_function<void() &&> cf{F{}};

    // cf();        // 编译失败

    std::move(cf)();
}
```

这里输出：

```text
rvalue
```

因为 `copyable_function<void() &&>` 的标准语义就是：

```cpp
std::move(f)()
```

而不是：

```cpp
f()
```

([en.cppreference.com](https://en.cppreference.com/w/cpp/utility/functional/copyable_function/operator().html?utm_source=chatgpt.com))

------

你可以再做一个实验：

```cpp
#include <functional>
#include <cstdio>

struct F {
    void operator()() &  { puts("&"); }
    void operator()() && { puts("&&"); }
};

int main()
{
    std::copyable_function<void()>   a{F{}};
    std::copyable_function<void()&>  b{F{}};
    std::copyable_function<void()&&> c{F{}};

    a();            // &
    std::move(a)(); // &

    b();            // &
    // std::move(b)();   // 编译错误

    std::move(c)(); // &&
}
```

这段代码最能体现 `copyable_function` 相对于 `std::function` 的核心价值：

它把 **cv/ref qualifier 纳入了类型系统**。

而 `std::function<void()>` 永远表现得像：

```cpp
std::copyable_function<void() const&>
```

内部 target 总是以左值方式调用，因此无法传播 `&&` 限定。([en.cppreference.com](https://en.cppreference.com/cpp/utility/functional/copyable_function?utm_source=chatgpt.com))

所以 GCC 16.1 输出四次 `lvalue` 并不是 bug，而是因为你声明的是：

```cpp
std::copyable_function<void()>
```

而不是：

```cpp
std::copyable_function<void() &&>
```