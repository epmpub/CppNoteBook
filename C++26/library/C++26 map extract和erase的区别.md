### 以下代码 map容器，extract和erase的区别

```c
#include <type_traits>
#include <print>
#include <map>
#include <string>
#include <string_view>


int main() {
    std::map<std::string,int,std::less<>> m {
        {"apple", 1},
        {"orange", 2}
    };

    m.erase("apple");

    for (const auto& [key, value] : m) {
        std::print("{}: {}\n", key, value);
    }

    auto nh = m.extract("orange");
    std::print("Extracted node has key: {} and value: {}\n", nh.key(), nh.mapped());

    m.try_emplace("banana", 10);
    m.insert_or_assign(
        std::string_view("pear"),
        20
    );

    for (const auto& [key, value] : m) {
        std::print("Finally: {:>6}: {}\n", key, value);
    }

}
```



```cpp
m.extract("orange");
```

看起来像是删除元素，但实际上 **`extract()` 的语义与 `erase()` 完全不同**。

## 1. extract() 返回 Node Handle

`extract()` 是 C++17 引入的节点操作（Node Handle）机制的一部分。

函数原型大致如下：

```cpp
node_type extract(const Key& key);
```

C++26 又增加了异构版本：

```cpp
template<class K>
node_type extract(K&& x);
```

你的代码：

```cpp
m.extract("orange");
```

实际上执行了：

```cpp
auto nh = m.extract("orange");
```

只是你没有接收返回值。

------

## 2. extract 做了什么

假设 map 为：

```cpp
{
    {"apple", 1},
    {"orange", 2}
}
```

执行：

```cpp
auto nh = m.extract("orange");
```

之后：

### map

```cpp
{
    {"apple", 1}
}
```

### nh

保存着：

```cpp
("orange", 2)
```

即：

```cpp
nh.key() == "orange"
nh.mapped() == 2
```

元素从容器中移除，但没有被销毁。

------

## 3. 与 erase 的区别

### erase

```cpp
m.erase("orange");
```

流程：

```text
从树中删除
↓
析构 pair<const Key,T>
↓
释放内存
```

元素彻底消失。

------

### extract

```cpp
auto nh = m.extract("orange");
```

流程：

```text
从树中摘下来
↓
保留节点内存
↓
返回 node_handle
```

元素仍然存在。

只是：

```cpp
属于 nh
而不属于 m
```

------

## 4. 为什么需要 extract

最大的用途：

### 修改 Key

map 的 Key 是 const：

```cpp
std::pair<const Key,T>
```

所以：

```cpp
it->first = "new_key";
```

非法。

例如：

```cpp
std::map<std::string,int> m {
    {"orange", 2}
};
```

不能：

```cpp
m.begin()->first = "banana";
```

编译失败。

------

利用 extract：

```cpp
auto nh = m.extract("orange");

nh.key() = "banana";

m.insert(std::move(nh));
```

结果：

```cpp
{
    {"banana", 2}
}
```

无需重新构造 value。

------

## 5. 零拷贝节点迁移

两个 map：

```cpp
std::map<std::string,int> m1 {
    {"orange", 2}
};

std::map<std::string,int> m2;
```

可以：

```cpp
auto nh = m1.extract("orange");

m2.insert(std::move(nh));
```

结果：

```cpp
m1 = {}
m2 = {{"orange",2}}
```

整个节点直接移动。

没有：

```cpp
std::string 拷贝
pair 拷贝
内存重新分配
```

------

## 6. 你的代码发生了什么

第一轮：

```cpp
{
    "apple"  -> 1
    "orange" -> 2
}
```

执行：

```cpp
m.erase("apple");
```

变成：

```cpp
{
    "orange" -> 2
}
```

打印：

```text
orange: 2
```

------

执行：

```cpp
m.extract("orange");
```

实际上：

```cpp
auto tmp = m.extract("orange");
```

map 变成：

```cpp
{}
```

但 `tmp` 立即被销毁。

因此：

```text
orange节点
↓
extract
↓
node_handle
↓
node_handle析构
↓
orange被销毁
```

效果最终等价于：

```cpp
m.erase("orange");
```

只是绕了个弯。

------

然后：

```cpp
m.try_emplace("banana", 10);
```

得到：

```cpp
{
    {"banana",10}
}
```

------

再执行：

```cpp
m.insert_or_assign(
    std::string_view("pear"),
    20
);
```

C++26 异构重载允许：

```cpp
std::string_view
```

直接作为查找键。

结果：

```cpp
{
    {"banana",10},
    {"pear",20}
}
```

打印：

```text
banana: 10
pear: 20
```

------

## 7. extract 最经典示例

```cpp
std::map<std::string,int> m {
    {"orange", 2}
};

auto node = m.extract("orange");

node.key() = "banana";

m.insert(std::move(node));

for (auto const& [k,v] : m)
{
    std::println("{} {}", k, v);
}
```

输出：

```text
banana 2
```

这是 `extract()` 最重要的设计目标：

结论：

> 从关联容器中取出一个节点，在不重新分配内存、不重新构造 value 的情况下修改 key 或转移到另一个容器。

因此可以把 `extract()` 理解成：

```text
erase + 返回节点所有权
```

而不是：

```text
erase + 返回值
```

返回的是整个红黑树（或哈希表）节点，而不是其中保存的数据副本。