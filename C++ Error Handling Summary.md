现代C++ 错误处理





# C++ Error Handling Summary

Here's a summary of C++ error handling approaches, including the C++23 `std::expected`:

## Traditional Error Handling Methods

### Exception Handling

```cpp
try {
    // Code that might throw
    throw std::runtime_error("Something went wrong");
} catch (const std::exception& e) {
    // Handle exception
    std::cerr << "Exception caught: " << e.what() << std::endl;
} catch (...) {
    // Catch all other exceptions
    std::cerr << "Unknown exception caught" << std::endl;
}
```

### Error Codes

```cpp
enum class ErrorCode { Success, FileNotFound, PermissionDenied };

ErrorCode openFile(const std::string& filename) {
    // Check if file exists
    if (!fileExists(filename)) {
        return ErrorCode::FileNotFound;
    }
    // Check permissions
    if (!hasPermission(filename)) {
        return ErrorCode::PermissionDenied;
    }
    return ErrorCode::Success;
}
```

## Modern C++ Error Handling

### std::optional (C++17)

```cpp
std::optional<int> parseInteger(const std::string& str) {
    try {
        return std::stoi(str);
    } catch (...) {
        return std::nullopt;
    }
}
```

### std::variant (C++17)

```cpp
std::variant<std::string, ErrorCode> readFile(const std::string& filename) {
    if (!fileExists(filename)) {
        return ErrorCode::FileNotFound;
    }
    // Read file contents
    return fileContents;
}
```

### std::expected (C++23)

The `std::expected` template is part of the C++23 standard and provides a way to return either a value or an error:

```cpp
#include <expected>
#include <string>

std::expected<int, std::string> divide(int a, int b) {
    if (b == 0) {
        return std::unexpected("Division by zero");
    }
    return a / b;
}

void use_divide() {
    auto result = divide(10, 2);
    if (result) {
        // Access the value
        std::cout << "Result: " << *result << std::endl;
    } else {
        // Access the error
        std::cout << "Error: " << result.error() << std::endl;
    }
    
    // Using value_or for default values
    int safe_result = divide(10, 0).value_or(-1);
}
```

Key features of `std::expected`:

- Combines error handling with return values
- Explicit error handling without exceptions
- Provides methods like `value()`, `error()`, `value_or()`, `and_then()`, and `or_else()`
- More type-safe than error codes
- More efficient than exceptions for expected error cases

Would you like me to explain any specific aspect of C++ error handling in more detail?