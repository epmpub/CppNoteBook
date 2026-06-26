

------

6.1.6 投影

sort() 和许多其他用于范围的算法一样，通常有一个额外的可选模板参数，一个投影:

```cpp
1 template<std::ranges::random_access_range R,
2 typename Comp = std::ranges::less,
3 typename Proj = std::identity>
4 requires std::sortable<std::ranges::iterator_t<R>, Comp, Proj>
5 ... sort(R&& r, Comp comp = {}, Proj proj = {});
```

可选的附加参数，允许在算法进一步处理之前为每个元素指定一个转换(投影)。
例如，sort() 允许指定要排序的元素的投影，而不是结果值的比较方式:

```cpp
1 std::ranges::sort(coll,
2 std::ranges::less{}, // still compare with <
3 [] (auto val) { // but use the absolute value
4 return std::abs(val);
5 });
```

这可能比以下代码更具可读性或更容易编程:cpp

```cpp
1 std::ranges::sort(coll,
2 [] (auto val1, auto val2) {
3 return std::abs(val1) < std::abs(val2);
4 });
```

有关完整示例，请参阅ranges/rangesproject.cpp。

默认的投影是std::identity()，只产生传递给它的参数，因此根本不执行投影/转换。(std::identity()
在<function> 中定义为一个新的函数对象)。
用户定义的投影只需要接受一个参数并为转换后的参数返回一个值。
元素必须可排序的要求考虑了投影:

```cpp
1 requires std::sortable<std::ranges::iterator_t<R>, Comp, Proj>
```

------

`std::identity` 是 C++20 引入的一个函数对象（function object）

作用非常简单：

“原样返回输入值，不做任何变换。”

定义等价于：

```cpp
struct identity
{
    template<class T>
    constexpr T&& operator()(T&& t) const noexcept
    {
        return std::forward<T>(t);
    }
};
```

即：

```cpp
identity(x) == x
```

它主要用于：

- ranges
- projection（投影）
- 泛型算法默认行为

你给出的：

```cpp
template<
    std::ranges::random_access_range R,
    typename Comp = std::ranges::less,
    typename Proj = std::identity>
requires std::sortable<
    std::ranges::iterator_t<R>,
    Comp,
    Proj>
sort(R&& r, Comp comp = {}, Proj proj = {});
```

这里：

```cpp
Proj
```

表示：

“排序前如何提取比较值”。

即 projection（投影）。

例如：

```cpp
struct Person
{
    std::string name;
    int age;
};
```

如果：

```cpp
std::vector<Person> people;
```

你想按：

```cpp
age
```

排序：

传统 STL：

```cpp
std::sort(begin(people), end(people),
          [](const auto& a, const auto& b)
          {
              return a.age < b.age;
          });
```

ranges：

```cpp
std::ranges::sort(people, {}, &Person::age);
```

这里：

```cpp
&Person::age
```

就是 projection。

算法内部逻辑近似：

```cpp
comp(proj(a), proj(b))
```

即：

```cpp
less(a.age, b.age)
```

如果用户没有提供：

```cpp
proj
```

怎么办？

就需要默认行为：

“不投影，直接比较元素本身”。

于是：

```cpp
std::identity
```

就作为默认 projection。

即：

```cpp
proj(x) == x
```

因此默认情况下：

```cpp
std::ranges::sort(v);
```

内部实际类似：

```cpp
less(identity(a), identity(b))
```

即：

```cpp
less(a, b)
```

所以：

`std::identity` 本质上是：

“空投影（no-op projection）”

它的存在是为了：

统一算法内部模型。

ranges algorithms 基本都采用：

```cpp
comp(proj(a), proj(b))
```

而不是：

```cpp
comp(a, b)
```

这样：

- 有 projection
- 无 projection

都能统一处理。

这是现代泛型设计的重要技巧。

你可以理解成：

```text
projection = compare 前的数据映射
identity   = 默认不映射
```

这在函数式编程中很常见。

例如：

数学：

genui{"math_block_widget_always_prefetch_v2":{"content":"f(x)=x"}}

identity function：

```text
f(x)=x
```

即恒等函数。

C++20 的：

```cpp
std::identity
```

本质上就是这个。

它经常出现于：

- ranges
- views
- projections
- comparator framework
- functional utilities

例如：

```cpp
std::ranges::min(v, {}, &T::member);
```

内部：

```cpp
comp(proj(a), proj(b))
```

即：

```cpp
less(a.member, b.member)
```

如果没 projection：

```cpp
proj = identity
```

则：

```cpp
less(a, b)
```

统一成立。

这是 ranges 设计中的关键抽象层。