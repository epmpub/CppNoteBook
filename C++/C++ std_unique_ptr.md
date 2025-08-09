# std::unique_ptr

代码:

```c++
#include <memory>

struct Data{};

// Function returning a unique_ptr handing off ownership to caller.
std::unique_ptr<Data> producer() { return std::make_unique<Data>(); }

// Function accepting a unique_ptr taking over ownership.
void consumer(std::unique_ptr<Data> data) {}

// Helps with Single Reponsibility Principle
// by separating resource management from logic
struct Holder {
    Holder() : data_{std::make_unique<Data>()} {}
    // implicitly defaulted move constructor && move assignment
    // implicitly deleted copy constructor && copy assignment
private:
    std::unique_ptr<Data> data_;
};

// shared_ptr has a fast constructor from unique_ptr
std::shared_ptr<Data> sptr = producer();

// Even in cases when manual resource management is required,
// a unique_ptr on the interface might be preferable:
void manual_handler(std::unique_ptr<Data> ptr) {
    Data* raw = ptr.release();
    // manual resource management
}

int main() {}
```



------

1. 基础定义

cpp

```cpp
#include <memory>

struct Data{};
```

- <memory> 是 C++ 标准库头文件，提供了智能指针（如 std::unique_ptr 和 std::shared_ptr）和相关工具。
- struct Data{} 定义了一个空的结构体 Data，这里只是作为示例，实际中可能包含数据成员或方法。

------

2. producer 函数

cpp

```cpp
std::unique_ptr<Data> producer() { return std::make_unique<Data>(); }
```

- **功能**: 这个函数创建一个 Data 对象的实例，并将其所有权封装在 std::unique_ptr<Data> 中返回给调用者。
- **std::make_unique**: C++14 引入的工具函数，安全高效地创建 std::unique_ptr。它在内部使用 new 分配内存并构造对象，避免了直接使用 new 的潜在异常安全问题。
- **所有权转移**: 返回时，std::unique_ptr 的所有权从函数内部转移到调用者。由于 std::unique_ptr 是独占所有权的智能指针，对象的所有权只能有一个持有者。

------

3. consumer 函数

cpp

```cpp
void consumer(std::unique_ptr<Data> data) {}
```

- **功能**: 这个函数接受一个 std::unique_ptr<Data> 参数，接管其所有权。

- **所有权转移**: 调用时，std::unique_ptr 的所有权从调用者转移到 consumer 函数的参数 data。当 consumer 函数结束时，data 超出作用域，std::unique_ptr 会自动释放其管理的 Data 对象（调用 delete）。

- **用法示例**:

  cpp

  ```cpp
  auto ptr = producer(); // 获取 unique_ptr
  consumer(std::move(ptr)); // 转移所有权给 consumer，ptr 变为空
  ```

------

4. Holder 结构体

cpp

```cpp
struct Holder {
    Holder() : data_{std::make_unique<Data>()} {}
    // implicitly defaulted move constructor && move assignment
    // implicitly deleted copy constructor && copy assignment
private:
    std::unique_ptr<Data> data_;
};
```

- **功能**: Holder 是一个封装了资源管理的类，展示了单一职责原则（Single Responsibility Principle, SRP）。

- **成员**:

  - data_: 一个 std::unique_ptr<Data>，管理一个 Data 对象的生命周期。
  - 构造函数使用 std::make_unique 初始化 data_。

- **所有权特性**:

  - **移动语义**: 因为 std::unique_ptr 不允许复制，Holder 的拷贝构造函数和拷贝赋值操作符被隐式删除（deleted），而移动构造函数和移动赋值操作符是默认实现的（defaulted）。
  - 这意味着 Holder 对象可以被移动（转移所有权），但不能被拷贝。

- **单一职责**: Holder 只负责管理 Data 的生命周期，逻辑部分可以交给其他类，从而分离资源管理和业务逻辑。

- **用法示例**:

  cpp

  ```cpp
  Holder h1;           // 创建 Holder 对象，内部有 unique_ptr<Data>
  Holder h2 = std::move(h1); // 移动 h1 到 h2，h1.data_ 变为空
  ```

------

5. 从 unique_ptr 到 shared_ptr

cpp

```cpp
std::shared_ptr<Data> sptr = producer();
```

- **功能**: 将 producer() 返回的 std::unique_ptr<Data> 转换为 std::shared_ptr<Data>。
- **转换机制**: std::shared_ptr 提供了一个从 std::unique_ptr 构造的构造函数。这个过程是高效的，因为它直接接管 std::unique_ptr 的所有权，而不是重新分配内存。
- **所有权变化**:
  - std::unique_ptr 是独占所有权，转换为 std::shared_ptr 后变为共享所有权。
  - producer() 返回的临时 std::unique_ptr 被移动到 sptr，之后 sptr 管理这个资源。
- **为什么这样做**:
  - 当需要多个对象共享同一资源时，std::shared_ptr 比 std::unique_ptr 更合适，因为它支持引用计数。
- **底层实现**: std::shared_ptr 会接管 std::unique_ptr 的裸指针，并初始化引用计数为 1。

------

6. manual_handler 函数

cpp

```cpp
void manual_handler(std::unique_ptr<Data> ptr) {
    Data* raw = ptr.release();
    // manual resource management
}
```

- **功能**: 这个函数展示如何从 std::unique_ptr 中释放裸指针以进行手动资源管理。

- **release()**:

  - std::unique_ptr::release() 放弃对资源的控制权，返回其管理的裸指针，并将 std::unique_ptr 置为空。
  - 与 delete 不同，release() 不会删除对象，调用者需要手动管理释放（例如调用 delete raw）。

- **使用场景**:

  - 当需要将资源传递给不支持智能指针的遗留代码时。
  - 或在特定情况下需要手动控制资源释放。

- **注意**:

  - 调用 release() 后，ptr 变为空，原始资源的所有权完全交给 raw。
  - 如果忘记手动释放 raw，会导致内存泄漏。

- **用法示例**:

  cpp

  ```cpp
  auto ptr = producer();
  manual_handler(std::move(ptr)); // 转移所有权，内部释放为 raw
  ```

------

总结与关键点

1. **std::unique_ptr**:
   - 表示独占所有权，适合资源管理的单一持有者场景。
   - 通过 std::make_unique 创建，避免直接使用 new。
   - 支持移动语义，不支持拷贝。
2. **std::shared_ptr**:
   - 表示共享所有权，适合多方需要访问同一资源的场景。
   - 可以从 std::unique_ptr 高效构造。
3. **所有权转移**:
   - producer -> consumer: 通过 std::move 转移 unique_ptr。
   - unique_ptr -> shared_ptr: 通过构造函数直接转换。
4. **单一职责原则**:
   - Holder 类将资源管理与逻辑分离，std::unique_ptr 确保资源安全。
5. **手动管理**:
   - release() 提供从智能指针到裸指针的过渡，但需要小心内存泄漏。

这段代码很好地展示了 C++ 智能指针的灵活性和资源管理能力。如果你有更具体的问题或需要进一步示例，请告诉我！