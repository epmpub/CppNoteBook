## **`std::function_ref`**

 是 C++26 引入的一个轻量级、非拥有的类型擦除可调用对象包装器。它本质上是一个可调用对象的引用视图，不拥有被调用的对象。

## 核心思想

`std::function_ref<R(Args...)>` 的行为类似于一个非拥有的 `std::function`。它接受任何可调用对象——函数指针、lambda、仿函数、`std::function`、`std::bind` 的结果等——只要其签名匹配。但关键区别在于：

**它不分配内存，不复制或移动被调用的对象，只存储一个指向函数体和一个指向上下文的指针（或类似的自定义可调用对象）。**

换句话说，它是一个可调用对象的引用，而不是可调用对象本身的包装器。

## 为什么需要它？

### `std::function` 的问题

- 拥有并复制/移动可调用对象
- 可能触发堆分配（除非小对象优化生效）
- 有类型擦除的开销（虚函数调用、可能的内存分配）

这些开销在某些场景下是不可接受的：

- 热路径回调
- 嵌入式环境
- 需要临时传递可调用对象的接口
- 不想在 API 中写模板参数的地方

### 裸指针/引用的问题

- 不能直接存储 lambda（需要手动转换成函数指针或做闭包装箱）
- 需要处理生命周期问题
- 模板参数会暴露实现细节

## 基本用法

```c
#include <functional>

void invoke(std::function_ref<void(int)> f) {
    f(42);  // 可以调用，和 std::function 一样
}

// 传一个 lambda
invoke([](int x) { std::print("{}", x); });

// 传函数指针
void print_int(int x) { std::print("{}", x); }
invoke(print_int);

// 传 std::function
std::function<void(int)> f = [](int x) { std::print("{}", x); };
invoke(f);
```

## 与 `std::function` 的关键区别

| 特性         | `std::function`         | `std::function_ref`            |
| ------------ | ----------------------- | ------------------------------ |
| 所有权       | 拥有（可复制、可移动）  | 不拥有（只是引用）             |
| 堆分配       | 可能                    | 从不                           |
| 可默认构造   | 是（为空状态）          | 否（必须绑定到有效可调用对象） |
| 可空状态     | 是（`nullptr` 检查）    | 否（设计上不允许）             |
| 生命周期管理 | 拥有，所以延长生命周期  | 不拥有，调用者必须保证有效     |
| 开销         | 虚函数 + 可能的内存分配 | 一个函数指针 + 一个数据指针    |

## 生命周期契约：必须保证有效性

```
std::function_ref<void()> bad_idea() {
    int x = 42;
    return [&x] { std::print("{}", x); };  // UB: lambda 引用栈上变量，
                                            // function_ref 不延长生命周期
}
```

`std::function_ref` 就像引用或指针：它假设绑定到的对象在调用时仍然存活。它不会延长临时对象的生命周期。

在你的例子中，使用者确实需要自行保证生命周期。

## 可能的实现原理

大多数实现会存储两个指针：

```c
struct function_ref<R(Args...)> {
    void*   ctx;          // 上下文/闭包指针
    R(*call)(void*, Args...);  // 指向静态的擦除函数的指针
};
```

构造时：

- 如果传入一个 lambda `[=](int x) { ... }`，`ctx` 指向 lambda 对象的地址，`call` 是一个静态函数模板实例化（例如 `invoke_lambda`），它会将 `ctx` 转回正确的 lambda 类型并调用。
- 如果是裸函数指针，`ctx` 可能为空，`call` 直接调用目标函数。

## 典型使用场景

1. **API 参数**：替代模板参数，避免编译时膨胀

   ```c++
   // 之前：模板化了整个函数
   template<typename F>
   void for_each(F f);
   
   // 之后：只擦除了调用签名
   void for_each(std::function_ref<void(int)> f);
   ```

2. **回调参数**：避免 `std::function` 的开销

   ```c++
   using Callback = std::function_ref<void(Event const&)>;
   class Widget {
       Callback on_click_;
   public:
       Widget(Callback cb) : on_click_(cb) {}
   };
   ```

3. **临时适配器**：快速包装不同签名

4. **泛型算法**：提供非模板化的可调用参数

## 总结

`std::function_ref` 填补了 "裸函数指针" 和 "`std::function`" 之间的空白。它提供了类型擦除的便利性，但放弃了所有权和为空状态，换来极低的开销。当你知道被调用的对象的生命周期超出调用范围、并且不想为 `std::function` 的堆分配或虚函数调用付账时，它就是正确的选择。