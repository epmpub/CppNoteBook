

**Enable use of `std::weak_ptr` as keys in unordered associative containers**

> 让 `std::weak_ptr` 可以直接作为 `std::unordered_map` 和 `std::unordered_set` 的 Key。

------

## C++26 之前的问题

对于有序容器：

```cpp
std::set<std::weak_ptr<Foo>,
         std::owner_less<std::weak_ptr<Foo>>> s;
```

是合法的。

因为：

```cpp
std::owner_less
```

能够比较两个智能指针是否指向同一个控制块（control block）。

------

但是对于无序容器：

```cpp
std::unordered_set<std::weak_ptr<Foo>> s;
```

通常编译失败。

原因是：

```cpp
unordered_set
```

需要：

```cpp
std::hash<Key>
operator==
```

而标准库没有提供：

```cpp
std::hash<std::weak_ptr<T>>
```

以及对应的 owner-based 相等比较。

------

## 为什么普通 hash 不行

假设：

```cpp
auto sp1 = std::make_shared<int>(42);

std::weak_ptr<int> w1 = sp1;
std::weak_ptr<int> w2 = sp1;
```

这里：

```cpp
w1
w2
```

来自同一个控制块。

逻辑上应该认为：

```cpp
w1 == w2
```

代表同一个对象。

------

再看：

```cpp
auto sp2 = std::shared_ptr<int>(
    sp1.get(),
    [](int*){}
);
```

虽然：

```cpp
sp1.get() == sp2.get()
```

但：

```cpp
sp1
sp2
```

拥有不同控制块。

因此：

```cpp
weak_ptr
```

的身份（identity）不能仅仅依赖：

```cpp
get()
```

返回的裸指针。

------

## 智能指针真正的身份

标准一直采用：

```cpp
owner_less
```

语义。

即：

```text
控制块(Control Block)
      ↓
决定对象身份
```

而不是：

```text
裸指针地址
```

------

例如：

```cpp
auto sp1 = std::make_shared<int>(42);

std::weak_ptr<int> w1 = sp1;
std::weak_ptr<int> w2 = sp1;
```

那么：

```cpp
std::owner_less{}(w1,w2)
```

和

```cpp
std::owner_less{}(w2,w1)
```

都返回：

```cpp
false
```

说明：

```cpp
w1
w2
```

拥有同一个 owner。

------

## C++26 的解决方案

新增两个工具：

### owner_hash

```cpp
std::owner_hash
```

### owner_equal

```cpp
std::owner_equal
```

同时：

```cpp
weak_ptr
shared_ptr
```

新增成员：

```cpp
owner_hash()
owner_equal()
```

相关支持。

------

## 示例

### unordered_set

```cpp
#include <memory>
#include <unordered_set>

struct Object {};

int main()
{
    std::unordered_set<
        std::weak_ptr<Object>,
        std::owner_hash,
        std::owner_equal
    > objects;
}
```

现在合法。

------

### unordered_map

```cpp
#include <memory>
#include <unordered_map>
#include <string>

struct Session {};

int main()
{
    std::unordered_map<
        std::weak_ptr<Session>,
        std::string,
        std::owner_hash,
        std::owner_equal
    > sessions;
}
```

合法。

------

## 实际用途

### 对象注册表

例如 GUI 系统：

```cpp
class Widget {};
```

你不想延长对象生命周期：

```cpp
std::shared_ptr<Widget>
```

会保持对象存活。

因此使用：

```cpp
std::weak_ptr<Widget>
```

作为 key：

```cpp
unordered_map<
    weak_ptr<Widget>,
    WidgetMetadata,
    owner_hash,
    owner_equal
>
```

------

### Observer 系统

```cpp
weak_ptr<Observer>
```

作为观察者身份。

对象销毁后：

```cpp
expired()
```

自动变为失效。

无需额外 ID。

------

### 缓存系统

缓存记录：

```cpp
weak_ptr<Resource>
```

对应统计信息：

```cpp
unordered_map<
    weak_ptr<Resource>,
    CacheStats,
    owner_hash,
    owner_equal
>
```

资源释放后：

```cpp
weak_ptr
```

自动失效。

------

## owner_before、owner_less、owner_hash 的关系

### C++11

```cpp
weak_ptr::owner_before()
```

用于比较控制块。

------

### C++11

```cpp
std::owner_less
```

用于：

```cpp
set
map
```

等有序容器。

------

### C++26

```cpp
std::owner_hash
std::owner_equal
```

用于：

```cpp
unordered_set
unordered_map
```

无序容器。

------

形成完整体系：

| 功能           | C++版本 | 工具               |
| -------------- | ------- | ------------------ |
| Owner 顺序比较 | C++11   | `owner_before()`   |
| 有序容器比较器 | C++11   | `std::owner_less`  |
| Owner 哈希     | C++26   | `std::owner_hash`  |
| Owner 相等判断 | C++26   | `std::owner_equal` |

------

## 示例：同一个对象只存一次

```cpp
auto sp = std::make_shared<int>(42);

std::weak_ptr<int> w1 = sp;
std::weak_ptr<int> w2 = sp;

std::unordered_set<
    std::weak_ptr<int>,
    std::owner_hash,
    std::owner_equal
> s;

s.insert(w1);
s.insert(w2);

std::println("{}", s.size());
```

输出：

```text
1
```

因为：

```cpp
w1
w2
```

共享同一个控制块。

`owner_equal` 判定它们是同一个 owner。

------

一句话总结：

C++26 通过引入 **`std::owner_hash`** 和 **`std::owner_equal`**，补齐了智能指针基于控制块（owner identity）的哈希与相等比较能力，使 `std::weak_ptr`（以及 `std::shared_ptr`）终于可以安全、标准地作为 `std::unordered_map` 和 `std::unordered_set` 的键。