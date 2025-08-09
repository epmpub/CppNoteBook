# std::merge 和 std::ranges::merge



```C++
#include <vector>
#include <algorithm>
#include <print>
#include <string>
#include <ranges>

int main() {
    std::vector<int> first{1, 2, 5};
    std::vector<int> second{2, 3, 4};
    std::vector<int> out;

    std::merge(
        begin(first), end(first),   // first input range
        begin(second), end(second), // second input range
        std::back_inserter(out));   // output range
    // out == {1, 2, 2, 3, 4, 5}

    std::println("out == {}", out);

    first = {5, 3, 1};
    second = {4, 2};
    out = {};

    std::ranges::merge(first, second, 
        std::back_inserter(out),
        std::greater<>{}); // Custom comparator
    // out = {5, 4, 3, 2, 1}

    std::println("out == {}", out);

    struct Value {
        int x;
        std::string label;
        bool operator<(const Value& v) const {
            return x < v.x;
        }
    };

    std::vector<Value> a{{0, "a1"}, {0, "a2"}, {1, "a3"}};
    std::vector<Value> b{{0, "b1"}, {1, "b2"}, {1, "b3"}};
    std::vector<Value> c;

    std::ranges::merge(a, b, std::back_inserter(c), std::less<>{});
    // c == {"a1", "a2", "b1", "a3", "b2", "b3"}
    
    std::println("c == {}", c | std::views::transform(&Value::label));
}
```

在 C++ 中，std::merge 和 std::ranges::merge 是用于合并两个有序序列的算法，但它们分别属于传统的 STL 算法（<algorithm>）和 C++20 引入的 Ranges 库（<ranges>）。以下是对两者的详细解释和比较。

------

1. **std::merge**

- **头文件**：<algorithm>

- **定义**：传统的 STL 算法，存在于 C++98 以来。

- **功能**：将两个有序范围合并为一个新的有序范围。

- **原型**：

  cpp

  ```cpp
  template <class InputIt1, class InputIt2, class OutputIt>
  OutputIt merge(InputIt1 first1, InputIt1 last1,
                 InputIt2 first2, InputIt2 last2,
                 OutputIt d_first);
  
  template <class InputIt1, class InputIt2, class OutputIt, class Compare>
  OutputIt merge(InputIt1 first1, InputIt1 last1,
                 InputIt2 first2, InputIt2 last2,
                 OutputIt d_first, Compare comp);
  ```

- **参数**：

  - first1, last1：第一个输入范围的起始和结束迭代器。
  - first2, last2：第二个输入范围的起始和结束迭代器。
  - d_first：输出范围的起始迭代器。
  - comp（可选）：自定义比较器，默认使用 <。

- **要求**：

  - 两个输入范围必须已按升序（或按 comp）排序。
  - 输出范围必须有足够空间容纳结果。

- **返回值**：指向输出范围末尾的迭代器。

- **示例**：

  cpp

  ```cpp
  #include <algorithm>
  #include <vector>
  #include <iostream>
  
  int main() {
      std::vector<int> v1 = {1, 3, 5};
      std::vector<int> v2 = {2, 4, 6};
      std::vector<int> result(6);
  
      std::merge(v1.begin(), v1.end(), v2.begin(), v2.end(), result.begin());
  
      for (int x : result) {
          std::cout << x << " "; // 输出: 1 2 3 4 5 6
      }
  }
  ```

------

2. **std::ranges::merge**

- **头文件**：<ranges> 和 <algorithm>

- **定义**：C++20 Ranges 库中的算法，提供更现代的接口。

- **功能**：与 std::merge 相同，但支持 Ranges 概念，接口更简洁。

