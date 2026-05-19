这里的 **`Protocol`** 是 Python typing 里**非常关键、但又经常被误解的一个概念**。


------

## 2️⃣ Protocol 做了什么？

```python
from typing import Protocol

class Validator(Protocol):
    def __call__(self, value: int) -> bool: ...
```

这行代码的真实含义是：

> **“任何对象，只要它能被 `()` 调用，
> 并且签名是 `(int) -> bool`，
> 就可以被当成 `Validator` 使用。”**

------

