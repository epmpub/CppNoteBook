你想要的是：**不每次显式写 `python3.14`，而让**

```bash
uv run python main.py
```

**自动等价于**

```bash
uv run python3.14 main.py
```

在 `uv` 里，**正确做法是“固定 Python 版本”，而不是改命令别名**。

------

## ✅ 推荐做法（官方 & 正统）

### 1️⃣ 在项目里 pin Python 3.14

```bash
uv python pin 3.14
```

这会在项目根目录生成：

```text
.python-version
```

内容类似：

```
3.14
```

之后 **所有 `uv run python ...` 都会自动用 3.14**：

```bash
uv run python main.py
```

✔ 等价于 `python3.14`
✔ 可复现
✔ CI / 团队一致
✔ uv 官方推荐

------

## 2️⃣ 一次性指定（不持久化）

如果你只是偶尔用 3.14：

```bash
uv run --python 3.14 python main.py
```

但你既然问“as”，说明你想**默认行为**，所以不推荐这个。

------

## ❌ 不推荐的方式

### ❌ Shell alias

```bash
alias python=python3.14
```

问题：

- 会影响系统 python
- uv **不会信任 shell alias**
- 跨平台/CI 会炸

------

## 🔍 uv 的真实逻辑（关键理解）

`uv run python` ≠ 系统 `python`

它做的是：

1. 查 `.python-version`
2. 查 `pyproject.toml`
3. 查 uv-managed Python
4. 最后才 fallback

所以**pin 版本是唯一正确答案**

------

## 🔥 最终结论（TL;DR）

> 想让
> `uv run python main.py`
> **等价于**
> `uv run python3.14 main.py`

### ✅ 只需要做一次：

```bash
uv python pin 3.14
```

------

如果你愿意，我可以帮你：

- 把 `pyproject.toml` 也一起规范好
- 对比 `uv` vs `pyenv` vs `conda`
- 给你一套 **Python 版本 + venv + lock 的工程级规范**

直接说 👍