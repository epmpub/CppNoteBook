Python Typing 使用决策树

# 🧭 Python Typing 使用决策树（工程版）

> **从上往下问，每一步只选一个 YES / NO**

------

## ① 这是「模块 / 系统边界」吗？

（API、JSON、DB、第三方、跨团队）

### ✅ YES → **必须 typing**

👉 进入 ②

### ❌ NO → 可选（继续往下）

------

## ② 输入数据「结构是否不透明 / 易写错」？

（dict、JSON、外部 payload）

### ✅ YES

➡ 使用 **形状约束**

- `TypedDict`（JSON / dict）
- `Protocol`（对象接口）

```python
class Payload(TypedDict):
    id: int
    name: str
```

❌ 不要用裸 `dict[str, Any]`

------

## ③ 我关心的是「你能做什么」而不是「你是谁」吗？

### ✅ YES

➡ 使用 **行为契约**

- `Protocol`
- `Callable`
- `ParamSpec`（装饰器）

```python
class Writer(Protocol):
    def write(self, s: str) -> int: ...
```

📌 典型：插件 / 回调 / hook

------

## ④ 这个值有「不同合法状态」吗？

（流程 / 权限 / 生命周期）

### ✅ YES

➡ 使用 **状态建模**

**先问一句关键问题：**

> 👉 不同状态下，**允许调用的 API 是否不同？**

------

### ④-A：只是逻辑分支不同

➡ `Literal`

```python
Status = Literal["draft", "published"]
```

------

### ④-B：状态会影响“能不能用”

➡ `Union + TypeGuard`

```python
def is_adult(p: Person) -> TypeGuard[Adult]: ...
```

------

### ④-C：非法状态必须“不存在”

➡ **拆类型（最强）**

```python
Person = Union[Minor, Adult]
```

📌 **这是 Python typing 的“顶级用法”**

------

## ⑤ 这是「集合 / 容器」吗？

### ✅ YES

➡ **永远不要写裸容器**

❌ 错误：

```python
def f(x: list): ...
```

✅ 正确：

```python
def f(x: Sequence[User]) -> dict[int, list[User]]:
    ...
```

工具：

- `Iterable`
- `Sequence`
- `Mapping`
- `TypeVar / Generic`

------

## ⑥ 这个值「应该被改吗」？

### ❌ 不应该

➡ **不变量约束**

- `Final`
- `frozen=True`
- `slots=True`

```python
MAX_RETRY: Final[int] = 3
@dataclass(frozen=True, slots=True)
class User: ...
```

📌 **你之前关心的“状态一致性”就在这里**

------

## ⑦ 这里是否存在「None / 不可能发生的情况」？

### ✅ YES

➡ 显式建模

- `Optional[T]`
- `Never`

```python
def unreachable(x: Never) -> None: ...
```

📌 配合 `match` / if → exhaustiveness check

------

## ⑧ 类型太长 / 语义不清晰？

### ✅ YES

➡ **表达意图**

- `TypeAlias`
- `Annotated`
- `Self`

```python
UserId = TypeAlias = int
Age = Annotated[int, ">=0"]
```

📌 **公共 API 强烈推荐**

------

## ⑨ 这是「不可信 → 可信」的边界吗？

（JSON / DB / 外部系统）

### ✅ YES

➡ **TypeGuard / 校验函数**

```python
def is_user(x: object) -> TypeGuard[User]: ...
```

❌ 不要到处 `cast`

------

# 🧠 一句话总原则（非常重要）

> **typing 应该：**
>
> - 出现在 **边界**
> - 出现在 **易错点**
> - 出现在 **长期维护代码**
>
> **不应该：**
>
> - 侵入 every line
> - 追求“类型覆盖率”

------

# 🧩 速记版（你可以直接存）

| 看到这种代码    | 第一反应                 |
| --------------- | ------------------------ |
| dict + 约定字段 | TypedDict                |
| 鸭子类型        | Protocol                 |
| if 状态分支     | Literal / TypeGuard      |
| Union 很多      | 拆状态类型               |
| list / dict     | Sequence / Mapping       |
| 常量            | Final                    |
| frozen DTO      | dataclass(frozen, slots) |
| 外部输入        | TypeGuard                |

------

## 下一步（如果你愿意继续）

我可以直接帮你做更“落地”的事：

- 🔧 **把这棵决策树变成 code review checklist**
- 📁 **给你一套“项目分层 typing 策略”（domain / api / infra）**
- ⚙️ **pyright 严格模式 + 忽略策略（实战配置）**

你选一个，我直接继续。