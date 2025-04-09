

# C++  std::complex



```C++
#include <iostream>
#include <complex>

int main(){
    // Complex numbers are specialized for float, double and long double
    std::complex<float> bottom_left{-2, -1.12};
    std::complex<float> top_right{0.7, 1.12};

    // real() and imag() methods to access the corresponding components
    for (float i = bottom_left.imag(); i < top_right.imag(); i+= 0.06) {
        for (float r = bottom_left.real(); r < top_right.real(); r+= 0.025) {

            std::complex<float> c{r,i};
            size_t iter = 0;
            // math functions
            for (std::complex<float> z=c; iter < 26 && abs(z) < 2; ++iter)
                z = z*z + c; // standard arithmetic operations

            // Print the mandelbrot set value for this "pixel"
            std::cout << static_cast<char>(iter+32);
        }
        std::cout << '\n';
    }
}
```





这段代码使用 C++ 的 <complex> 库实现了一个简单的曼德布罗集（Mandelbrot Set）可视化程序。它通过迭代复数计算并输出字符来表示曼德布罗集的边界和内部区域。以下是对代码的详细解释：

------

**代码结构**

1. **头文件**：
   - <iostream>：用于标准输入输出。
   - <complex>：提供 std::complex 类型和相关复数操作。
2. **主要内容**：
   - 定义复平面区域。
   - 迭代计算曼德布罗集。
   - 输出可视化结果。

------

**代码逐步解释**

**1. 定义复平面区域**

cpp

```cpp
std::complex<float> bottom_left{ -2, -1.12 };
std::complex<float> top_right{ 0.7, 1.12 };
```

- **std::complex<float>**：
  - 表示复数，模板参数指定浮点类型（这里是 float）。
  - 构造函数接受实部和虚部：{real, imag}。
- **bottom_left**：
  - 实部：-2，虚部：-1.12。
  - 表示复平面左下角坐标。
- **top_right**：
  - 实部：0.7，虚部：1.12。
  - 表示复平面右上角坐标。
- **区域**：
  - 实部范围：[-2, 0.7]。
  - 虚部范围：[-1.12, 1.12]。
  - 这是一个常见的曼德布罗集可视化区域。

------

**2. 遍历复平面**

cpp

```cpp
for (float i = bottom_left.imag(); i < top_right.imag(); i += 0.06) {
    for (float r = bottom_left.real(); r < top_right.real(); r += 0.025) {
```

- **外层循环（虚部）**：
  - bottom_left.imag()：返回虚部 -1.12。
  - top_right.imag()：返回虚部 1.12。
  - 步长 0.06：每次增加 0.06，控制垂直分辨率。
  - 行数 ≈ (1.12 - (-1.12)) / 0.06 ≈ 37.33，约 37 行。
- **内层循环（实部）**：
  - bottom_left.real()：返回实部 -2。
  - top_right.real()：返回实部 0.7。
  - 步长 0.025：每次增加 0.025，控制水平分辨率。
  - 列数 ≈ (0.7 - (-2)) / 0.025 = 2.7 / 0.025 = 108。
- **效果**：
  - 遍历复平面网格，每个点 (r, i) 表示一个像素。

------

**3. 曼德布罗集迭代**

cpp

```cpp
std::complex<float> c{ r, i };
size_t iter = 0;
for (std::complex<float> z = c; iter < 26 && abs(z) < 2; ++iter)
    z = z * z + c;
```

- **c{ r, i }**：
  - 当前网格点的复数坐标，例如 c = r + i*i（i 是虚单位）。
- **z = c**：
  - 初始化 z 为当前点 c，这是曼德布罗集计算的起点。
- **迭代公式**：
  - z = z * z + c：
    - 曼德布罗集的核心公式：z_{n+1} = z_n^2 + c。
    - 从 z_0 = 0 开始，但这里直接用 z_0 = c（可能是简化）。
- **终止条件**：
  - iter < 26：最多迭代 26 次。
  - abs(z) < 2：如果 |z| ≥ 2，则认为点发散，不属于曼德布罗集。
  - std::abs(z)：计算复数的模，sqrt(real^2 + imag^2)。
- **iter**：
  - 记录迭代次数，表示点是否快速发散。

------

**4. 输出字符**

cpp

```cpp
std::cout << static_cast<char>(iter + 32);
```

- **iter + 32**：
  - iter 范围：[0, 26]。
  - 加 32 映射到 ASCII 可打印字符范围：[32, 58]。
    - 32：空格 ' '。
    - 33：'!'。
    - 58：':'。
- **static_cast<char>**：
  - 将整数转换为字符。
- **效果**：
  - 发散慢（iter 小）：低 ASCII 值（如空格）。
  - 发散快或不发散（iter 大）：高 ASCII 值（如字母或标点）。
  - 形成曼德布罗集的字符可视化。

------

**5. 换行**

cpp

```cpp
std::cout << '\n';
```

- 每行结束后换行，形成二维字符图案。

------

**关键技术点**

1. **std::complex**：
   - 支持复数运算（加法、乘法、abs 等）。
   - real() 和 imag() 访问实部和虚部。
2. **曼德布罗集**：
   - 定义：对于复数 c，如果迭代 z = z^2 + c（z_0 = 0）不发散（|z| 保持小于 2），则 c 属于集合。
   - 这里简化从 z = c 开始，实际应为 z = 0，但结果仍能近似可视化。
3. **字符可视化**：
   - 使用 ASCII 字符表示迭代次数，模拟灰度效果。
   - 步长（0.025 和 0.06）决定分辨率。

------

**输出示例**

运行代码会生成类似以下的字符图案（简化版，实际 37 行 × 108 列）：

```text
                           :!  
                        :!!!  
                      :!!!!   
                    :!!!!!!   
                  :!!!!!!!!   
                :!!!!!!!!!!   
              :!!!!!!!!!!!!   
            :!!!!!!!!!!!!!!   
          :!!!!!!!!!!!!!!!!   
        :!!!!!!!!!!!!!!!!!!   
      :!!!!!!!!!!!!!!!!!!!!   
    :!!!!!!!!!!!!!!!!!!!!!!   
  :!!!!!!!!!!!!!!!!!!!!!!!!   
:!!!!!!!!!!!!!!!!!!!!!!!!!!   
  :!!!!!!!!!!!!!!!!!!!!!!!!   
    :!!!!!!!!!!!!!!!!!!!!!!   
      :!!!!!!!!!!!!!!!!!!!!   
        :!!!!!!!!!!!!!!!!!!   
          :!!!!!!!!!!!!!!!!   
            :!!!!!!!!!!!!!!   
              :!!!!!!!!!!!!   
                :!!!!!!!!!!   
                  :!!!!!!!!   
                    :!!!!!!   
                      :!!!!   
                        :!!!  
                           :!  
```

- **空格**：表示快速发散的区域（外部）。
- **标点**：表示边界或慢发散区域。
- **高值字符**：表示不发散的内部（iter = 26）。

------

**可能的改进或注意事项**

1. **正确初始化 z**：

   - 曼德布罗集应从 z = 0 开始：

     cpp

     ```cpp
     std::complex<float> z{0, 0};
     ```

     当前代码从 z = c 开始，可能偏离标准定义。

2. **分辨率调整**：

   - 减小步长（例如 0.01）可提高细节。

3. **颜色或图形输出**：

   - 使用图形库（如 SFML）替代字符输出。

------

**总结**

- **功能**：绘制曼德布罗集的字符可视化。
- **std::complex**：处理复数运算。
- **输出**：用字符表示迭代次数，形成图案。

如果你有具体问题（例如修改步长或修复 z 初始化），欢迎提问！