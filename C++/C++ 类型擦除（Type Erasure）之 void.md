C++ 类型擦除（Type Erasure）之 void*

这段代码展示了一种基于 **类型擦除（Type Erasure）** 的实现，用于在 C++ 中以统一的方式管理不同类型的对象，而无需依赖虚函数或继承层次结构。GenericHolder 是一个类型擦除的包装器，能够持有任何满足特定接口（Interface 概念）的对象，并提供统一的调用方式。以下是对代码的逐步解释。

代码:

```c++
#include <memory>
#include <string>
#include <utility>
#include <iostream>

// Concept representing the interface (optional):
template <typename T> concept Interface = requires (T t) {
  { t.operation() } -> std::same_as<int>;
};

// Owning variant of a generic holder
struct GenericHolder {
  // Only the constructor is specific to each type:
  template<Interface T>
  GenericHolder(std::unique_ptr<T> ptr) {
    // operation_ and destroy_ remember the type
    operation_ = [](void* blob) {
      return static_cast<T*>(blob)->operation(); 
    };
    destroy_ = [](void* blob) {
      delete static_cast<T*>(blob); 
    };
    blob_ = ptr.release();
  }
  ~GenericHolder() {
    if (blob_) destroy_(this->blob_);
  }

  // Move only (can be made copyable by addition of a clone_ fp)
  GenericHolder(const GenericHolder&) = delete;
  GenericHolder& operator=(const GenericHolder&) = delete;

  // Move operations
  GenericHolder(GenericHolder&& other)
      : blob_(std::exchange(other.blob_, nullptr)),
        operation_(std::exchange(other.operation_, nullptr)),
        destroy_(std::exchange(other.destroy_, nullptr)) {}
  GenericHolder& operator=(GenericHolder&& other) {
      if (blob_) destroy_(blob_);
      blob_ = std::exchange(other.blob_, nullptr);
      operation_ = std::exchange(other.operation_, nullptr);
      destroy_ = std::exchange(other.destroy_, nullptr);
      return *this;
  }

  // Actual interface
  int operation() { return operation_(this->blob_); }

private:
  // Generic storage, note that adding a new operation breaks ABI
  void *blob_;
  int (*operation_)(void*);
  void (*destroy_)(void*);
};

// Implementations are unrelated and have no virtual methods
struct ImplA {
    int operation() { return rank; }
    int rank;
};

struct ImplB {
    int operation() { return std::stoi(text); }
    std::string text;
};

void user(GenericHolder data) {
    int v = data.operation();
    std::cout << v << "\n";
}

int main() {
    user(GenericHolder(std::make_unique<ImplA>(10))); // OK, prints 10
    user(GenericHolder(std::make_unique<ImplB>("42"))); // OK, prints 42
}
```



------

**代码分解**

**1. 接口概念（Interface）**

cpp

```cpp
template <typename T>
concept Interface = requires (T t) {
    { t.operation() } -> std::same_as<int>;
};
```

- **含义**: 定义一个概念 Interface，要求类型 T 必须有一个无参数的成员函数 operation()，且返回类型为 int。
- **作用**: 用于约束 GenericHolder 接受的类型，确保它们支持 operation() 接口。
- **可选性**: 注释中提到这是可选的，因为即使不使用概念，模板构造函数也能通过编译时检查实现类似约束。

------

**2. GenericHolder 类型擦除类**

GenericHolder 是核心类，负责擦除具体类型并提供统一接口。

**成员变量**

cpp

```cpp
private:
    void *blob_;
    int (*operation_)(void*);
    void (*destroy_)(void*);
```

- **blob_**: 一个 void* 指针，存储具体实现对象的地址，隐藏了类型信息。
- **operation_**: 函数指针，指向一个接受 void* 并返回 int 的函数，用于调用底层对象的 operation()。
- **destroy_**: 函数指针，指向一个接受 void* 并销毁对象的函数，用于清理资源。

**构造函数**

cpp

```cpp
template<Interface T>
GenericHolder(std::unique_ptr<T> ptr) {
    operation_ = [](void* blob) {
        return static_cast<T*>(blob)->operation();
    };
    destroy_ = [](void* blob) {
        delete static_cast<T*>(blob);
    };
    blob_ = ptr.release();
}
```

- **参数**: 接受一个 std::unique_ptr<T>，其中 T 必须满足 Interface 概念。
- **初始化**:
  - **operation_**: 赋值为一个 lambda，捕获类型 T，将 void* 转换为 T* 并调用其 operation()。
  - **destroy_**: 赋值为一个 lambda，负责删除 T* 类型的对象。
  - **blob_**: 从 std::unique_ptr 释放原始指针并存储。
- **类型擦除**: 通过将具体类型 T 的操作封装为函数指针，隐藏了 T 的具体信息。

**析构函数**

cpp

```cpp
~GenericHolder() {
    if (blob_) destroy_(this->blob_);
}
```

