# std::is_constant_evaluated()

`std::is_constant_evaluated()` 是 C++20 引入的一个标准库函数，用于在编译期检测当前表达式是否在**常量求值（constant evaluation）**上下文中执行。它主要用于编写既能用于编译期（`constexpr`）又能用于运行期的代码，并针对不同场景选择不同的实现逻辑。

---

## **核心功能**
- **返回值**：
  - 如果在 **编译期常量求值** 的上下文中调用（如 `constexpr` 变量初始化、模板参数、`static_assert` 等），返回 `true`。
  - 如果在 **运行期** 调用，返回 `false`。
- **典型用途**：
  - 在 `constexpr` 函数中，针对编译期和运行期选择不同的分支逻辑。
  - 避免在编译期触发未定义行为（UB），同时允许运行期的灵活处理。

---

## **基本用法**
### 1. **头文件**
```cpp
#include <type_traits>  // C++20 起
```

### 2. **示例代码**
```cpp
#include <iostream>
#include <type_traits>

constexpr int square(int x) {
    if (std::is_constant_evaluated()) {  // 编译期分支
        return x * x;
    } else {                            // 运行期分支
        std::cout << "Runtime call\n";
        return x * x;
    }
}

int main() {
    constexpr int a = square(10);  // 编译期调用，is_constant_evaluated() == true
    int b = square(20);           // 运行期调用，is_constant_evaluated() == false

    std::cout << a << ", " << b << "\n";
}
```
**输出**：
```
Runtime call
100, 400
```

---

## **典型应用场景**
### 1. **避免编译期 UB**
在 `constexpr` 函数中，某些操作（如指针解引用、数组越界）在编译期是严格禁止的，但在运行期是允许的。可以用 `is_constant_evaluated()` 提供安全路径：
```cpp
constexpr int safe_access(const int* p, int idx) {
    if (std::is_constant_evaluated() && (p == nullptr || idx < 0)) {
        throw "Compile-time UB detected!";  // 编译期报错
    }
    return p[idx];  // 运行期允许（但需确保安全）
}
```

### 2. **优化编译期计算**
某些计算（如数学函数）在编译期可以用更严格的算法，而在运行期用更高效的实现：
```cpp
constexpr double sqrt_approx(double x) {
    if (std::is_constant_evaluated()) {
        // 编译期使用高精度但慢的算法
        return /* constexpr-friendly sqrt */;
    } else {
        // 运行期使用快速近似（如 SIMD 优化）
        return /* fast runtime sqrt */;
    }
}
```

### 3. **兼容 `constexpr` 和非 `constexpr` 代码**
某些操作（如动态内存分配）在 C++20 后可以在 `constexpr` 中使用，但需要区分上下文：
```cpp
constexpr void* allocate(std::size_t n) {
    if (std::is_constant_evaluated()) {
        return nullptr;  // 编译期不支持 malloc
    } else {
        return std::malloc(n);  // 运行期分配
    }
}
```

---

## **注意事项**
1. **调用位置限制**：
   - 必须直接在 `constexpr` 函数或常量求值上下文中调用，否则行为未定义。
   - 错误示例：
     ```cpp
     bool foo() {
         return std::is_constant_evaluated();  // UB：非常量上下文
     }
     ```

2. **逻辑分支**：
   - 编译器必须能在编译期确定 `if (std::is_constant_evaluated())` 的真假，因此不能用于运行期动态分支：
     ```cpp
     // 错误用法：无法优化
     int bar(int x) {
         bool is_const = std::is_constant_evaluated();  // 运行期才确定
         return is_const ? x * x : x + x;
     }
     ```

3. **编译器支持**：
   - 需要支持 C++20 的编译器（GCC 9+, Clang 10+, MSVC 19.28+）。

---

## **总结**
| 特性         | 说明                                                     |
| ------------ | -------------------------------------------------------- |
| **用途**     | 区分编译期和运行期逻辑                                   |
| **返回值**   | `true`（编译期常量求值）或 `false`（运行期）             |
| **典型场景** | 避免编译期 UB、优化计算、兼容 `constexpr`/非 `constexpr` |
| **限制**     | 必须在 `constexpr` 或常量上下文中调用                    |

通过合理使用 `std::is_constant_evaluated()`，可以写出更灵活、安全的 `constexpr` 代码，同时兼顾编译期和运行期的需求。