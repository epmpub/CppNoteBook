# C++17 static_pointer_cast, dynamic_pointer_cast, const_pointer_cast, reinterpret_pointer_cast

Four C++17 cast-style functions that can convert the pointed-to type of std::shared_ptr without affecting ownership.

When handling smart pointers, we might need to change the pointed-to type without changing the pointed-to object.

With *std::unique_ptr*, this is fairly straightforward. However, if you try to do the same with *std::shared_ptr*, you need to be careful to maintain the shared ownership.

C++17 simplified this process by introducing four cast-style functions:

- *std::static_pointer_cast*
- *std::dynamic_pointer_cast*
- *std::const_pointer_cast*
- *std::reinterpret_pointer_cast*

```C++
#include <memory>
#include <print>
#include <type_traits>

struct File {
    int fd_;
};

struct Base {
    virtual ~Base() = default;
};
struct Derived : Base {
    ~Derived() override = default;
};

int main() {
    // const_cast
    auto ptr = std::make_shared<int>(42);
    // decltype(ptr) == std::shared_ptr<int>
    // ptr.use_count() == 1

    std::println("ptr.use_count() == {}\n", ptr.use_count());
    static_assert(std::is_same_v<decltype(ptr), std::shared_ptr<int>>);

    // A new instance sharing the same ownership,
    // but pointing to const int*
    auto cv1 = std::const_pointer_cast<const int>(ptr); // C++17
    // decltype(cv1) == std::shared_ptr<const int>
    // ptr.use_count() == cv1.use_count() == 2

    std::println("ptr.use_count() == {}, cv1.use_count() == {}\n", ptr.use_count(), cv1.use_count());
    static_assert(std::is_same_v<decltype(cv1), std::shared_ptr<const int>>);

    // Consuming conversion
    auto cv2 = std::const_pointer_cast<int>(std::move(cv1)); // C++20
    // decltype(cv2) == std::shared_ptr<int>
    // ptr.use_count() == 2, cv2.use_count() == 2
    // cv1 == nullptr

    std::println("ptr.use_count() == {}, cv2.use_count() == {}", ptr.use_count(), cv2.use_count());
    std::println("(cv1 == nullptr) == {}\n", cv1 == nullptr);
    static_assert(std::is_same_v<decltype(cv2), std::shared_ptr<int>>);

    // reinterpret_cast
    auto file = std::make_shared<File>(42);
    // decltype(file) == std::shared_ptr<File>

    std::println("file.use_count() == {}\n", file.use_count());
    static_assert(std::is_same_v<decltype(file), std::shared_ptr<File>>);

    auto fd = std::reinterpret_pointer_cast<int>(file);
    // decltype(fd) == std::shared_ptr<int>
    // file.use_count() == fd.use_count() == 2
    // *fd == 42

    std::println("file.use_count() == {}, fd.use_count() == {}", file.use_count(), fd.use_count());
    std::println("*fd == {}\n", *fd);
    static_assert(std::is_same_v<decltype(fd), std::shared_ptr<int>>);

    // static_cast
    auto derived = std::make_shared<Derived>();
    // decltype(derived) == std::shared_ptr<Derived>
    
    static_assert(std::is_same_v<decltype(derived), std::shared_ptr<Derived>>);

    auto base = std::static_pointer_cast<Base>(std::move(derived));
    // decltype(base) == std::shared_ptr<Base>
    // base.use_count() == 1, derived == nullptr

    std::println("base.use_count() == {}", base.use_count());
    std::println("(derived == nullptr) == {}\n", derived == nullptr);
    static_assert(std::is_same_v<decltype(base), std::shared_ptr<Base>>);

    // dynamic_cast
    derived = std::dynamic_pointer_cast<Derived>(base);
    // derived.use_count() == 2, base.use_count() == 2

    std::println("derived.use_count() == {}, base.use_count() == {}", derived.use_count(), base.use_count());
}
```

我来详细解释这段 C++ 代码，它展示了智能指针 std::shared_ptr 的各种类型转换操作，包括 const_cast、reinterpret_cast、static_cast 和 dynamic_cast。

------

代码结构

这段代码使用以下头文件：

- <memory>：提供智能指针 std::shared_ptr
- <print>：提供 std::println 用于格式化输出（C++23 特性）
- <type_traits>：提供 std::is_same_v 用于类型检查

定义了两个简单的结构体：

