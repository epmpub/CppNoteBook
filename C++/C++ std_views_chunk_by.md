

# std::views::chunk_by

C++23`std::views::chunk_by`与 C++20 类似`std::views::split`；

但是，与 split 不同，它使用二元谓词进行操作。

当谓词返回 false 时，两个测试元素之间将开始一个新的块



```c++

#include <ranges>
#include <vector>
#include <cmath>
#include <iostream>


int main()
{
	std::vector<int> data{ 1,2,-4,-2,-1,8,-7,3,4,-5,-5 };
	auto same_sign = [](int left, int right) {
		return std::signbit(left) == std::signbit(right);
		};
	for (const auto& chunk : data | std::views::chunk_by(same_sign)) {
		// Iterate over chunks:
		// {1,2}, {-4,-2,-1}, {8,7,3,4}, {-5,-5}
		for (int i : chunk) {
			// Iterate over elements in chunk
			std::cout << i << ' ';
		}
		std::cout << '\n';
	}
}

```

same_sign 是一个 lambda 表达式，定义了一个二元谓词。

功能：

- 接受两个整数 left 和 right。
- 使用 std::signbit 检查它们的符号位是否相同。
  - std::signbit(x) 返回 true 如果 x 是负数（符号位为 1），false 如果是正数或零。
- 返回 true 表示两数符号相同，false 表示不同。

用途：用于 std::views::chunk_by 分组。

