**C++26 `std::indirect`** 是通过提案 **P3019R14** 引入的新类型，定义在 `<memory>` 头文件中。它属于 **“复合类设计词汇类型”（Vocabulary Types for Composite Class Design）** 的一部分（另一个是 `std::polymorphic`）。

### 核心作用

`std::indirect<T, Allocator = std::allocator<T>>` 是一个**值语义（value-like semantics）的包装器**，内部持有一个**动态分配**的 `T` 对象。

- 复制 `indirect` 时，会**深度复制**内部的 `T` 对象（而不是复制指针）。
- 移动 `indirect` 时，转移所有权（移动后处于 *valueless* 状态）。
- `const` 传播：通过 `const indirect` 访问时，内部对象也是 `const` 的。
- 提供类似普通 `T` 的值语义，但存储在堆上（适合大对象、不可复制或需要多态行为的成员）。

它常用于**复合类**（composite classes）中，作为成员变量，避免值语义大对象拷贝或手动管理指针。

### 基本用法

```cpp
#include <memory>
#include <string>
#include <iostream>

struct BigData {
    std::string content = "hello";
};

class MyClass {
    std::indirect<BigData> data_;

public:
    MyClass() : data_(std::in_place) {}   // 默认构造内部 BigData

    void modify() {
        data_->content += " world";        // 正确使用 ->
        std::cout << data_->content << '\n';
    }

    const std::string& get_content() const {
        return data_->content;
    }
};

int main() {
    MyClass obj;
    obj.modify();                    // 正确
    // obj->modify();                // 错误！会报你遇到的错误
}
```

### 关键特性

- **构造**：
  - `indirect()`：默认构造（使用 `T` 的默认构造）。
  - `indirect(std::in_place, args...)`：带参数构造 `T`。
  - 支持 allocator-aware 构造。

- **访问**：
  - `operator->` / `operator*`：访问内部对象。
  - `valueless_after_move()`：检查是否已被移动（valueless 状态）。

- **比较**：支持 `==`、`!=`、`<=` 等（比较内部 `T`）。

- **pmr 版本**：
  ```cpp
  std::pmr::indirect<BigData> pmr_data;  // 使用 polymorphic_allocator
  ```

### 与其它类型的对比

| 类型                   | 语义          | 存储位置 | 复制行为     | 典型用途                 |
| ---------------------- | ------------- | -------- | ------------ | ------------------------ |
| `T`                    | 值语义        | 栈/成员  | 深度复制     | 小对象                   |
| `std::unique_ptr<T>`   | 独占所有权    | 堆       | 不可复制     | 独占动态对象             |
| `std::shared_ptr<T>`   | 共享所有权    | 堆       | 浅复制       | 共享所有权               |
| **`std::indirect<T>`** | **值语义**    | 堆       | **深度复制** | **复合类中的大对象成员** |
| `std::polymorphic<T>`  | 值语义 + 多态 | 堆       | 深度复制     | 多态值成员（类型擦除）   |

### 注意事项

- 移动后对象变为 *valueless*，此时访问 `operator*` / `operator->` 是未定义行为（需先检查 `valueless_after_move()`）。
- `T` 不能是数组、cv-qualified 类型、`in_place_t` 等。
- `T` 可以是不完整类型（incomplete type），便于前向声明。
- 几乎所有操作都支持 `constexpr`（在支持的环境下）。

### Feature Test Macro

```cpp
#if __cpp_lib_indirect >= 202502L
// 支持 std::indirect
#endif
```

**提案**：P3019R14  
**cppreference**：[`std::indirect`](https://en.cppreference.com/cpp/memory/indirect)

`std::indirect` 和 `std::polymorphic` 是 C++26 中非常实用的“现代 C++ 复合类设计”工具，显著减少了手动管理动态成员的 boilerplate 代码，尤其适合需要值语义但又希望堆分配的场景。

需要完整构造/赋值示例、与 `std::polymorphic` 的对比，还是在复合类中的实际应用场景？随时告诉我！