```C++
#include <iostream>
#include <vector>
#include <algorithm> // for std::sample
#include <random>    // for std::mt19937

int main() {
    // 输入序列
    std::vector<int> input = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

    // 输出序列（预分配空间）
    std::vector<int> output(3);

    // 设置随机数生成器
    std::random_device rd; // 获取随机种子
    std::mt19937 gen(rd()); // 使用 Mersenne Twister 引擎

    // 使用 std::sample 随机抽取 3 个元素
    std::sample(input.begin(), input.end(), output.begin(), 3, gen);

    // 输出结果
    std::cout << "Random sample: ";
    for (int x : output) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

