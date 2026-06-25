

在 C++ 函数式编程里：

- **Currying（柯里化）**：把“多参数函数”转换成“一连串单参数函数”
- **Partial Application（偏函数应用）**：固定部分参数，生成一个新函数

它们经常一起出现，但不是对立关系。

------

# 一、先看原始函数

例如：

```cpp
int add(int a, int b) {
    return a + b;
}
```

这是一个普通二元函数：

```text
(a, b) -> result
```

------

# 二、柯里化（Currying）

柯里化会把：

```text
(a, b) -> result
```

变成：

```text
a -> (b -> result)
```

也就是：

```cpp
auto add_curried = [](int a) {
    return [a](int b) {
        return a + b;
    };
};
```

调用方式：

```cpp
auto add5 = add_curried(5);

std::cout << add5(3); // 8
```

这里：

```cpp
add_curried(5)
```

返回的是：

```cpp
(int b) -> int
```

------

# 三、偏函数应用（Partial Application）

偏函数应用的目标是：

> “固定部分参数”

例如：

```cpp
auto add5 = [](int b) {
    return add(5, b);
};
```

或者：

```cpp
auto add5 = std::bind(add, 5, std::placeholders::_1);
```

这里：

```text
(a,b)->c
```

固定了：

```text
a = 5
```

得到：

```text
b -> c
```

------

# 四、两者关系

关键点：

> 柯里化非常容易实现偏函数应用。

因为：

```cpp
add_curried(5)
```

本质上已经完成：

```text
固定第一个参数
```

所以：

```text
Currying + 调用部分参数
=
Partial Application
```

------

# 五、真正区别

## 柯里化：改变函数结构

它改变的是：

```text
f(a,b,c)
```

变成：

```text
f(a)(b)(c)
```

是“函数形式变化”。

------

## 偏函数：固定参数

它改变的是：

```text
f(a,b,c)
```

变成：

```text
g(b,c)
```

是“参数绑定”。

------

# 六、数学上看

## 柯里化

把：

```text
f : (A × B) → C
```

转换成：

```text
f : A → (B → C)
```

这是函数表示形式转换。

------

## 偏函数

给定：

```text
f(a,b)
```

固定：

```text
a=x
```

得到：

```text
g(b)=f(x,b)
```

这是参数约束。

------

# 七、它们不是 opposite（相反）

真正“接近相反”的其实是：

- Currying（柯里化）
- Uncurrying（反柯里化）

例如：

```cpp
f(a)(b)
```

恢复成：

```cpp
f(a,b)
```

这才是数学意义上的逆过程。

------

# 八、在 C++ 中为什么不常见？

因为 C++：

- 默认不是纯函数语言
- lambda 捕获已经足够强
- `std::bind`
- closure
- template
- generic lambda

已经能轻松实现 partial application。

所以：

C++ 更多使用：

- closure
- bind
- lambda capture

而不是正式 currying。

------

# 九、现代 C++ 的典型写法

例如：

```cpp
auto add = [](int a, int b) {
    return a + b;
};

auto partial_add =
    [](int a) {
        return [a](int b) {
            return a + b;
        };
    };
```

这里：

- 形式上是 currying
- 使用上是 partial application

二者融合在一起。

------

# 十、一句话总结

可以这样理解：

| 概念                              | 本质                           |
| --------------------------------- | ------------------------------ |
| Currying（柯里化）                | 把多参数函数拆成链式单参数函数 |
| Partial Application（偏函数应用） | 固定部分参数生成新函数         |
| 二者关系                          | Currying 让 Partial 更自然     |
| 是否相反                          | 不是                           |
| Currying 的反操作                 | Uncurrying                     |

你甚至可以把它理解成：

> Currying 是“函数变形”
> Partial Application 是“函数定制”