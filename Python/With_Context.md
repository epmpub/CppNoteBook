

## ❶ 你只是“定义了协议”，但**从未进入协议**

`__enter__ / __exit__` **不会自动执行**。

> 👉 **只有 `with` 语句才会触发它们**

你现在的代码是：

```python
p = Person
```

这里发生的事情只有一件：

- `p` 指向 **类对象 `Person`**
- 没有实例
- 没有 `with`
- 没有作用域结束

所以 **什么都不会调用**

------

## ❷ 就算你写成 `p = Person()`，也还是不会打印

很多人会以为这样就行了：

```python
p = Person()
```

但依然 **不会打印**。

原因是：

> **Context Manager ≠ 构造器**
>
> `__enter__ / __exit__` 与 `__init__` **完全无关**

只有：

```python
with Person() as p:
    ...
```

------

## ❸ 正确触发的最小示例（但你这版仍然是错的）

你可能会改成：

```python
class Person:
    def __enter__(self):
        return self

    def __exit__(self):
        print("clean something...")

with Person() as p:
    pass
```

👉 **这仍然会报错**

------

## ❹ 关键点：`__exit__` 的签名是强约束的

Python 的 context manager **不是鸭子类型**，而是**协议类型**。

### 正确签名必须是：

```python
def __exit__(self, exc_type, exc, tb) -> bool | None:
    ...
```

你的：

```python
def __exit__(self):
    ...
```

会导致：

```text
TypeError: __exit__() takes 1 positional argument but 4 were given
```

------

## ❺ 完全正确、最小、可工作的版本

```python
class Person:
    def __enter__(self):
        print("enter")
        return self

    def __exit__(self, exc_type, exc, tb):
        print("clean something...")

if __name__ == "__main__":
    with Person() as p:
        print("working...")
```

输出：

```text
enter
working...
clean something...
```

------

## ❻ 这背后的**真正语义**（重点）

### `with` 做了什么（等价展开）

```python
mgr = Person()
obj = mgr.__enter__()
try:
    ...
finally:
    mgr.__exit__(exc_type, exc, tb)
```

所以：

| 行为             | 是否发生 |
| ---------------- | -------- |
| 创建实例         | ✅        |
| 调用 `__enter__` | ✅        |
| 离开作用域       | ✅        |
| 调用 `__exit__`  | ✅        |

而你原来的代码：

```python
p = Person
```

连 **第一步都没发生**。

------

## ❼ 再加一条容易踩的坑（你这个例子正好踩中）

### `__exit__` ≠ 析构函数

- `__exit__` **由 `with` 调用**
- `__del__` **由 GC 决定，且不可靠**

你这个例子正好印证了之前那句话：

> **资源释放必须有“语法级作用域”，而不是“对象生命周期”**

------

## 最后一句总结（请记住）

> **Context Manager 不是“有 `__exit__` 就会执行”**
> **而是“只有 `with` 才会执行 `__exit__`”**

如果你愿意，下一步我可以：

- 把这个 `Person` 改成一个**真正安全的资源类**
- 或用 `typing.Protocol` 把 context manager 的**协议约束写出来**

你想继续哪条？