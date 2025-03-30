 `std::span` 

---

在 C++ 中使用 `std::span` 传递参数是一种高效且安全的方式，尤其适合处理连续内存数据（如数组、`std::vector`、原生指针等）。以下是关键点和示例：

### **1. 基本用法**

`std::span` 是一个轻量级的非拥有视图，用于表示连续内存序列。传递参数时，可以替代传统的 **指针 + 大小** 或 **起始/结束迭代器**。

#### **函数声明**
```cpp
#include <span>

// 接受一个 int 类型的连续序列
void process_data(std::span<int> data) {
    for (auto& elem : data) {
        // 处理每个元素
    }
}
```

#### **调用示例**
```cpp
std::vector<int> vec = {1, 2, 3};
int arr[] = {4, 5, 6};
process_data(vec);    // 直接传递 vector
process_data(arr);     // 传递数组
process_data({vec.data() + 1, 2}); // 传递子范围：{2, 3}
```

---

### **2. 传递 `const` 数据**
如果函数不修改数据，应使用 `span<const T>`：
```cpp
void read_data(std::span<const int> data) {
    for (const auto& elem : data) {
        std::cout << elem << " ";
    }
}
```

---

### **3. 按值传递 vs 按引用传递**
- **按值传递**：`std::span` 本身很小（通常为 2 个指针大小），适合按值传递。
- **按引用传递**：仅当需要修改 `span` 自身（如改变其指向的范围）时使用，但罕见。

```cpp
// 正确：按值传递
void process_data(std::span<int> data);

// 需要修改 span 自身时（不常见）
void resize_span(std::span<int>& data, size_t new_size);
```

---

### **4. 处理固定大小数据**
可以指定 `span` 的静态长度（`N`），但通常更推荐动态长度（更灵活）：
```cpp
// 接受长度为 3 的 int 数组
void process_fixed(std::span<int, 3> data);

int arr[3] = {1, 2, 3};
process_fixed(arr); // OK
```

---

### **5. 传递原生指针和大小**
将 **指针 + 大小** 转换为 `span`：
```cpp
void external_lib_func(int* ptr, size_t size) {
    process_data(std::span<int>(ptr, size));
}
```

---

### **6. 注意事项**
- **数据有效性**：`span` 不拥有数据，需确保底层数据在 `span` 使用期间有效。
- **连续内存**：仅支持连续容器（如 `std::vector`, `std::array`, 原生数组），不支持 `std::list`。
- **C++20 支持**：需编译器支持 C++20 或使用兼容库（如 Microsoft GSL）。

---

### **7. 完整示例**
```cpp
#include <iostream>
#include <span>
#include <vector>

void print(std::span<const int> data) {
    for (const auto& elem : data) {
        std::cout << elem << " ";
    }
    std::cout << "\n";
}

void square_elements(std::span<int> data) {
    for (auto& elem : data) {
        elem *= elem;
    }
}

int main() {
    std::vector<int> vec = {1, 2, 3, 4};
    int arr[] = {5, 6, 7};

    print(vec);        // 输出: 1 2 3 4
    print(arr);        // 输出: 5 6 7

    square_elements(vec);
    square_elements({arr, 3});

    print(vec);        // 输出: 1 4 9 16
    print(arr);        // 输出: 25 36 49
}
```

---

### **总结**
- 使用 `std::span` 替代指针 + 大小，提升代码可读性和安全性。
- 优先使用 `span<const T>` 表示只读数据。
- 确保数据在 `span` 生命周期内有效。