- **作用**: 如果 blob_ 不为空，调用 destroy_ 函数释放资源，确保没有内存泄漏。

**移动语义**

cpp

```cpp
GenericHolder(GenericHolder&& other)
    : blob_(std::exchange(other.blob_, nullptr)),
      operation_(std::exchange(other.operation_, nullptr)),
      destroy_(std::exchange(other.destroy_, nullptr)) {}

GenericHolder& operator=(GenericHolder&& other) {
    if (blob_) destroy_(blob_);
    blob_ = std::exchange(other.blob_, nullptr);
    operation_ = std::exchange(other.operation_, nullptr);
    destroy_ = std::exchange(other.destroy_, nullptr);
    return *this;
}
```

- **移动构造**: 将 other 的资源（blob_、operation_、destroy_）转移过来，并将 other 置为空。
- **移动赋值**: 先清理当前资源，然后接管 other 的资源。
- **std::exchange**: 转移值并将原值设为 nullptr，确保移动后 other 处于有效状态。
- **不可拷贝**: 禁用了拷贝构造函数和赋值运算符（可通过添加 clone_ 函数支持拷贝）。

**接口**

cpp

```cpp
int operation() { return operation_(this->blob_); }
```

- **作用**: 调用存储的 operation_ 函数指针，传递 blob_，实现统一的操作接口。

------

**3. 实现类**

cpp

```cpp
struct ImplA {
    int operation() { return rank; }
    int rank;
};

struct ImplB {
    int operation() { return std::stoi(text); }
    std::string text;
};
```

- **ImplA**: 实现 operation()，返回一个整数成员 rank。
- **ImplB**: 实现 operation()，将字符串 text 转换为整数并返回。
- **特点**: 这两个类没有继承关系，也没有虚函数，但都满足 Interface 概念。

------

**4. 使用示例**

cpp

```cpp
void user(GenericHolder data) {
    int v = data.operation();
    std::cout << v << "\n";
}

user(GenericHolder(std::make_unique<ImplA>(10))); // 输出 10
user(GenericHolder(std::make_unique<ImplB>("42"))); // 输出 42
```

- **user 函数**: 接受 GenericHolder，调用其 operation() 并打印结果。
- **调用**:
  - ImplA(10): 创建一个 rank = 10 的对象，operation() 返回 10。
  - ImplB("42"): 创建一个 text = "42" 的对象，operation() 返回 42。
- **结果**: 类型被擦除后，user 无需知道具体类型即可操作。

------

**工作原理**

1. **构造时**:
   - GenericHolder 接受 std::unique_ptr<T>，将 T 的操作封装为函数指针。
   - blob_ 存储原始指针，operation_ 和 destroy_ 记住如何操作和销毁。
2. **调用时**:
   - operation() 通过函数指针调用底层对象的 operation()。
3. **销毁时**:
   - 析构函数调用 destroy_ 释放资源。

------

**类型擦除的特点**

- **无虚函数**: 实现类（如 ImplA、ImplB）无需继承或使用虚函数。
- **函数指针**: 使用函数指针代替虚表，减少虚函数调用的间接性。
- **移动语义**: 支持高效的资源转移，但默认不可拷贝。
- **ABI 稳定性**: 添加新操作（如新的函数指针）会破坏二进制兼容性（注释中提到）。

------

**中文解释**

**概念**

- **Interface**: 定义一个接口，要求类型有返回 int 的 operation()。
- **GenericHolder**: 类型擦除的持有者，统一管理不同实现。

**实现**

- **构造**: 用模板接受具体类型，封装其操作和销毁逻辑。
- **存储**: 用 void* 存储对象，用函数指针存储行为。
- **移动**: 支持资源转移，避免拷贝（可扩展为支持拷贝）。

**使用**

- **ImplA 和 ImplB**: 两个独立类型，满足接口但无继承关系。
- **user**: 通过 GenericHolder 调用 operation()，无需关心具体类型。

**输出**

- ImplA(10) 输出 10。
- ImplB("42") 输出 42。

------

**优缺点**

**优点**

1. **灵活性**: 支持任何满足接口的类型，无需继承。
2. **性能**: 避免虚表，使用函数指针可能更高效。
3. **资源管理**: 通过 std::unique_ptr 确保安全释放。

**缺点**

1. **不可拷贝**: 默认只支持移动（需额外实现 clone_ 支持拷贝）。
2. **扩展性**: 添加新操作需要修改 GenericHolder，破坏 ABI。
3. **复杂性**: 实现和管理比传统多态更复杂。

------

**总结**

这段代码展示了一种轻量级的类型擦除实现，适用于需要统一接口但不想依赖虚函数的场景。它通过函数指针和模板结合，实现了运行时多态，同时保持编译时类型安全。如果你有进一步问题或想扩展功能（如支持拷贝），请告诉我！