- File：包含一个整数 fd_
- Base 和 Derived：展示继承关系，Base 是虚基类，Derived 是派生类

------

1. const_cast（通过 std::const_pointer_cast）

示例代码

cpp

```cpp
auto ptr = std::make_shared<int>(42);  // 创建指向 int 的共享指针
auto cv1 = std::const_pointer_cast<const int>(ptr);  // 转换为 const int
auto cv2 = std::const_pointer_cast<int>(std::move(cv1));  // 从 const int 转换回 int
```

解释

- **初始状态**：ptr 是一个 std::shared_ptr<int>，指向值为 42 的整数，使用计数为 1。
- **std::const_pointer_cast<const int>**：
  - 将 ptr 转换为 std::shared_ptr<const int> 类型。
  - 两个指针共享同一对象的所有权，使用计数变为 2。
  - cv1 指向的整数是只读的（const）。
- **std::const_pointer_cast<int>（带 move）**：
  - 从 cv1（const int）转换回非 const 的 int。
  - 使用 std::move 转移 cv1 的所有权，之后 cv1 变为 nullptr。
  - ptr 和 cv2 共享所有权，使用计数仍为 2。

输出

```text
ptr.use_count() == 1
ptr.use_count() == 2, cv1.use_count() == 2
ptr.use_count() == 2, cv2.use_count() == 2
(cv1 == nullptr) == true
```

------

2. reinterpret_cast（通过 std::reinterpret_pointer_cast）

示例代码

cpp

```cpp
auto file = std::make_shared<File>(42);  // 创建指向 File 的共享指针
auto fd = std::reinterpret_pointer_cast<int>(file);  // 转换为指向 int 的指针
```

解释

- **初始状态**：file 是一个 std::shared_ptr<File>，File 包含一个整数成员 fd_，初始化为 42。
- **std::reinterpret_pointer_cast<int>**：
  - 将 file 的指针重新解释为 std::shared_ptr<int>。
  - 两个指针共享同一块内存的所有权，使用计数变为 2。
  - fd 指向的内存被当作 int 解析，*fd 输出 fd_ 的值 42。
- **注意**：这种转换假设内存布局兼容（这里依赖 File 的第一个成员是 int），否则可能是未定义行为。

输出

```text
file.use_count() == 1
file.use_count() == 2, fd.use_count() == 2
*fd == 42
```

------

3. static_cast（通过 std::static_pointer_cast）

示例代码

cpp

```cpp
auto derived = std::make_shared<Derived>();  // 创建指向 Derived 的共享指针
auto base = std::static_pointer_cast<Base>(std::move(derived));  // 转换为指向 Base
```

解释

- **初始状态**：derived 是一个 std::shared_ptr<Derived>，指向 Derived 对象。
- **std::static_pointer_cast<Base>**：
  - 将 derived 转换为 std::shared_ptr<Base>（向上转型）。
  - 使用 std::move 转移所有权，derived 变为 nullptr。
  - base 独自拥有对象，使用计数为 1。
- **特点**：适用于已知类型关系的静态转换，不涉及运行时检查。

输出

```text
base.use_count() == 1
(derived == nullptr) == true
```

------

4. dynamic_cast（通过 std::dynamic_pointer_cast）

示例代码

cpp

```cpp
derived = std::dynamic_pointer_cast<Derived>(base);  // 从 Base 转换回 Derived
```

解释

- **初始状态**：base 是一个 std::shared_ptr<Base>，实际指向 Derived 对象。
- **std::dynamic_pointer_cast<Derived>**：
  - 将 base 转换为 std::shared_ptr<Derived>（向下转型）。
  - 运行时检查类型是否匹配（需要多态类型，即虚函数）。
  - 成功后，base 和 derived 共享所有权，使用计数变为 2。
- **特点**：安全的动态类型转换，若类型不匹配则返回 nullptr。

输出

```text
derived.use_count() == 2, base.use_count() == 2
```

------

关键概念总结

1. **std::const_pointer_cast**：
   - 用于在 const 和非 const 之间转换。
   - 不改变底层对象，仅改变指针类型。
2. **std::reinterpret_pointer_cast**：
   - 重新解释内存，适用于低级操作，小心未定义行为。
3. **std::static_pointer_cast**：
   - 静态类型转换，适用于已知的继承关系，无运行时开销。
4. **std::dynamic_pointer_cast**：
   - 动态类型转换，适用于多态类型，运行时检查安全性。

注意事项

