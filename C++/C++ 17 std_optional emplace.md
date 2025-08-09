# C++ 17 std::optional emplace

```C++
    std::optional<std::string> opt;
    opt.emplace(5, 'a');
    //print opt
    std::cout << opt.value() << std::endl;
    return 0;
```



这段代码使用了 C++17 引入的 `std::optional` 类型。我来解释每一行：

```cpp
std::optional<std::string> opt;
```

这行创建了一个 `std::optional` 对象 `opt`，它可以包含一个 `std::string` 类型的值或者不包含任何值（为空）。初始状态下，`opt` 为空。

```cpp
opt.emplace(5, 'a');
```

这行使用 `emplace()` 方法在 `opt` 内部原地构造一个 `std::string` 对象。这里调用的是 `std::string` 的构造函数，参数为 `(5, 'a')`，这会创建一个包含 5 个字符 'a' 的字符串。换句话说，这等价于 `std::string(5, 'a')`，结果是字符串 `"aaaaa"`。

`emplace()` 方法的好处是直接在 `optional` 对象内部构造目标对象，而不需要先构造一个临时的 `std::string` 然后再复制或移动到 `optional` 中。

```cpp
std::cout << opt.value() << std::endl;
```

这行代码使用 `value()` 方法获取 `optional` 中存储的值并打印出来。如果此时 `opt` 为空（没有值），调用 `value()` 会抛出 `std::bad_optional_access` 异常。在这个例子中，由于前面已经通过 `emplace()` 设置了值，所以 `opt` 不为空，打印出的结果会是 `"aaaaa"`。

整个代码的输出会是：

```
aaaaa
```

如果想要更安全地访问 `optional` 中的值，可以使用 `opt.value_or(默认值)` 或检查 `opt.has_value()` 后再访问。





# C++ 中的 emplace 方法总结

C++ 中的 `emplace` 系列方法主要用于容器类，它们的共同点是允许就地构造元素，避免创建临时对象，从而提高性能。以下是 C++ 标准库中支持 `emplace` 操作的主要组件：

## 标准容器

### 顺序容器

- **`std::vector`**: `emplace_back()`, `emplace()`
- **`std::deque`**: `emplace_back()`, `emplace_front()`, `emplace()`
- **`std::list`**: `emplace_back()`, `emplace_front()`, `emplace()`
- **`std::forward_list`** (C++11): `emplace_after()`, `emplace_front()`

### 关联容器

- **`std::set`/`std::multiset`**: `emplace()`, `emplace_hint()`
- **`std::map`/`std::multimap`**: `emplace()`, `emplace_hint()`
- **`std::unordered_set`/`std::unordered_multiset`**: `emplace()`, `emplace_hint()`
- **`std::unordered_map`/`std::unordered_multimap`**: `emplace()`, `emplace_hint()`

### 容器适配器

- **`std::stack`**: `emplace()`
- **`std::queue`**: `emplace()`
- **`std::priority_queue`**: `emplace()`

## 工具类

- **`std::optional`** (C++17): `emplace()`
- **`std::variant`** (C++17): `emplace<Index>()`, `emplace<Type>()`
- **`std::any`** (C++17): `emplace<Type>()`
- **`std::pair`**: 没有 `emplace` 方法，但可以通过 `std::piecewise_construct` 实现类似效果
- **`std::tuple`**: 没有 `emplace` 方法，但构造时支持原地构造

## 智能指针

- **`std::unique_ptr`**: `reset(T* ptr)` (不叫 emplace 但有类似功能)
- **`std::shared_ptr`**: `reset(T* ptr)` (不叫 emplace 但有类似功能)
- **`std::make_unique`/`std::make_shared`**: 工厂函数，实现类似 emplace 的原地构造

## 算法

标准算法库中没有直接称为 `emplace` 的算法，但有一些算法支持就地构造的概念：

- **`std::transform`**: 可以将结果直接构造到目标容器
- **`std::generate`**: 可以生成新元素

## 字符串

标准的 `std::string` 和 `std::wstring` 有以下 emplace 相关方法：

- **`std::basic_string`**: `emplace()`, `emplace_back()`

## C++20 新增

- **`std::span`**: 无 emplace 方法（它是非拥有型容器）
- **`std::format`**: 无 emplace 方法

## 特点对比

所有这些 `emplace` 方法的共同特点是：

1. 直接在容器内存中构造对象
2. 避免不必要的临时对象创建和复制/移动操作
3. 通常接受可变参数模板，将参数完美转发给对象构造函数

`emplace` 相比于 `insert` 等方法的优势在高性能场景尤为明显，特别是当存储的对象构造成本较高或移动成本较高时。