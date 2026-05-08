#### `Boost.Coroutine` / `Boost.Coroutine2` 的核心目标：

> 在 C++20 coroutine 出现之前，为 C++ 提供“可暂停/恢复”的协程能力。

它主要解决：

# “函数不能中途挂起后再继续执行”的问题

------

普通函数：

```cpp
void f()
{
    step1();
    step2();
    step3();
}
```

特点：

- 一次执行到底
- return 后栈销毁
- 无法“暂停后恢复”

但很多场景天然需要：

```text
执行一点
暂停
以后继续
```

例如：

- generator
- async IO
- pipeline
- 状态机
- 游戏脚本
- cooperative multitasking

于是就有了 coroutine。

------

# 一、Boost.Coroutine 的本质

它基于：

- `Boost.Context`
- stack switching

实现：

# stackful coroutine（有栈协程）

即：

```text
Coroutine A
  own stack

Coroutine B
  own stack
```

因此：

- 可以在任意深度 suspend
- 保留完整调用栈

------

# 二、它解决了什么实际问题？

------

# 1. Generator（生成器）

这是最经典用途。

没有 coroutine：

```cpp
std::vector<int> data;
```

必须一次性生成全部数据。

------

有 coroutine：

```cpp
yield(1);
yield(2);
yield(3);
```

像 Python：

```python
yield
```

一样。

------

Boost.Coroutine2：

```cpp
boost::coroutines2::coroutine<int>::pull_type source(
    [](push_type& yield)
    {
        yield(1);
        yield(2);
        yield(3);
    });
```

调用方：

```cpp
for (int v : source)
{
    std::cout << v;
}
```

------

这解决：

# “惰性生成（lazy generation）”

问题。

特别适合：

- parser
- tokenizer
- tree traversal
- streaming
- infinite sequence

------

# 2. Callback Hell

以前 async：

```cpp
read(..., [](auto){
    write(..., [](auto){
        close(...);
    });
});
```

非常难维护。

Coroutine：

```cpp
read();
write();
close();
```

同步风格。

------

虽然早期 Boost.Coroutine 不像现代 async/await 那么完整。

但：

# 它证明了“挂起恢复”模型是正确方向。

------

# 3. 状态机爆炸

很多逻辑本质是：

```text
state A
 → wait
state B
 → wait
state C
```

传统写法：

```cpp
switch(state)
{
}
```

Coroutine：

```cpp
while (...)
{
    ...
    yield();
}
```

代码自然很多。

------

# 三、为什么叫 stackful coroutine？

因为：

# 每个 coroutine 有独立栈

例如：

```text
Coroutine A stack
 ├── f1
 ├── f2
 └── f3

Coroutine B stack
 ├── g1
 └── g2
```

Suspend 时：

- 整个调用链保留

因此：

# 可以在深层函数里 suspend

例如：

```cpp
f1()
{
    f2();
}

f2()
{
    f3();
}

f3()
{
    yield(); // OK
}
```

这是 stackful 最大优势。

------

# 四、Boost.Coroutine vs Coroutine2

------

# Boost.Coroutine（旧）

问题：

- API 复杂
- 性能一般
- exception handling 麻烦
- allocator 问题
- move support 差

后来：

# 被 Coroutine2 替代

------

# Boost.Coroutine2（新版）

底层：

- 基于 Boost.Context
- 更现代 C++

特点：

- move-only
- 性能更好
- 更安全
- allocator 更合理
- 更适合 generator

------

现在实际：

# Coroutine2 基本是 generator/fiber 工具

而不是现代 async runtime。

------

# 五、它和 C++20 coroutine 的根本区别

这是重点。

------

# Boost.Coroutine

是：

# stackful coroutine

------

# C++20 coroutine

是：

# stackless coroutine

------

# 六、stackful vs stackless

------

# 1. Boost.Coroutine（stackful）

每个协程：

```text
独立 stack
```

因此：

```cpp
f1()
{
    f2();
}

f2()
{
    yield(); // OK
}
```

任意深度 suspend。

------

代价：

- 需要分配 stack
- context switch
- assembly
- stack memory

------

# 2. C++20 coroutine（stackless）

没有独立栈。

编译器：

# 把函数改写成状态机

类似：

```cpp
switch(state)
{
case 0:
...
case 1:
...
}
```

因此：

- 不切换 stack
- 无需汇编
- 更轻量

------

但：

# 只能在 coroutine 函数内部 suspend

例如：

```cpp
co_await ...
```

必须：

```cpp
co_await
```

一路上传播。

深层普通函数：

```cpp
void f3()
{
    co_await ... // 不行
}
```

必须：

```cpp
task<> f3()
```

整个调用链 coroutine 化。

这是 stackless 最大限制。

------

# 七、性能对比

------

# Boost.Coroutine

成本：

```text
save registers
switch stack
jump
```

类似 fiber switch。

通常：

- 几十~几百 cycles

------

# C++20 coroutine

成本：

```text
state machine jump
```

更轻。

通常：

- 几个到几十 cycles

因此：

# async IO 场景

C++20 更适合。

------

# 八、表达能力对比

------

# Boost.Coroutine 更强

因为：

# 真正保留调用栈

可以：

- deep suspend
- recursive suspend
- 任意层 yield

写法非常自然。

------

# C++20 coroutine 更受限制

但：

- 更高性能
- 更容易优化
- 更适合 IO runtime

------

# 九、为什么现代 C++ 更偏向 C++20 coroutine？

因为：

# async IO 不需要完整 stack

IO await：

```text
socket read
timer
db query
```

本质：

- 挂起点明确
- suspend 点少

因此：

stackless 更划算。

------

# 十、为什么 Go/Lua/Erlang 还大量使用 stackful？

因为：

它们追求：

# “任意代码都能 suspend”

例如：

```go
foo()
{
    bar()
}

bar()
{
    baz()
}

baz()
{
    network_read()
}
```

任意层自动 suspend。

这是 stackful 最大优势。

------

# 十一、现代 Boost 生态的趋势

现在：

```text
Boost.Context
    ↓
Boost.Fiber
```

仍是 stackful 世界。

而：

```text
Boost.Asio awaitable
```

已经走向：

# C++20 stackless coroutine

路线。

------

# 十二、一句话总结

------

# Boost.Coroutine / Coroutine2

解决：

> C++ 缺少“可暂停/恢复函数”的问题。

它提供：

- yield/resume
- generator
- cooperative multitasking
- stackful coroutine

底层依赖：

- Boost.Context
- stack switching

------

# 与 C++20 coroutine 的核心区别

| 对比         | Boost.Coroutine | C++20 coroutine        |
| ------------ | --------------- | ---------------------- |
| 类型         | stackful        | stackless              |
| 是否有独立栈 | 有              | 没有                   |
| 是否切 stack | 是              | 否                     |
| suspend 能力 | 任意深度        | 必须 coroutine 链      |
| 实现方式     | context switch  | compiler state machine |
| 性能         | 较重            | 更轻                   |
| 表达能力     | 更强            | 更受限                 |
| async IO     | 一般            | 非常适合               |
| 依赖汇编     | 是              | 否                     |

------

可以把它们理解为：

```text
Boost.Coroutine
≈ Go goroutine / Lua coroutine

C++20 coroutine
≈ Rust async/await
```