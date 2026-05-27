## sentinel



用哨兵计数和定义范围

范围可以不仅仅是容器或一对迭代器，也可以通过以下方式定义:

• 同一类型的开始和结束迭代器
• 开始迭代器和哨兵(可能是不同类型的结束标记)
• 开始迭代器和count
• 数组

```c++
std::vector<int> coll{1, 2, 3, 4, 5, 6, 7, 8, 9};
auto pos5 = std::ranges::find(coll, 5);
if (std::ranges::distance(pos5, coll.end()) >= 5)
{
  for (int val : std::views::counted(pos5, 5))
  {
    std::cout << val << ' ';
  }
}
std::cout
    << '\n';
```

这里，std::views::counted(pos5, 3) 创建了一个视图，该视图表示从pos5 所引用的元素开始的三
个元素。注意，counted() 不会检查是否存在元素(传递的计数过高会导致未定义行为)，开发者要确
保代码的有效性。因此，需要使用std::ranges::distance()，检查是否有足够的元素(注意，若集合没
有随机访问迭代器，这种检查的开销可能会很大)。

若知道有一个值为5 的元素后面至少有两个元素，也可以这样:

```cpp
1 // if we know there is a 5 and at least two elements behind:
2 for (int val : std::views::counted(std::ranges::find(coll, 5), 3)) {
3 std::cout << val << ' ';
4 }
```

计数可能为0，则该范围为空。
注意，只有当你确实有一个迭代器和一个计数时，才可以使用counted()。若已经有了一个范围，
并且只想处理前n 个元素，请使用std::views::take()。





```cpp
#include <iostream>
#include <algorithm> // for for_each()
struct NullTerm
{
  bool operator==(auto pos) const {
      return *pos == '\0';// end is where iterator points to ’\0’
  }
};
int main()
{
  const char *rawString = "hello world";
                          // iterate over the range of the begin of rawString and its end:
                          for (auto pos = rawString; pos != NullTerm {};++pos){
                              std::cout << ' ' << *pos;
                          }std::cout
                          << '\n';
                          // call range algorithm with iterator and sentinel:
                          std::ranges::for_each(rawString,  // begin of range
                                                NullTerm{}, // end is null terminator
                                                [](char c)
                                                {
                                                  std::cout << ' ' << c;
                                                });std::cout
                          << '\n';
}
```



`NullTerm` 作为 sentinel（哨兵）并不是“必须存在”，而是：

它解决了“范围结束位置无法提前知道”的问题。

你的代码本质上是在把：

```cpp
const char*
```

当成一个“以 `'\0'` 结尾的流式范围（stream-like range）”。

这里最大的特点是：

没有真正的 end iterator。

传统 STL 算法：

```cpp
std::for_each(begin, end, func);
```

要求：

- begin
- end

必须是同类型 iterator。

例如：

```cpp
char arr[] = "hello";

std::for_each(arr, arr + 5, f);
```

这里：

```cpp
arr + 5
```

是明确已知的结束位置。

但：

```cpp
const char* rawString = "hello world";
```

这里只有：

```cpp
begin = rawString
```

没有：

```cpp
end
```

因为 C 字符串长度是运行时扫描得到的。

传统做法：

```cpp
std::strlen(rawString)
```

先扫描一次：

```cpp
auto end = rawString + std::strlen(rawString);
```

然后：

```cpp
std::for_each(rawString, end, f);
```

问题：

字符串被扫描了两次：

1. strlen()
2. for_each()

而 sentinel 的设计允许：

“边遍历边判断结束条件”。

于是：

```cpp
std::ranges::for_each(rawString,
                      NullTerm{},
                      func);
```

实际内部逻辑近似：

```cpp
while (!(pos == end))
{
    func(*pos);
    ++pos;
}
```

这里：

```cpp
end
```

不是 iterator。

而是：

```cpp
NullTerm{}
```

它只是一个“结束判定器”。

这就是 sentinel 的本质：

“结束条件对象（termination condition object）”

而不是：

“结束位置 iterator”。

这是 ranges 相对于传统 STL 的重要升级。

以前 STL 假设：

```text
range = [begin, end)
```

即：

结束位置必须能表示为 iterator。

但现实很多数据结构不是这样：

- C string
- 网络流
- socket
- generator
- coroutine
- lazy range
- 无限序列
- mmap 流
- token stream

它们很多：

“只有开始，没有真正的 end iterator”。

sentinel 正是为此引入。

C++20 正式把：

```cpp
iterator != iterator
```

扩展成：

```cpp
iterator != sentinel
```

例如：

```cpp
std::counted_iterator
std::default_sentinel
```

就是标准库经典 sentinel 模型。

比如：

```cpp
auto i = std::counted_iterator(ptr, n);

std::ranges::for_each(i,
                      std::default_sentinel,
                      f);
```

这里：

```cpp
default_sentinel
```

根本不是 iterator。

它只是：

“count == 0 时结束”。

这和你的：

```cpp
*pos == '\0'
```

本质一样。

你的代码：

```cpp
bool operator==(auto pos) const {
    return *pos == '\0';
}
```

含义是：

“只要当前位置指向 '\0'，range 就结束。”

因此：

```cpp
NullTerm
```

是：

“逻辑结束条件”

而不是：

“物理结束位置”。

这是 ranges/sentinel 最核心的思想。

为什么 C++20 要这么设计？

因为 iterator-pair 模型有根本缺陷：

它默认：

“end 必须提前可达”。

但很多现代数据源：

根本不知道 end。

例如：

网络包：

```text
recv() until EOF
```

协程：

```cpp
co_yield
```

生成器：

```cpp
views::iota()
```

无限流：

```cpp
views::repeat()
```

它们没有：

```cpp
end iterator
```

所以：

sentinel 是对 iterator model 的泛化。

旧模型：

```text
iterator == iterator
```

新模型：

```text
iterator == termination-condition
```

这是 ranges 的理论基础之一。

实际上：

sentinel 的概念来自：

- Rust iterator
- functional lazy sequence
- stream processing
- generator model

它让 STL 从：

“容器算法”

演化成：

“流式算法”。