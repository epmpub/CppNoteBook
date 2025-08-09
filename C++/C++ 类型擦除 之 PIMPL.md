# C++ 类型擦除 之 PIMPL 

The inheritance approach to type erasure allows users to work simultaneously with multiple implementations.

However, if we only require one implementation at a time, we can choose a simpler and faster approach using the PIMPL pattern.

```c++
#include <memory>
#include <string>
#include <iostream>

// Header:
struct SomeInterface {
    int operation() const;
    SomeInterface();
    SomeInterface(SomeInterface&&) = default;
    ~SomeInterface();
private:    
    struct Implementation;
    std::unique_ptr<Implementation> pimpl_;   
};

// User only depends on SomeInterface, implementations behind
// SomeInterface can be swapped without affecting user.
void user(SomeInterface data) {
    int v = data.operation();
    std::cout << v << "\n";
}

// Implementation A
struct SomeInterface::Implementation {
    int operation() { return rank; }
    int rank;
};

SomeInterface::~SomeInterface() = default;
SomeInterface::SomeInterface()
  : pimpl_(std::make_unique<SomeInterface::Implementation>(10)) {}

int SomeInterface::operation() const {
    return pimpl_->operation();
}

// Implementation B
struct SomeInterface::Implementation {
    int operation() { return std::stoi(text); }
    std::string text;
};

SomeInterface::~SomeInterface() = default;
SomeInterface::SomeInterface() 
  : pimpl_(std::make_unique<SomeInterface::Implementation>("42")) {}

int SomeInterface::operation() const {
    return pimpl_->operation();
}
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/oz5cKf47h)