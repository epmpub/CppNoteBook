## `std::piecewise_construct`

 是 C++11 引入的一个标签对象（tag object），主要用于告诉某些容器或类：

> **不要先构造一个完整对象再传进去，而是分别用不同参数直接构造各个成员对象。**

最典型的应用场景是 `std::pair`。

------

## 问题背景

假设有一个 `std::pair`：

```cpp
std::pair<std::string, std::vector<int>> p(
    "hello",
    std::vector<int>(10, 42)
);
```

这里：

1. 构造 `std::string`
2. 构造临时 `std::vector`
3. 移动到 pair 中

虽然已经比较高效，但有时我们希望：

```cpp
first 直接用某些参数构造
second 直接用另外一些参数构造
```

而不是先构造临时对象。

------

## pair 的难题

考虑：

```cpp
std::pair<std::string, std::vector<int>> p(
    ???,
    ???
);
```

假如我们想调用：

```cpp
std::string(5, 'A')
```

构造 first，

同时调用：

```cpp
std::vector<int>(10, 42)
```

构造 second。

pair 如何知道：

```cpp
(5, 'A')
```

属于 first，

而

```cpp
(10, 42)
```

属于 second？

因为构造函数参数已经混在一起了。

------

## piecewise_construct 的解决方案

标准库提供：

```cpp
std::piecewise_construct
```

配合：

```cpp
std::forward_as_tuple(...)
```

使用：

```cpp
std::pair<std::string, std::vector<int>> p(
    std::piecewise_construct,
    std::forward_as_tuple(5, 'A'),
    std::forward_as_tuple(10, 42)
);
```

等价于：

```cpp
first  = std::string(5, 'A');
second = std::vector<int>(10, 42);
```

直接在 pair 内部原位构造。

------

## pair 内部大概长什么样

标准库实现类似：

```cpp
template<class... Args1,
         class... Args2>
pair(
    piecewise_construct_t,
    tuple<Args1...> first_args,
    tuple<Args2...> second_args
);
```

然后：

```cpp
std::apply(
    [&](auto&&... args){
        new (&first)
        T1(std::forward<decltype(args)>(args)...);
    },
    first_args
);

std::apply(
    [&](auto&&... args){
        new (&second)
        T2(std::forward<decltype(args)>(args)...);
    },
    second_args
);
```

本质上就是：

```cpp
tuple -> 参数包展开 -> 构造对象
```

------

## map 中为什么经常出现

最经典场景：

```cpp
std::map<int, std::string> m;
```

map 的元素类型实际上是：

```cpp
std::pair<const int, std::string>
```

当你：

```cpp
m.emplace(
    std::piecewise_construct,
    std::forward_as_tuple(1),
    std::forward_as_tuple(5, 'A')
);
```

实际上构造：

```cpp
pair<const int, string>(
    1,
    string(5, 'A')
);
```

避免额外临时对象。

------

## 为什么有了 emplace 还需要 piecewise_construct？

很多人第一次学时都会疑惑。

例如：

```cpp
m.emplace(1, "hello");
```

已经很好用了。

但对于复杂类型：

```cpp
struct Person
{
    Person(std::string name,
           int age,
           double salary);
};
```

map：

```cpp
std::map<int, Person>
```

如果：

```cpp
m.emplace(
    1,
    Person("Tom", 18, 1000)
);
```

会先构造：

```cpp
Person(...)
```

再移动进去。

而：

```cpp
m.emplace(
    std::piecewise_construct,
    std::forward_as_tuple(1),
    std::forward_as_tuple(
        "Tom",
        18,
        1000
    )
);
```

直接构造：

```cpp
Person("Tom",18,1000)
```

到 map 节点中。

------

## C++17以后为何较少看到

因为有了：

```cpp
try_emplace()
```

例如：

```cpp
m.try_emplace(
    1,
    "Tom",
    18,
    1000
);
```

标准库内部实际上就是利用类似 piecewise 的机制完成：

```cpp
Person("Tom",18,1000)
```

原位构造。

因此很多代码不再显式写：

```cpp
piecewise_construct
```

------

## 本质理解

`std::piecewise_construct` 可以理解为：

```cpp
"把多个参数包分别分发给多个成员对象构造函数"
```

例如：

```cpp
pair<T1,T2>
```

需要：

```cpp
Args1... -> T1
Args2... -> T2
```

于是：

```cpp
piecewise_construct
+
tuple<Args1...>
+
tuple<Args2...>
```

形成了一个「参数包拆分协议」。

------

总结：

`std::piecewise_construct` 解决的是：

> 一个对象（如 pair）内部有多个成员对象，每个成员对象都需要自己的构造参数包，而普通函数参数无法表达“这组参数属于第一个成员、那组参数属于第二个成员”的问题。

它本质上是：

```cpp
piecewise_construct
+ tuple
+ perfect forwarding
```

实现的“多成员原位构造机制”，最常见于 `std::pair`、`std::map::emplace` 和标准库容器节点构造。

```cpp
// Piecewise_construct_app.cpp : This file contains the 'main' function. Program execution begins and ends there.
//

#include <iostream>
#include <vector>
#include <utility>
#include <string>

int main()
{
	std::vector<std::pair< int, std::string>> vec;
	vec.emplace_back(
		std::piecewise_construct, 
		std::tuple{ 42 },
		std::tuple{ 10,'a' }
	);
	for(auto& [i, s] : vec)
		std::cout << i << ' ' << s << '\n';
}

```

参考:https://www.youtube.com/watch?v=XSUlr7yAClc



