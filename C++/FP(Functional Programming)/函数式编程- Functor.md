# 函数式编程- Functor（函子）

这个才是数学意义上的 Functor。

它和 C++ “仿函数对象” 完全不同。

------

# 三、Functor 的本质

Functor 的核心：

> “可被映射（map）的上下文”

关键不是“函数”。

而是：

```
容器/context
```

------

## 经典定义

Functor 必须提供：

```
map / fmap
```

即：

```
(a -> b)
```

能够作用到：

```
F<a>
```

得到：

```
F<b>
```

------

# 四、最经典例子：std::optional

例如：

```
std::optional<int> x = 5;
```

你希望：

```
x + 1
```

但 optional 不是 int。

于是：

```
Functor 的意义：
把函数提升到上下文里
```

------

## map 操作

例如：

```
auto fmap = [](auto opt, auto func) {
    if (opt)
        return std::optional{func(*opt)};
    return std::optional<std::invoke_result_t<
        decltype(func), decltype(*opt)
    >>{};
};
```

使用：

```
auto r = fmap(x, [](int v) {
    return v + 1;
});
```

得到：

```
optional<int>{6}
```

------

# 五、Functor 真正解决的问题

它解决：

```
“函数如何作用于容器内部”
```

例如：

| Context   |
| --------- |
| optional  |
| vector    |
| future    |
| coroutine |
| async     |
| expected  |

Functor 让：

```
普通函数
```

自动适配：

```
带上下文的数据
```

------

# 六、数学本质

Functor 保持：

```
结构不变
```

只改变：

```
内部值
```

例如：

```
optional<int>
```

映射后：

```
optional<string>
```

但：

```
optional 这个结构没变
```

