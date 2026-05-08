## SSO（Small String Optimization，小字符串优化）

本质思想非常简单：

> “短字符串不要分配堆内存，直接存在 string 对象内部。”

------

# 一、为什么需要 SSO

正常情况下：

```cpp
std::string s = "hello";
```

理论上：

```text
string对象
   ↓
heap内存："hello"
```

即使只有 5 个字符：

- 也要：
  - malloc
  - free
  - allocator
  - heap metadata
  - cache miss

代价很大。

而现实世界：

大量字符串都很短：

- 文件名
- JSON key
- HTTP header
- 用户名
- UI 文本
- 日志 tag

因此：

> “短字符串频繁堆分配”非常浪费。

------

# 二、SSO 的核心做法

string 内部直接内置一个小 buffer：

例如：

```cpp
class string
{
    char* heap_ptr;

    char small[16];
};
```

当字符串很短：

```cpp
"hello"
```

直接放：

```text
small[16]
```

不分配堆内存。

------

# 三、示意图

## 没有 SSO

```text
std::string
+--------+
| ptr ---|-------> heap: "hello"
| size   |
| cap    |
+--------+
```

需要 heap allocation。

------

## 有 SSO

```text
std::string
+-------------------+
| small buffer      |
| h e l l o \0      |
| size              |
+-------------------+
```

完全不访问 heap。

------

# 四、什么时候触发 SSO

不同实现不同。

常见：

| STL实现   | SSO大小     |
| --------- | ----------- |
| MSVC STL  | 15 chars    |
| libc++    | 22~23 chars |
| libstdc++ | 15 chars    |

例如：

```cpp
std::string s = "hello";
```

通常不会 malloc。

但：

```cpp
std::string s(1000, 'A');
```

会转为 heap allocation。

------

# 五、SSO 带来的收益

## 1. 避免 malloc/free

这是最大收益。

heap allocation 很贵：

- 锁
- allocator bookkeeping
- fragmentation
- cache miss

SSO 直接避免。

------

## 2. cache locality 更好

字符串数据和对象本体在一起：

```text
[string object + chars]
```

CPU cache 命中率更高。

------

## 3. 减少内存碎片

大量小字符串：

- 不再污染 heap
- allocator 压力下降

------

## 4. 移动构造更快

小字符串直接 memcpy 即可。

------

# 六、SSO 的代价

## 1. string 对象变大

为了内置 buffer：

```cpp
sizeof(std::string)
```

通常：

- 24 bytes
- 32 bytes

比纯指针大。

------

## 2. 实现复杂

需要：

- small mode
- heap mode
- 状态切换

属于 STL 实现里的复杂部分。

------

# 七、为什么现代 string 性能好了很多

很大原因就是：

## SSO + move semantics

以前：

```cpp
return std::string;
```

可能疯狂 malloc。

现在：

- 小字符串：
  - SSO
- 大字符串：
  - move

性能提升巨大。

------

# 八、EASTL 为什么特别重视 SSO

游戏引擎里：

大量：

- entity name
- shader name
- asset path
- UI text

都很短。

因此：

> “减少小字符串 heap allocation”收益极大。

所以 EASTL 的：

```cpp
eastl::string
```

会特别强调：

- aggressive SSO
- fixed_string
- allocator control

------

# 九、现代 C++ 相关替代方案

## string_view

```cpp
std::string_view
```

进一步优化：

- 不复制
- 不分配
- 只引用

但：

- 不拥有数据

和 SSO 是不同方向。

------

# 十、一句话理解

SSO 本质就是：

> “把短字符串直接塞进 string 对象内部，避免堆分配。”

它是现代 `std::string` 性能提升最关键的优化之一。