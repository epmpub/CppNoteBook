

#  std::stack

```c++
#include <stack>
#include <functional>
#include <iostream>

// Operations that also return an undo-lambda
auto add(int& state, int operand) {
    state += operand;
    return [operand](int& state) { state -= operand; };
}

auto substract(int& state, int operand) {
    state -= operand;
    return [operand](int& state) { state += operand; };
}

auto multiply(int& state, int operand) {
    state *= operand;
    return [operand](int& state) { state /= operand; };
}

auto divide(int& state, int operand) {
    int orig = state;
    state /= operand;
    return [operand, rem = orig - state * operand](int& state) { 
        state *= operand;
        state += rem;
    };
}

int main() {
    // Apply operations and store the rollback in a std::stack.
    std::stack<std::function<void(int&)>> rollback;

    int state = 0;
    rollback.push(add(state, 10));
    // state == 10

    std::cout << "state == " << state << "\n";

    rollback.push(divide(state, 3));
    // state == 3

    std::cout << "state == " << state << "\n";

    // revert last operation
    rollback.top()(state);
    rollback.pop();
    // state == 10

    std::cout << "state == " << state << "\n";

    rollback.push(multiply(state, 2));
    // state == 20

    std::cout << "state == " << state << "\n";

    while (!rollback.empty()) {
        rollback.top()(state);
        rollback.pop();
    }
    // state == 0

    std::cout << "state == " << state << "\n";
}
```



这段代码展示了一种基于“操作和撤销”模式的 C++ 实现，使用 std::stack 存储撤销操作（以 lambda 函数的形式），并通过 std::function 提供类型擦除。代码定义了四种算术操作（加、减、乘、除），每种操作不仅修改状态，还返回一个对应的撤销操作。以下是逐步解释。

------

代码概览

- 定义了四个函数（add、substract、multiply、divide），每个函数：
  - 修改一个整数状态（state）。
  - 返回一个 lambda 函数，用于撤销该操作。
- 在 main 中，使用 std::stack 存储撤销操作，展示如何应用和撤销这些操作。

------

关键组件

1. **头文件**

cpp

```cpp
#include <stack>
#include <functional>
#include <iostream>
```

- <stack>：提供 std::stack，用于存储撤销操作。
- <functional>：提供 std::function，作为 lambda 的类型擦除容器。
- <iostream>：用于输出。
- **add 函数**

cpp

```cpp
auto add(int& state, int operand) {
    state += operand;
    return [operand](int& state) { state -= operand; };
}
```

- **功能**：
  - 将 operand 加到 state 上。
  - 返回一个 lambda，用于撤销（减去 operand）。
- **示例**：
  - state = 0, operand = 10 → state = 10。
  - 返回的 lambda：state -= 10 → state = 0。
- **substract 函数**

cpp

```cpp
auto substract(int& state, int operand) {
    state -= operand;
    return [operand](int& state) { state += operand; };
}
```

- **功能**：
  - 从 state 中减去 operand。
  - 返回一个 lambda，用于撤销（加上 operand）。
- **示例**：
  - state = 10, operand = 5 → state = 5。
  - 返回的 lambda：state += 5 → state = 10。
- **multiply 函数**

cpp

```cpp
auto multiply(int& state, int operand) {
    state *= operand;
    return [operand](int& state) { state /= operand; };
}
```

- **功能**：
  - 将 state 乘以 operand。
  - 返回一个 lambda，用于撤销（除以 operand）。
- **示例**：
  - state = 10, operand = 2 → state = 20。
  - 返回的 lambda：state /= 2 → state = 10。
- **divide 函数**

cpp

```cpp
auto divide(int& state, int operand) {
    int orig = state;
    state /= operand;
    return [operand, rem = orig - state * operand](int& state) { 
        state *= operand;
        state += rem;
    };
}
```

- **功能**：
  - 将 state 除以 operand（整数除法）。
  - 保存原始值 orig，计算余数 rem = orig - state * operand。
  - 返回一个 lambda，用于撤销（乘以 operand 并加上余数）。
- **示例**：
  - state = 10, operand = 3：
    - orig = 10, state = 10 / 3 = 3, rem = 10 - 3 * 3 = 1。
  - 返回的 lambda：
    - state = 3 * 3 + 1 = 10 → state = 10。
- **注意**：
  - 整数除法会丢失余数，撤销时需要 rem 恢复原始值。
- **main 函数**

cpp

```cpp
int main() {
    std::stack<std::function<void(int&)>> rollback;
    int state = 0;

    rollback.push(add(state, 10));
    std::cout << "state == " << state << "\n"; // state == 10

    rollback.push(divide(state, 3));
    std::cout << "state == " << state << "\n"; // state == 3

    rollback.top()(state);
    rollback.pop();
    std::cout << "state == " << state << "\n"; // state == 10

    rollback.push(multiply(state, 2));
    std::cout << "state == " << state << "\n"; // state == 20

    while (!rollback.empty()) {
        rollback.top()(state);
        rollback.pop();
    }
    std::cout << "state == " << state << "\n"; // state == 0
}
```

- **std::stack<std::function<void(int&)>> rollback**：
  - 存储撤销操作，std::function 擦除 lambda 的具体类型。
- **操作序列**：
  1. **add(state, 10)**：
     - state = 0 + 10 = 10。
     - 压入撤销 lambda：state -= 10。
     - 输出：state == 10。
  2. **divide(state, 3)**：
     - state = 10 / 3 = 3，余数 1。
     - 压入撤销 lambda：state = 3 * 3 + 1。
     - 输出：state == 3。
  3. **撤销**：
     - rollback.top()(state) 调用栈顶 lambda：state = 3 * 3 + 1 = 10。
     - rollback.pop() 移除该 lambda。
     - 输出：state == 10。
  4. **multiply(state, 2)**：
     - state = 10 * 2 = 20。
     - 压入撤销 lambda：state /= 2。
     - 输出：state == 20。
  5. **清空栈**：
     - while (!rollback.empty()) 循环执行所有撤销操作：
       - state = 20 / 2 = 10（multiply 的撤销）。
       - state = 10 - 10 = 0（add 的撤销）。
     - 输出：state == 0。

------

为什么这样工作？

1. **撤销模式**：
   - 每个操作返回一个闭包（lambda），捕获操作数并定义逆操作。
   - std::stack 按 LIFO（后进先出）顺序存储撤销操作，适合回滚。
2. **std::function**：
   - 提供类型擦除，允许存储不同 lambda（只要签名是 void(int&)）。
3. **整数除法的特殊处理**：
   - divide 需要保存余数，确保撤销时恢复原始值。

------

输出

```text
state == 10
state == 3
state == 10
state == 20
state == 0
```

------

使用场景

- **事务回滚**：
  - 在数据库或状态机中，记录操作以便失败时恢复。
- **命令模式**：
  - 实现可撤销的命令（如文本编辑器的“撤销”功能）。
- **调试**：
  - 跟踪和恢复状态变化。

------

总结

- 代码通过返回 lambda 实现操作的撤销，使用 std::stack 管理回滚。
- add、substract、multiply 是简单逆操作，divide 处理余数以确保精确恢复。
- 展示了 C++ 中闭包和容器结合的强大功能，适用于需要撤销的场景。
- 