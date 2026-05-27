Function Object（函数对象，Functor）

本质上是：“行为像函数的对象”。

即：

- 可以保存状态
- 可以像函数一样调用
- 本质是对象
- 通过 `operator()` 实现调用语法

最基础形式：

```cpp
struct Add
{
    int operator()(int a, int b) const
    {
        return a + b;
    }
};

Add add;

add(1, 2); // 像函数一样调用
```

这里：

```cpp
add(1,2)
```

实际上是：

```cpp
add.operator()(1,2)
```

这就是 function object。

它解决的核心问题是：

“把行为（behavior）变成值（value）。”

这是现代泛型编程、函数式编程、STL 的核心思想之一。

在 C 语言里：

函数和数据是分离的：

```c
int add(int a,int b);
```

函数：

- 不能保存状态
- 不能作为类型系统的一部分
- 不能携带上下文

而 function object：

```cpp
struct Compare
{
    bool descending;

    bool operator()(int a, int b) const
    {
        return descending ? a > b : a < b;
    }
};
```

这里：

```cpp
Compare{true}
```

本身就是：

“一个可调用值”。

即：

行为被对象化。

这带来了巨大灵活性。

例如 STL：

```cpp
std::sort(v.begin(), v.end(), Compare{true});
```

算法不再依赖固定逻辑。

而是：

“接收行为参数”。

这是 STL 最核心设计之一。

Function object 主要解决：

1. 行为参数化
2. 泛型算法定制
3. 状态化函数
4. 编译期优化
5. 高性能回调
6. 函数组合

这是比“普通函数指针”更高级的抽象。

函数指针：

```cpp
bool (*cmp)(int,int)
```

问题：

- 无状态
- 无法内联（通常）
- 类型信息弱
- 不支持模板特化
- 不支持捕获

而 function object：

- 可内联
- 可优化
- 有类型
- 可携带状态
- 零开销抽象

因此 STL 大量采用 functor。

例如：

```cpp
std::less<>
std::greater<>
std::plus<>
std::multiplies<>
std::identity
```

这些全是 function objects。

例如：

```cpp
std::less<>
```

近似：

```cpp
struct less
{
    template<class T, class U>
    constexpr bool operator()(T&& a, U&& b) const
    {
        return a < b;
    }
};
```

所以：

```cpp
std::ranges::sort(v);
```

默认内部：

```cpp
less(a, b)
```

而：

```cpp
std::identity
```

则是：

```cpp
struct identity
{
    template<class T>
    constexpr T&& operator()(T&& x) const
    {
        return std::forward<T>(x);
    }
};
```

作用：

“不改变输入值”。

即：

数学上的：

genui{"math_block_widget_always_prefetch_v2":{"content":"f(x)=x"}}

identity function。

它主要用于：

- projection 默认值
- ranges
- pipeline
- 泛型算法统一化

例如：

```cpp
comp(proj(a), proj(b))
```

默认：

```cpp
proj = identity
```

于是：

```cpp
comp(a, b)
```

自然成立。

而：

```cpp
std::bind
```

则是另一类 function object。

它解决：

“函数偏绑定（partial application）”。

例如：

```cpp
int add(int a, int b)
{
    return a + b;
}
```

你想固定一个参数：

```cpp
auto add10 = std::bind(add, 10, std::placeholders::_1);

add10(5); // 15
```

这里：

```cpp
add10
```

已经变成：

```text
f(x)=add(10,x)
```

即：

部分参数预绑定。

这是函数式编程里的：

- partial application
- currying（部分相关）

思想。

本质上：

```cpp
std::bind
```

把：

```text
function + bound arguments
```

组合成：

“新的 function object”。

内部大概类似：

```cpp
struct BindObject
{
    int fixed = 10;

    int operator()(int x) const
    {
        return add(fixed, x);
    }
};
```

所以：

bind 本质是：

“函数对象生成器”。

不过现代 C++ 中：

```cpp
std::bind
```

已经大量被 lambda 替代。

因为：

lambda：

- 更直观
- 更安全
- 错误更少
- 类型更清晰

例如：

```cpp
auto add10 = [](int x)
{
    return add(10, x);
};
```

通常优于：

```cpp
std::bind
```

现在：

function object 生态大概分：

1. Stateless functor

```cpp
std::less<>
std::identity
```

1. Stateful functor

```cpp
Compare{descending=true}
```

1. Lambda closure object

```cpp
[x](...) {}
```

lambda 本质也是 function object。

编译器实际生成：

```cpp
struct Lambda
{
    int x;

    auto operator()(...) const
    {
    }
};
```

1. Binder/composer

```cpp
std::bind
std::not_fn
ranges::views
```

1. Type-erased callable

```cpp
std::function
```

这是更高层封装。

Function object 在现代 C++ 的意义非常深。

它实际上是：

“把行为纳入类型系统”。

这是 STL、Ranges、Coroutine、Execution Model、Functional Programming 的基础。

从抽象层面看：

Function Object 是：

“First-class behavior object（第一类行为对象）”。

即：

行为可以：

- 传递
- 存储
- 组合
- 返回
- 模板化
- 编译期优化

这是现代 C++ 泛型体系的核心之一。