std::clamp

在 C++ 里，`std::clamp` 属于 `<algorithm>` 头文件提供的一个函数，它是 Min - Max 算法中的一部分。其作用是把一个值限制在指定的最小值和最大值范围之内。

### 函数原型

cpp

```cpp
template< class T >
const T& clamp( const T& v, const T& lo, const T& hi );

template< class T, class Compare >
const T& clamp( const T& v, const T& lo, const T& hi, Compare comp );
```

### 参数解释

- `v`：待限制的值。
- `lo`：下限值，也就是最小值。
- `hi`：上限值，也就是最大值。
- `comp`：可选的比较函数对象，用于比较元素大小。

### 返回值

- 若 `v` 小于 `lo`，则返回 `lo`。
- 若 `v` 大于 `hi`，则返回 `hi`。
- 若 `v` 处于 `lo` 和 `hi` 之间（包含边界），则返回 `v`。

### 示例代码

cpp

```cpp
#include <iostream>
#include <algorithm>

int main() {
    int value = 15;
    int min_value = 10;
    int max_value = 20;

    int clamped_value = std::clamp(value, min_value, max_value);

    std::cout << "Original value: " << value << std::endl;
    std::cout << "Clamped value: " << clamped_value << std::endl;

    return 0;
}
```

### 代码解释

- 在上述代码中，`value` 为 15，`min_value` 是 10，`max_value` 是 20。由于 15 处于 10 和 20 之间，所以 `std::clamp` 函数会返回 15。
- 若 `value` 小于 10，函数会返回 10；若 `value` 大于 20，函数则会返回 20。

### 应用场景

`std::clamp` 在很多场景下都能发挥作用，比如：

- **游戏开发**：限制角色的生命值、魔法值等属性在合理范围。
- **图形处理**：将颜色值限定在 0 到 255 之间。
- **温度控制**：确保温度处于安全范围。