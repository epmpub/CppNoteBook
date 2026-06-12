 **C++26 为 `std::span<const T>` 增加的 initializer_list 构造能力**

在 C++20/C++23 中：

```cpp
void use_data(std::span<const int> data);

int main()
{
    use_data({1,2,3,4});   // ❌ 编译失败
}
```

原因是：

```cpp
{1,2,3,4}
```

的类型是：

```cpp
std::initializer_list<int>
```

而 C++23 的 `std::span` 并不能直接从 `initializer_list` 构造。

因此只能这样写：

```cpp
use_data(std::array{1,2,3,4});
```

或者：

```cpp
std::vector<int> v{1,2,3,4};
use_data(v);
```

------

### C++26 的改进

C++26 新增了构造函数：

```cpp
template<class T>
constexpr span(std::initializer_list<T>) noexcept;
```

（仅适用于 `span<const T>`）

因此：

```cpp
void use_data(std::span<const int> data);

int main()
{
    use_data({1,2,3,4});   // ✅ C++26
}
```

现在合法。

编译器大致会转换成：

```cpp
std::initializer_list<int> tmp{1,2,3,4};

std::span<const int> s(tmp.begin(), tmp.size());

use_data(s);
```

------

### 为什么只支持 `span<const T>`

下面是不允许的：

```cpp
void modify(std::span<int> data);

modify({1,2,3,4});    // ❌
```

因为：

```cpp
initializer_list
```

中的元素是只读的。

实际上：

```cpp
std::initializer_list<int>
```

内部相当于：

```cpp
const int*
```

所以只能构造：

```cpp
std::span<const int>
```

不能构造：

```cpp
std::span<int>
```

否则会破坏常量性。

------

### 解决了什么问题

以前很多 API 写成：

```cpp
void process(std::span<const int>);
```

调用时比较啰嗦：

```cpp
process(std::array{1,2,3,4});

process(std::vector<int>{1,2,3,4});
```

C++26 后可以直接：

```cpp
process({1,2,3,4});
```

尤其适合：

- 测试代码
- 配置数据
- 查表数据
- 数学运算接口

例如：

```cpp
#include <span>
#include <print>

void print_values(std::span<const int> values)
{
    for (auto v : values)
        std::print("{} ", v);

    std::println();
}

int main()
{
    print_values({1,2,3,4,5});   // C++26
}
```

输出：

```text
1 2 3 4 5
```

------

### 生命周期问题

很多人担心：

```cpp
use_data({1,2,3,4});
```

中的数组会不会立即销毁？

不会。

与普通函数参数一样：

```cpp
use_data({1,2,3,4});
```

创建的 `initializer_list` 临时对象会持续到整个函数调用结束。

因此：

```cpp
void use_data(std::span<const int> data)
{
    // data 在这里是安全的
}
```

完全没问题。

但是：

```cpp
std::span<const int> s = {1,2,3,4}; // ⚠️
```

或者：

```cpp
auto s = std::span<const int>{ {1,2,3,4} };
```

如果把 span 保存起来，离开该语句后底层数据就销毁了，`s` 会变成悬空视图（dangling span）。

因此这个特性主要是为了：

```cpp
foo({1,2,3,4});   // 推荐
```

而不是：

```cpp
std::span<const int> s = {1,2,3,4}; // 危险
```

这也是该提案在标准委员会中讨论最多的问题之一。