- **原型**：

  cpp

  ```cpp
  namespace std::ranges {
      template <std::input_iterator I1, std::sentinel_for<I1> S1,
                std::input_iterator I2, std::sentinel_for<I2> S2,
                std::weakly_incrementable O, class Comp = std::less<>,
                class Proj1 = std::identity, class Proj2 = std::identity>
      requires std::mergeable<I1, I2, O, Comp, Proj1, Proj2>
      merge_result<I1, I2, O>
      merge(I1 first1, S1 last1, I2 first2, S2 last2, O result,
            Comp comp = {}, Proj1 proj1 = {}, Proj2 proj2 = {});
  
      template <ranges::input_range R1, ranges::input_range R2,
                std::weakly_incrementable O, class Comp = std::less<>,
                class Proj1 = std::identity, class Proj2 = std::identity>
      requires std::mergeable<ranges::iterator_t<R1>, ranges::iterator_t<R2>, O, Comp, Proj1, Proj2>
      merge_result<ranges::borrowed_iterator_t<R1>, ranges::borrowed_iterator_t<R2>, O>
      merge(R1&& r1, R2&& r2, O result,
            Comp comp = {}, Proj1 proj1 = {}, Proj2 proj2 = {});
  }
  ```

- **参数**：

  - 两种重载：
    1. **迭代器版本**：类似 std::merge，但支持哨兵（sentinel）和投影（projection）。
    2. **范围版本**：直接接受 Ranges 对象（如 std::vector）。
  - first1, last1 / r1：第一个输入范围。
  - first2, last2 / r2：第二个输入范围。
  - result：输出范围的起始迭代器。
  - comp（可选）：比较器，默认 std::less<>。
  - proj1, proj2（可选）：投影函数，默认 std::identity（直接使用元素）。

- **要求**：

  - 输入范围必须已排序。
  - 输出范围必须有足够空间。

- **返回值**：

  - merge_result<I1, I2, O> 结构，包含三个迭代器：
    - in1：第一个输入范围的结束位置。
    - in2：第二个输入范围的结束位置。
    - out：输出范围的结束位置。

- **示例**：

  cpp

  ```cpp
  #include <ranges>
  #include <algorithm>
  #include <vector>
  #include <iostream>
  
  int main() {
      std::vector<int> v1 = {1, 3, 5};
      std::vector<int> v2 = {2, 4, 6};
      std::vector<int> result(6);
  
      auto [in1, in2, out] = std::ranges::merge(v1, v2, result.begin());
  
      for (int x : result) {
          std::cout << x << " "; // 输出: 1 2 3 4 5 6
      }
  }
  ```

------

主要区别

| 特性     | std::merge                | std::ranges::merge           |
| -------- | ------------------------- | ---------------------------- |
| 引入版本 | C++98                     | C++20                        |
| 接口     | 迭代器对 (begin, end)     | 支持迭代器对和 Ranges 对象   |
| 返回值   | 输出迭代器 (OutputIt)     | merge_result 结构            |
| 比较器   | 可选，默认 <              | 可选，默认 std::less<>       |
| 投影     | 不支持                    | 支持（proj1, proj2）         |
| 哨兵支持 | 不支持（需要 end 迭代器） | 支持（last 可以是哨兵）      |
| 约束     | 手动确保迭代器有效        | 编译期检查（std::mergeable） |

------

详细对比

1. **接口灵活性**：

   - std::merge 要求显式传递迭代器对，适合传统代码。
   - std::ranges::merge 支持直接传递容器（如 v1, v2），更简洁。

2. **投影功能**：

   - std::ranges::merge 支持投影函数，允许在比较前转换元素。例如：

     cpp

     ```cpp
     struct Item { int value; };
     std::vector<Item> v1 = {{1}, {3}};
     std::vector<Item> v2 = {{2}, {4}};
     std::vector<Item> result(4);
     std::ranges::merge(v1, v2, result.begin(),
                        std::less<>{}, // 比较器
                        &Item::value); // 投影到 value
     ```

3. **返回值**：

   - std::merge 只返回输出结束迭代器。
   - std::ranges::merge 返回结构化结果，便于进一步处理。

4. **类型安全**：

   - std::ranges::merge 使用 Concepts（如 std::mergeable），在编译期检查输入和输出类型的兼容性。

------

使用场景

- **std::merge**：
  - 适用于传统代码或需要兼容旧标准的场景。
  - 手动管理迭代器，适合低级控制。
- **std::ranges::merge**：
  - 现代 C++ 项目，追求简洁和类型安全。
  - 需要投影或哨兵支持的复杂场景。

------

总结

- std::merge 是经典算法，依赖迭代器对，简单直接。
- std::ranges::merge 是 Ranges 版本，提供更现代的接口，支持范围、投影和哨兵。
- 两者功能相同，但 std::ranges::merge 更灵活、更安全，适合新代码。