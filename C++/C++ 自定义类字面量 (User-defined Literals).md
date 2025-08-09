# C++ 自定义类字面量 (User-defined Literals)

在 C++11 及更高版本中，你可以为自定义类创建用户定义字面量(User-defined Literals)，这使得代码更加直观和易读。以下是实现自定义类字面量的方法：

## 基本语法

用户定义字面量通过定义字面量运算符来实现，语法如下：

```cpp
ReturnType operator"" _suffix(Parameters);
```

## 实现方式

### 1. 整数字面量运算符

```cpp
#include <iostream>

class Distance {
    int meters;
public:
    Distance(int m) : meters(m) {}
    void show() const { std::cout << meters << " meters\n"; }
};

Distance operator"" _m(unsigned long long m) {
    return Distance(static_cast<int>(m));
}

int main() {
    auto d = 100_m;  // 创建100米的距离
    d.show();        // 输出: 100 meters
}
```

### 2. 浮点数字面量运算符

```cpp
class Temperature {
    double celsius;
public:
    Temperature(double c) : celsius(c) {}
    void show() const { std::cout << celsius << "°C\n"; }
};

Temperature operator"" _c(long double temp) {
    return Temperature(static_cast<double>(temp));
}

int main() {
    auto t = 36.6_c;  // 创建36.6°C的温度
    t.show();         // 输出: 36.6°C
}
```

### 3. 字符串字面量运算符

```cpp
#include <string>

class Greeting {
    std::string msg;
public:
    Greeting(const char* str, size_t len) : msg(str, len) {}
    void say() const { std::cout << msg << "\n"; }
};

Greeting operator"" _greet(const char* str, size_t len) {
    return Greeting(str, len);
}

int main() {
    auto g = "Hello"_greet;  // 创建问候语
    g.say();                 // 输出: Hello
}
```

## 更复杂的例子

```cpp
#include <iostream>
#include <string>
#include <cmath>

class Complex {
    double real, imag;
public:
    Complex(double r, double i = 0.0) : real(r), imag(i) {}
    
    friend Complex operator"" _i(long double imag) {
        return Complex(0.0, static_cast<double>(imag));
    }
    
    Complex operator+(const Complex& other) const {
        return Complex(real + other.real, imag + other.imag);
    }
    
    void print() const {
        std::cout << real << (imag >= 0 ? "+" : "") << imag << "i\n";
    }
};

int main() {
    Complex c1 = 3.0 + 4.0_i;  // 3 + 4i
    Complex c2 = 2.5 - 1.2_i;   // 2.5 - 1.2i
    Complex c3 = c1 + c2;
    
    c1.print();  // 输出: 3+4i
    c2.print();  // 输出: 2.5-1.2i
    c3.print();  // 输出: 5.5+2.8i
}
```

## 注意事项

1. 用户定义字面量必须以下划线 `_` 开头（标准库的字面量不使用下划线）
2. 可以重载的字面量类型：
   - 整型 (`unsigned long long`)
   - 浮点型 (`long double`)
   - 字符 (`char`)
   - 字符串 (`const char*`, `size_t`)
   - 原始字符 (`const char*`)
3. 字面量运算符可以定义为类的成员函数或全局函数
4. 字面量运算符通常应声明为 `constexpr` 以支持编译期计算

用户定义字面量可以极大地提高代码的可读性，特别是在涉及单位、特殊值或领域特定语言的场景中。