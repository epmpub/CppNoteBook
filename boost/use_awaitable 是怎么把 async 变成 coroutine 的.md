## use_awaitable 是怎么把 async 变成 coroutine 的?



这个问题问得很到位——你已经在看 Asio 协程模型的“发动机”了。

我先给你一句结论版，然后再一层层拆开：

------

# 🧠 一句话结论

> `use_awaitable` 做的事是：
> **把“callback 完成通知”转换成“协程恢复（resume）”**

也就是：

```
async + callback
        ↓
async + co_await（协程挂起/恢复）
```

------

# 🔍 从你熟悉的代码出发

## 原始 async 写法

```cpp
socket.async_read_some(buffer, handler);
```

👉 handler 会在未来被调用

------

## coroutine 写法

```cpp
co_await socket.async_read_some(buffer, use_awaitable);
```

👉 看起来没有 handler，但其实：

> handler 被“藏起来了”，变成了恢复协程的机制

------

# 🔧 Step 1：Asio 的一个关键设计（Completion Token）

Boost.Asio 所有 async API 都长这样：

```cpp
async_xxx(args..., CompletionToken token);
```

👉 `token` 决定：

> “这个 async 操作完成后，结果怎么交付？”

------

## 常见 token：

| token           | 行为                   |
| --------------- | ---------------------- |
| callback        | 调用回调               |
| `use_future`    | 返回 future            |
| `use_awaitable` | 返回 awaitable（协程） |

------

# ⚡ Step 2：use_awaitable 做了什么？

当你写：

```cpp
co_await socket.async_read_some(..., use_awaitable);
```

内部发生的是：

------

## 1️⃣ 返回一个 awaitable 对象

```cpp
auto op = async_read_some(..., use_awaitable);
```

👉 这个 `op` 是一个“可等待对象”

------

## 2️⃣ 触发 awaiter 机制

编译器会调用：

```cpp
op.operator co_await()
```

得到一个 awaiter，包含三个关键函数：

------

### 🧩 awaiter 三件套

```cpp
await_ready()   // 是否立即完成？
await_suspend() // 挂起时做什么？
await_resume()  // 恢复时返回什么？
```

------

# 🔥 Step 3：核心魔法（await_suspend）

这是关键中的关键👇

```cpp
await_suspend(coroutine_handle h)
```

👉 Asio 在这里做的事是：

------

## ✅ 把 coroutine handle 当成“callback”

```cpp
socket.async_read_some(buffer,
    [h](error_code ec, size_t n) {
        保存结果;
        h.resume();  // 恢复协程
    });
```

------

💥 看到了吗？

这就是关键转换：

```
callback → coroutine resume
```

------

# 🧠 Step 4：执行流程（完整）

当你执行：

```cpp
co_await socket.async_read_some(...)
```

实际流程是：

------

### ① 进入 await_suspend

```
注册 async_read_some
把 coroutine handle 存进 handler
挂起当前协程
```

------

### ② IO 发生

```
内核 → epoll / IOCP → Asio
```

------

### ③ handler 被调用

```cpp
handler(ec, n)
{
    保存结果
    coroutine_handle.resume()
}
```

------

### ④ 协程恢复

```cpp
std::size_t n = await_resume();
```

------

# 🎯 对比一下（你会更清晰）

## ❌ 传统 callback

```cpp
async_read_some(..., [&](ec, n){
    // continuation
});
```

------

## ✅ coroutine（展开后本质）

```cpp
async_read_some(..., [h](ec, n){
    h.resume();
});
```

👉 continuation = coroutine

------

# 🔬 Step 5：use_awaitable 本质结构（简化版）

你可以把它脑补成这样：

```cpp
struct use_awaitable_t {};

template<typename AsyncOp>
struct awaitable_op
{
    AsyncOp op;
    result_type result;

    bool await_ready() { return false; }

    void await_suspend(coroutine_handle h)
    {
        op([this, h](auto ec, auto value){
            result = value;
            h.resume();
        });
    }

    result_type await_resume()
    {
        return result;
    }
};
```

👉 真实实现更复杂（支持 executor / error / cancellation）

但核心逻辑就是这个

------

# ⚡ 一个更深的 insight（你现在可以理解了）

你之前问：

> 为什么现代 C++ callback 越来越倾向“一次性 continuation”

现在答案很清晰了：

------

> coroutine = continuation 的“语言级实现”

------

也就是说：

```
callback → continuation（手写）
future → continuation（封装）
coroutine → continuation（编译器生成）
```

------

# 🔥 一句话终极总结

> `use_awaitable` 的作用就是：
> **把 async 操作的 completion handler，替换成“恢复协程”的机制**

------

# 🚀 如果你想继续深入（已经到很高阶了）

你可以继续挖这几个点（非常值）：

### 👉 1. awaiter / awaitable 概念（C++ 标准层）

### 👉 2. Asio executor + coroutine 如何绑定线程

### 👉 3. cancellation（协程取消机制）

### 👉 4. co_spawn 内部实现（协程调度器）

------

如果你愿意，我可以带你看：

👉 **co_spawn 是怎么把 coroutine 挂到 io_context 的**
👉 或者画一张完整链路：`epoll → handler → resume coroutine`

这基本就是“写网络框架”的视角了 😏