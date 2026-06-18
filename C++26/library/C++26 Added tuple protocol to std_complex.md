`Added tuple protocol to std::complex` 是 C++26 的一个小而优雅的库改进，来源于 **P2819R2**。

它的目标是：

> 让 `std::complex<T>` 像 `std::pair`、`std::tuple` 一样支持 Tuple Protocol。

换句话说，C++26 之后：

```cpp
std::complex<double>
```

被视为一个拥有两个元素的 tuple-like 类型：

```text
element 0 -> real()
element 1 -> imag()
```

------

### C++23 之前

假设：

```cpp
std::complex<double> z{3.0, 4.0};
```

访问实部和虚部只能写：

```cpp
z.real()
z.imag()
```

例如：

```cpp
auto r = z.real();
auto i = z.imag();
```

------

而下面的写法不合法：

```cpp
auto [r, i] = z;
```

编译失败。

因为：

```cpp
std::complex
```

不是 tuple-like 类型。

------

### C++26 新增 Tuple Protocol

标准库增加了：

```cpp
std::tuple_size<std::complex<T>>
std::tuple_element<0, std::complex<T>>
std::tuple_element<1, std::complex<T>>
```

以及：

```cpp
std::get<0>(complex)
std::get<1>(complex)
```

------

因此：

```cpp
std::complex<double> z{3.0, 4.0};

auto [r, i] = z;
```

现在合法。

输出：

```text
r = 3
i = 4
```

------

### 结构化绑定

这是最直观的收益。

C++26：

```cpp
#include <complex>

std::complex z{1.5, 2.5};

auto [re, im] = z;
```

等价于：

```cpp
auto re = z.real();
auto im = z.imag();
```

------

### std::get

也可以：

```cpp
std::complex z{10.0, 20.0};

std::println("{}", std::get<0>(z));
std::println("{}", std::get<1>(z));
```

输出：

```text
10
20
```

对应：

```text
0 -> real
1 -> imag
```

------

### tuple_size

现在：

```cpp
static_assert(
    std::tuple_size_v<
        std::complex<double>
    > == 2
);
```

成立。

------

### tuple_element

```cpp
using C = std::complex<double>;

static_assert(
    std::same_as<
        std::tuple_element_t<0, C>,
        double
    >
);

static_assert(
    std::same_as<
        std::tuple_element_t<1, C>,
        double
    >
);
```

成立。

------

### 与 Ranges 的配合

很多现代泛型代码要求：

```cpp
tuple-like
```

例如：

```cpp
pair
tuple
array
subrange
```

以前：

```cpp
complex
```

无法参与。

现在：

```cpp
complex
```

也属于 tuple-like。

例如泛型函数：

```cpp
template<class T>
void print_pair_like(const T& x)
{
    std::println(
        "{} {}",
        std::get<0>(x),
        std::get<1>(x)
    );
}
```

以前：

```cpp
print_pair_like(std::complex{1.0,2.0});
```

失败。

现在成功。

------

### 为什么委员会这么做？

因为数学上：

```cpp
complex<T>
```

天然就是：

```text
(real, imag)
```

二元对象。

而且标准早已规定：

```cpp
real()
imag()
```

访问接口。

补上 tuple protocol 几乎没有兼容性风险，却能获得：

- Structured Binding
- std::get
- tuple-like Concepts
- 泛型算法兼容性

收益很大。

------

### 一个完整示例

```cpp
#include <complex>
#include <tuple>
#include <print>

int main()
{
    std::complex z{3.0, 4.0};

    auto [re, im] = z;

    std::println("re = {}", re);
    std::println("im = {}", im);

    std::println("{}", std::get<0>(z));
    std::println("{}", std::get<1>(z));

    static_assert(
        std::tuple_size_v<decltype(z)> == 2
    );
}
```

输出：

```text
re = 3
im = 4
3
4
```

------

### 与 std::pair 的对比

C++26 之后：

```cpp
std::pair<int,int>
```

和：

```cpp
std::complex<int>
```

都满足 tuple protocol：

```cpp
auto [a,b] = pair;
auto [re,im] = complex;
```

语法完全一致。

------

总结一下：

C++26 的 **Added tuple protocol to `std::complex`** 本质上是把 `std::complex<T>` 纳入标准的 *tuple-like* 体系，为其增加：

```cpp
std::get<0>()
std::get<1>()
std::tuple_size
std::tuple_element
```

从而支持结构化绑定、泛型 tuple 算法和 tuple-like Concepts。数学含义上：

```text
std::complex<T>
≈ tuple<T, T>
≈ (real, imag)
```

因此这是一个非常自然且实用的改进。