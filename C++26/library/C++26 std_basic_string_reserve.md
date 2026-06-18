## C++26 删除 `std::basic_string::reserve()` 无参重载



```c
#include <string>

int main(){
    std::string s = "Hello, C++26!";
    // s.reserve(); // C++26 之后，reserve() 已经废弃；
    s.shrink_to_fit(); // C++26 之后，shrink_to_fit 变得更有效率，能真正释放内存
    s.clear();         // 现在 clear() 也能真正释放内存了！
    s.shrink_to_fit(); // 现在调用 shrink_to_fit 后，s 的容量应该会变为 0，完全释放内存
    return 0;
}
```

# 1. 原本 `std::basic_string::reserve()` 有两个重载

在 C++23 及更早版本中：

```cpp
void reserve(size_type new_cap);
void reserve();   // 非标准/扩展/历史遗留实现中存在
```

但“无参数版本”并不是标准设计的一部分语义核心。

------

## ❗关键点：标准语义其实只有一个

标准定义的是：

```cpp
void reserve(size_type new_cap);
```

含义是：

> 至少分配 `new_cap` 个字符容量（capacity >= new_cap）

------

# 2. 那“无参数 reserve()”是什么？

```c

```



在一些实现（历史 libstdc++ / libc++ / MSVC 扩展）中出现过：

```cpp
std::string s;
s.reserve();
```

它的行为通常是：

```text
capacity -> size（或者略微优化 shrink/fit）
```

但问题是：

### ❌ 它语义不清晰

因为它可能被解释为：

- shrink to fit？
- no-op？
- force reallocation？
- optimize capacity?
- align internal buffer?

不同标准库实现行为并不一致。

------

# 3. C++26 的决策：删除该重载

C++26 移除：

```cpp
basic_string::reserve()
```

（无参数版本）

------

# 4. 为什么要删？

核心原因有三个：

------

## 4.1 语义重叠（redundant API）

C++11 起已经有：

```cpp
s.shrink_to_fit();
```

用于表达：

> 释放多余容量（non-binding request）

而 `reserve()` 无参版本和它高度重叠。

------

## 4.2 容易误用（最重要）

开发者常误以为：

```cpp
s.reserve();
```

等价于：

```cpp
s.shrink_to_fit();
```

但实际上：

- 可能不缩容
- 可能不释放内存
- 可能甚至不做任何事

👉 这是“看起来有意义，但实际上不可靠”的 API

------

## 4.3 标准语义统一

现代 C++ 内存模型倾向于：

| 操作          | 语义                        |
| ------------- | --------------------------- |
| reserve(n)    | 扩容（capacity >= n）       |
| shrink_to_fit | 收缩（non-binding request） |
| resize(n)     | 改变 size                   |
| clear()       | size = 0                    |

而：

```cpp
reserve()
```

不属于任何一类。

------

# 5. 删除后的替代方式

## ❌ 旧写法（C++23及以前）

```cpp
std::string s = "hello";
s.reserve();
```

------

## ✅ C++26 推荐替代

### 如果想“释放冗余内存”

```cpp
s.shrink_to_fit();
```

------

### 如果想“提前扩容”

```cpp
s.reserve(expected_size);
```

------

### 如果想“强制重新分配”

```cpp
std::string tmp;
tmp.swap(s);
```

或：

```cpp
s = std::string(s);
```

------

# 6. 实现层面影响

实际上：

- 很多 STL 实现的 `reserve()` 无参版本内部只是调用 `shrink_to_fit()`
- 或者直接 no-op

删除后：

```text
编译期错误替代运行期歧义
```

------

# 7. 一句话总结

> C++26 删除 `std::basic_string::reserve()` 无参重载，是为了消除一个语义模糊且与 `shrink_to_fit()` 重叠的历史 API，使字符串容量管理接口回归“reserve = 扩容，shrink_to_fit = 收缩”的单一职责模型。

------

如果你需要，我可以顺便把 C++26 对 `std::string` 相关 capacity 管理（SSO、growth factor、propagate behavior）一起整理成一张“现代 std::string 内存模型图”。