是的，**命名空间别名（Namespace Alias）** 是 **C++98** 就已经引入的功能，比 C++11 还早。

例如：

```cpp
namespace rng = std::ranges;
```

这种语法从最早的 ISO C++ 标准（1998）开始就支持。

示例：

```cpp
namespace very_long_namespace_name
{
    void func() {}
}

namespace vln = very_long_namespace_name;

int main()
{
    vln::func();
}
```

------

### 标准演进

| 特性                                           | 引入标准 |
| ---------------------------------------------- | -------- |
| Namespace Alias                                | C++98    |
| using declaration (`using std::cout;`)         | C++98    |
| using directive (`using namespace std;`)       | C++98    |
| Alias Template (`using Vec = std::vector<T>;`) | C++11    |
| Inline Namespace                               | C++11    |

很多人会混淆下面几个东西：

#### 1. Namespace Alias（C++98）

```cpp
namespace rng = std::ranges;
```

给命名空间起别名。

------

#### 2. Using Declaration（C++98）

```cpp
using std::cout;
using std::ranges::find;
```

把某个名字引入当前作用域。

------

#### 3. Using Directive（C++98）

```cpp
using namespace std;
```

把整个命名空间中的名字引入当前作用域。

------

#### 4. Type Alias（C++11）

```cpp
using StringVec = std::vector<std::string>;
```

等价于：

```cpp
typedef std::vector<std::string> StringVec;
```

但语法更现代。

------

#### 5. Alias Template（C++11）

```cpp
template<typename T>
using Vec = std::vector<T>;
```

这是 `typedef` 做不到的。

例如：

```cpp
Vec<int> nums;
Vec<std::string> words;
```

------

所以：

```cpp
namespace rng = std::ranges;
```

这个语法并不是 C++11 新特性，而是从 **C++98** 就存在了。只是因为 `std::ranges` 本身是 **C++20** 才引入的，所以你只能在 C++20 及以后写：

```cpp
namespace rng = std::ranges;
```

本质上是：

- `namespace 别名 = 原命名空间;` —— C++98
- `std::ranges` —— C++20

两者组合后看起来像是现代语法，但实际上别名机制已经有将近 30 年历史了。