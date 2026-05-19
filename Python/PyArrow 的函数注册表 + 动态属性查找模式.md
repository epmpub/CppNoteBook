### PyArrow 的**函数注册表 + 动态属性查找**模式



## 一、整体架构

```
┌─────────────────────────────────────────┐
│         Python 层 (用户代码)              │
│   pc.count_distinct(array)              │
└──────────────┬──────────────────────────┘
               │ 1. 属性查找
               ▼
┌─────────────────────────────────────────┐
│      Python Compute 模块                 │
│   __getattr__("count_distinct")         │
└──────────────┬──────────────────────────┘
               │ 2. 查询注册表
               ▼
┌─────────────────────────────────────────┐
│        函数注册表 (Registry)              │
│   {"count_distinct": Function对象}       │
└──────────────┬──────────────────────────┘
               │ 3. 返回函数对象
               ▼
┌─────────────────────────────────────────┐
│         C++ 层 (实际计算)                 │
│   count_distinct 的实现代码               │
└─────────────────────────────────────────┘
```

## 二、详细实现步骤

### 步骤 1: C++ 层函数注册

在 PyArrow 的 C++ 代码中，每个计算函数都会被注册到全局注册表：

```cpp
// C++ 代码示例 (arrow/compute/registry.cc)
class FunctionRegistry {
private:
    std::unordered_map<std::string, std::shared_ptr<Function>> functions_;
    
public:
    // 注册函数
    Status AddFunction(std::shared_ptr<Function> func) {
        functions_[func->name()] = func;
        return Status::OK();
    }
    
    // 查找函数
    Result<std::shared_ptr<Function>> GetFunction(const std::string& name) {
        auto it = functions_.find(name);
        if (it == functions_.end()) {
            return Status::KeyError("Function not found");
        }
        return it->second;
    }
};

// 注册 count_distinct 函数
auto count_distinct_func = std::make_shared<CountDistinctFunction>();
GetFunctionRegistry()->AddFunction(count_distinct_func);

// 注册 sum 函数
auto sum_func = std::make_shared<SumFunction>();
GetFunctionRegistry()->AddFunction(sum_func);
```

### 步骤 2: Python 绑定层

使用 Cython 将 C++ 的注册表暴露给 Python：

```python
# pyarrow/_compute.pyx (Cython 代码)

# C++ 函数注册表的 Python 包装
cdef class FunctionRegistry:
    cdef:
        shared_ptr[CFunctionRegistry] registry
    
    def get_function(self, name):
        """从注册表获取函数"""
        cdef CResult[shared_ptr[CFunction]] result
        result = self.registry.get().GetFunction(name.encode())
        if not result.ok():
            raise KeyError(f"Function '{name}' not found")
        return pyarrow_wrap_function(result.ValueOrDie())

# 全局注册表实例
_global_registry = FunctionRegistry()

def get_function(name):
    """公共 API：通过名称获取函数"""
    return _global_registry.get_function(name)

def list_functions():
    """列出所有注册的函数"""
    return _global_registry.list_all_functions()

def call_function(name, args, options=None):
    """通用函数调用接口"""
    func = get_function(name)
    return func.call(args, options)
```

### 步骤 3: 动态属性查找

在 Python 的 `compute` 模块中实现 `__getattr__`：

```python
# pyarrow/compute.py

from pyarrow._compute import get_function, call_function, list_functions

# 模块级别的 __getattr__
def __getattr__(name):
    """
    当访问 pc.count_distinct 时，如果在模块中找不到这个属性，
    就会调用这个函数
    """
    try:
        # 尝试从注册表获取函数
        func = get_function(name)
        
        # 创建一个便捷的包装函数
        def wrapper(*args, **kwargs):
            return call_function(name, args, kwargs.get('options'))
        
        # 保留函数的元信息
        wrapper.__name__ = name
        wrapper.__doc__ = func.doc if hasattr(func, 'doc') else None
        
        return wrapper
    
    except KeyError:
        raise AttributeError(
            f"module 'pyarrow.compute' has no attribute '{name}'"
        )

# 也可以直接调用
def call_function(function_name, args, options=None):
    """通用调用接口"""
    return _call_function(function_name, args, options)
```

## 三、实际运行流程

让我们跟踪 `pc.count_distinct(array)` 的完整执行流程：

```python
import pyarrow.compute as pc
import pyarrow as pa

array = pa.array([1, 2, 2, 3, 3, 3])
result = pc.count_distinct(array)
```

**详细执行步骤：**

