# C++ 23 std::out_ptr的用法

C++23 的 std::out_ptr 是一个用于处理 C 风格指针输出的工具，通常用于将智能指针适配到需要输出参数的 C 风格 API。

cpp

```cpp
std::unique_ptr<FILE, decltype(&fclose)> file(fopen("example.txt", "r"), &fclose);
```

我们可以用 std::out_ptr（假设使用 C++23 或类似工具）来改写 fopen 的调用，让它与 std::unique_ptr 配合工作。

使用 std::out_ptr 的背景

std::out_ptr 的主要用途是将智能指针（比如 std::unique_ptr）的原始指针地址传递给需要输出参数的 C 风格函数（比如 fopen 这种返回 FILE* 的函数），并在调用后自动将结果转移到智能指针管理。

但是需要注意的是，fopen 并不直接适合 std::out_ptr，因为 fopen 的用法是返回一个 FILE*，而不是通过输出参数修改指针（比如 FILE**）。**

**std::out_ptr 更适合像这样的函数：int some_c_function(T** out)，其中 out 是一个输出参数。

为了展示 std::out_ptr 的用法，我会先改写代码假设一个类似 fopen 的函数，它通过输出参数返回 FILE*，然后再回到 fopen 的实际用法。

------

假设的场景：改写为类似 fopen 的函数

假设我们有一个函数 open_file，它的签名是：

cpp

```cpp
int open_file(const char* filename, const char* mode, FILE** out);
```

这个函数通过 out 参数返回 FILE*，并返回一个状态码（0 表示成功）。我们可以用 std::out_ptr 来改写：

使用 std::out_ptr 的代码

cpp

```cpp
#include <memory>
#include <cstdio>
#include <iostream>
#include <memory> // 包含 std::out_ptr (C++23)

int open_file(const char* filename, const char* mode, FILE** out) {
    *out = fopen(filename, mode); // 模拟通过输出参数返回 FILE*
    return (*out != nullptr) ? 0 : -1; // 0 表示成功
}

int main() {
    // 定义 std::unique_ptr，使用 fclose 作为删除器
    std::unique_ptr<FILE, decltype(&fclose)> file(nullptr, &fclose);

    // 使用 std::out_ptr 适配 FILE** 参数
    if (open_file("example.txt", "r", std::out_ptr(file)) == 0) {
        char buffer[256];
        while (fgets(buffer, sizeof(buffer), file.get())) {
            printf("%s", buffer);
        }
    } else {
        std::cout << "Failed to open file\n";
    }

    // file 超出作用域时，std::unique_ptr 自动调用 fclose
    return 0;
}
```

解释：

1. **std::out_ptr(file)**：
   - std::out_ptr 是一个工具，它将 std::unique_ptr 转换为一个临时的 FILE**，可以传递给 open_file 的第三个参数。
   - 在 open_file 调用后，std::out_ptr 会自动将 FILE* 转移到 file（即 std::unique_ptr）中管理。
   - 同时，std::out_ptr 确保如果 open_file 失败，file 不会接管一个无效的指针。
2. **删除器**：
   - std::unique_ptr 仍然使用 fclose 作为删除器，确保 FILE* 在析构时被正确关闭。
3. **好处**：
   - 使用 std::out_ptr 可以更安全地与 C 风格 API 交互，避免手动管理指针的麻烦。

------

回到 fopen 的实际用法

fopen 的签名是 FILE* fopen(const char* filename, const char* mode)，它直接返回 FILE*，而不是通过输出参数。因此，std::out_ptr 并不直接适用，因为 std::out_ptr 设计是为了处理类似 FILE** 这种输出参数的场景。

对于 fopen，原来的写法已经是最简洁和合适的：

cpp

```cpp
std::unique_ptr<FILE, decltype(&fclose)> file(fopen("example.txt", "r"), &fclose);
```

如果非要用某种方式改写（比如为了展示 std::out_ptr 的类似思想），我们需要封装 fopen 成一个带有输出参数的函数：

封装 fopen 并使用 std::out_ptr

cpp

```cpp
#include <memory>
#include <cstdio>
#include <iostream>
#include <memory> // 包含 std::out_ptr (C++23)

// 封装 fopen 为带有输出参数的函数
int wrapped_fopen(const char* filename, const char* mode, FILE** out) {
    *out = fopen(filename, mode);
    return (*out != nullptr) ? 0 : -1;
}

int main() {
    std::unique_ptr<FILE, decltype(&fclose)> file(nullptr, &fclose);

    // 使用 std::out_ptr 调用封装的函数
    if (wrapped_fopen("example.txt", "r", std::out_ptr(file)) == 0) {
        char buffer[256];
        while (fgets(buffer, sizeof(buffer), file.get())) {
            printf("%s", buffer);
        }
    } else {
        std::cout << "Failed to open file\n";
    }

    return 0;
}
```



使用lambda关闭文件:

```C++
#include <memory>
#include <cstdio>

int main() {
    // 使用 lambda 表达式作为删除器
    auto deleter = [](FILE* fp) { if (fp) fclose(fp); };
    std::unique_ptr<FILE, decltype(deleter)> file(fopen("example.txt", "r"), deleter);

    if (file) {
        char buffer[256];
        while (fgets(buffer, sizeof(buffer), file.get())) {
            printf("%s", buffer);
        }
    }

    return 0;
}
```



```C++
#include <string>
#include <iostream>
#include <memory>

extern "C" {
	void get_data(int** ptr) {
		int* result = (int*)malloc(sizeof(int));
		*result = 42;
		*ptr = result;
	}
}
int main()
{
	std::unique_ptr<int, decltype([](int* ptr) { free(ptr); }) > somethings;
	get_data(std::inout_ptr(somethings));
	std::cout << *somethings << std::endl;
	return 0;
}
```



解释：

- 我们定义了一个 wrapped_fopen 函数，将 fopen 的行为封装成带有 FILE** 输出参数的形式。
- 然后使用 std::out_ptr 来适配 std::unique_ptr，让它接收 wrapped_fopen 的输出。

------

总结

- 如果直接使用 fopen，原始代码已经是最合适的：std::unique_ptr<FILE, decltype(&fclose)> file(fopen("example.txt", "r"), &fclose);。
- std::out_ptr 更适合需要输出参数的 C 风格函数（比如 int func(T** out)）。
- 如果一定要用 std::out_ptr，需要将 fopen 封装成带有输出参数的函数（如 wrapped_fopen），然后用 std::out_ptr 适配。

