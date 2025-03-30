现代C++提供了多种错误处理机制，开发者可根据具体场景选择最合适的方式。以下是最常用的方法及其典型应用场景：

---

### 1. **异常处理（Exceptions）**
- **核心机制**：`try`/`catch` + `throw`
- **特点**：
  - 适合不可恢复错误（如资源不足、逻辑错误）
  - 自动栈展开，资源通过RAII自动清理
  - 性能开销较高（需权衡是否启用）
- **代码示例**：
  ```cpp
  void loadConfig() {
      if (!fileExists("config.json")) 
          throw std::runtime_error("Config file missing");
      // ...
  }
  
  try {
      loadConfig();
  } catch (const std::exception& e) {
      std::cerr << "Error: " << e.what() << std::endl;
  }
  ```

---

### 2. **类型安全的错误码：`std::error_code`（C++11）**
- **核心机制**：
  - 通过`std::error_code`封装错误码和分类
  - 避免“魔术数字”，支持跨平台错误码（如系统错误）
- **典型场景**：
  - 需要明确错误分类时（如网络错误、文件IO错误）
  - 禁用异常的项目（如嵌入式、高性能计算）
- **代码示例**：
  ```cpp
  std::error_code readFile(const std::string& path, std::string& data) {
      std::ifstream file(path);
      if (!file) return std::make_error_code(std::errc::no_such_file_or_directory);
      // ...
      return {}; // 空表示成功
  }
  
  std::string data;
  if (auto ec = readFile("data.txt", data); ec) {
      std::cerr << "Error: " << ec.message() << std::endl;
  }
  ```

---

### 3. **可选值：`std::optional`（C++17）**
- **核心机制**：
  - 表示值可能存在或缺失（如查找失败）
  - 不携带错误原因，适合简单场景
- **代码示例**：
  ```cpp
  std::optional<int> parseNumber(const std::string& s) {
      try {
          return std::stoi(s);
      } catch (...) {
          return std::nullopt;
      }
  }
  
  if (auto num = parseNumber("123")) {
      use(*num);
  } else {
      std::cout << "Invalid number" << std::endl;
  }
  ```

---

### 4. **预期值-错误包装：`std::expected`（C++23）**
- **核心机制**：
  - 函数返回可能的值或错误对象（类似Rust的`Result`）
  - 强制调用者处理两种状态
- **代码示例**：
  ```cpp
  std::expected<std::string, std::error_code> readData() {
      if (fail) return std::unexpected(std::make_error_code(std::errc::io_error));
      return "data";
  }
  
  auto result = readData();
  if (result) {
      process(*result);
  } else {
      handleError(result.error());
  }
  ```

---

### 5. **第三方库支持（如Boost.Outcome）**
- **核心机制**：
  - 提供类似`std::expected`的功能（C++23前）
  - 支持更复杂的错误传播（如链式错误）
- **代码示例**：
  ```cpp
  outcome::result<int> computeValue() {
      if (error) return outcome::failure(Error::InvalidInput);
      return 42;
  }
  
  auto result = computeValue();
  if (result.has_value()) {
      use(result.value());
  } else {
      logError(result.error());
  }
  ```

---

### 6. **契约（Contracts）**（提案中，未来可能支持）
- **核心机制**：
  - 通过`[[expects]]`、`[[ensures]]`等属性定义接口约束
  - 在调试阶段捕获前置/后置条件违规
- **示例**：
  ```cpp
  void push(int x) [[expects: !isFull()]] [[ensures: !isEmpty()]] {
      // ...
  }
  ```

---

### **选择建议**
| 场景         | 推荐方案                 |
| ------------ | ------------------------ |
| 不可恢复错误 | 异常（如内存耗尽）       |
| 明确错误分类 | `std::error_code`        |
| 简单值缺失   | `std::optional`          |
| 强制错误处理 | `std::expected`（C++23） |
| 禁用异常     | 错误码或`std::expected`  |
| 高性能场景   | 错误码或`std::expected`  |

---

### **最佳实践**
1. **优先RAII**：确保异常安全时资源自动释放。
2. **避免异常滥用**：高频路径或关键系统慎用异常。
3. **明确错误语义**：通过自定义错误类型（如`enum class`）提高可读性。
4. **组合使用**：如用`std::expected`包装`std::error_code`，兼顾灵活性和信息量。

现代C++的趋势是**显式、类型安全且无开销的错误处理**，结合传统机制与新特性，开发者可构建健壮且高效的错误处理策略。