```python
# 步骤 1: Python 解释器查找 pc.count_distinct
# - 首先在 pc 模块的 __dict__ 中查找 'count_distinct'
# - 没找到，调用 pc.__getattr__('count_distinct')

# 步骤 2: __getattr__ 被调用
def __getattr__(name):  # name = "count_distinct"
    # 步骤 2.1: 调用 get_function
    func = get_function("count_distinct")
    
    # 步骤 2.2: get_function 查询 C++ 注册表
    # C++ 层: registry.GetFunction("count_distinct")
    # 返回一个 Function 对象
    
    # 步骤 2.3: 创建 Python 包装函数
    def wrapper(*args, **kwargs):
        return call_function("count_distinct", args, kwargs.get('options'))
    
    return wrapper

# 步骤 3: 返回的 wrapper 函数被调用
wrapper(array)  # 等价于 call_function("count_distinct", [array], None)

# 步骤 4: call_function 执行
def call_function(name, args, options):
    # 4.1: 再次从注册表获取函数对象
    func = get_function(name)
    
    # 4.2: 调用 C++ 层的实际实现
    # 传递参数到 C++ 的 count_distinct 实现
    # C++ 执行计算
    # 返回结果
    return func.execute(args, options)

# 步骤 5: 返回结果
# result = 3 (数组中有 3 个不同的值)
```

## 四、实践验证

让我写一个简化版本来演示这个模式：

```python
# 模拟 PyArrow 的注册表机制
class SimplifiedComputeModule:
    """模拟 pyarrow.compute 模块"""
    
    def __init__(self):
        # 函数注册表
        self._registry = {}
        
        # 注册一些函数
        self._register_function("count_distinct", self._count_distinct_impl)
        self._register_function("sum", self._sum_impl)
        self._register_function("mean", self._mean_impl)
    
    def _register_function(self, name, impl):
        """注册函数到注册表"""
        self._registry[name] = impl
    
    def _count_distinct_impl(self, array):
        """count_distinct 的实际实现"""
        return len(set(array))
    
    def _sum_impl(self, array):
        """sum 的实际实现"""
        return sum(array)
    
    def _mean_impl(self, array):
        """mean 的实际实现"""
        return sum(array) / len(array)
    
    def call_function(self, name, *args, **kwargs):
        """通用调用接口"""
        if name not in self._registry:
            raise KeyError(f"Function '{name}' not found")
        return self._registry[name](*args, **kwargs)
    
    def list_functions(self):
        """列出所有注册的函数"""
        return list(self._registry.keys())
    
    def __getattr__(self, name):
        """动态属性查找"""
        if name.startswith('_'):
            # 避免无限递归
            raise AttributeError(f"No attribute '{name}'")
        
        if name not in self._registry:
            raise AttributeError(
                f"module 'compute' has no function '{name}'"
            )
        
        # 返回一个包装函数
        def wrapper(*args, **kwargs):
            return self.call_function(name, *args, **kwargs)
        
        wrapper.__name__ = name
        return wrapper

# 使用示例
compute = SimplifiedComputeModule()

# 查看所有函数
print("所有函数:", compute.list_functions())

# 方式 1: 通过属性访问（动态查找）
data = [1, 2, 2, 3, 3, 3]
print("count_distinct:", compute.count_distinct(data))  # 触发 __getattr__
print("sum:", compute.sum(data))
print("mean:", compute.mean(data))

# 方式 2: 通过字符串调用（直接查表）
print("count_distinct:", compute.call_function("count_distinct", data))

# 方式 3: 验证动态生成
print("\n验证动态特性:")
print("'count_distinct' in dir(compute):", 'count_distinct' in dir(compute))  # False
print("hasattr(compute, 'count_distinct'):", hasattr(compute, 'count_distinct'))  # True

# 尝试访问不存在的函数
try:
    compute.non_existent()
except AttributeError as e:
    print(f"错误: {e}")
```

## 五、优势与劣势

### 优势：

```python
# 1. 扩展性：轻松添加新函数
# C++ 层只需注册新函数，Python 层自动可用
compute._register_function("new_func", lambda x: x * 2)
print(compute.new_func(5))  # 10，立即可用！

# 2. 性能：减少 Python 开销
# 实际计算在 C++ 层，Python 只负责路由

# 3. 一致性：统一的调用接口
# 所有函数都遵循相同的调用约定

# 4. 灵活性：支持多种调用方式
pc.count_distinct(array)  # 属性方式
pc.call_function("count_distinct", [array])  # 字符串方式
```

### 劣势：

```python
# 1. 类型检查困难
# Pylance/mypy 无法静态分析动态生成的属性

# 2. IDE 自动完成受限
# IDE 不知道有哪些函数可用

# 3. 调试复杂
# 错误堆栈更深，难以追踪

# 4. 文档生成困难
# 自动文档工具可能无法识别动态函数
```

## 六、实际应用场景

这种模式在很多库中都有应用：

```python
# 1. NumPy 的 ufuncs
# numpy 也使用类似的注册机制

# 2. TensorFlow/PyTorch
# 操作符注册表

# 3. SQLAlchemy
# 函数表达式的动态生成

# 4. 插件系统
# 动态加载和注册插件
```

## 总结

PyArrow 的**函数注册表 + 动态属性查找**模式：

1. **注册表**：C++ 层维护一个函数名到实现的映射
2. **动态查找**：Python 层通过 `__getattr__` 拦截属性访问
3. **延迟绑定**：函数在首次访问时才查找和包装
4. **统一接口**：所有函数通过统一的 `call_function` 执行

这种设计让 PyArrow 既保持了 C++ 的性能，又提供了 Python 的便捷性，但也导致了静态类型检查工具的困难。