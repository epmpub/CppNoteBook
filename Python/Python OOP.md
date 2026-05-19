python OOP

------

## 1️⃣ 继承（Inheritance）——**“是一个”关系（is-a）**

👉 **子类 = 特化的父类**

```python
class Animal:
    def speak(self):
        print("animal sound")

class Dog(Animal):
    def speak(self):
        print("wang")

d = Dog()
d.speak()   # wang
```

**语义重点**

- `Dog` **就是** `Animal`
- 可替换性（LSP）：`Dog` 能放到任何 `Animal` 使用的地方

❌ 反例（滥用继承）：

```python
class Car(Engine):  # ❌ car 不是 engine
    pass
```

------

## 2️⃣ 组合（Composition）——**“有一个” + 强生命周期（has-a）**

👉 **对象内部创建 / 持有另一个对象**
👉 **生命周期绑定**

```python
class Engine:
    def start(self):
        print("engine start")

class Car:
    def __init__(self):
        self.engine = Engine()   # 强组合

    def run(self):
        self.engine.start()

c = Car()
c.run()
```

**语义重点**

- `Car` **拥有** `Engine`
- `Car` 死亡，`Engine` 也没有存在意义
- 最推荐的设计方式（👉 *“优先组合而非继承”*）

------

## 3️⃣ 聚合（Aggregation）——**“有一个” + 弱生命周期**

👉 和组合几乎一样，但 **不负责创建、不控制生命周期**

```python
class Engine:
    def start(self):
        print("engine start")

class Car:
    def __init__(self, engine: Engine):
        self.engine = engine     # 聚合（外部注入）

    def run(self):
        self.engine.start()

engine = Engine()
car1 = Car(engine)
car2 = Car(engine)   # 同一个 engine
```

**语义重点**

|            | 组合   | 聚合   |
| ---------- | ------ | ------ |
| 创建者     | 内部   | 外部   |
| 生命周期   | 强绑定 | 弱绑定 |
| 是否可共享 | 否     | 是     |

👉 **工程上**：

- 组合 = “我负责你的一生”
- 聚合 = “我只是用你”

------

## 4️⃣ 接口（Interface）——**行为契约，不关心实现**

### 🐍 Python 没有 `interface` 关键字

但有 **3 种常见方式** 👇

------

### 4.1 抽象基类（ABC）——**运行期约束**

```python
from abc import ABC, abstractmethod

class Payment(ABC):
    @abstractmethod
    def pay(self, amount: int):
        pass

class Alipay(Payment):
    def pay(self, amount: int):
        print(f"Alipay pay {amount}")

class WechatPay(Payment):
    def pay(self, amount: int):
        print(f"Wechat pay {amount}")
def checkout(p: Payment):
    p.pay(100)
```

**特点**

- 强约束
- 类继承链显式
- 适合 SDK / 框架

------

### 4.2 Protocol（你之前问过）——**结构化接口（duck typing++）**

```python
from typing import Protocol

class Payment(Protocol):
    def pay(self, amount: int) -> None: ...
class ApplePay:
    def pay(self, amount: int):
        print(f"ApplePay {amount}")
def checkout(p: Payment):
    p.pay(100)
```

✔️ **不需要继承**
✔️ **mypy / pyright 静态检查**
❌ 运行期不检查

👉 **非常 Pythonic + 工程友好**

------

### 4.3 纯鸭子类型（最原始）

```python
class Cash:
    def pay(self, amount):
        print(amount)

def checkout(p):
    p.pay(100)
```

❌ 没约束
❌ IDE / 重构不友好

------

## 5️⃣ 一张对比表（重点）

| 关系 | 关键词 | 是否继承 | 生命周期 | 使用场景    |
| ---- | ------ | -------- | -------- | ----------- |
| 继承 | is-a   | ✅        | 绑定     | 类型层级    |
| 组合 | has-a  | ❌        | 强       | 核心部件    |
| 聚合 | uses-a | ❌        | 弱       | 资源/服务   |
| 接口 | can-do | ❌        | 无       | 解耦 / 插件 |

------

## 6️⃣ 一句工程建议（很重要）

> **90% 的业务代码：组合 + 接口（Protocol）**
> **继承只用于稳定的抽象层**

