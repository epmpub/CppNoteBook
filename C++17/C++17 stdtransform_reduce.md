# C++17 std::transform_reduce

```C++
/*
#include <array>
#include <iostream>
#include <numeric>

int main()
{
	std::array coll{ 1,2,3,4 };

	auto twice = [](int v) { return v * 2; };

	// 计算每个元素的两倍的和：
	std::cout << "sum of doubles: "
		<< std::transform_reduce(coll.cbegin(), coll.cend(),  // 范围
			0, std::plus{}, twice)
		<< '\n';

	// 计算每个元素的平方的和：
	std::cout << "sum of squared: "
		<< std::transform_reduce(coll.cbegin(), coll.cend(),  // 范围
			0L, std::plus{},
			[](auto v) {
				return v * v;
			})
		<< '\n';
}
*/

#include <array>
#include <iostream>
#include <numeric>
#include <string>

int main()
{
    std::array coll1{ 1,2,3,4};
    std::array coll2{ 1, 2, 3, 4, 5 };

    // 计算每个元素的平方的和：
    std::cout << "sum of squared:         "
        << std::transform_reduce(coll1.cbegin(), coll1.cend(),    // 第一个范围
            coll1.cbegin(),                  // 第二个范围
            0L)
        << '\n';

    // 计算每两个相应元素的差的乘积：
    std::cout << "product of differences: "
        << std::transform_reduce(coll1.cbegin(), coll1.cend(),    // 第一个范围
            coll2.cbegin(),                  // 第二个范围
            1L,
            std::multiplies{}, std::minus{})
        << '\n';

    // 先把两个相应元素转换成的字符串连接起来，再连接所有上一步生成的字符串：
    std::cout << "sum of combined digits: "
        << std::transform_reduce(coll1.cbegin(), coll1.cend(),    // 第一个范围
            coll2.cbegin(),                  // 第二个范围
            std::string{}, std::plus{},
            [](auto x, auto y) {
                return std::to_string(x) + std::to_string(y) + " ";
            })
        << '\n';
}
```



### 代码解释

#### 1. 计算每个元素的平方的和

cpp

```cpp
std::cout << "sum of squared:         "
    << std::transform_reduce(coll1.cbegin(), coll1.cend(),    // 第一个范围
        coll1.cbegin(),                  // 第二个范围
        0L)
    << '\n';
```



这里使用了`std::transform_reduce`的二元操作版本：

- **输入**：两个范围都是`coll1`自身

- **初始值**：`0L`（长整型 0）

- **操作**：默认使用`operator*`进行转换，然后用`operator+`进行归约

- 效果

  ：计算两个范围中对应元素的乘积之和（即点积）。由于两个范围都是

  ```
  coll1
  ```

  ，所以结果等价于每个元素的平方和：

  plaintext

  ```plaintext
  1*1 + 2*2 + 3*3 + 4*4 = 1 + 4 + 9 + 16 = 30
  ```

#### 2. 计算每两个相应元素的差的乘积

cpp

```cpp
std::cout << "product of differences: "
    << std::transform_reduce(coll1.cbegin(), coll1.cend(),    // 第一个范围
        coll2.cbegin(),                  // 第二个范围
        1L,
        std::multiplies{}, std::minus{})
    << '\n';
```

这里使用了自定义的二元操作：

- **输入**：第一个范围是`coll1`，第二个范围是`coll2`

- **初始值**：`1L`（长整型 1）

- **转换操作**：`std::minus{}`（计算两个元素的差）

- **归约操作**：`std::multiplies{}`（将所有差相乘）

- 效果

  ：plaintext

  ```plaintext
  (1-1) * (2-2) * (3-3) * (4-4) = 0 * 0 * 0 * 0 = 0
  ```

  注意：虽然

  ```
  coll2
  ```

  有 5 个元素，但只处理到

  ```
  coll1
  ```

  的长度（4 个元素）。

#### 3. 字符串连接操作

cpp

```cpp
std::cout << "sum of combined digits: "
    << std::transform_reduce(coll1.cbegin(), coll1.cend(),    // 第一个范围
        coll2.cbegin(),                  // 第二个范围
        std::string{}, std::plus{},
        [](auto x, auto y) {
            return std::to_string(x) + std::to_string(y) + " ";
        })
    << '\n';
```

这里使用了更复杂的自定义操作：

- **输入**：两个范围分别是`coll1`和`coll2`

- **初始值**：空字符串`std::string{}`

- **转换操作**：将两个整数转换为字符串并连接，中间加空格

- **归约操作**：`std::plus{}`（连接所有转换后的字符串）

- 效果

  ：plaintext

  ```plaintext
  "11 " + "22 " + "33 " + "44 " = "11 22 33 44 "
  ```

### 总结

`std::transform_reduce`是一个灵活的算法，可以：

1. 对两个范围的元素进行转换和归约
2. 自定义转换操作和归约操作
3. 处理不同类型的初始值（如数值、字符串等）

这个算法在数学计算、数据处理和字符串操作中都非常有用。注意当处理两个范围时，第二个范围的长度至少要和第一个范围一样长，否则会导致未定义行为。



