std::meta::nonstatic_data_members_of 返回的是 std::vector<<<std::meta::info>>

官方行为（C++26 Reflection）

cpp

```cpp
constexpr auto members = std::meta::nonstatic_data_members_of(^^T, ctx);
// members 的类型是： std::vector<std::meta::info>
```

为什么设计成返回 std::vector？

- 返回值数量是运行时（其实是编译时）可变的，vector 最自然。
- 方便使用 std::ranges、size()、operator[] 等操作。
- 但也正因为返回 vector，它涉及编译期分配，所以不能直接用于 template for 等强 constexpr 场景。

这就是为什么我们几乎总是要配合 std::define_static_array 使用：

cpp

```cpp
// 正确且常见的写法
template for (constexpr auto mem : 
    std::define_static_array(
        std::meta::nonstatic_data_members_of(^^T, ctx))) 
{
    // ...
}
```

类似返回 vector 的函数（常见）

- std::meta::members_of(^^T)
- std::meta::enumerators_of(^^Enum)
- std::meta::bases_of(^^T)
- std::meta::annotations_of(...)

这些函数大多返回 std::vector<<std::meta::info>>，所以都需要 define_static_array 来“固化”成静态数组。

------

需要我给你一个干净的工具函数封装版本吗？比如：

```cpp
template<typename T>
constexpr auto get_members() {
    constexpr auto ctx = std::meta::access_context::unchecked();
    return std::define_static_array(
        std::meta::nonstatic_data_members_of(^^T, ctx));
}
```

这样以后调用就简单多了。需要的话我直接给你。