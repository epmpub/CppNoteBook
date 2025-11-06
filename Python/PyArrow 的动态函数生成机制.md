## PyArrow 的动态函数生成机制

PyArrow 的 `compute` 模块中的许多函数实际上是从 C++ 后端动态注册和生成的。让我展示一下这个过程：

### 1. **函数注册表机制**

PyArrow 维护了一个函数注册表（Function Registry），C++ 层定义的计算函数会被注册到这个表中：

```python
import pyarrow.compute as pc

# 查看所有注册的函数
print(len(pc.list_functions()))  # 会显示数百个函数
print(pc.list_functions()[:10])  # 查看前10个

# 获取特定函数的信息
func_info = pc.get_function("count_distinct")
print(func_info)
```

### 2. **动态属性访问**

PyArrow 使用 Python 的 `__getattr__` 机制来动态创建函数访问：

```python
# 简化版示例，展示原理
class ComputeModule:
    def __getattr__(self, name):
        # 当访问 pc.count_distinct 时
        # 实际上是从注册表中查找并返回对应的函数
        try:
            return get_function_from_registry(name)
        except KeyError:
            raise AttributeError(f"No function named {name}")
```

### 3. **实际代码示例**

在 PyArrow 的源码中（`python/pyarrow/_compute.pyx`），你可以看到类似这样的实现：

```python
# 这是伪代码，展示概念
def call_function(function_name, *args, **kwargs):
    """通过名称调用注册的函数"""
    func = _get_function(function_name)  # 从 C++ 注册表获取
    return func.call(args, kwargs)

# 动态生成 Python 包装器
def _make_python_wrapper(func_name):
    def wrapper(*args, **kwargs):
        return call_function(func_name, *args, **kwargs)
    return wrapper
```

### 4. **验证动态生成**

你可以通过这段代码验证：

```python
import pyarrow.compute as pc

# 方法1: 通过字符串调用函数
result1 = pc.call_function("count_distinct", [birthdays_table["years"]])

# 方法2: 通过属性访问
result2 = pc.count_distinct(birthdays_table["years"])

print(result1 == result2)  # True，两种方式等价

# 查看函数对象
print(type(pc.count_distinct))  # 显示它是一个动态生成的函数包装器
```

### 5. **为什么这样设计？**

这种设计有几个优势：

```python
# 优势1: 扩展性 - 可以轻松添加新函数而不修改 Python 代码
# C++ 层添加新函数后，Python 层自动可用

# 优势2: 性能 - 实际计算在 C++ 层完成
import pyarrow.compute as pc
import pyarrow as pa

data = pa.array(range(1000000))
# 这个调用实际上是：
# Python -> 查找注册表 -> 调用 C++ 函数 -> 返回结果
result = pc.sum(data)

# 优势3: 一致性 - 所有计算函数使用相同的调用约定
```

### 6. **类型提示的挑战**

这就是为什么 Pylance 会报错：

```python
# pyarrow/compute.pyi (类型存根文件)
# 需要手动维护每个函数的类型签名

def count_distinct(array: Array, ...) -> Scalar: ...
def sum(array: Array, ...) -> Scalar: ...
# ... 数百个函数

# 但如果 C++ 层添加了新函数，类型存根可能还没更新
# 导致 Pylance 找不到类型信息
```

### 7. **查看源码**

你可以实际查看这个机制：

```python
import pyarrow.compute as pc
import inspect

# 查看 compute 模块的属性
print(pc.__dict__.keys())  # 看看哪些是预定义的

# 尝试访问一个不存在的函数
try:
    pc.non_existent_function
except AttributeError as e:
    print(f"错误: {e}")

# 查看 count_distinct 的实现
print(inspect.getfile(pc.call_function))  # 查看实际的实现文件
```

## 总结

PyArrow 使用**函数注册表 + 动态属性查找**的模式，让 Python 层能够调用 C++ 层注册的所有计算函数。这种设计让库更灵活、性能更好，但也导致静态类型检查工具（如 Pylance）难以完全跟踪所有可用的函数。

这就是为什么你的代码能正常运行，但 Pylance 会报警告的根本原因！