

##  **`WSA_WAIT_EVENT_0`**

** 不是“第一个事件”，而是“返回值的基准常量（base offset）”**

------

# 🧠 正确理解方式

你可以把它想成：

```
返回值 = WSA_WAIT_EVENT_0 + 事件索引
```

所以：

| 返回值                 | 含义                     |
| ---------------------- | ------------------------ |
| `WSA_WAIT_EVENT_0 + 0` | 第 0 个事件（events[0]） |
| `WSA_WAIT_EVENT_0 + 1` | 第 1 个事件（events[1]） |
| `WSA_WAIT_EVENT_0 + 2` | 第 2 个事件（events[2]） |

------

# 🔍 那为什么叫 “EVENT_0”？

这里容易误导 👇

👉 它的名字像“第0个事件”，
👉 **但实际上它只是一个常量，比如数值 0**

所以：

```
WSA_WAIT_EVENT_0 == 0   // 通常是这样
```

👉 真正的 index 是“偏移量”

------

# 🔧 用你代码对应一下

```cpp
DWORD ret = WSAWaitForMultipleEvents(...);
DWORD index = ret - WSA_WAIT_EVENT_0;
```

👉 本质就是：

```
index = ret - 0
```

------

# ⚠️ 为什么不直接返回 index？

这是 Windows API 的统一设计（很关键）：

它需要在一个返回值里表达多种状态：

| 返回值                 | 含义        |
| ---------------------- | ----------- |
| `WSA_WAIT_EVENT_0 + i` | 第 i 个事件 |
| `WSA_WAIT_TIMEOUT`     | 超时        |
| `WSA_WAIT_FAILED`      | 失败        |

👉 所以必须留出“编号空间”

------

# 🎯 一句话总结

👉 **`WSA_WAIT_EVENT_0` 不是事件，而是“第 0 个事件的返回值基准”**

你可以记成：

```
index = 返回值 - 基准值
```

------

# 🚀 再补一个你已经快接触到的本质

这个设计其实是从 Win32 API 继承来的，比如：

```cpp
WaitForMultipleObjects
```

👉 Winsock 只是复用了同一套语义

------

如果你继续往下走，我可以帮你打通最后一层：

👉 为什么 IOCP 完全不需要这种“index + 事件数组”的机制（这才是 Windows 网络模型的精髓）