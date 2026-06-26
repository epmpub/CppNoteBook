

编译期动态对象生命周期模型（constexpr object lifetime model）

```cpp
constexpr
```

只是把代码提前到编译期执行。

实际上从 C++20 开始，标准已经允许编译器在常量求值期间模拟完整的对象生命周期：

```text
创建对象
构造对象
动态分配
移动对象
销毁对象
释放内存
```

这套机制通常被称为：

```text
constexpr object lifetime model
```

或者：

```text
Constant Evaluation Object Model
```

------

# C++11时代

当年的 constexpr 非常受限：

```cpp
constexpr int square(int x)
{
    return x * x;
}
```

允许：

```cpp
constexpr int x = square(5);
```

但是：

```cpp
constexpr std::string s("hello");
```

根本不允许。

因为：

```text
动态内存
构造函数
析构函数
```

几乎都被禁止。

你可以把 C++11 constexpr 理解成：

```text
编译期表达式计算器
```

而不是：

```text
编译期执行环境
```

------

# C++14

允许循环：

```cpp
constexpr int sum()
{
    int s = 0;

    for(int i=0;i<10;i++)
        s += i;

    return s;
}
```

但是仍然没有对象生命周期管理。

------

# C++20重大变化

P0784《More constexpr containers》

引入：

```cpp
new
delete
allocator
```

的 constexpr 支持。

例如：

```cpp
constexpr int f()
{
    auto p = new int(42);

    int v = *p;

    delete p;

    return v;
}

static_assert(f() == 42);
```

合法。

这在 C++17 是完全不可能的。

------

# 编译器实际上做了什么

对于：

```cpp
constexpr int f()
{
    auto p = new int(42);

    int v = *p;

    delete p;

    return v;
}
```

编译器并不会真的调用 malloc。

它会建立一个虚拟内存模型：

```text
Compile-time Heap

+----------+
| int = 42 |
+----------+
      ^
      p
```

然后模拟：

```text
new
construct
read
destroy
delete
```

整个过程。

------

# 生命周期追踪

考虑：

```cpp
struct X
{
    int value;

    constexpr X(int v)
        : value(v)
    {
    }

    constexpr ~X()
    {
    }
};

constexpr int f()
{
    X x(42);

    return x.value;
}
```

编译器会记录：

```text
1. storage created
2. constructor called
3. object alive
4. value read
5. destructor called
6. storage released
```

这与运行时完全一致。

------

# 为什么必须跟踪析构

例如：

```cpp
struct Tracker
{
    int* p;

    constexpr Tracker()
        : p(new int(42))
    {
    }

    constexpr ~Tracker()
    {
        delete p;
    }
};
```

调用：

```cpp
constexpr int f()
{
    Tracker t;

    return *t.p;
}
```

编译器必须知道：

```text
t 离开作用域时
析构函数被调用
delete 被执行
```

否则会产生编译期内存泄漏。

------

# constexpr中的内存泄漏

例如：

```cpp
constexpr int f()
{
    auto p = new int(42);

    return *p;
}
```

编译失败。

因为：

```text
new 了
没有 delete
```

标准要求：

```text
所有 constexpr 分配
必须在常量求值结束前释放
```

因此：

```cpp
static_assert(f() == 42);
```

报错。

------

# 编译器维护对象图

实际上编译器内部维护类似：

```text
Object #1
 ├─ type = int
 ├─ state = alive
 └─ storage = heap

Object #2
 ├─ type = Tracker
 ├─ state = alive
 └─ member -> Object #1
```

然后跟踪：

```text
construct
move
destroy
```

事件。

------

# 为什么 std::string 能 constexpr

现代 string：

```cpp
constexpr std::string s("hello");
```

编译器大致模拟：

```text
allocate buffer
construct chars
copy chars
destroy string
free buffer
```

全部在编译期完成。

因此：

```cpp
constexpr auto n =
    std::string("hello").size();
```

合法。

------

# 为什么 inplace_vector 难实现

你的例子：

```cpp
std::inplace_vector<NonTrivial,10>
```

执行：

```cpp
emplace_back()
```

实际上涉及：

```text
placement new
object lifetime begin
```

执行：

```cpp
pop_back()
```

涉及：

```text
destructor call
object lifetime end
```

执行：

```cpp
clear()
```

涉及：

```text
for each element:
    destroy()
```

编译器必须准确知道：

```text
第几个元素活着
第几个元素已经销毁
```

------

# 为什么 GCC 16 暂时拒绝

假设：

```cpp
inplace_vector<string,10>
```

内部布局：

```text
storage[10]
size = 3

slot0 alive
slot1 alive
slot2 alive
slot3 dead
...
```

执行：

```cpp
pop_back()
```

后：

```text
slot2 destroyed
```

执行：

```cpp
emplace_back()
```

后：

```text
slot2 reconstructed
```

这属于：

```text
storage reuse
lifetime restart
```

这是 constexpr 中最复杂的部分之一。

目前 libstdc++ 尚未完全实现。

因此源码直接写了类似：

```cpp
if consteval
{
    if constexpr(!is_trivially_destructible_v<T>)
        __builtin_unreachable();
}
```

意思是：

```text
编译期先别处理复杂生命周期
只支持 trivial 类型
```

------

# C++26正在解决什么

大量库设施变为 constexpr：

- `std::inplace_vector`
- `std::hive`
- `<memory>`未初始化算法
- `std::stable_sort`
- 更多容器操作

这些都依赖于：

```text
编译期对象生命周期管理
```

能力。

标准已经允许：

```cpp
constexpr std::vector<int>
constexpr std::string
constexpr std::inplace_vector<T>
```

理论上完整工作。

但实现难度极高。

------

# 一个直观理解

把现代 constexpr 编译器想象成一个微型虚拟机：

```text
Compile-Time VM

Stack
Heap
Object Lifetime Table
Destructor Queue
Allocation Tracker
```

当看到：

```cpp
constexpr auto result = f();
```

编译器实际上是在这个 VM 里执行 `f()`。

执行结束时必须满足：

```text
所有对象正确析构
所有内存正确释放
没有悬空引用
没有生命周期违规
```

否则：

```text
不是一个常量表达式
```

编译失败。

这也是为什么你的 `std::inplace_vector<NonTrivial>` 理论上符合 C++26 标准，但当前 GCC 16 仍然报：

```text
only trivial types are supported at compile time
```

因为标准已经具备这套生命周期模型，而标准库实现还没有把所有容器操作接入这套模型。