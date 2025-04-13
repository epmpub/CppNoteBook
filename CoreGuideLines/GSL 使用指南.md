# Microsoft Guidelines Support Library (MS-GSL) 使用指南

MS-GSL (Microsoft Guidelines Support Library) 是一个小型开源库，旨在帮助开发者更安全地编写C++代码，特别是遵循C++核心准则(C++ Core Guidelines)。它提供了一系列有用的工具和类型，以减少常见编程错误。

## 1. 安装MS-GSL

### 通过包管理器安装
- **vcpkg**: `vcpkg install ms-gsl`
- **Conan**: `conan install ms-gsl/3.1.0@`

### 手动安装
1. 从GitHub克隆仓库:
   ```bash
   git clone https://github.com/microsoft/GSL.git
   ```
2. 将`include`目录添加到项目的包含路径中

## 2. 核心组件和使用

### 2.1 边界检查类型

#### `gsl::span<T>`
- 替代原始指针，提供安全的数组/容器视图
- 自动进行边界检查

```cpp
#include <gsl/span>

void process_data(gsl::span<int> data) {
    for (auto& item : data) {
        // 安全访问，自动边界检查
        item *= 2;
    }
}

int arr[10] = {1,2,3,4,5,6,7,8,9,10};
process_data(arr);  // 自动转换数组为span
```

#### `gsl::multi_span<T, N>`
- 多维数组的安全视图

```cpp
#include <gsl/multi_span>

void process_matrix(gsl::multi_span<int, 2> matrix) {
    for (int i = 0; i < matrix.extent(0); ++i) {
        for (int j = 0; j < matrix.extent(1); ++j) {
            // 安全的多维访问
            matrix[i][j] = i + j;
        }
    }
}

int matrix[3][3];
process_matrix(matrix);
```

### 2.2 所有权和生命周期管理

#### `gsl::owner<T>`
- 明确标记拥有资源所有权的指针

```cpp
#include <gsl/pointers>

gsl::owner<int*> p = new int[100];
// ... 使用p ...
delete[] p;  // 明确所有权，提醒需要手动释放
```

#### `gsl::not_null<T>`
- 保证指针或智能指针不为null

```cpp
#include <gsl/pointers>

void safe_print(gsl::not_null<const char*> str) {
    // 保证str不会是nullptr
    std::cout << str.get();
}

const char* msg = "Hello";
safe_print(msg);  // 如果msg是nullptr，会触发断言
```

### 2.3 实用工具

#### `gsl::finally`
- 确保资源清理的RAII包装器

```cpp
#include <gsl/util>

void example() {
    FILE* file = fopen("data.txt", "r");
    auto cleanup = gsl::finally([&] { 
        if (file) fclose(file); 
    });
    
    // 使用文件...
    // 无论是否抛出异常，文件都会被关闭
}
```

#### `gsl::narrow_cast`和`gsl::narrow`
- 安全的类型转换

```cpp
#include <gsl/narrow>

int main() {
    int32_t big = 1'000'000;
    int16_t small = gsl::narrow<int16_t>(big);  // 会抛出异常，因为值太大
    
    // 或者使用narrow_cast(不检查)
    small = gsl::narrow_cast<int16_t>(big);  // 可能截断，但不会抛出异常
}
```

### 2.4 合约检查

#### 前置/后置条件检查

```cpp
#include <gsl/assert>

int divide(int a, int b) {
    Expects(b != 0);  // 前置条件检查
    int result = a / b;
    Ensures(result * b == a);  // 后置条件检查
    return result;
}
```

## 3. 与STL的互操作

MS-GSL与STL容器有良好的互操作性：

```cpp
#include <vector>
#include <gsl/span>

void process_vector(std::vector<int>& v) {
    gsl::span<int> s{v};
    for (auto& item : s) {
        item *= 2;
    }
}

std::vector<int> numbers = {1, 2, 3, 4, 5};
process_vector(numbers);
```

## 4. 最佳实践

1. **优先使用`span`替代原始指针数组**：减少缓冲区溢出风险
2. **用`not_null`标记非空指针**：消除空指针解引用
3. **使用`owner`明确资源所有权**：提高代码可读性和安全性
4. **利用`finally`管理资源**：确保异常安全
5. **使用合约检查**：明确函数的前置/后置条件

## 5. 注意事项

1. MS-GSL主要提供编译时和调试时的安全检查，发布版本中许多检查会被禁用
2. 某些功能(如边界检查)可能有性能开销
3. 不是所有编译器都完全支持，确保你的编译器兼容

通过合理使用MS-GSL，可以显著提高C++代码的安全性和可靠性，同时保持高性能。