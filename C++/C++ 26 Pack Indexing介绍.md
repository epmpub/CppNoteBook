# C++ 26 Pack Indexing介绍

**Pack Indexing** 是 C++26 引入的一项核心语言特性，用于直接访问模板参数包（Parameter Pack）中的特定元素。它通过一个直观的索引语法（...[I]）极大地简化了参数包的操作，避免了之前版本中繁琐的递归模板或标准库工具（如 std::tuple）的间接方法。这一特性由 Corentin Jabot 和 Pablo Halpern 在提案 P2662R3 中提出，并被纳入 C++26 标准。

以下是对 Pack Indexing 的详细介绍：

------

背景

在 C++11 中引入了参数包（Parameter Pack），允许模板接受不定数量的类型或值参数（如 typename... Ts 或 Ts... args）。虽然参数包通过折叠表达式（C++17）和扩展语法提供了强大的功能，但直接提取特定元素却一直是个难题。开发者通常需要：

- 使用递归模板。
- 借助 std::tuple 和 std::get。
- 编写复杂的布尔表达式。

这些方法虽然可行，但往往代码冗长、可读性差，且编译性能开销较大。Pack Indexing 的引入解决了这一痛点，提供了一种原生的、简洁的访问方式。

------

语法和基本用法

Pack Indexing 使用 ...[I] 语法，其中：

- ... 表示参数包的展开。
- [I] 是索引，I 必须是编译期常量表达式，指定要访问的包中第几个元素（从 0 开始计数）。

它适用于以下场景：

1. **函数参数包（Function Parameter Pack）**：

   cpp

   ```cpp
   template <typename... Ts>
   auto get_first(Ts... args) {
       return args...[0];  // 返回第一个参数
   }
   ```

   - 调用：get_first(1, 2.5, "hello") 返回 1。

2. **模板参数包（Template Parameter Pack）**：

   cpp

   ```cpp
   template <typename... Ts>
   using FirstType = Ts...[0];  // 获取第一个类型
   ```

   - FirstType<int, double, char> 等价于 int。

3. **结构绑定中的包（Structured Binding Pack）**：

   cpp

   ```cpp
   template <typename Tup>
   auto get_second(Tup tup) {
       auto [...elems] = tup;
       return elems...[1];  // 返回第二个元素
   }
   ```

4. **Lambda 捕获中的包（Lambda Capture Pack）**：

   cpp

   ```cpp
   template <std::size_t I, typename... Args>
   auto foo(Args... args) {
       return [...members = args](Args...[I] op) {
           return members...[I] + op;
       };
   }
   ```

------

特点和限制

1. **编译期索引**：

   - 索引 I 必须是编译期常量，例如 0、sizeof...(Ts)-1。

   - 不支持运行时索引（如变量 int i），否则会编译错误：

     cpp

     ```cpp
     int idx = 1;
     args...[idx];  // 错误：idx 不是常量表达式
     ```

2. **范围检查**：

   - 如果索引超出包的大小（如空包或越界），程序是非法（ill-formed），编译器会报错：

     cpp

     ```cpp
     static_assert(get_first() == 1);  // 错误：空包无法索引
     ```

3. **仅限于标识符**：

   - Pack Indexing 只能用于简单的标识符（id-expression），不支持复杂表达式：

     cpp

     ```cpp
     std::forward<Args>(args)...[I];  // 错误：复杂表达式不可索引
     std::forward<Args...[I]>(args...[I]);  // 正确
     ```

4. **引用类别保留**：

   - 访问参数时，默认得到的是左值引用。若要保留原始引用类别（如右值引用），需显式转换：

     cpp

     ```cpp
     auto&& arg = static_cast<Args...[I]&&>(args...[I]);
     ```

5. **不支持负数索引或切片**：

   - 当前不支持从末尾索引（如 Python 的负数索引）或切片（如 Ts...[1:3]），但这些可能是未来扩展方向。

------

示例代码

以下是一个综合示例，展示 Pack Indexing 的用法：

cpp

```cpp
#include <print>

template <typename... Ts>
auto first_plus_last(Ts... values) {
    return values...[0] + values...[sizeof...(values) - 1];
}

template <std::size_t I, typename... Ts>
auto get_at(Ts... args) {
    return args...[I];
}

int main() {
    std::println("First + Last: {}", first_plus_last(1, 2, 3, 4, 5));  // 输出: 6 (1 + 5)
    std::println("Element at 2: {}", get_at<2>(10, 20, 30, 40));     // 输出: 30
}
```

------

实际应用

1. **简化元编程**：
   - 直接访问类型包中的特定类型，无需递归或 std::tuple_element。
   - 示例：提取第一个类型用于类型别名。
2. **参数处理**：
   - 在变参函数中直接操作特定参数，提升代码可读性。
3. **结构化绑定**：
   - 配合 C++17 的结构绑定，轻松访问解构后的元素。

------

与之前方法的对比

在 C++26 之前，提取第 N 个元素通常需要：

- **递归模板**：

  cpp

  ```cpp
  template <std::size_t I, typename T, typename... Ts>
  struct NthType {
      using type = typename NthType<I - 1, Ts...>::type;
  };
  template <typename T, typename... Ts>
  struct NthType<0, T, Ts...> {
      using type = T;
  };
  ```

- **使用 std::tuple**：

  cpp

  ```cpp
  template <std::size_t I, typename... Ts>
  auto get_nth(Ts... args) {
      return std::get<I>(std::forward_as_tuple(std::forward<Ts>(args)...));
  }
  ```

相比之下，Pack Indexing 的 args...[I] 更直观、更高效，减少了编译期开销和代码复杂度。

------

未来展望

Pack Indexing 是参数包操作的一个重要进步，但仍有一些未解决的需求：

- **反向索引**：如 Ts...[-1] 表示最后一个元素。
- **切片**：如 Ts...[1:3] 提取子包。
- **运行时索引**：支持动态索引（需更大语言改动）。

这些功能已在提案中讨论，可能在 C++29 或更晚版本实现。

------

总结

Pack Indexing（C++26）通过 ...[I] 语法为参数包提供了数组式的索引能力，使代码更简洁、可读性更高，同时保持编译期的高效性。它是 C++ 模板元编程的一次重要进化，填补了长期以来访问参数包元素的空白。对于开发者来说，这是一个值得关注和掌握的新工具，尤其在处理变参模板时能显著提升开发体验。