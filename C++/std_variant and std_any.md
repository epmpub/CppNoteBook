std variant and std any

```c++
#include <iostream>
#include <vector>
#include <algorithm> // for std::sample
#include <random>    // for std::mt19937
#include <string>

#include <variant>   // for std::variant
#include <any>

int main() {
	// 输入序列
	using VarType = std::variant<int, std::string>;

	std::vector<VarType> input = { "C++","Rust","Golang","Python","Ruby","C#",1,2,3 };


	// 输出序列（预分配空间）
	std::vector<VarType> output(3);

	// 设置随机数生成器
	std::random_device rd; // 获取随机种子
	std::mt19937 gen(rd()); // 使用 Mersenne Twister 引擎

	// 使用 std::sample 随机抽取 3 个元素
	std::sample(input.begin(), input.end(), output.begin(), 3, gen);

	// 输出结果
	std::cout << "Random sample: ";

	for (const auto& item : output) {
		if (std::holds_alternative<std::string>(item)) {
			std::cout << std::get<std::string>(item) << " ";
		} else if (std::holds_alternative<int>(item)) {
			std::cout << std::get<int>(item) << " ";
		} else {
			std::cout << "[unknown] ";
		}
	}

	//std::cout << std::endl;

	// 使用 std::visit 来处理 std::variant 中的不同类型
	//for (const auto& item : output) {
	//	std::visit([](const auto& value) {
	//		std::cout << value << " ";
	//		}, item);
	//}

	// 使用 std::any 来处理不同类型的输出
	//for (const auto& item : output) {
	//	try {
	//		std::cout << std::any_cast<const char*>(item) << " ";
	//	} catch (const std::bad_any_cast&) {
	//		try {
	//			std::cout << std::any_cast<int>(item) << " ";
	//		} catch (const std::bad_any_cast&) {
	//			std::cout << "[unknown] ";
	//		}
	//	}
	//}

	return 0;
}
```

