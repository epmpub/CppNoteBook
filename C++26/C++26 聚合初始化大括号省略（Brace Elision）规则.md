在 C++26 中，通过核心提案 **[P3106R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p3106r1.html)**（由 James Touton 提出），C++ 标准对**聚合初始化（Aggregate Initialization）中的大括号省略（Brace Elision）规则进行了全面澄清和规范**，并作为缺陷报告（Defect Report, DR）合入标准。 [1, 2] 

这一改动的核心目的是**修复旧标准中大括号省略在边界情况下的语义矛盾、消除了编译器之间的实现分歧，并修正了此前未考虑到的数组和空结构体初始化漏洞**。 [3] 

------

## 一、 什么是大括号省略（Brace Elision）？

在 C++ 中，当一个聚合体（如结构体或数组）内部包含另一个子聚合体时，标准允许在初始化时**省去内层的大括号**。 [4] 

```cpp
struct Point { int x, y; };
struct Line { Point start; Point end; };

// 完整写法的初始化：
Line l1 = { {1, 2}, {3, 4} };

// 触发大括号省略（Brace Elision）的扁平写法：
Line l2 = { 1, 2, 3, 4 }; // 允许！1,2 自动填充 start，3,4 自动填充 end
```

这种设计虽然带来了书写的便利性，但由于老版标准描述不够严谨，在遇到**未知边界数组**和**空子聚合体**时，标准条文引发了逻辑悖论。 [3] 

------

## 二、 C++26 修复和规范的核心问题

## 1. 修复了“未知边界数组”推导的悖论 (CWG 2149)

在旧标准中，声明一个未知边界数组（如 `int arr[]`）时，标准规定**“数组的长度由初始化列表中的元素个数决定”**。但是，当引入大括号省略时，这个定义就失效了。 [3] 

**致命的逻辑矛盾漏洞：**

```cpp
struct Agg { int x, y; };
Agg arr[] = { 1, 2, 3, 4 }; 
```

- **根据数组规则**：初始化列表里有 `1, 2, 3, 4` 共 **4 个**元素，因此 `arr` 的长度应该是 `4`（即 `Agg arr[4]`）。
- **根据大括号省略规则**：因为 `Agg` 需要两个 `int`，`1,2` 结合为第一个元素，`3,4` 结合为第二个元素。实际只构造了 **2 个** `Agg` 对象。 [3] 

这导致数组长度在标准描述中同时既是 4 又是 2。

- **C++26 修复**：重写了计算规则。明确规定：在确定数组或聚合体的显式初始化元素数量时，**必须先应用大括号省略的平铺展开逻辑**。计算出实际消耗并成功分配给子对象的结构化元素个数后，才能确定最终的数组长度（即上例中 `arr` 的长度确定为 `2`）。 [5] 

## 2. 删除了多余的“空子聚合体”特例限制

在旧标准（如 C++20/C++23）中，有一个关于空嵌套结构体的严格限制：**如果一个聚合体中包含一个完全没有任何成员的空结构体（或只有静态成员的结构体），在这个空结构体后面如果还有其他非空成员，那么大括号省略是不允许的**。 [4, 5] 

```cpp
struct Empty {};
struct Sub { int val; };
struct MyAgg {
    Empty e;
    Sub s;
};

// C++20/C++23 规则：
MyAgg a = { {}, 42 }; // 必须为空结构体写一个 {}
// MyAgg b = { 42 };  // 错误！旧标准禁止在此处省略 e 的大括号并让 42 飘过去
```

- **C++26 修复**：P3106R1 直接**删除了这一条过时的段落**（删除了原标准中的 Paragraph 14）。因为现代 C++ 的初始化逻辑完全有能力在不借助这条特例的情况下，优雅地处理空结构体。现在，大括号省略规则变得更加通用和统一。 [5] 

## 3. 规范了元素过多（Overflow）时的非法处理

旧标准对“当平铺大括号时，传入的初始化参数超过了聚合体实际能容纳的数量”这一场景的文字描述带有歧义，导致部分编译器抛出硬错误，部分编译器表现异常。 [5] 

- **C++26 修复**：统一并严密化了表述。如果初始化列表（Initializer List）在经过大括号省略匹配后，**剩余的参数无法与任何后续的聚合体成员匹配（即参数溢出），该程序直接被定义为不合法（ill-formed）**。 [5] 

------

## 三、 总结与意义

C++26 对大括号省略规则的澄清，属于对语言规范**核心语义层面的“修辞与逻辑大扫除”**。

- **对开发者的影响**：几乎没有破坏性改变，反而让那些以前在 GCC、Clang 或 MSVC 之间编译结果不一致的复杂嵌套数组/结构体代码（例如由宏生成或元编程生成的平铺初始化代码），获得了**完全一致且安全可预测的编译行为**。
- **支持状态**：该提案已被作为 **缺陷报告 (DR)** 采纳，这意味着它不仅适用于 C++26 模式，各大编译器也会将此修复逆向推导应用到 C++20 和 C++23 的标准实现中。现行的 **Clang 17+** 和 **GCC 14+** 已经内置了这一套规范后的处理逻辑。 [1, 2] 

您在实际编写涉及 `std::array` 嵌套或者大数组初始化时，是否遇到过必须写双层大括号（如 `{{ ... }}`）否则报错的困惑呢？我们可以结合具体的代码结构来聊聊它的进化！ [6, 7] 

[1] [https://cppreference.com](https://cppreference.com/cpp/26)

[2] [https://cppreference-45864d.gitlab-pages.liu.se](https://cppreference-45864d.gitlab-pages.liu.se/en/cpp/compiler_support/26.html)

[3] [https://www.open-std.org](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p3106r0.html)

[4] [https://olympiads.ru](https://olympiads.ru/zaoch/2018-19/lang_docs/cppreference.com/reference/en/cpp/language/aggregate_initialization.html)

[5] [https://www.open-std.org](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p3106r1.html)

[6] [https://stackoverflow.com](https://stackoverflow.com/questions/50598248/why-does-initialization-of-array-of-pairs-still-need-double-braces-in-c14)

[7] [https://stackoverflow.com](https://stackoverflow.com/questions/11734861/when-can-outer-braces-be-omitted-in-an-initializer-list)