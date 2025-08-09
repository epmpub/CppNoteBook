# `std::front_inserter`、`std::back_inserter` 和 `std::inserter`

在 C++ 标准库中，`std::front_inserter`、`std::back_inserter` 和 `std::inserter` 是**插入迭代器适配器**，用于将元素插入到容器中，而不是覆盖现有元素。它们常用于标准算法（如 `std::copy`、`std::transform`）的输出位置，使算法能够将结果直接插入到容器的特定位置。

以下是它们的详细解释和区别：

---

### **1. `std::back_inserter`（尾部插入迭代器）**
- **功能**：将元素插入到容器的**末尾**。
- **底层操作**：调用容器的 `push_back` 方法。
- **适用容器**：支持 `push_back` 的容器（如 `std::vector`、`std::deque`、`std::list`）。
- **特点**：
  - 元素按原顺序依次添加到容器尾部。
  - 时间复杂度：对于 `std::vector` 是均摊 O(1)，其他容器通常为 O(1)。
- **示例**：
  ```cpp
  #include <vector>
  #include <algorithm>
  #include <iterator>
  
  int main() {
      std::vector<int> src = {1, 2, 3};
      std::vector<int> dst;
  
      // 将 src 的元素复制到 dst 的末尾
      std::copy(src.begin(), src.end(), std::back_inserter(dst));
      // dst 结果为 {1, 2, 3}
  }
  ```

---

### **2. `std::front_inserter`（头部插入迭代器）**
- **功能**：将元素插入到容器的**头部**。
- **底层操作**：调用容器的 `push_front` 方法。
- **适用容器**：支持 `push_front` 的容器（如 `std::list`、`std::deque`）。
- **特点**：
  - 元素按**逆序**插入到容器头部（每次插入到最前面）。
  - 时间复杂度：通常为 O(1)。
  - **不适用 `std::vector`**（因为它不支持 `push_front`）。
- **示例**：
  ```cpp
  #include <list>
  #include <algorithm>
  #include <iterator>
  
  int main() {
      std::list<int> src = {1, 2, 3};
      std::list<int> dst;
  
      // 将 src 的元素逆序插入到 dst 的头部
      std::copy(src.begin(), src.end(), std::front_inserter(dst));
      // dst 结果为 {3, 2, 1}
  }
  ```

---

### **3. `std::inserter`（指定位置插入迭代器）**
- **功能**：将元素插入到容器的**指定位置**。
- **底层操作**：调用容器的 `insert` 方法。
- **适用容器**：所有支持 `insert` 方法的容器（如 `std::vector`、`std::list`、`std::set`、`std::map`）。
- **特点**：
  - 需要指定插入的初始位置（迭代器）。
  - 每次插入后，迭代器会自动更新到新插入元素之后的位置，保持后续插入的顺序正确。
  - 时间复杂度：依赖容器的 `insert` 方法（如 `std::vector` 中间插入为 O(N)）。
- **示例**：
  ```cpp
  #include <vector>
  #include <algorithm>
  #include <iterator>
  
  int main() {
      std::vector<int> src = {1, 2, 3};
      std::vector<int> dst = {0, 4};
  
      // 在 dst 的第二个位置（元素 4 之前）插入 src 的元素
      std::copy(src.begin(), src.end(), std::inserter(dst, dst.begin() + 1));
      // dst 结果为 {0, 1, 2, 3, 4}
  }
  ```

---

### **三者的核心区别**
| 特性           | `std::back_inserter`     | `std::front_inserter`    | `std::inserter`                               |
| -------------- | ------------------------ | ------------------------ | --------------------------------------------- |
| **插入位置**   | 容器末尾                 | 容器头部                 | 指定位置                                      |
| **底层方法**   | `push_back`              | `push_front`             | `insert`                                      |
| **适用容器**   | 支持 `push_back` 的容器  | 支持 `push_front` 的容器 | 所有支持 `insert` 的容器                      |
| **元素顺序**   | 保持原序                 | 逆序                     | 保持原序（从指定位置开始）                    |
| **时间复杂度** | 通常 O(1)（如 `vector`） | O(1)（如 `list`）        | 依赖容器和位置（如 `vector` 中间插入为 O(N)） |

---

### **使用场景**
1. **`std::back_inserter`**：  
   - 需要将元素顺序添加到容器末尾（如构建队列、动态数组）。
   - 示例：`std::copy(src.begin(), src.end(), std::back_inserter(dst))`.

2. **`std::front_inserter`**：  
   - 需要逆序插入元素到容器头部（如构建栈或逆序链表）。
   - 示例：`std::copy(src.begin(), src.end(), std::front_inserter(dst))`.

3. **`std::inserter`**：  
   - 需要在任意位置插入元素（如合并两个有序容器）。
   - 示例：`std::set_union(a.begin(), a.end(), b.begin(), b.end(), std::inserter(result, result.begin()))`.

---

### **注意事项**
- **容器兼容性**：  
  - `std::front_inserter` 不能用于 `std::vector`（因为它没有 `push_front`）。
  - `std::inserter` 可以用于关联容器（如 `std::set`），但插入位置可能需要符合容器的排序规则。

- **迭代器失效**：  
  - 对 `std::vector` 使用 `std::inserter` 时，多次插入可能导致迭代器失效，但 `std::inserter` 内部会自动更新迭代器位置。

---

### **总结**
- 使用 `std::back_inserter` 将元素顺序添加到末尾。
- 使用 `std::front_inserter` 将元素逆序插入到头部（仅适用支持 `push_front` 的容器）。
- 使用 `std::inserter` 在任意指定位置插入元素（最灵活，但需注意性能）。

这些插入迭代器简化了容器操作，使标准算法可以直接输出到容器的不同位置，无需手动管理迭代器。