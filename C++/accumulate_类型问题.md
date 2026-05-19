

0 == 0.0 

演示代码:



```C++
#include <ranges>
#include <print>
#include <vector>
#include <algorithm>
#include <numeric>
#include <iostream>

int main() {

	//算术运算:
	// 
	//if (0 == 0.0)
	//{
	//	std::print("0 == 0.0\n");
	//}
	//else {
	//  std::print("0 != 0.0\n");
	//}

	//编译时断定:
	//static_assert(std::is_same_v<decltype(0),decltype(0.0)>);

	//std::print("0 type is: {}\n", typeid(0).name());

	//std::print("0.0 type is: {}\n", typeid(0.0).name());


	//数据准备:
	std::vector v{ 1.1,2.2,3.3,4.4,5.5,6.6 };

    //传统C++11  std::accumulate
	//If left to type inference, op operates on values of the same type as init which can result in unwanted casting of the iterator elements. For example, std::accumulate(v.begin(), v.end(), 0) likely does not give the result one wishes for when v is of type std::vector<double>.

   //auto sum = std::accumulate(v.begin(), v.end(), 0.0); // 0  BUG !!!!   0 != 0.0  
   //std::cout << "std::accumulate: " << sum << std::endl;

   //// C++23 std::ranges::accumulate
   auto result = std::ranges::fold_left(v, 0, std::plus{});  // C++23 的正确用法  f(f(f(f(init, x1), x2), ...), xn),
   std::cout << "std::ranges::fold_left: " << result << std::endl;

   return 0;
}
```

