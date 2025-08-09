# C++ 类型擦除 之 通过继承实现

Type erasure is a technique that allows the decoupling of the bit representation of implementation from the user.

This permits the user to be resilient against changes in the implementation and/or to operate on different implementations.

The simplest C++ approach to type erasure is through inheritance.

```C++
#include <memory>
#include <string>
#include <iostream>

// Abstract class, C++ doesn't have a notion of an interface:
class SomeInterface {
public:
    // Pure virtual methods that define an interface
    virtual int operation() = 0; 
    // Virtual destructor
    virtual ~SomeInterface() = default;
};

// Concrete implementations have to implement all pure methods,
// but can have any additional methods and contain arbitrary data.
struct ImplA : SomeInterface {
    ImplA(int rank) : rank(rank) {}
    int operation() override {
        return rank;
    }
    int rank;
};

struct ImplB : SomeInterface {
    ImplB(std::string text) : text(text) {}
    int operation() override {
        return std::stoi(text);
    }
    std::string text;
};

// User only depends on SomeInterface, can accept any type
// that implements SomeInterface:
void user(std::unique_ptr<SomeInterface> data) {
    int v = data->operation();
    std::cout << v << "\n";
}

user(std::make_unique<ImplA>(10)); // OK, prints 10
user(std::make_unique<ImplB>(std::string{"42"})); // OK, prints 42
```

