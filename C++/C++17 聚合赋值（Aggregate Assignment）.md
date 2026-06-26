聚合赋值

```cpp
Compare{true}
```

中的：

```cpp
true
```

会直接初始化成员：

```cpp
descending
```

因为：

```cpp
struct Compare
{
    bool descending;

    bool operator()(int a, int b) const
    {
        return descending ? a > b : a < b;
    }
};
```

是一个 aggregate（聚合类型）

而：

```cpp
Compare{true}
```

使用的是：

```cpp
aggregate initialization
```

聚合初始化。

等价于：

```cpp
Compare tmp;
tmp.descending = true;
```

或者更准确地说：

```cpp
Compare tmp{ .descending = true };
```

因此：

```cpp
descending == true
```

于是：

```cpp
return a > b;
```

即降序排序。

例如：

```cpp
std::vector<int> v{1,5,2,4};

std::sort(v.begin(), v.end(), Compare{true});
```

内部比较：

```cpp
Compare{true}(a,b)
```

即：

```cpp
a > b
```

结果：

```text
5 4 2 1
```

而：

```cpp
Compare{false}
```

则：

```cpp
a < b
```

结果：

```text
1 2 4 5
```

这里本质上是：

“用对象保存算法策略”。

即：

```cpp
Compare{true}
```

表示：

“降序比较器对象”。

这是 function object 最经典用途之一：

“状态化行为（stateful behavior）”。

因为普通函数：

```cpp
bool compare(int a,int b);
```

无法携带：

```cpp
descending
```

这种状态。

而 function object 可以：

```cpp
struct Compare
{
    bool descending;
};
```

于是：

“行为 + 状态”

合并进同一个对象。

这也是 STL 大量采用 functor/lambda 的原因。