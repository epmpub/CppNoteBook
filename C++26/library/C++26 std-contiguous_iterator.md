**C++26 DR20: Converting contiguous iterators to pointers**（提案 **P3349R1**）是对 `std::contiguous_iterator` 概念的重要**缺陷修复**（Defect Report）。

### 背景问题

C++20 引入了 `std::contiguous_iterator` 概念，其核心保证是：

> 迭代器所指向的元素在内存中**连续存储**（contiguous in memory），因此 `&*it + n` 等价于 `&*(it + n)`。

但是，**标准中长期存在歧义**：

- 标准库算法和某些设施（如 `std::to_address`、`std::span` 构造、`ranges::subrange` 等）**假设**能可靠地将 `contiguous_iterator` 转换为底层指针。
- 但原规范中**没有强制要求** `std::to_address(i)` 对所有 `contiguous_iterator` 都必须返回有效指针。
- 一些自定义迭代器虽然满足 `contiguous_iterator` 概念，但无法可靠转换为指针，导致标准库实现行为不一致或性能下降。

### C++26 的修复（DR20）

**主要变更**：

1. **加强 `contiguous_iterator` 概念的要求**：
   - 现在明确要求：对于满足 `contiguous_iterator<I>` 的迭代器 `i`，`std::to_address(i)` 必须是合法的且返回正确的指针。

2. **改进 `std::to_address`**：
   - 增强其对 contiguous iterators 的支持。
   - 允许标准库在内部直接使用指针算术来优化算法（例如 `std::copy`、`std::sort` 等）。

3. **规范库函数的行为**：
   - 允许标准库在处理 `[i, s)` 范围时，将迭代器操作替换为指针操作：
     ```cpp
     // 实现可优化为：
     std::copy(i, s, out);  
     // 等价于使用 to_address(i) 进行指针版本的操作
     ```

### 主要影响

- **对用户自定义迭代器**：
  - 如果你实现了一个 `contiguous_iterator`，现在**必须**正确支持 `std::to_address(it)`（通常通过 `std::addressof(*it)` 或返回内部指针）。
  - 这使得自定义 contiguous 迭代器更容易被标准库高效使用。

- **对标准库实现**：
  - 允许更多激进的优化（尤其是 `ranges` 算法和 `std::span`）。
  - 提升了 `vector`、`string`、`array` 等 contiguous 容器的性能一致性。

- **与 `std::to_address` 的关系**：
  ```cpp
  #include <iterator>
  #include <vector>
  
  std::vector<int> v = {1,2,3,4};
  auto it = v.begin();
  
  int* p = std::to_address(it);   // C++26 后更可靠
  ```

### Feature Test / DR 标记

在 cppreference 等文档中标记为 **DR20**（C++26 Defect Report）。

**提案**：  
- [P3349R1 Converting Contiguous Iterators To Pointers](https://wg21.link/P3349)

### 总结

这是一个**清理性质**的 DR，修复了 C++20 `contiguous_iterator` 概念在实际使用中的一个重要漏洞。它让 **“contiguous = 可以安全转为指针”** 这个直观语义在标准中真正落地，从而提升了 Ranges 和算法的性能，并让自定义 contiguous 迭代器行为更可预测。

如果你在实现自定义 contiguous iterator，或者在使用 `std::to_address` 时遇到问题，这个 DR 正是为了解决这类场景。需要具体实现示例吗？