- 所有这些转换保留了 std::shared_ptr 的引用计数机制。
- 使用 std::move 可以转移所有权，避免不必要的拷贝。
- static_assert 用于编译期验证类型正确性。

这段代码展示了 C++ 中智能指针类型转换的强大功能，适用于需要灵活管理对象所有权和类型的场景。





## 2 信息完整性

在你的代码中：

cpp

```cpp
derived = std::dynamic_pointer_cast<Derived>(base);  // 从 Base 转换回 Derived
```

信息是否会丢失取决于具体情况，但从你提供的完整代码上下文来看，**在这个例子中信息不会丢失**。让我详细解释原因，以及在什么情况下可能会丢失信息。

------

分析代码上下文

1. **初始状态**：
   - base 是一个 std::shared_ptr<Base>，它是通过 std::static_pointer_cast<Base>(std::move(derived)) 从一个 std::shared_ptr<Derived> 转换来的。
   - 实际上，base 管理的对象是一个 Derived 类型的实例，因为它最初是由 std::make_shared<Derived>() 创建的。
   - Base 是一个多态基类（因为它有虚析构函数 virtual ~Base() = default），Derived 是其派生类。
2. **std::dynamic_pointer_cast<Derived>(base)**：
   - std::dynamic_pointer_cast 用于将 std::shared_ptr<Base> 动态转换为 std::shared_ptr<Derived>。
   - 由于 base 实际上指向一个 Derived 对象，转换会成功，derived 将获得一个新的 std::shared_ptr<Derived>，它指向同一个对象。
   - 使用计数从 1 增加到 2，因为 base 和 derived 现在共享同一对象的所有权。
3. **结果**：
   - 转换前后，底层对象始终是同一个 Derived 实例。
   - std::dynamic_pointer_cast 通过运行时类型信息（RTTI）检查确保 base 指向的对象确实是 Derived 或其派生类，因此不会丢失信息。

------

信息是否丢失？

在这个特定场景中，**信息不会丢失**，原因如下：

- **对象完整性**：base 和 derived 指向的底层对象没有改变，只是通过不同的类型（Base* 或 Derived*）访问它。
- **多态支持**：由于 Base 是多态类（有虚函数），std::dynamic_pointer_cast 能正确识别对象的实际类型并返回适当的指针。
- **数据保留**：Derived 的所有成员数据和虚函数表在转换前后都保持不变。你可以通过 derived 访问到完整的 Derived 对象，而不仅仅是 Base 的部分。

输出验证

代码中的输出：

```text
derived.use_count() == 2, base.use_count() == 2
```

表明 base 和 derived 共享同一个对象，且转换成功。

------

什么时候会丢失信息？

尽管在这个例子中信息不会丢失，但在其他情况下使用 std::dynamic_pointer_cast 可能会导致“信息丢失”或转换失败：

1. **对象不是目标类型**：

   - 如果 base 指向的不是 Derived 或其派生类，而是 Base 或其他不相关的派生类，std::dynamic_pointer_cast<Derived> 会失败，返回一个空指针（nullptr）。

   - 示例：

     cpp

     ```cpp
     auto base = std::make_shared<Base>();  // 纯 Base 对象
     auto derived = std::dynamic_pointer_cast<Derived>(base);
     // derived == nullptr，因为 base 不是 Derived 类型
     ```

   - 在这种情况下，无法访问 Derived 特有的信息，因为对象本身就不包含这些信息。

2. **切片问题（非智能指针相关）**：

   - 如果直接将 Derived 对象赋值给 Base（而不是通过指针或智能指针），会发生对象切片（slicing），丢失派生类信息。但你的代码使用的是智能指针，不会发生这种情况。

3. **类型信息丢失**：

   - 如果基类没有虚函数（即非多态类），std::dynamic_pointer_cast 将无法工作，因为它依赖 RTTI。你的代码中 Base 有虚析构函数，所以这一点不适用。

------

结论

在你的代码中，derived = std::dynamic_pointer_cast<Derived>(base) **不会丢失信息**，因为：

- base 实际上指向一个 Derived 对象。
- std::dynamic_pointer_cast 成功恢复了对完整 Derived 对象的访问。
- 底层对象的状态和数据在整个过程中保持不变。

如果你通过 derived 访问对象的成员或调用虚函数，你会得到与原始 Derived 对象完全一致的结果。如果有具体的使用场景或代码想进一步讨论，可以告诉我，我会帮你分析！