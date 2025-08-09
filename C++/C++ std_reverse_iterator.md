# std::reverse_iterator

在 C++ 中，std::reverse_iterator 是一个标准库提供的迭代器适配器，用于以逆序遍历容器中的元素。它通常与标准容器的正向迭代器（如 iterator）配合使用，通过包装正向迭代器来实现反向遍历的功能。std::reverse_iterator 定义在 <iterator> 头文件中，是 STL（标准模板库）的一部分。

基本概念

- **正向迭代器**：从容器的开头向结尾移动（递增）。
- **逆向迭代器**：从容器的结尾向开头移动（递减）。
- std::reverse_iterator 本质上是对一个正向迭代器的封装，但它的行为是反向的。例如，当你对 reverse_iterator 执行 ++ 操作时，它实际上会让底层的正向迭代器执行 -- 操作。

构造

std::reverse_iterator 通常通过容器的 rbegin() 和 rend() 方法来获取：

- rbegin()：返回指向容器最后一个元素的逆向迭代器。
- rend()：返回指向容器第一个元素之前位置的逆向迭代器。

你也可以通过正向迭代器显式构造一个 reverse_iterator，例如：

cpp

```cpp
std::vector<int> vec = {1, 2, 3, 4};
std::reverse_iterator<std::vector<int>::iterator> rit(vec.end());
```

工作原理

std::reverse_iterator 的核心思想是调整迭代器的行为：

- 它的 * 操作返回的是底层正向迭代器指向的前一个元素。
- 递增操作（++）会使底层迭代器递减（--）。
- 递减操作（--）会使底层迭代器递增（++）。

这种设计确保了逆向迭代器可以无缝地与正向迭代器的工作方式保持一致。

示例代码

以下是一个简单的使用示例：

cpp

```cpp
#include <iostream>
#include <vector>
#include <iterator>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};

    // 使用逆向迭代器遍历
    std::cout << "Reverse order: ";
    for (std::reverse_iterator<std::vector<int>::iterator> it = vec.rbegin(); it != vec.rend(); ++it) {
        std::cout << *it << " "; // 输出：5 4 3 2 1
    }
    std::cout << std::endl;

    return 0;
}
```

成员函数

std::reverse_iterator 提供了一些有用的成员函数：

- base()：返回底层的正向迭代器。
- \* 和 ->：访问当前元素。
- ++ 和 --：移动迭代器。

例如：

cpp

```cpp
std::vector<int> vec = {1, 2, 3};
auto rit = vec.rbegin();
std::cout << *rit << std::endl; // 输出：3
auto base_it = rit.base();      // 返回 vec.end()
```

注意事项

1. **边界**：逆向迭代器的 rend() 对应于正向迭代器的 begin() 之前的位置，因此不能解引用 rend()。
2. **底层迭代器要求**：std::reverse_iterator 需要底层迭代器至少是双向迭代器（BidirectionalIterator），因为它依赖于 -- 操作。
3. **性能**：由于只是对正向迭代器的简单包装，std::reverse_iterator 的开销非常小。

应用场景

- 需要逆序遍历容器时，例如打印元素、反向处理数据等。
- 与算法（如 std::sort）结合使用时，可以通过逆向迭代器指定反向顺序。

总之，std::reverse_iterator 是一个简单而强大的工具，它扩展了 C++ 迭代器的灵活性，让开发者可以轻松实现反向遍历操作。