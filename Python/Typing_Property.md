

> **如何在 Python 这种“不强制封装”的语言里，用 \*typing + property\* 做出“接近强类型语言”的 API 约束。**

下面我按 **可落地的工程层级** 来讲。

------

# 一、目标：什么叫“强约束 API”？

在 Python 里，“强约束”通常意味着：

1. ❌ 不允许非法状态
2. ❌ 不允许随意赋值
3. ✅ IDE / mypy / pyright 能提前报错
4. ✅ 运行时也能兜底
5. ✅ 对外 API 稳定，对内可演化

------

# 二、核心组合拳

> **`private field + @property + typing + runtime validation`**

这是 Python 里最稳的一套。

------

# 三、基础模板（强烈推荐）

```python
from typing import Final

class Person:
    _name: str
    _age: int

    def __init__(self, name: str, age: int):
        self._name = name
        self.age = age  # 强制走 property

    @property
    def name(self) -> str:
        return self._name  # 只读

    @property
    def age(self) -> int:
        return self._age

    @age.setter
    def age(self, value: int) -> None:
        if value < 0 or value > 150:
            raise ValueError("age must be in [0, 150]")
        self._age = value
```

### 你得到了什么？

| 能力              | 是否具备 |
| ----------------- | -------- |
| IDE 类型提示      | ✅        |
| mypy/pyright 检查 | ✅        |
| 非法值阻断        | ✅        |
| 外部不能乱写 name | ✅        |
| 内部可自由改实现  | ✅        |

------

# 四、用 `Final` 防止“语义被破坏”

```python
from typing import Final

class Person:
    species: Final[str] = "Homo sapiens"
Person.species = "Alien"  # ❌ mypy/pyright 报错
```

⚠️ `Final` 是 **静态约束**，运行时不拦，但 IDE 会拦。

------

# 五、用 `Literal` 限定合法值域

```python
from typing import Literal

class User:
    _role: Literal["admin", "user", "guest"]

    @property
    def role(self) -> Literal["admin", "user", "guest"]:
        return self._role

    @role.setter
    def role(self, value: Literal["admin", "user", "guest"]) -> None:
        self._role = value
```

IDE 会直接提示非法值：

```python
user.role = "root"  # ❌
```

------

# 六、用 `Protocol` 定义“最小可用接口”

```python
from typing import Protocol

class HasAge(Protocol):
    @property
    def age(self) -> int: ...
def is_adult(p: HasAge) -> bool:
    return p.age >= 18
```

✔ 不要求继承
✔ 只关心 API 契约
✔ 非常适合大型系统

------

# 七、禁止“动态加字段”（高级）

```python
class Person:
    __slots__ = ("_name", "_age")
p.foo = 123  # ❌ AttributeError
```

这一步 **非常重要**，能防止 API 被“偷偷扩展”。

------

# 八、冻结对象（接近不可变）

```python
class FrozenPerson:
    __slots__ = ("_name", "_age", "_frozen")

    def __init__(self, name: str, age: int):
        self._frozen = False
        self._name = name
        self._age = age
        self._frozen = True

    def __setattr__(self, key, value):
        if getattr(self, "_frozen", False):
            raise AttributeError("object is frozen")
        super().__setattr__(key, value)
```

现在你得到了 **接近 C++ const / Rust immutability 的效果**。

------

# 九、真实工程级“强约束 Person”

```python
from typing import Final

class Person:
    __slots__ = ("_name", "_age")

    species: Final[str] = "Homo sapiens"

    def __init__(self, name: str, age: int):
        self._name = name
        self.age = age

    @property
    def name(self) -> str:
        return self._name

    @property
    def age(self) -> int:
        return self._age

    @age.setter
    def age(self, value: int) -> None:
        if not isinstance(value, int):
            raise TypeError("age must be int")
        if value < 0 or value > 150:
            raise ValueError("invalid age")
        self._age = value
```

✔ 强类型
✔ 强约束
✔ API 清晰
✔ 实现可替换

------

# 十、一句话工程总结（重点）

> **Python 的“强约束”不是靠 `private`，
> 而是靠 `typing + property + discipline + tooling`。**

如果你愿意，我可以下一步直接给你：

- **mypy / pyright 配置模板**
- **对比 dataclass vs 手写 class 的约束强度**
- **如何设计“不会后悔”的公共 API**