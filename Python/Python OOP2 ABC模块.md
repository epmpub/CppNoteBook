好，这里我**系统 + 偏工程实战**地把 `abc` 模块讲清楚。
我会刻意对比你前面问过的 **Protocol / typing / API 约束**，让你知道 **什么时候该用 abc，什么时候不该用**。

------

# 一、`abc` 模块是干什么的？

```
abc = Abstract Base Classes
```

👉 **用途一句话：**

> **在运行期（runtime）强制子类“必须实现某些方法”**

这是 Python 里**最接近 Java / C# 接口 + 抽象类的东西**。

------

# 二、最基础用法（你必须会）

```python
from abc import ABC, abstractmethod

class Payment(ABC):

    @abstractmethod
    def pay(self, amount: int) -> None:
        pass
```

### ❌ 不能直接实例化

```python
p = Payment()
# TypeError: Can't instantiate abstract class Payment
```

------

### ✅ 子类必须实现所有抽象方法

```python
class Alipay(Payment):
    def pay(self, amount: int) -> None:
        print(f"pay {amount}")
a = Alipay()   # OK
```

------

### ❌ 漏实现就报错（运行期）

```python
class BadPay(Payment):
    pass

BadPay()
# TypeError: Can't instantiate abstract class BadPay
```

⚠️ **注意：不是定义时报错，而是实例化时报错**

------

# 三、抽象方法 ≠ 只能是 method

## 1️⃣ 抽象属性（property）

```python
class Repo(ABC):

    @property
    @abstractmethod
    def db_name(self) -> str:
        pass
class MySQLRepo(Repo):
    @property
    def db_name(self) -> str:
        return "mysql"
```

------

## 2️⃣ 抽象 classmethod / staticmethod

```python
class Serializer(ABC):

    @classmethod
    @abstractmethod
    def loads(cls, data: str):
        pass
```

------

# 四、抽象类 ≠ 不能有实现

👉 **这是和“接口”的关键区别**

```python
class BaseService(ABC):

    def log(self, msg: str):
        print(f"[LOG] {msg}")

    @abstractmethod
    def run(self):
        pass
class Job(BaseService):
    def run(self):
        self.log("running job")
```

📌 抽象类 =

- 一部分 **模板**
- 一部分 **必须实现的点**

👉 非常适合 **Template Method 模式**

------

# 五、`abc` 的核心机制（重要）

### 抽象方法是如何被追踪的？

```python
Payment.__abstractmethods__
# frozenset({'pay'})
```

- Python 在 **类创建时**
- 统计所有 `@abstractmethod`
- 存在未实现 → 禁止实例化

------

# 六、`abc` ≠ 接口（这是很多人误解的）

| 特性           | abc  | Java interface |
| -------------- | ---- | -------------- |
| 是否允许实现   | ✅    | Java 8 前 ❌    |
| 是否强制继承   | ✅    | ✅              |
| 是否运行期检查 | ✅    | ❌              |
| 多继承         | ✅    | Java ❌         |

------

# 七、`register()` —— 虚拟子类（高级但危险）

```python
class Flyable(ABC):
    @abstractmethod
    def fly(self): ...
class Bird:
    def fly(self):
        print("fly")
Flyable.register(Bird)
isinstance(Bird(), Flyable)  # True
```

⚠️ **注意**

- 不检查方法是否真的存在
- 只影响 `isinstance / issubclass`
- 不影响 IDE / type checker

👉 **极少用，99% 项目不建议**

------

# 八、ABC + 多继承（真实工程会遇到）

```python
class Logger(ABC):
    @abstractmethod
    def log(self, msg: str): ...

class FileLogger:
    def log(self, msg: str):
        print("file:", msg)

class Service(FileLogger, Logger):
    pass
Service()   # OK
```

👉 **只要 MRO 中有实现即可**

------

# 九、abc vs Protocol（你之前问过的重点对比）

| 维度           | abc    | Protocol |
| -------------- | ------ | -------- |
| 约束时机       | 运行期 | 静态     |
| 是否继承       | 必须   | 不需要   |
| duck typing    | ❌      | ✅        |
| IDE / 类型推断 | 一般   | 非常好   |
| 插件式架构     | 一般   | ⭐⭐⭐⭐⭐    |

### 结论（非常重要）

> **abc 是“框架控制点”**
> **Protocol 是“业务解耦点”**

------

# 十、什么时候「一定要」用 abc？

✅ **适合 abc 的场景**

1. SDK / 框架
2. BaseXXX 类（如 `BaseCommand`, `BaseRepository`）
3. 希望**运行时立即失败**
4. 团队规模大，约束 > 灵活

❌ **不适合 abc 的场景**

- 快速业务代码
- 插件系统
- 第三方实现（你无法控制继承）

------

# 十一、一个真实工程示例（推荐）

```python
class BaseRepo(ABC):

    def save(self, obj):
        self._validate(obj)
        self._save(obj)

    def _validate(self, obj):
        if obj is None:
            raise ValueError()

    @abstractmethod
    def _save(self, obj):
        pass
class SqlRepo(BaseRepo):
    def _save(self, obj):
        print("save to sql")
```

👉 **模板方法 + 抽象扩展点**

------

# 十二、总结一句话（给你做设计决策用）

> **abc = 强约束、强控制、运行期兜底**
> **Protocol = 解耦、可扩展、类型系统友好**

------

如果你愿意，下一步我可以帮你：

- 🔍 深挖 `abc` 的 **底层实现（`__isabstractmethod__`）**
- 🔁 用 **Protocol 重写 abc 示例，对比设计差异**
- 🧠 总结 **Python OOP 设计决策树（什么时候选什么）**

你更想继续哪一个？