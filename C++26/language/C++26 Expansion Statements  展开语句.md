Expansion Statements / 展开语句

在 C++26 中，**[P1306R5 提案（Expansion Statements / 展开语句）](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p1306r5.html)** 引入了备受瞩目的 **`template for`** 语法。这是 C++ 元编程史上的里程碑式飞跃：**它允许我们在编译期直接“循环”展开参数包（Parameter Packs）、元组（Tuples）以及编译期范围（Ranges），彻底告别了过去晦涩丑陋的折叠表达式展开和复杂的递归模板诡计**。 [1, 2] 

------

## 历史痛点：编译期“循环”的噩梦

在 C++23 之前，如果你想遍历一个异构容器（如 `std::tuple`）或参数包中的每一个元素，并对它们执行某种操作，你无法使用常规的 `for` 循环，因为 `for` 是运行时逻辑，要求每次迭代的类型完全一致。

为了在编译期实现遍历，开发者们被迫发明了极其晦涩的绕路技巧： [1, 2] 

```cpp
// 传统的 C++ 绕路写法：使用 C++17 折叠表达式（视觉噪音极大，且不支持 break 语句）
template<typename... Args>
void print_all(Args&&... args) {
    (std::println("{}", args), ...); 
}

// 遍历 tuple 更是需要复杂的 std::apply 配合变长通用 Lambda：
std::apply([](auto&&... args) {
    (std::println("{}", args), ...);
}, my_tuple);
```

**致命缺陷**：折叠表达式只是一行连缀展开，不仅极难阅读，而且**完全不支持在“循环”内部执行 `break` 或 `continue` 进行提早终止控制**。 [1, 2] 

## C++26 的解决方案：`template for` [3] 

C++26 的 `template for` 本质上是一个**代码打桩生成器（Code Stamper）**。它在编译期运行时会将循环体内的逻辑针对每个元素硬性“拷贝”铺平展开，其展开生成的每一步（$S_0; S_1; ... S_{n-1}$）内部，**迭代变量可以拥有截然不同的类型或不同的 `constexpr` 属性值**。 [1, 2] 

根据 P1306R5 标准定义，`template for` 的展开分为以下三大应用场景： [1, 2] 

## 1. 异构元组展开（Destructuring Expansion）

你可以通过类似结构化绑定（Structured Bindings）的语法，直接遍历一个 `std::tuple` 或任何满足 tuple-like 协议的自定义类： [1, 4] 

```cpp
#include <tuple>
#include <print>

void demo_tuple() {
    auto my_tuple = std::make_tuple(42, "C++26", 3.14);

    // C++26 完美通过：直接在编译期铺平遍历异构集合
    template for (auto&& elem : my_tuple) {
        // 在每一次展开步骤中，elem 的实际类型都会发生演变 (int -> const char* -> double)
        std::println("Value: {}", elem);
    }
}
```

- **编译器视角**：它会将上述代码原地解包为 3 个独立的代码块，完美避免了任何运行期开销。 [2, 4] 

## 2. 参数包与表达式列表展开（Enumerating Expansion）

可以直接处理变长参数包或者由花括号包围的显式表达式列表： [1, 4] 

```cpp
template <typename... Args>
void process_pack(Args&&... args) {
    // 针对传递进来的动态参数包逐一解包
    template for (auto&& arg : {args...}) {
        if constexpr (std::is_pointer_v<std::decay_t<decltype(arg)>>) {
            if (arg == nullptr) break; // 🌟 杀手级能力：支持传统循环控制！
        }
        std::println("{}", arg);
    }
}
```

## 3. 编译期常量范围展开（Iterating Expansion） [5, 6] 

如果你传递一个在编译期就能确定大小（Random-Access Range）且可在编译期评估的 `constexpr` 容器，它也可以安全展开： [1, 4] 

```cpp
#include <array>

constexpr std::array<int, 4> indices = {0, 1, 2, 3};

void compile_time_loop() {
    // 强制静态展开循环
    template for (constexpr int idx : indices) {
        // 这里的 idx 具备真切的编译期常量属性，可直接充当模板参数！
        std::println("Step {}", idx);
    }
}
```

