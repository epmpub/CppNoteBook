

Python Typing 的 7 大使用场景分类

------

## Ⅰ️⃣ **形状约束（Structural / Shape Typing）**

> “我不关心你是谁，只关心你**长什么样**”

### 工具

- `Protocol`
- `TypedDict`
- `NamedTuple`

### 典型问题

- 鸭子类型无法静态验证
- dict 的隐式字段约定

### 使用场景

```python
class HasHi(Protocol):
    def hi(self) -> None: ...
class UserPayload(TypedDict):
    id: int
    name: str
```

### 什么时候用

✅ 模块边界
✅ 第三方对象适配
✅ API 入参 / JSON

🚫 不用于内部小函数

------

## Ⅱ️⃣ **状态建模（State / Variant Typing）**

> “不是一个类型，而是**不同状态下的不同合法形态**”

### 工具

- `Literal`
- `Union`
- `TypeGuard`
- `TypeAlias`

### 典型问题

- if 判断太多
- 非法状态只能运行时发现

### 使用场景

```python
Status = Literal["draft", "published"]

def is_published(x: Doc) -> TypeGuard[PublishedDoc]: ...
```

### 什么时候用

✅ 工作流 / 生命周期
✅ 权限 / 金融 / 流程
✅ 你之前讨论的「状态一致性」

------

## Ⅲ️⃣ **接口契约（Behavior / Capability）**

> “你能做什么，而不是你是什么”

### 工具

- `Protocol`
- `Callable`
- `ParamSpec`
- `Concatenate`

### 典型问题

- callback / hook 乱传
- 高阶函数不可推理

### 使用场景

```python
from typing import Callable

Handler = Callable[[int, str], bool]
class Writer(Protocol):
    def write(self, s: str) -> int: ...
```

### 什么时候用

✅ 框架 / 插件系统
✅ 回调 API
✅ 装饰器

------

## Ⅳ️⃣ **集合与泛型语义（Container Semantics）**

> “这个容器里**装的是什么**？”

### 工具

- `Iterable[T]`
- `Sequence[T]`
- `Mapping[K, V]`
- `TypeVar`
- `Generic`

### 典型问题

- `list` / `dict` 语义模糊
- 返回值结构不清晰

### 使用场景

```python
def group(xs: Sequence[User]) -> dict[int, list[User]]:
    ...
```

### 什么时候用

✅ 所有非 trivial 集合
🚫 不要写裸 `list` / `dict`

------

## Ⅴ️⃣ **不变量 / 约束（Immutability & Intent）**

> “这个东西**不能被改 / 不能为 None / 不能扩展**”

### 工具

- `Final`
- `NotRequired`
- `Required`
- `Literal`
- `Never`

### 典型问题

- 魔法修改
- None 滥用
- 不可能到达的分支

### 使用场景

```python
from typing import Final, Never

MAX_RETRY: Final[int] = 3
def impossible(x: Never) -> None: ...
```

### 什么时候用

✅ 常量
✅ exhaustiveness check
✅ match / if 完整性

------

## Ⅵ️⃣ **API 表达能力增强（Readability / Intent）**

> “让签名**自己说明意图**”

### 工具

- `TypeAlias`
- `Annotated`
- `Self`

### 典型问题

- 类型太长
- 语义不清晰

### 使用场景

```python
UserId = TypeAlias = int
def clone(self) -> Self: ...
Age = Annotated[int, ">= 0"]
```

### 什么时候用

✅ 公共 API
✅ Domain 概念
🚫 内部临时变量

------

## Ⅶ️⃣ **安全边界（Unsafe → Safe Transition）**

> “从不可信数据 → 强约束世界”

### 工具

- `TypeGuard`
- `assert_type`
- `cast`（慎用）

### 典型问题

- JSON / DB / 外部输入
- runtime 校验后，类型系统仍不知道

### 使用场景

```python
def is_user(x: object) -> TypeGuard[User]:
    ...
```

### 什么时候用

✅ IO 边界
🚫 不要在纯内部逻辑用 cast

------

# 一张「速查表」（工程实用）

| 你遇到的问题  | 用什么                  |
| ------------- | ----------------------- |
| dict 字段约定 | `TypedDict`             |
| 鸭子类型      | `Protocol`              |
| 状态分支      | `Union + TypeGuard`     |
| 回调 / hook   | `Callable / ParamSpec`  |
| API 可读性    | `TypeAlias / Annotated` |
| 不可变常量    | `Final`                 |
| 不可能发生    | `Never`                 |

------

# 一个**非常重要的原则**（给你这种工程取向的人）

> **typing 不是全覆盖工具
> 它应该：
> ① 出现在“边界”
> ② 出现在“易错点”
> ③ 出现在“长期维护代码”**

如果你愿意，下一步我可以帮你做一件**非常工程化**的事：

- 给你一份 **「Python Typing 使用决策树」**
- 或把你现有项目的 typing **分层规划一遍**

你选一个。