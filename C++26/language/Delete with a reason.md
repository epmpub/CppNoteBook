https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2573r2.html#alt-syntax



C++26 引入的 **“delete with a reason”**（带原因的 `= delete`）解决了一个问题：**让你在禁止某些函数调用时，能够提供更清晰的诊断信息**。

在 C++11–C++23 中，你可以写：

```cpp
void myfunc(const auto&) = delete;
```

如果有人调用它，编译器会报错：

```
error: use of deleted function 'void myfunc(const auto&)'
```

但是报错信息通常很笼统，不说明为什么不能调用。

C++26 引入了带理由的 delete：

```cpp
void myfunc(const auto&) = delete("stop using implicit conversion");
```

这样当有人调用时，编译器可以输出更有意义的信息：

```
error: 'myfunc(const auto&)' is deleted: stop using implicit conversion
```

------

### 解决的问题

1. **更清晰的诊断**
   - 帮助使用者理解为什么函数被禁用。
   - 比普通 `= delete` 提供上下文，减少调试成本。
2. **自文档化 API**
   - 代码本身告诉别人：“这个函数不应该被调用，原因是……”
   - 避免单纯依赖注释或外部文档。
3. **控制隐式转换**
   - 很常见的用例是阻止隐式类型转换或模板实例化。
   - 例如你不想 `myfunc(4.6)` 调用自动生成的模板版本，带理由的 delete 就可以明确指出。

------

简单总结：
**C++26 的 `= delete("reason")` 让编译器报错信息更具可读性和可维护性，是一种在编译期进行 API 约束的工具。**

下面是一个完整的 C++26 示例，演示 `delete with a reason` 的典型用途：禁止隐式转换，并给出明确的编译期提示。

```cpp
#include <iostream>

// 只允许 float
void myfunc(float value)
{
    std::cout << "myfunc(float): " << value << '\n';
}

// 禁止其它类型
template<typename T>
void myfunc(const T&)
    = delete("myfunc() only accepts float; "
             "perform an explicit conversion if needed");

int main()
{
    myfunc(4.6f);      // OK

    // 以下调用会触发编译错误
    myfunc(42);
    // myfunc("hello");
    // myfunc(3.14);

    return 0;
}
```

使用 GCC 16：

```bash
g++ -std=c++2c main.cpp
```

可能看到类似诊断：

```text
error: use of deleted function
'void myfunc(const T&) [with T = int]'

note: deleted: myfunc() only accepts float;
perform an explicit conversion if needed
```

------

这个特性特别适合替代过去的写法：

```cpp
template<typename T>
void myfunc(const T&) = delete;
```

以前用户只能看到：

```text
error: use of deleted function
```

现在可以直接看到：

```text
error: use of deleted function
note: myfunc() only accepts float;
perform an explicit conversion if needed
```

------

另一个非常实用的例子是禁止裸指针接口：

```cpp
#include <memory>

void process(std::unique_ptr<int> ptr)
{
}

template<typename T>
void process(T*)
    = delete("Use std::unique_ptr instead of raw pointers");

int main()
{
    auto p = std::make_unique<int>(42);

    process(std::move(p)); // OK

    int x = 10;
    process(&x);           // 编译失败
}
```

诊断信息会明确告诉调用者：

```text
Use std::unique_ptr instead of raw pointers
```

这比写在文档里更有效，因为错误会直接出现在编译器输出中。

因此，`delete with a reason` 本质上是：

```cpp
= delete;
```

的增强版。

它并没有增加新的语言能力，而是把：

```cpp
"为什么禁止调用"
```

变成了编译器可以显示的正式诊断信息，使 API 设计者能够在编译期向用户传达意图。