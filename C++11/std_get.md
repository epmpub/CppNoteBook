C++中 std::get 是主要用来访问tuple-like 的数据结构

**`std::get<N>` 核心设计目标就是访问所有 tuple-like（类元组）结构化绑定兼容类型**，是这类容器统一的下标访问接口。

## 哪些类型属于 tuple-like，支持 std::get

标准规定只要满足 tuple-like 概念，就可以用 `std::get<索引>`、`std::tuple_size`、`std::tuple_element`：

1. `std::tuple<Ts...>` 元组（最典型）
2. `std::pair<T1,T2>` 键值对（固定大小 2 的元组）
3. `std::array<T, N>` 固定长度数组
4. 结构体绑定自定义类型（C++17 结构化绑定支持的自定义类型）
5. `std::ranges::subrange`、部分标准库视图适配器等标准库类型