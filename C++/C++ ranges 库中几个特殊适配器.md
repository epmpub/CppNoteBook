C++ Ranges 库中几个特殊适配器 

---

以下是关于 C++ Ranges 库中几个特殊适配器 **`owning_view`**、**`ref_view`**、**`views::all`** 和 **`views::common`** 的详细说明及示例代码。这些适配器用于管理范围的生命周期、引用或兼容性。

### **1. `ref_view`**

- **作用**：将范围包装为一个非拥有（non-owning）的引用视图，确保范围在使用期间有效。
- **特点**：
  - 不持有数据所有权，仅引用原始范围。
  - 必须确保原始范围的声明周期长于 `ref_view`。
- **示例**：
  ```cpp
  #include <ranges>
  #include <vector>
  #include <iostream>
  
  int main() {
      std::vector<int> vec = {1, 2, 3};
  
      // 创建对 vec 的引用视图
      std::ranges::ref_view ref_vec = std::views::all(vec);
  
      // 修改原始数据会影响视图
      vec.push_back(4);
  
      for (int x : ref_vec) {
          std::cout << x << " "; // 输出: 1 2 3 4
      }
  }
  ```

---

### **2. `owning_view`**
- **作用**：持有范围的独占所有权（类似 `std::unique_ptr`），用于管理临时或需要延长生命周期的范围。
- **特点**：
  - 拥有数据所有权，可以安全传递。
  - 适用于右值范围（如临时容器）。
- **示例**：
  ```cpp
  #include <ranges>
  #include <vector>
  #include <iostream>
  
  int main() {
      // 临时 vector 被移动到 owning_view
      auto owned = std::ranges::owning_view(std::vector{1, 2, 3});
  
      // 即使原始临时对象销毁，owned 仍有效
      for (int x : owned) {
          std::cout << x << " "; // 输出: 1 2 3
      }
  }
  ```

---

### **3. `views::all`**
- **作用**：自动为范围生成安全的视图（`ref_view` 或 `owning_view`）。
- **特点**：
  - 对左值返回 `ref_view`。
  - 对右值返回 `owning_view`。
- **示例**：
  ```cpp
  #include <ranges>
  #include <vector>
  #include <iostream>
  
  void process(std::ranges::view auto range) {
      for (int x : range) std::cout << x << " ";
  }
  
  int main() {
      std::vector<int> vec = {1, 2, 3};
  
      // 对左值生成 ref_view
      auto view1 = std::views::all(vec);
      process(view1); // 输出: 1 2 3
  
      // 对右值生成 owning_view
      auto view2 = std::views::all(std::vector{4, 5, 6});
      process(view2); // 输出: 4 5 6
  }
  ```

---

### **4. `views::common`**
- **作用**：将范围转换为具有“通用”迭代器的范围，兼容传统 C++ 代码（要求 `begin()` 和 `end()` 返回同一类型）。
- **场景**：需要将新式范围传递给接受旧式迭代器对的接口。
- **示例**：
  ```cpp
  #include <ranges>
  #include <vector>
  #include <algorithm> // 传统算法
  
  int main() {
      std::vector<int> vec = {3, 1, 4, 1, 5};
  
      // 生成一个传统迭代器兼容的范围
      auto common_range = vec | std::views::filter([](int x) { return x > 2; })
                              | std::views::common;
  
      // 传递给传统算法（如 std::sort）
      std::sort(common_range.begin(), common_range.end());
  
      for (int x : common_range) {
          std::cout << x << " "; // 输出: 1 3 4 5（排序后）
      }
  }
  ```

---

### **5. 联合使用示例**
```cpp
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    // 创建一个临时范围并用 owning_view 持有
    auto owned = std::views::all(std::vector{1, 2, 3});

    // 转换为通用范围以兼容旧接口
    auto common = owned | std::views::common;

    // 使用传统算法（如 std::accumulate）
    int sum = 0;
    std::for_each(common.begin(), common.end(), [&sum](int x) { sum += x; });
    std::cout << sum; // 输出: 6
}
```

---

### **总结**
| 适配器              | 作用                                 | 适用场景                          |
| ------------------- | ------------------------------------ | --------------------------------- |
| **`ref_view`**      | 非拥有引用视图                       | 需要安全引用现有范围              |
| **`owning_view`**   | 持有范围所有权                       | 管理临时或需要移动的范围          |
| **`views::all`**    | 自动选择 `ref_view` 或 `owning_view` | 简化视图生成（左值/右值自动处理） |
| **`views::common`** | 生成传统迭代器兼容的范围             | 兼容旧代码或传统算法              |

- **生命周期**：优先使用 `views::all` 简化代码，显式使用 `ref_view` 或 `owning_view` 控制所有权。
- **兼容性**：`views::common` 是连接 Ranges 和传统代码的桥梁。