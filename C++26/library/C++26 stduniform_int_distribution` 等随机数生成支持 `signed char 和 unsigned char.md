**`std::uniform_int_distribution` 等随机数生成支持 `signed char` 和 `unsigned char`**

（P4037R1）解释。

这是 C++26 中的一个**缺陷修复（Defect Report）**提案，已被采纳，建议回溯应用到 C++11（`<random>` 引入之时）。

### 背景问题

在 C++11~C++23 中，`<random>` 头文件中的整数分布（尤其是 `std::uniform_int_distribution<IntType>`）对模板参数 `IntType` 有严格限制：

```cpp
// 标准规范（旧版）
template<class IntType = int>
class uniform_int_distribution;
```

**要求**：`IntType` 必须是以下之一：
- `short`, `int`, `long`, `long long`
- `unsigned short`, `unsigned int`, `unsigned long`, `unsigned long long`

**使用 `signed char` 或 `unsigned char`**（以及 `char` 在某些平台上是 signed 时）会导致**未定义行为（UB）**。

**常见痛点**：
- 生成随机字节（`unsigned char`）非常常见（如加密、图像处理、网络缓冲、哈希等）。
- 开发者被迫使用 `uint8_t` / `int8_t`（如果平台支持）或 `unsigned short` 等变通方案，但 `uint8_t` 通常是 `unsigned char` 的别名，仍可能触发 UB。
- 引擎（如 `std::mt19937`）的 `result_type` 也不支持 `char` 类型。

```cpp
// 之前：UB 或编译错误
std::uniform_int_distribution<unsigned char> dist(0, 255);
auto rng = std::mt19937{std::random_device{}()};
unsigned char byte = dist(rng);   // 危险！
```

### 提案解决方案（P4037）

- **扩展支持**：将 `signed char` 和 `unsigned char`（以及 `char`，取决于其底层类型）正式加入允许的 `IntType` 列表。
- **对所有整数分布**生效，包括：
  - `uniform_int_distribution`
  - `binomial_distribution`
  - `geometric_distribution`
  - `poisson_distribution`
  - `discrete_distribution` 等
- **对随机引擎**（`linear_congruential_engine`、`mersenne_twister_engine` 等）的 `UIntType` / `result_type` 也进行相应调整，确保一致性。
- **实现要求**：内部必须正确处理这些窄整数类型的范围、模运算、避免溢出等问题。

### 效果与益处

- **合法且安全**：
  ```cpp
  // C++26 及 DR 回溯后
  std::uniform_int_distribution<unsigned char> dist(0, 255);
  std::uniform_int_distribution<signed char>   sdist(-128, 127);
  
  std::vector<unsigned char> bytes(1024);
  std::ranges::generate(bytes, [&] { return dist(rng); });
  ```
- **向后兼容**：旧代码行为不变，仅扩展了之前 UB 的情况。
- **性能**：无额外开销，实现通常直接使用底层整数运算。
- **一致性**：`char` 类型家族现在在随机数生成中与其他整数类型平等对待，符合开发者直觉。

### 实际意义

- 大幅简化**随机字节生成**、**随机字符串**、**噪声生成**、**测试数据**等常见场景。
- 减少对第三方库或自定义分布的依赖。
- 修复了长期存在的“纸面 UB”问题（许多实现实际上已支持，但标准未背书）。

Cppreference 的 C++26 页面已记录此变更，libc++、MSVC 等主要实现正在/已经跟进（部分作为 DR 回溯）。

这是一个典型的“修复历史遗留限制、让标准更实用”的 C++26 改进提案。完整细节可查阅 **P4037R1** 论文。