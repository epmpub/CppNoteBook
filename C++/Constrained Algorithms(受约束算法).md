C++20 的 Constrained Algorithms（受约束算法）本质上是：

“带有 Concepts 类型约束的 STL 算法。”

它解决的是传统 STL 算法几个长期问题：

- 模板错误信息极其难读
- 参数要求隐藏在文档里
- SFINAE 行为不可预测
- iterator/category 要求不明确
- 泛型算法缺乏“语义约束”

C++20 通过：

- Concepts
- Ranges
- requires clause

重新定义了 STL 算法体系。

最核心变化：

旧 STL：

```cpp
std::sort(begin, end);
```

编译器不会提前检查：

- 是否随机迭代器
- 是否可交换
- 是否可比较

直到模板深层实例化才爆炸。

错误信息通常几百行。

例如：

```cpp
std::list<int> lst;
std::sort(lst.begin(), lst.end());
```

错误巨大。

因为：

```cpp
std::sort
```

要求 RandomAccessIterator。

但 list 只有 BidirectionalIterator。

而 C++20 constrained algorithm：

```cpp
std::ranges::sort(lst);
```

会直接报：

```text
constraint not satisfied
```

并明确指出：

```text
random_access_range required
```

这就是 constrained 的含义：

“模板拥有显式语义约束。”

典型定义（简化）：

```cpp
template<
    std::random_access_iterator I,
    std::sentinel_for<I> S,
    class Comp = ranges::less,
    class Proj = identity
>
requires std::sortable<I, Comp, Proj>
constexpr I
sort(I first, S last, Comp comp = {}, Proj proj = {});
```

这里出现了：

```cpp
std::random_access_iterator
std::sortable
std::sentinel_for
```

这些都是 Concept。

它们相当于：

“编译期接口契约”。

即：

```cpp
sort
```

不再接受“任意类型”，而只接受：

“满足排序语义的类型”。

这是现代 C++ 泛型编程的重要转变。

以前：

“鸭子类型 + 模板碰碰运气”

现在：

“形式化语义接口”。

C++20 的 constrained algorithms 主要来自：

```cpp
std::ranges
```

例如：

旧版：

```cpp
std::find(begin(v), end(v), 3);
```

新版：

```cpp
std::ranges::find(v, 3);
```

区别不仅是省略 begin/end。

更重要的是：

算法本身带约束。

例如：

```cpp
template<input_range R, class T>
constexpr borrowed_iterator_t<R>
find(R&& r, const T& value);
```

这里：

```cpp
input_range
```

明确要求：

- 必须可迭代
- 必须符合 range 概念

如果不满足：

立即约束失败。

而不是深层模板报错。

Constrained Algorithms 还引入了 Projection（投影）。

例如：

```cpp
struct Person
{
    std::string name;
    int age;
};

std::vector<Person> people;

std::ranges::sort(people, {}, &Person::age);
```

这里：

```cpp
&Person::age
```

是 projection。

等价于：

```cpp
[](const Person& p){ return p.age; }
```

这比传统：

```cpp
std::sort(..., lambda)
```

更简洁。

这是 ranges algorithms 的重要增强。

C++20 constrained algorithms 的核心设计目标：

1. 提前失败（fail early）
2. 更清晰错误信息
3. 语义化泛型
4. 更安全的模板系统
5. 与 ranges 无缝结合
6. 替代 iterator-pair API

其背后哲学接近：

- Rust trait bounds
- Haskell typeclass constraints
- Swift protocol constraints

例如：

Rust：

```rust
fn sort<T: Ord>(...)
```

对应 C++：

```cpp
template<std::totally_ordered T>
```

这是 C++ 泛型系统逐渐“类型理论化”的体现。

一个很重要的变化：

传统 STL：

```cpp
iterator + iterator
```

C++20：

```cpp
range
```

传统：

```cpp
std::copy(a.begin(), a.end(), b.begin());
```

现代：

```cpp
std::ranges::copy(a, b.begin());
```

于是：

算法开始“理解容器”。

这也是 ranges 的核心。

常见 constrained algorithms：

- `std::ranges::sort`
- `std::ranges::find`
- `std::ranges::copy`
- `std::ranges::transform`
- `std::ranges::for_each`
- `std::ranges::fold_left`（C++23）
- `std::ranges::contains`（C++23）

它们几乎都是：

“concept + range + projection” 的组合。

最后总结一句：

Constrained Algorithms 本质上是：

“具有形式化语义约束的现代 STL 泛型算法系统。”

它让 C++ 模板从：

“无限制元编程”

逐渐演化为：

“可验证的编译期接口系统”。