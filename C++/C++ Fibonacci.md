

# Fibonacci的3中写法 

```C++
#include <iostream>
#include <array>
#include <utility>

template<int I>struct Fib
{
	static const int val = Fib<I - 1>::val + Fib<I - 2>::val;
};

template<>struct Fib<0>
{
	static const int val = 0;
};

template<>struct Fib<1>
{
	static const int val = 1;
};


template<size_t ...I>
int fib_impl(std::index_sequence<I...>, const int i)
{
	constexpr std::array<int, sizeof...(I)> a = { Fib<I>::val... };
	return a[i];
}

int fib(const int i)
{
	return fib_impl(std::make_index_sequence<47>(), i);
}

int main()
{
	std::cout << fib(46) << std::endl;
}
```



// use coroutine

```C++
std::generator<int> fibonacci() {
    int a = 0, b = 1;
    while (true) {
        co_yield std::exchange(a, std::exchange(b, a + b));
    }
}

int answer_to_the_universe() {
    auto rng = fibonacci() | std::views::drop(6) | std::views::take(3);
    return std::ranges::fold_left(std::move(rng), 0, std::plus{});
}

```



```C++
#include <cmath>
#include <iostream>
constexpr double sqrt_5 = 2.23606797749979; // Precomputed value of sqrt(5)

constexpr int fib(const int i)
{
   if (i == 0) return 0;
   if (i == 1) return 1;
   return static_cast<int>((std::pow(1 + sqrt_5, i) - std::pow(1 - sqrt_5, i)) / (std::pow(2, i) * sqrt_5));
}

int main()
{
   std::cout << fib(45) << '\n';
   return 0;
}

```

![](D:\0.products\myBook\Fibonaiic-3.png)

```C++
#include <iostream>
template<int I>struct Fib
{
	static const int val = Fib<I - 1>::val + Fib<I - 2>::val;
};

template<>struct Fib<0>
{
	static const int val = 0;
};

template<>struct Fib<1>
{
	static const int val = 1;
};

int main(int argc, const char* [])
{
	std::cout << Fib<46>::val << '\n';
}

```

要实现一个高效的 Fibonacci 函数，避免使用递归模板（如示例中的 Fib 模板类）带来的编译期开销或运行时递归开销，我们可以使用**迭代方法**或**动态规划**，以降低时间复杂度到 O(n) 并且空间复杂度为 O(1)。以下是一个高效的非模板实现的 Fibonacci 函数，计算第 n 个 Fibonacci 数（n 从 0 开始，F(0) = 0, F(1) = 1）。

高效 Fibonacci 函数

cpp

```cpp
#include <iostream>

long long fibonacci(int n) {
    if (n < 0) return 0; // 无效输入
    if (n == 0) return 0;
    if (n == 1) return 1;

    long long prev = 0, curr = 1;
    for (int i = 2; i <= n; ++i) {
        long long next = prev + curr;
        prev = curr;
        curr = next;
    }
    return curr;
}

int main() {
    std::cout << fibonacci(46) << '\n'; // 输出第 46 个 Fibonacci 数
    return 0;
}
```

说明

1. **算法**：
   - 使用迭代方法，维护两个变量 prev 和 curr，分别表示 F(i-2) 和 F(i-1)。
   - 在每次循环中，计算 F(i) = F(i-1) + F(i-2)，然后更新 prev 和 curr。
   - 这种方法避免了递归的栈开销和重复计算。
2. **时间复杂度**：O(n)，因为只需循环 n-1 次。
3. **空间复杂度**：O(1)，只使用固定数量的变量（prev 和 curr）。
4. **数据类型**：
   - 使用 long long 以处理较大的 Fibonacci 数。例如，F(46) = 1,836,311,903，适合 long long（64 位整数）的范围。
   - 如果需要更大的 n，可以使用大整数库（如 GMP 或 Boost.Multiprecision）。
5. **与模板版本对比**：
   - 模板版本（如 Fib<I>）是编译期计算，适合在编译时确定值的场景，但会导致大量模板实例化，增加编译时间。
   - 运行时迭代版本更灵活，适合动态输入，且运行时效率高。

输出

对于 n = 46，运行上述代码将输出：

```text
1836311903
```

进一步优化

- **矩阵快速幂**：如果需要计算非常大的 n，可以使用矩阵形式：

  ```text
  | F(n+1) F(n)   | = | 1 1 |^n * | F(1) F(0) |
  | F(n)   F(n-1) |   | 1 0 |     | F(0) F(-1)|
  ```

  结合快速幂算法，时间复杂度可降到 O(log n)。但对于 n ≤ 46，这种方法复杂度较高，迭代方法已足够高效。

- **常量时间（查找表）**：如果 n 的范围较小（如 n ≤ 100），可以预计算 Fibonacci 数并存储在数组中，查询时 O(1)。但这会增加空间复杂度到 O(n)。

注意事项

- 如果 n 较大（如 n > 93），long long 可能会溢出，需使用大整数类型。
- 输入验证（如 n < 0）可根据需求调整。

如果您需要矩阵快速幂实现或其他特定优化，请告诉我！