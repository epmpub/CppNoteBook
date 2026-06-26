C++26 disallow binding a returned reference to a temporary 

在 C++26 中，通过核心提案 **[P2748R5](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2748r5.html)**（由 Brian Bi 提出），C++ 标准引入了一项重大的安全性改进：**正式将“在 return 语句中将返回的引用绑定到临时表达式”的行为标记为非法（ill-formed）**。 [1, 2] 

简单来说，在 C++26 模式下，**很多以前能编译通过、但在运行时会产生悬空引用（Dangling Reference）的 Bug 代码，现在直接变成编译期错误（Compilation Error）**。 [2, 3] 

------

## 1. 为什么要修改？（历史痛点）

在 C++26 之前，C++ 拥有一种**临时对象生命周期延长（Lifetime Extension）**机制。例如： [4, 5] 

```cpp
const std::string& s = std::string("hello"); // 允许！临时对象生命周期延长到与 s 相同
```

但是，标准明确规定：**这种生命周期延长绝对不会发生在 `return` 语句中**。 [1, 5] 

```cpp
// C++26 之前：允许编译，但会引发严重的运行时崩溃
const std::string& get_string() {
    return std::string("hello"); // 临时对象在 return 结束时就被销毁了！
} 
```

在旧标准中，上述代码可以顺利编译通过。尽管主流编译器（如 Clang/GCC）会对此发出警告（Warning），但由于标准认为它是“合法的”（虽然属于未定义行为 UB），在复杂的泛型代码或隐式类型转换中，编译器往往很难甚至无法捕捉到这种错误，从而留下了巨大的内存安全隐患。 [2, 6, 7] 

------

## 2. C++26 具体的标准变更

C++26 标准直接**删除了**旧标准中关于“`return` 语句中绑定的临时对象生命周期不延长”的温和描述，取而代之的是一条铁律： [1] 

> “在返回类型为引用的函数中，如果 `return` 语句将返回的引用绑定到了一个临时表达式，则程序是非法的（ill-formed）。” [1] 

这意味着，编译器必须在编译时拦截此类代码并直接报错。 [2, 7] 

## 典型错误示例（现在无法通过编译）

## 示例 A：直接返回字面量创建的临时对象

```cpp
const int& get_num() {
    return 42; // 错误！42 产生一个临时 int，禁止绑定到返回引用
}
```

## 示例 B：完美转发 / 万能引用中的隐式临时对象

这是一个在泛型编程中极其容易踩中的隐式陷阱：

```cpp
template <typename T>
const T& max_value(const T& a, const T& b) {
    return a > b ? a : b;
}

int main() {
    // max_value 的参数绑定到了 1.0(double) 隐式转换出来的临时 int 上！
    // 最终返回了一个绑定到该临时的引用。
    const int& res = max_value<int>(5, 1.0); // C++26 直接在编译期拒绝此代码
}
```

## 示例 C：类型转换产生的临时对象

```cpp
struct Derived : Base {};
Derived get_derived();

Base& get_base() {
    return get_derived(); // 错误！通过切片隐式转换产生了 Base 类型的临时对象
}
```

------

## 3. 特殊的例外情况：`std::is_convertible`

为了不破坏现有的模板元编程和类型推导逻辑，标准对这一规则做了一个极小的保护性例外： [1, 2] 

- **例外**：在标准库特征检测工具（如 `std::is_convertible`）内部用于推导类型的“虚构函数（Invented Function）”中，这种绑定依然是被允许的。这确保了底层的模板特征（Type Traits）检测不会因为这一安全限制而产生编译死锁。 [1, 2, 8] 

------

## 4. 总结与意义

这项改动是 Modern C++ 迈向**内存安全（Memory Safety）**的关键一步。根据提案作者在 Bloomberg（170万行 C++ 代码）和 Chromium（3900万行 C++ 代码）开源库上的测试，该改动在几乎不破坏合法历史代码的前提下，精准抓出了大量隐藏的悬空引用 Bug。 [8, 9, 10] 

C++26 的核心哲学正在发生变化：**能让编译器在编译期挡住的内存 Bug，就绝对不留到运行时去崩溃。** [2, 3] 

------

如果你想了解编译器如何具体实现这一检测，或者想知道如何安全地重构这些被 C++26 拒绝的历史代码，我们可以针对具体的代码场景继续聊聊！

[1] [https://www.sandordargo.com](https://www.sandordargo.com/blog/2025/06/11/cpp26-no-binding-to-returned-reference-to-temporary)

[2] [https://app.daily.dev](https://app.daily.dev/posts/c-26-disallow-binding-a-returned-reference-to-a-temporary-occ7olfqx)

[3] [https://isocpp.org](https://isocpp.org/blog/2025/07/cpp26-disallow-binding-a-returned-reference-to-a-temporary-sandor-dargo)

[4] [https://stackoverflow.com](https://stackoverflow.com/questions/11560339/returning-temporary-object-and-binding-to-const-reference)

[5] [https://pvs-studio.com](https://pvs-studio.com/en/blog/posts/cpp/1006/)

[6] [https://www.reddit.com](https://www.reddit.com/r/cpp/comments/1la86fp/c26_disallow_binding_a_returned_reference_to_a/)

[7] [https://www.open-std.org](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2748r3.html)

[8] [https://www.open-std.org](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2748r5.html)

[9] [https://news.hada.io](https://news.hada.io/topic?id=28018)

[10] [https://www.infoq.com](https://www.infoq.com/news/2026/04/cpp-26-reflection-safety-async/)