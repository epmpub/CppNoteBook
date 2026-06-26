## C++ 26 Hashing support for std::chrono value classes

C++26（提案阶段，已在 WG21 中逐步合入 POC / implementation exploration）中关于 **`std::chrono` 哈希支持（hashing support for value classes）** 的核心目标，是让 `std::chrono` 的时间类型能够“开箱即用”地作为无序容器 key，而不需要用户手写 hash functor。

------

## 1. 背景问题（C++23 及之前）

在 C++20 / C++23 中：

```cpp
std::unordered_map<std::chrono::system_clock::time_point, int> m;
```

通常是**不成立或不可移植的**，原因是：

- `std::hash` 并未对所有 `std::chrono` value types 提供标准特化
- 不同标准库实现（libstdc++ / libc++ / MSVC STL）行为不一致
- 即使某些实现支持，也属于扩展，不是标准保证

因此用户必须手动写：

```cpp
struct Hash {
    size_t operator()(std::chrono::system_clock::time_point const& tp) const {
        return std::hash<std::chrono::system_clock::rep>{}(tp.time_since_epoch().count());
    }
};
```

------

## 2. C++26 的改进目标

C++26 引入（或标准化）方向是：

> 为 `std::chrono::duration` 和 `std::chrono::time_point` 提供一致、可移植的 `std::hash` 支持。

核心覆盖：

- `std::chrono::duration<Rep, Period>`
- `std::chrono::time_point<Clock, Duration>`
- 相关 value types（依赖 duration 的类型）

------

## 3. 设计思路（关键点）

### 3.1 hash 基于“底层 representation”

对 `duration`：

```cpp
hash(duration) ≈ hash(rep value)
```

本质上：

- 直接对 `rep`（整数/浮点）进行 hash
- 不依赖 period（如 seconds / milliseconds）

------

对 `time_point`：

```cpp
hash(time_point) ≈ hash(duration since_epoch)
```

即：

```cpp
return hash(tp.time_since_epoch().count());
```

------

### 3.2 避免精度/单位混淆

C++26 设计重点之一：

不同单位必须归一化到同一 representation：

```cpp
std::chrono::seconds
std::chrono::milliseconds
```

如果表示同一时间点，应 hash 一致（或规范化后一致）。

------

### 3.3 constexpr + noexcept

目标特性：

- `constexpr std::hash`（允许编译期计算）
- `noexcept`
- 与 `<functional>` 完全一致语义

------

## 4. 标准行为（预期）

### 4.1 duration

```cpp
std::hash<std::chrono::milliseconds>{}(ms)
```

等价于：

```cpp
std::hash<long long>{}(ms.count())
```

------

### 4.2 time_point

```cpp
std::hash<std::chrono::system_clock::time_point>{}(tp)
```

等价于：

```cpp
std::hash<duration>{}(tp.time_since_epoch())
```

------

## 5. 使用效果

### 5.1 unordered_map 直接可用

```cpp
#include <chrono>
#include <unordered_map>

using namespace std::chrono;

std::unordered_map<system_clock::time_point, int> events;

int main() {
    auto now = system_clock::now();
    events[now] = 42;
}
```

C++26 下目标状态：

- 无需自定义 hash
- 无需 specialization hack
- 标准库直接支持

------

## 6. 与现有 std::hash 的关系

C++26 并不是新增一套 hash 机制，而是：

> 补齐 `std::hash<T>` 对 chrono value types 的标准特化

仍然使用：

```cpp
namespace std {
  template<class T> struct hash;
}
```

只是增加：

```cpp
template<class Rep, class Period>
struct hash<std::chrono::duration<Rep, Period>>;

template<class Clock, class Duration>
struct hash<std::chrono::time_point<Clock, Duration>>;
```

------

## 7. 设计约束与争议点

### 7.1 precision loss vs semantic equality

不同 duration 类型：

- 1s
- 1000ms

语义等价，但 hash 是否必须一致？

标准倾向：

- **hash 不强制跨单位一致**
- 但实现通常会 normalize 到 common duration

------

### 7.2 floating-point duration

如果：

```cpp
duration<double>
```

存在：

- NaN / -0.0 / precision issues

hash 必须定义稳定行为（通常依赖 bit representation）

------

## 8. 总结

C++26 的 chrono hashing 改进本质是：

- 统一 `std::chrono::duration` / `time_point` 的 hash 行为
- 基于 `.count()` 或 epoch duration
- 使 chrono 类型可直接用于 unordered associative containers
- 消除跨标准库实现差异

------

如果你需要，我可以进一步拆：

- P2592 / PXXXX 相关提案原文设计细节
- libstdc++ / libc++ 当前实现差异
- 或 chrono hash 与 `strong typedef` / `chrono::zoned_time` 的关系