------

## `template for` vs 传统运行时 `for`

为了便于记忆，我们可以通过下表清晰地看出它与常规循环的本质差别：

| 特性 [1, 2, 3, 6, 7]           | 传统 `for` / 范围 `for`    | C++26 `template for`                             |
| :----------------------------- | :------------------------- | :----------------------------------------------- |
| **执行时期**                   | 运行期（Runtime）          | **编译期代码生成（Compile-time Expansion）**     |
| **循环步变量类型**             | 所有迭代步骤中必须绝对一致 | **每一步均可具有完全不同的异构类型**             |
| **控制流（break / continue）** | 支持                       | **全面支持**（折叠表达式无法做到这一点）         |
| **开销行为**                   | 产生跳转指令与条件判定开销 | **零运行期开销**（代码在编译期已被完全硬性铺平） |

------

## 与 C++26 静态反射（P2996）的梦幻联动 [8, 9] 

`template for` 最让人兴奋的终极形态，是它能与 **C++26 的另一个头等特性——静态反射（Static Reflection）** 产生无缝化学反应。 [7, 10] 

借由反射，我们可以直接提取出一个类的所有成员变量，并在编译期利用 `template for` 自动生成诸如 JSON 序列化、打印日志等过去极度繁琐的工程代码： [1] 

```cpp
#include <experimental/meta> // C++26 反射头文件
#include <print>

struct User {
    int id;
    std::string name;
    bool is_admin;
};

template <typename T>
void auto_print_fields(const T& obj) {
    // 1. 获取该结构体所有成员变量的反射元数据集合（编译期 range）
    constexpr auto fields = std::meta::members_of(^T); 

    // 2. 利用 template for 静态遍历这些成员元数据
    template for (constexpr auto field : fields) {
        // 3. 将元数据拼接（Splice）回真实代码，直接读取对应成员的值！
        std::println("{}: {}", std::meta::name_of(field), obj.[:field:]);
    }
}
```

##  总结

C++26 的 `Expansion Statements` 终结了现代 C++ 在处理“变长泛型数据”时长达数年的语法痛苦期。它用最符合人类直觉的 `for` 循环外壳，包装了最强大的编译期铺平技术，让模板元编程的门槛大幅降低。 [2, 11, 12, 13] 

你现在负责的项目里，是否有大量手写的 **`std::tuple` 访问器**、**变长完美转发处理器** 或者 **复杂的 `std::index_sequence` 递归**？我们可以聊聊如何用 `template for` 把它们彻底重构成通俗易懂的直观代码！

[1] [https://stackoverflow.com](https://stackoverflow.com/questions/79691380/what-is-template-for-in-c26)

[2] [https://stackoverflow.com](https://stackoverflow.com/questions/79691380/what-is-template-for-in-c26)

[3] [https://cppreference.com](https://cppreference.com/cpp/language/template_for)

[4] [https://www.open-std.org](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p1306r5.html)

[5] [https://www.reddit.com](https://www.reddit.com/r/cpp/comments/1jgi1yg/c26_expansion_tricks/)

[6] [https://www.open-std.org](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p1306r5.html)

[7] [https://www.reddit.com](https://www.reddit.com/r/cpp/comments/1jgi1yg/c26_expansion_tricks/)

[8] [https://www.open-std.org](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p1306r3.pdf)

[9] [https://www.reddit.com](https://www.reddit.com/r/cpp/comments/1dnquk4/implementation_of_token_sequence_expressions_p3294/)

[10] [https://indico.cern.ch](https://indico.cern.ch/event/1471803/contributions/6966807/attachments/3281291/5863853/CHEP26_cpp_reflection_jolly_chen_260526_.pdf)

[11] [https://www.sandordargo.com](https://www.sandordargo.com/blog/2025/04/30/cpp26-constexpr-library-changes)

[12] [https://isocpp.org](https://isocpp.org/files/papers/P1789R3.pdf)

[13] [https://github.com](https://github.com/cplusplus/papers/issues/156)