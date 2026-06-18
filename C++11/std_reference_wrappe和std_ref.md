std::reference_wrapper 通常和 std::ref / std::cref 配合使用。

std::ref 是专门用来创建 std::reference_wrapper 的辅助函数，这是最常见、最推荐的用法。1. 正确且推荐的写法

```cpp
#include <functional>
#include <iostream>

int main() {
    int a = 10;
    double b = 3.14;

    // 使用 std::ref 创建 reference_wrapper
    std::reference_wrapper<int> rw1 = std::ref(a);      // 非 const
    std::reference_wrapper<double> rw2 = std::ref(b);

    // std::cref 用于 const 引用
    std::reference_wrapper<const int> rw3 = std::cref(a);

    rw1.get() = 42;          // 修改 a 的值

    std::cout << a << '\n';  // 42
}
```

2. 常见搭配场景

| 场景                   | 推荐写法                                                 | 说明            |
| ---------------------- | -------------------------------------------------------- | --------------- |
| 普通变量               | std::ref(x)                                              | 最常用          |
| const 变量 / 不想修改  | std::cref(x)                                             | const reference |
| 存入 vector / 容器     | std::vector<std::reference_wrapper<int>> v{std::ref(x)}; | 常用            |
| lambda / bind / thread | std::thread(f, std::ref(x))                              | 传递引用        |
| 直接构造（不推荐）     | std::reference_wrapper<int> rw(x);                       | 可以，但繁琐    |

3. 直接构造 vs 使用 std::ref

cpp

```cpp
int x = 100;

// 方式1：推荐（使用 std::ref）
auto rw1 = std::ref(x);

// 方式2：直接构造（可以，但很少这么写）
std::reference_wrapper<int> rw2(x);     // 需要显式写类型
std::reference_wrapper<int> rw3 = x;    // ❌ 不能这样，编译错误
```

小结：

- std::reference_wrapper<T> 是类型。
- std::ref() / std::cref() 是创建这个类型的函数。
- 实际编程中，99% 的情况都是通过 std::ref 来生成 reference_wrapper。

------

回答你的问题：
是的，std::reference_wrapper 几乎总是和 std::ref / std::cref 配合使用，这是标准用法。

需要我给你更多实际场景的例子吗？（比如在 std::thread、std::sort、ranges 中的用法）